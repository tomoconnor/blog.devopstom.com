---
title: Terraforming RabbitMQ on AWS for Celery
date: "2022-01-31T01:00:01.123Z"
description: ""
---

I've been making things with Django since around 2009.  
Usually I just deploy a thing onto a DigitalOcean droplet, and move on.  

For an interesting challenge this time around, I thought I'd make a thing that's as AWS native as I possibly can, whilst keeping within the Free Tier. 

I wrote [last week](https://blog.devopstom.com/aws-cloudfront-and-cloudflare-dns/) about the difficulties surrounding the fact that the AWS Free Tier doesn't include Route53. 

This particular Django project has a 'requirement' that some of the processing of requests is actually queued, because it can be a bit on the time consuming side, potentially, so it'd make sense if that happens via a task queue. 

My usual go-to for Django-based task queueing is [Celery](https://docs.celeryproject.org/en/stable/index.html).  In the past when I've built a django/celery app, I've usually had Django and Celery running in separate Docker containers on the same droplet, both talking to a RabbitMQ in yet another container - but this time 
I'm looking for a bit of a different solution. 

So I did a bit of digging around, and found that Celery seems to support SQS on some level, so I could deploy a SQS, and have Celery connect to that, and drop RabbitMQ.

## SQS
I created a little terraform component to deploy a SQS, and drop the SQS endpoint, ID and ARN as outputs, but also create ssm_parameters for them, then added a fairly simple little bit of Python/boto3 code to enable Django's settings.py to pull the parameters directly from ParameterStore. 

This actually all worked fine, until I ran [Prowler](https://github.com/prowler-cloud/prowler), which pointed out that SQS was unencrypted.  
I suspect it actually wouldn't make any difference in this case, as there's nothing sensible being sent on the SQS, but for best practices, I redeployed SQS with encryption enabled.  

And then everything broke. 

Encrypted SQS has to be accessed over HTTPS (makes sense, if you think about it, as there's no point in connecting insecurely to an encrypted thing).

No matter what configuration options I tried for Celery, setting `USE_SSL` and a variety of `broker_transport_options`, I was unable to get it to connect over https.  So SQS was now no longer suitable as a queue transport. 

*Much shouting ensued.*

## Back to RabbitMQ
Bearing in mind that one of the restrictions on this current project is keeping everything within the AWS Free Tier, and that the EC2 instance type within the Free Tier is the `t2.micro`, the idea of running Docker on a `t2.micro` containing the Django web app, the Celery Worker, and a RabbitMQ broker isn't terribly appealing. 
The options are: 
1. Everything on a `t2.micro` - **No.** Like, it might work, but it's not something I particularly want to consider right now. 
2. RabbitMQ Docker container living in AWS Fargate - This is a really appealing solution, but Fargate is outside the Free Tier (*Why is that?*)
3. AmazonMQ - Rabbit Flavoured - Yum. 

I'm pretty sure when I get a bit further along with this project, I will end up with an EC2 instance for the Django webapp, but because I'll only be able to have one of them, I want to keep EC2 for something else. 

Fortunately, AmazonMQ has a Free Tier offering, of 750 hours per month for 12 months of a mq.t2.micro or mq.t3.micro (If the t3.micro is available free-tier here, why not as EC2?) -- And a choice of either ActiveMQ or RabbitMQ.  
As far as I can tell, Celery has no support for ActiveMQ (*yet?*) - so I'm choosing Rabbit. 

So I destroyed the SQS, and dropped the sqs terraform code, and replaced it with a chunk of code that creates an `aws_mq_broker` resource instead.  

I also chose this point in time to create a VPC with some subnets we'll use later. 
AmazonMQ will get deployed to a public subnet for the time being, as I still need to be able to access it from my dev PC.  Ideally I'd shove it in a private subnet, and VPN to it, but VPNs aren't within the free tier, and again, I don't want to use up my EC2 allowance on an OpenVPN tunnel either. 

I have a neat idea forming in my head as I write this about a potential way to get around this, but that'll have to wait for another day.

So with a Security Group (and NACL) limiting access to the RabbitMQ broker tied to my static IPv4 and IPv6 subnets, it doesn't actually make all that much difference that it's in a public subnet, as it's really only public to me. 
I created the admin password with `random_password` resource, and stored its value in AWS Secrets Manager, and the username and URLs in Parameter Store. 

Whilst the AWS provider for Terraform contains the resources to create the broker, it doesn't have the resource types for managing RabbitMQ.  Fortunately there is a RabbitMQ provider for terraform (https://registry.terraform.io/providers/cyrilgdn/rabbitmq/latest/docs).

```hcl
provider "rabbitmq" {
  # Configuration options
  endpoint = data.aws_ssm_parameter.rmq_console.value
  username = data.aws_ssm_parameter.rmq_admin_username.value
  password = data.aws_secretsmanager_secret_version.password_version.secret_string
}
```

So I can now create the provider, using data from SSM ParameterStore and SecretsManager, completely removing the requirement to keep the password anywhere other than inside Secrets Manager.

Now I can use that rabbitmq provider to create a vhost, user, queues, exchanges and binding to tie it all together.  The documentation is straightforward, although seems to have been written for oldterraform with the `"${blah}"` variable format. 

So for creating a RabbitMQ User, I have this
```hcl
resource "random_password" "rmq_example_password" {
  length  = 20
  special = true
  lower   = true
  upper   = true
  number  = true
}
resource "aws_secretsmanager_secret" "rmq_example_password" {
  name = "${local.global_key}-mq-example-password"
}
resource "aws_secretsmanager_secret_version" "rmq_example_password_value" {
  secret_id     = aws_secretsmanager_secret.rmq_example_password.id
  secret_string = random_password.rmq_example_password.result
}

resource "rabbitmq_user" "example" {
  name     = "example_app"
  password = random_password.rmq_example_password.result
}

resource "rabbitmq_permissions" "example" {
  user  = rabbitmq_user.example.name
  vhost = "/"
  permissions {
    configure = "^$"
    read      = ".*"
    write     = ".*"
  }
}

resource "aws_ssm_parameter" "param_example_username" {
  name  = "/${var.environment}/celery-user-example"
  type  = "String"
  value = rabbitmq_user.example.name
}
```

All the secrets are stored in Secrets Manager, as we saw previously, and username is stored in ParameterStore. 

So I can use the `get_parameter` and `get_secret` helper functions I added to the Django project to allow things like the RabbitMQ user and password to be pulled directly from those stores now, and Celery thrives anew.

## Conclusion
I'm absolutely flabbergasted that I wasn't able to get Celery to connect over SSL.  I suspect I may revisit this at a later date, and if it really is a bug, look into it more, but for now, I'm pretty happy with Rabbit-flavoured Amazon MQ. 

This is one of those things where you discover just how much power there is in Terraform.  I could've configured the RabbitMQ queues and users and exchanges equally well with Ansible, but then I'd be adding another tool, and another stage.

Look out for a blog in a week or so with some more information on Prowler for AWS Security assessments.

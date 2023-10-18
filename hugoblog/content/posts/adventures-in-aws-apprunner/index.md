---
title: Adventures in AWS App Runner 
date: "2022-09-03T01:10:01.123Z"
description: "Getting started with a relatively new AWS service"
---

So, a few months back, I applied for the AWS Community Builder (CB) programme, and this time, I was accepted.  This is the first in a series of new AWS articles I'm working on.

Within a day of joining the CB Programme, I had a challenge, and the source for this article.

[Mentor Match](https://github.com/jonodrew/mentor-match) is an application originally written for the UK Civil Service LGBTQ+ network. It's currently deployed on Heroku, but given that Heroku have recently announced plans to discontinue their Free Dyno's plan (GRRRRRR!), I've been working on some Terraform code to deploy it to AWS. 

Mentor Match is already Dockerised, so options for deployment are pretty varied.  
On the 'complex' end of the spectrum, we have Elastic Kubernetes Service (EKS), but that's out of scope for both this article, this deployment, and any budget :D. 

There's also AWS Elastic Container Service (ECS) (and Fargate) - Fargate is by far one of my all-time favourite AWS offerings.  It's pretty straightforward to take a docker-compose file and convert it into a Task Definition to run an application on ECS. 

We could also deploy an instance on EC2, install Docker engine, and docker-compose, then just deploy the application that way. 

Most interesting of all, there's the relatively new [AWS App Runner](https://aws.amazon.com/apprunner/).  App Runner was released as General Availability in May 2021, but this is my first time using the service. 

Gotta say, I am impressed.  I wrote the terraform in an evening, without having the actual AWS account to test against, so it was mostly a case of 'try what usually works' and hope for the best. 

We start, assuming that being a fairly fresh account, we're to create a VPC, so because I hate the tedious crap associated with making VPCs and subnetting, just gonna import the `terraform-aws-modules/vpc/aws` module. 

This VPC is actually only going to be used for AWS Elasticache for Redis, because that's a requirement for the MentorMatch application .  Because it only needs a Redis, and managing an EC2 instance isn't what I particularly want to do, Elasticache is a pretty neat choice for a single instance of `cache.t4g.micro`. 

So configuring the VPC with `intra_subnets` only, no Public Subnets with Internet Gateway, no Private Subnets with expensive NAT Gateway. (This decision comes back to haunt me later.)

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "mentor-match-vpc"
  cidr = "10.0.0.0/16"

  azs           = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
  intra_subnets = ["10.0.1.0/26", "10.0.1.64/26", "10.0.1.128/26"]

  enable_dns_hostnames = true
  enable_dns_support   = true
  enable_nat_gateway = false
  enable_vpn_gateway = false

  tags = {
    Terraform   = "true"
    Environment = "dev"
    Project     = "mentor-match"
  }
}
```
Into the intra subnets, I [launch](https://github.com/tomoconnor/mentor-match/blob/iac-deployment/iac/terraform/elasticache_redis.tf) an `aws_elasticache_cluster` with a single instance of Redis 6.2. Using my current favourite pattern of using `random_password` resource to create a password, store it in AWS Parameter Store (as a SecureString), and also use it for the admin password for the `aws_elasticache_user` resource.
Ideally Secrets Manager is better, but Parameter Store is cheaper. 

Next up, is App Runner, and interestingly, App Runner can integrate with a VPC, to some extent.  In fact, it's rather ingenious. 
Behind the scenes of App Runner, unsurprisingly, is AWS Fargate.  Given that deployment methods for App Runner are either a Docker Container or Code, and the code mechanism builds you a Docker container, finding out that it's Fargate under the hood shouldn't be a surprise. 
App Runner deployments are launched into an AWS-owned VPC, with an integrated (hidden to user interaction), Network Load Balancer. If you deploy an App Runner VPC Connector, feeding it the requisite Subnets and Security Groups, you can effectively cross-connect to your VPC, for application backend access to RDS databases, or APIs running in private subnets, or in this case, AWS Elasticache. 

The deep-dive into how this all works is fascinating, but much better explained within [AWS' own blog on the topic](https://aws.amazon.com/blogs/containers/deep-dive-on-aws-app-runner-vpc-networking/). 
The upshot is, that despite being connected with the customer-owned VPC, internet traffic goes out via the App Runner VPC, whilst application specific VPC traffic is handled elegantly by 'Hyperplane', transparently to the customer who deployed their Application with App Runner. 

Putting all this together, we now have subnets where Redis lives, the application front end deployed in App Runner, communicating happily. 

I did a little bit of the nice-to-have stuff, like adding a domain via Route 53, but also connecting the deployed App Runner service to Route 53 with `aws_apprunner_custom_domain_association`, and got the frontend of the application accessible from the internet. 

Now, remember earlier I said that choosing totally isolated subnets would come back to bite me? 
Part of Mentor Match is an asynchronous task queue worker, using Celery, and talking to Redis on the backend.  However, given that the Celery backend workers have no web interface (nor would you want them to!), App Runner isn't a great choice for deploying these.  But also, because the entire application is already dockerised, deploying just a tiny instance of Worker to ECS/Fargate on its own should be pretty painless, right? 

So, I write the code to deploy an ECS cluster, and set up the Task Definition to pull the worker image out of ECR, and run it in a Teeny Little Fargate task, and because everything is terraformed, when it comes to the Environment Variables bit of the Task Definition, I can do stuff like this: 
```hcl
   environment = [
          {
          name  = "SERVICE_URL"
          value = "https://${aws_apprunner_service.mmweb.service_url}"
          },
          {
            name = "FLASK_ENV"
            value = "development"
          },
          {
            name = "REDIS_URL"
            value = "redis://${aws_elasticache_cluster.redis.cache_nodes.0.address}:${aws_elasticache_cluster.redis.cache_nodes.0.port}/0"
          }
        ]
```
But it doesn't work when I come to deploy it, There's an odd error message about being unable to access ECR, and suddenly it occurs to me why this is, because the subnet that we're running in (same old intra subnet that Redis lives in), has no access to the rest of the internet in any way. 
So I need to deploy some VPC Endpoints. Quite a few. 
2 for ECR, one for the API, one for DKR (Docker).  3 for ECS and management of ECS (ecs.agent, ecs.telemetry, and ecs), one for Cloudwatch Logs, and a VPC Endpoint Gateway for S3 (because ECR image layers live in S3).
https://github.com/tomoconnor/mentor-match/blob/iac-deployment/iac/terraform/vpce.tf

This project has inadvertently (by my desire to avoid costly NAT Gateways) become accidentally more secure than it needs to be.  I could probably have gotten away with deploying either everything in Public Subnets (but ick.), or deploy public subnets for the Workers to live in, and everything else in Intra or Private. 

Anyway. Where was I? 
App Runner.  I reckon for me this shows real potential.  I'm currently still Angry With Heroku for discontinuing their Free Dynos.  I've had to move a few projects off their free tier infrastructure to elsewhere over the past couple of weeks.  I reckon that for some use-cases, App Runner will prove really useful. I highly recommend having a look at the Deep Dive linked above, there's some really fascinating insight in there to how this service works, that also hints at the under-the-hood information about other bits of services.

Usually, when I'm playing around with a new service, I like to do the first deployment as ClickOps, and get the feel for it via the AWS Console.  This time around, I kinda had my hand forced to doing it as Terraform First, not having had access to the AWS account for deployment for a few days.  It's been an interesting, yet reassuringly pleasant experience that although App Runner is *probably* not aimed directly at me as a potential customer, the capabilities for managing it via Terraform are still mature and usable. 












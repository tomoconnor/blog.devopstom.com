---
title: IPv6 Only EC2 - Is it now a reality? 
date: "2022-01-31T19:00:01.123Z"
description: "Yesnaby."
---

Short answer: No. 

Actual answer: Nearly, but not quite.

My initial curiosity around this was the following passing thought: 
> “If EC2 supports IPv6 Only instances, and AWS Systems Manager Agent supports IPv6, then can I have access via Session Manager and an Egress-Only Internet Gateway – and remove the requirement for a costly NAT gateway for private subnets?”

Let's break that down into some smaller tasks. 
* Create an IPv6-only Subnet with a `::/0` route pointing at an Egress-only Internet Gateway (EIGW)
* Create an instance in that subnet.
* Enable AWS Systems Manager/Session Manager via Quick Start (it's the easiest way to get going).

The IPv6-only subnet is easy, and the EIGW too. 

The first unexpected error I encountered was that *“IPv6 addresses are not supported on t2.micro”* – Which was a surprise, to say the least. The t3.micro does, but that’s not in the free tier (for this exercise, I don’t mind that, but it’s worth bearing in mind.)

Hopefully the Free Tier will be updated to include `t3.micro` as `t2.xxxx` generations get phased out? 

With a new EC2 instance running, and in an IPv6-Only subnet, behind an EIGW, there's no way to access it, short of enabling EC2 Serial Console. 

So, I launched a second instance, in a second IPv6-Only subnet, but behind an Internet Gateway (IGW), so I can SSH to it over IPv6.

I could’ve created a dual-stack subnet, but where’s the fun in that? Then all I had to do was add a Security Group rule allowing SSH access in to the first instance from the second one. 
Basically creating an IPv6 Bastion Host.

First things first, let’s update the instance with the usual apt-get update. Connections to security.ubuntu.com are fine, but we get the following error:

```
Could not connect to eu-west-1.ec2.archive.ubuntu.com:80 (54.229.116.227), connection timed out 
Could not connect to eu-west-1.ec2.archive.ubuntu.com:80 (34.253.189.82), connection timed out 
Could not connect to eu-west-1.ec2.archive.ubuntu.com:80 (54.246.214.20), connection timed out 
Could not connect to eu-west-1.ec2.archive.ubuntu.com:80 (54.229.225.193), connection timed out 
Could not connect to eu-west-1.ec2.archive.ubuntu.com:80 (34.253.229.19), connection timed out 
Could not connect to eu-west-1.ec2.archive.ubuntu.com:80 (34.241.117.189), connection timed out
```

That’s slightly concerning, and something of a bellwether of things to come.

Ok, we can work around that, at the slight expense of connecting to a non-AWS ubuntu mirror. 
We’ll just use [HEAnet](https://www.heanet.ie/), who’ve had an ubuntu mirror on IPv6 for as long as I can remember.

```
deb https://ftp.heanet.ie/mirrors/ubuntu/ focal main restricted universe multiverse
deb-src https://ftp.heanet.ie/mirrors/ubuntu/ focal main restricted universe multiverse

deb https://ftp.heanet.ie/mirrors/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://ftp.heanet.ie/mirrors/ubuntu/ focal-updates main restricted universe multiverse

deb https://ftp.heanet.ie/mirrors/ubuntu/ focal-security main restricted universe multiverse
deb-src https://ftp.heanet.ie/mirrors/ubuntu/ focal-security main restricted universe multiverse

deb https://ftp.heanet.ie/mirrors/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://ftp.heanet.ie/mirrors/ubuntu/ focal-backports main restricted universe multiverse
```

The interesting (yet unsurprising thing) is that the new instance does have an IPv4 address, it’s just not usable for anything other than access to the IMDS as it has an IP of `169.254.219.27/32`
IPv6 IMDS access is possible, but I didn't enable it. 

To get back on track, let’s see if AWS SSM can connect to the instance, and we can use that for remote access instead of bastioned SSH. 
In the logfile `/var/log/amazon/ssm/amazon-ssm-agent.log`, we can see this:
```
2022-01-24 12:03:57 INFO [ssm-agent-worker] Entering SSM Agent hibernate - RequestError: send request failed
caused by: Post "https://ssm.eu-west-1.amazonaws.com/": dial tcp 52.95.125.3:443: i/o timeout
```
Ah.  That's a bit of a problem.  That implies that there's no IPv6 records for `ssm.eu-west-1.amazonaws.com`. 

**Can we create a VPC endpoint to get around this limitation?**

No.

> `VPC Endpoints do not support IPv6-native subnet subnet-09e6740a5b8c3c0fa.`

So, that’s the end of SSM, so if we do have IPv6-Only instances, we’ll always need either a ‘public’ IPV6 subnet with a bastion, or a dual-stack subnet with a bastion in.

## What else doesn't work:

Well, a surprising number of sites don’t have AAAA records for their services. GitHub is slowly rolling out IPv6 support it seems, so GitHub Pages works, but you can’t do a git clone from a repository over IPv6.

**Can we start a thing from a public docker repo?**

``Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)``

No. 

To fix, edit `/etc/docker/daemon.json` and add this block

```json
{
    "registry-mirrors": ["https://registry.ipv6.docker.com"]
}
```

And restart the docker daemon.

**Can I pull a docker image from ECR (Elastic Container Registry)?**

No.

```
dig -t AAAA 1xxxxxxxxxx5.dkr.ecr.eu-west-1.amazonaws.com

; <<>> DiG 9.16.1-Ubuntu <<>> -t AAAA 1xxxxxxxxxx5.dkr.ecr.eu-west-1.amazonaws.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7786
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;1xxxxxxxxxx5.dkr.ecr.eu-west-1.amazonaws.com. IN AAAA

;; ANSWER SECTION:
1xxxxxxxxxx5.dkr.ecr.eu-west-1.amazonaws.com. 26 IN CNAME nlb3-3090dfc6de4af029.elb.eu-west-1.amazonaws.com.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Mon Jan 24 15:38:32 UTC 2022
;; MSG SIZE  rcvd: 113
```

So how can I get application code onto this EC2 instance? SCPing it over feels like such a backwards step.

Even if I did manage to deploy an application, I’d have to either proxy or tunnel connections to other AWS Services.

```
ubuntu@i-086ec760bbd5c2961:~$ host -6 sqs.eu-west-1.amazonaws.com
;; connection timed out; no servers could be reached
```

Amazingly (after all this, **any** service that supports IPv6 connectivity is amazing), PyPi, the Python Package repository worked out of the box, I didn’t need to reconfigure anything!  That was just to install `aws-cli` -- however...

aws-cli from inside the instance is similarly fraught.

```
2022-01-24 17:34:03,101 - MainThread - botocore.endpoint - DEBUG - Sending http request: <AWSPreparedRequest stream_output=False, method=POST, url=https://ec2.eu-west-1.amazonaws.com/, headers={'Content-Type': b'application/x-www-form-urlencoded; charset=utf-8', 'User-Agent': b'aws-cli/1.22.41 Python/3.8.10 Linux/5.11.0-1022-aws botocore/1.23.41', 'X-Amz-Date': b'20220124T173403Z',
...
urllib3.exceptions.ConnectTimeoutError: (<botocore.awsrequest.AWSHTTPSConnection object at 0x7ff16f5ada30>, 'Connection to ec2.eu-west-1.amazonaws.com timed out. (connect timeout=60)')
```
I tried creating a dual-stack Application Load Balancer (I also tried a Network Load Balancer) in front of my IPv6 instance, but for some reason it got stuck on Provisioning, even overnight, with no error messages. 😞

Interestingly enough, I was playing around with deploying an IPv6-Only instance with Terraform over the weekend and found that if you are configuring a Network ACL (NACL) with Terraform, and want to specify ICMPv6, you must specify its Protocol Number, since “icmpv6” or “icmp6”, which is how you often see it in the Linux OS, is not supported.


```hcl
public_inbound_acl_rules = [
    {
      rule_number     = 92
      rule_action     = "allow"
      from_port       = 1024
      to_port         = 65535
      protocol        = 58 # ICMPv6
      ipv6_cidr_block = "::/0"
      icmp_type = 129 # echo reply
      icmp_code = 0
    },
    {
      rule_number = 94
      rule_action = "allow"
      from_port   = 1024
      to_port     = 65535
      protocol    = "icmp"
      cidr_block  = "0.0.0.0/0"
      icmp_type = 0
      icmp_code = 0
    },
```

I am rapidly running out of use-cases where I can see IPv6-Only instances being any use at all.

To summarise: 
* SSM Session Manager cannot connect via ipv6, so we need to have a bastion host still :(  
* default apt/yum repositories on Amazon Linux and Ubuntu need replacing with ones which do support IPv6. 
* aws-cli does not work, neither does `boto3`, because seemingly none of the `amazonaws.com` endpoints for the API have a listener on IPv6.

So, I’m not sure what the use of an IPv6-Only instance is right now. 

It almost feels like it was rushed out in time for re:Invent. 

Perhaps I’m the first person to try and use it in anger?

GitHub could do with enabling IPv6 on the endpoints for raw.githubusercontent.com and the main github.com repository endpoints.

Docker’s IPv6 repository seems to work well, though.

It’d be awesome if AWS made some changes. 
1. Full support for IPv6 access to every API under `*.amazonaws.com`
2. Make `t3.micro` part of the Free Tier as well as `t2.micro`
3. VPC Endpoints that could attach to IPv6-Only subnets.
4. Session Manager support (although I suspect this would resolve itself once (1) is fixed.
5. Make `eu-west-1.ec2.archive.ubuntu.com` dual-stack.

### Footnotes:
Yesnaby is an area of Orkney, but [The Meaning of Liff](https://tmoliff.blogspot.com/2011/06/yesnaby-n.html) gives it the meaning of "Yes, maybe" which means "No".
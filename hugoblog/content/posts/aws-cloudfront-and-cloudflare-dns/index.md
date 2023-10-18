---
title: AWS Cloudfront and Cloudflare DNS
date: "2022-01-23T21:11:01.123Z"
description: "Multi-cloud Free Tiers"
---

One of the things that I find most surprising about AWS's Free Tier, is that it doesn't include Route 53.  

I mean, you get a very ample amout of DynamoDB storage (25GB), a million Lambda requests a month - always free.. And 12 months of free EC2 micro instance, and RDS -- enough to build out a small project or home lab without incurring fees (Be careful, though, and always set up billing alerts.).
But not to offer any free tier for Route 53 seems rather odd, a web service without DNS is kinda anaemic.

Cloudflare, on the other hand, offer a really nice free DNS service - with a nice API to interact with it.  An API that's [well supported](https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs) by Terraform.
A lot of the things I build in my spare time have are fronted by Cloudflare, and pretty much every domain I own uses their DNS. Including this one. 

So on Saturday afternoon, I thought I'd have a play around with terraforming a Cloudfront CDN to front a S3 bucket, but to use Cloudflare's DNS instead of Route 53. 

## The Plan:
* Create a S3 bucket in eu-west-1
* Create an ACM Certificate for `cdn.wibblesplat.com` in us-east-1 (ACM certificates for Cloudfront need to live in us-east-1)
* Create the ACM Domain Validation records in Cloudflare's zonefile for `wibblesplat.com`
* Create a Cloudfront Distribution for `cdn.wibblesplat.com` using the ACM cert, and origin of the S3 bucket.
* Add a CNAME (with no proxying) to point `cdn.wibblesplat.com` at the distribution domain name from Cloudfront.

My code for this solution is [here](https://github.com/tomoconnor/terraform-cloudfront-and-cloudflare)

You'll need to set up a Cloudflare API Token with Edit Zone permissions for your selected zone, or else you get this error:
```
╷
│ Error: no zone found
│ 
│   with data.cloudflare_zone.primary,
│   on cloudflare_data.tf line 1, in data "cloudflare_zone" "primary":
│    1: data "cloudflare_zone" "primary" {
│ 
╵
```

I haven't actually tested this with Cloudflare in Proxy mode, because two layers of CDN seems ridiculous - as if Cache Invalidation wasn't a hard enough problem already.  So really it's just using Cloudflare for DNS - and would work equally well with any other Terraformable DNS provider. 

My main reason for doing it this way, rather than having S3 in 'static website mode' is that I still, to this day struggle to get the fine balance of ACL controls and permissions correct.  Doing it this way with Cloudfront and an OAI means at least I can leave the bucket private, and also have more control in future, if I want to do anything with Lambda@Edge, for example. 

If anyone from AWS happens to read this, It'd be lovely if the Free Tier included one public hosted zone on Route 53. 
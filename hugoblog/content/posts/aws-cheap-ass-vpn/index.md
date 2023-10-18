---
title: Building a Cheap-ass VPN for AWS 
date: "2022-02-06T01:10:01.123Z"
description: "Home/Office to VPC connectivity for a lower cost"
---

For me, one of the most appealing features of AWS is the ability to connect securely to an entire Virtual Private Cloud (VPC) of resources over a Virtual Private Network (VPN).  

Unlike the VPN technology that's so commonly and popularly marketed on youtube adverts 🙄, a VPN to a VPC is a secure private link between your router and a virtual router within AWS.

AWS offer a Site-to-Site VPN as a managed service, but configuration can be a bit of a minefield to the uninitiated.  
Ideally you need to have a router capable of BGP, or else you'll have to create a lot of static routes. 

There's a lot of components to a AWS Managed Site-to-Site VPN: You need to configure a Customer Gateway, and Virtual Private Gateway, and the Routes via BGP, plus the IPSec config. 

It also costs **$36.50** per tunnel per month for an always-on tunnel.

What I've been playing with recently is a Terraform module for deploying an ultra-cheap VPN based around Strongswan running on a `t4g.nano` instance.

The `t4g.nano` is the current cheapest instance type available on AWS. They only have 0.5GB of RAM, but do have 2 ARM/Gravitron vCPUs.

They only cost **$3.06** a month for On-Demand usage, so that's over $33 less than the Site-to-Site Managed option.

Data Transfer needs to be considered, but Inbound transfer is free, so in order to make it up to that $33/month you're paying for on Managed VPN, you'd need to do ~360GB outbound data per month.

Magical.

## Terraforming The VPN
I created a module which encapsulates the following resources that are required to make the VPN function. 
* `aws_eip` - Need an Elastic IP (EIP) to simplify local-end firewall configuration. 
* `module_ec2_instance` - using the [terraform/aws/ec2_instance](https://registry.terraform.io/modules/terraform-aws-modules/ec2-instance/aws/latest) module.
* `aws_security_group` - Comprises the following inbound/outbound rules:
  * SSH-In (for initial config) 
  * ISAKMP In (udp/500)
  * NAT-T In (udp/4500)
  * ESP In (IP Protocol 50)
  * AH In (IP Protocol 51)

The `cheap_ass_vpn` [module](https://github.com/tomoconnor/cheap-ass-vpn) outputs the endpoint EIP for ease of connecting to the local firewall/gateway

All Security Group rules are IP restricted to a static IP address via a variable `allow_list_ipv4_prefix`

The Outbound rules are mostly identical, except they also allow HTTP and HTTPS for software updates.

One final rule in the Outbound block allows all traffic out to the rest of the VPC. 
```hcl
resource "aws_security_group_rule" "all_out_to_vpc" {
  type              = "egress"
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = [data.aws_vpc.this.cidr_block]
  security_group_id = aws_security_group.allow_vpn_traffic.id
  description       = "All Traffic out to VPC"
}
```

## VPN Config with StrongSwan
The configuration of the VPN terminator / endpoint / ec2 instance was by far the hardest part of this entire project. 

List of things that caught me out:
* Old Documentation for Strongswan vs new configuration via Swanctl. 
* Swanctl doesn't load connections and credentials on reload out of the box - I had to add a post-up script command to tell it to load them.
* The Mysterious `updown` command - possibly because of earlier documentation issues, I didn't realise that the correct syntax for the `updown` script was `/usr/sbin/updown iptables` -- so it never created the policy-based routing rules in iptables. 
* Masquerade (NAT) mode - also iptables, but to allow the traffic from my local network to reach other subnets in the VPC.
* Don't forget to set `net.ipv4.ip_forward` or else nothing'll work.
* Disable `source/destination check` in the EC2 instance configuration.
* Network ACLs will trip you up on every possible opportunity - make sure you remember to add intra-subnet rules if required. 
* `iptables-persistent` is your friend
* iptables logging is extremely noisy. 

That said, it does just work.  
Configuring my Fortigate to accept the tunnel was surprisingly straightforward, I think it only took me 4 attempts at a SA proposal to find a combination that both ends were satisfied with.  

I ended up with IKEv2, PSK and the following settings:
```
Phase 1: AES256-SHA256 dh14,5
Phase 2: AES256-SHA256 dh14,5
```
Once I had this working, I created a custom AMI in my account, and fed that value back to the Terraform VPN Module. 

I may have a poke around and see what other combinations also work, but for now I'm just rejoicing in the fact that I have a working site to site VPN that doesn't require a Costly VPN Gateway managed service.

The next thing I'm thinking about doing is making a physical switch with an ESP8266 so I can have a nice big toggle switch for enable/disabling the VPN. (Realistically, all that ESP will do is call a Lambda which will start/stop the remote-end VPN instance.)
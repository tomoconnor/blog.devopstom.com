---
title: "Lightning Post: Dumping MS DNS to Bind"
date: "2013-05-28T19:59:58.000Z"
description: "Too short for a full blog post"
warning: ancient
---

This is the first in a series of Lightning Posts, short snippets that I don't really have the time to write up into a full post, but they're interesting nonetheless.

Lightning Post 1: How to export DNS data from Microsoft DNS to a zone file.

> "Why'd you wanna do that?", I hear you cry.

Well, It's entirely possible to use BIND (or PowerDNS, for that matter) as a DNS server instead of the integrated MS DNS service that's bundled with Windows Server.

When you create an Active Directory, a process creates some service records, like `_ldap._tcp.ForestDnsZones.yourdomain.tld` and so on.

Well, these aren't impossible to create by hand, but it's nice to have a dump for these things at least initially. 

So: 

Login as Administrator, and load up a Powershell console:

 

`dnscmd YourDomainController.tld /ZoneExport YourDomain.fqdn.tld YourDmain.fqdn.tld.txt` 

Then you can look in `%windir%/system32/dns/*` and find the txt files  containing your zone data.

 

Done.
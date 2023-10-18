---
title: "A manifest for Agile Devops"
date: "2011-12-20T17:00:00.000Z"
description: "Does scrum really fit for DevOps?"
warning: ancient
---

I’ve decided.  We need to start doing [points poker](https://en.wikipedia.org/wiki/Planning_poker) here at Baseblack if we’re going to carry on this Agile DevOps thing. 

I’ve got to admit, the first time I came across the Agile methodology was quite late in my career.  In the past, prioritisation of “operations” projects was reasonably first come first serve, or by order of priority (frequently, business need, and seldom operational requirement).

For software development teams, Agile is a pretty good, native fit.  The concepts embodied by stories and sprints fit a development team very cleanly.  When it comes to systems administration and engineering, or what I’ve come to refer to as DevOps, Agile can be a bit more awkward initially. 

Operations teams across the globe will tell you that their tasks are intrinsically more “sprawly”, and that interconnections between tasks are frequently more complex.  

The truth of the matter is, that frequently there is no simple and sensible way to break up a task into entirely unconnected subtasks.  Something which can bugger up Agile, if you’re too hard and fast with the requirements and rules by which you play the game.

Pretty early on in this new job, I started looking at the previous DevOp Engineer’s puppet manifests. They were *mostly* ok, but with some absolute crazy meatballs thrown in for good measure.  

It’s actually a common fault of Sysadmins to want to throw out the previous team’s work and start afresh, but in this case it actually was easier to start fresh than repair the foibles and cockups of the old code.   

Given that I’d already spent 3-4 days reading and trying to interpret the state of the system, and it was blatently apparent that there were too many bits of “wouldn’t it be cool if we hacked this in to make it do X”, and not enough actual hard and fast config to make things work.  

I’ll put that one down to my predecessor not being very puppet-savvy.

One of the big reasons this sprint overran was that the discovery process (first 3-4 days) was mostly involved with exploring the state of the systems, and what we wanted to accomplish.  In the old manifests, there were huge chunks of code that installed numerous applications, which would be easier to manage and integrate if modularised.  

A good proportion of time in the implementation phase went into creating lots of individual modules for various applications and packages.

As I was saying earlier about interconnected tasks, this wasn’t just a Fix Puppet sprint.

The background to fixing puppet was to enable the faster building of new machines from unboxing to users logging in.  

There were some massively weird problems with the internal DNS, using Bind9, and the old DHCP server was prone to some peculiar lease issues, and it was running on a physical VM host, when it probably ought to have been a VM guest.  Fixing DNS would best be done whilst fixing DHCP.  Fixing DNS meant installing PowerDNS, which in turn means installing Postgresql. Setting up DNS Slaves means installing PowerDNS on multiple servers and configuring Postgres replication.

There’s no way that I’m building out multiple copies of anything without Puppet, so there’s  the first bit of recursive loop.

The way to untangle this is to realise that puppet doesn’t need a puppetmaster to run manifests.  All you need to do is write the puppet configs and then use the puppet agent itself to run the manifests from files.  You can then use that to bootstrap a puppetmaster, or a DNS server, or just get a sense of how it will all fit together when you do the final server buildout.

I’m going to leave this here.  I think the general conclusions to draw are the following. 

1. Agile is great.. It doesn’t fit all teams, and it’s worth trying.  If it doesn’t fit, no worries.  If it does, cool.
1. Planning is the biggest stage of any project, or at least, should be. 
1. Infrastructure projects shouldn’t be forced into the traditional Agile Sprint, because they tend to become a lot more sprawly on investigation of the actual problem than they look at first glance.

I’m about to post the articles on [Postgres replication](/postgres-replication-91), and the technical portion of this article.


---
title: "FakeRAID and Virtualisation"
date: "2013-11-14T19:59:58.000Z"
description: "Here Be Dragons"
warning: ancient
---

I've been tinkering with Virtualisation quite a bit recently.

For a new project, without an allocated budget, I was asked to provide some simple Virtualisation capability, to hold them over until they get budget approval, and can buy their own hardware.

I managed to rescue a Dell R510 server from the scrap heap, only to discover that it contains a Dell S300 "FakeRAID" card, that's not supported by Linux (so KVM, Xen et al are out).  It's also not supported by VMWare ESXi, so that's out.  The only OS that *is* supported is Windows, and Windows Server.

Dell's driver set for the card suggests that Server 2008R2 is the latest, supported server.  It actually turns out that not only is the driver compatible with the installer or Server 2012, but given the way the local storage array is presented to the Server OS, that it's also compatible with Hyper-V.

So from spending a few weeks digging into VMWare vCenter and so on, to now having one host running Hyper-V, it's been quite an interesting journey.  Not only do I now have a working server (utilising a FakeRAID card - Something I'm not entirely happy with, but it's definitely better than nothing), but also the ability (much to my surprise), to run Linux VM guests on it.

Installing Windows Server 2012 on this R510 is surprisingly easy, but does need the Dell S300 drivers to be on a USB stick ready for the installer to ask for them.  When you get to the bit where you partition the disk, you need to provide a path to the driver .inf file, and then after that, it just appears as one big-ass hard disk, ready to be partitioned.

I allocated a 70GB chunk for the OS installation, and left the rest to be allocated later on when I installed Hyper-V.

I'd heard before that Hyper-V supports Linux quite happily, but never actually had a chance to experiment with it. 

Suffice to say, so far, I'm very impressed.  Not so impressed I'd ditch VMWare for the grand scheme of things, regarding implementing Virt. around here, but impressed enough to hand this server over to the team for them to use.

In the next few weeks, I'll be trying to get my hands on the vCloud (Ops, Automation, Chargeback, etc.. ) toolsuite, for some further experimentation.  Looking forward to it enormously.
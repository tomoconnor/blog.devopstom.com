---
title: "COTS Series: Enterprise Patch Management"
date: "2017-05-15T19:59:58.000Z"
description: "Update all the things!"
---

This might read a little bit like an advert, but it’s not.  I’m not getting paid for this.  I like to talk about things that make my life easier, and there’s going to be a few more in this mini-series.

I’m just gonna come out and say it.  Patch Management is hard, but it needn’t be.  

The following comes from my personal experience of managing patches and software updates in an enterprise environment for the last 4-odd years.

I’m primarily targeting Windows here, because it seems to generate the most patches, and seems to be the biggest embuggerance to keep updated.  Linux systems tend to be easier - on Debian systems, I use `apticron` and `unattended-upgrades`.  For Redhat/CentOS et al, there’s `yum-updatesd` / `yum-cron` and friends.

In the beginning, there was WSUS.  That’s Windows Server Update Service - basically a role you install on Windows Server 2012.  It looks at your Active Directory, and finds your systems, then identifies patches and downloads them.  Then you stage patches into sets and push them out to your servers and workstations.  

Or at least, that’s what you think you’re doing.  There’s not really any very good way with native WSUS to check that it’s doing that, because the reporting is utter pants.

I have struggled with WSUS in the past for days on end to try and ensure that it’s behaving itself, and patching the right patches onto the right systems.

And then you come into work one day, and discover that WSUS has corrupted its internal database, and it’s now lying to you about the state of patching on your enterprise.


So about a year ago, I removed it from our service catalogue, and replaced it with a rather nifty application that I discovered at the Infosec Europe conference/trade show.

Shavlik (although, seemingly now called Ivanti but I’ll probably forget and refer to it as Shavlik).

The basic premise of the idea is this.  You install the console onto a server, and it goes off and hunts down things on your domain.  It can also patch your hypervisors, which is dead handy, because VMware’s update utility also blows.

When it’s got a list of stuff on your domain, it’ll go and figure out what patches are available from software vendors (not just Microsoft, but a whole raft of others including Adobe, Firefox, Google, 7zip, and many many more).

Then you can create schedules - although the defaults are pretty sane, something like:

* “Check for and Download fresh updates every hour”
* “Scan the domain every morning”

If you want to schedule automatic patch deployment and reboots, you can do this too, with a little dialog box to pop up to logged on users saying something you can customise.  

There’s the features that WSUS lacks, like the ability to actually work, and patch stuff; and there’s the reporting too, so you can easily generate pretty graphs and pictures for people who are interested in that kind of thing.

I have evaluated a number of patch management tools in the search for one that works, and the biggest selling point on any of them, for me is the ability to answer the following question:

*“Does it Just Work?”*

I don’t want to be forced to install agents on my endpoints and servers - they’re busy enough doing the things that they do without being cluttered up with any further software.

I don’t want to have to babysit the process.  I like to set it, and get on with doing other things, and have it notify me at some point when it’s finished.

I don’t want to have to trigger it to hunt for new updates, or new systems that need patching - that kind of thing *has* to be automatic, and it is.

Most importantly, I don’t want to have to struggle with the software to get the information that I need, out of it. 

Fortunately, Shavlik/Ivanti  does meet these requirements, and it just fucking works.

Whilst there is an argument perhaps, in some the minds of some along the lines of “Why should we pay for this, when WSUS provides it for free” or possibly “Well, we could write our own scripts”.  I suggest perhaps that you bear in mind the amount of staff time it takes to babysit WSUS, and debug edge cases in your scripts, and when you count up the hours over a few months, you’ll probably find it would have paid for itself.  


**If there is one easy take-home message from the recent cyber attacks, it is that there is absolutely no excuse not to have up-to-date software on enterprise networks.**

As systems administrators, you have the future of your employer’s business in your hands, and you have to do everything you can to ensure that there is a future for it.
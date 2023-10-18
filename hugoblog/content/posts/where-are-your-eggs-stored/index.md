---
title: Where are your eggs stored?
date: "2011-04-22T23:46:37.121Z"
description: A long post about not relying on one provider.
warning: ancient
---
When I was growing up, one of the things that particularly interested me about the English language were idioms and proverbs.  

I think today, whilst many are still suffering the effects of the week, we should look a little more closely at one particular proverb, and perhaps its effective meaning today.

"Don't put all your eggs in one basket" :- This phrase is commonly (and some might say, [incorrectly](https://herbison.com/herbison/broken_eggs.html)) attributed to Miguel Cervantes (in Don Quixote), but some sources have reported its usage as early as 1600.  Also of little surprise is that many other historical cultures had similar phrases.

OK.  We've established that historical peoples knew about having redundancy in their Ova storage and distribution methods, so pray-tell, why has this fantastic tradition been forgotten?

I am, of course, talking about the recent (21/04/11) Amazon EC2 and related services outage.  [Reddit, Foursquare and Quora](https://web.archive.org/web/20110518133927/http://www.informationweek.com/news/cloud-computing/infrastructure/229402054) are the big 3 companies who've been very public about their outage, but I wonder how many smaller companies and startups who rely on Amazon services for their server needs are also ending up out of pocket (due to lost revenues), or simply offline.

So the problem is this.  Amazon are fucking cheap, in comparison to pretty much any other VPS solution.  This is a royal pain in the arsehole, from a systems engineering point of view, because Amazon also price all of their other services similarly cheap.  S3 is Seriously Cheap Storage, (they should have called it SCS perhaps).   There's also the Load-balancer and cloudfront CDN frontend, again, incredibly cheap.  There's a real movement towards building ones entire infrastructure around the Amazon cloud, and I think this is the problem.  Amazon even offer a DNS service (Route 53), so you can serve your website's DNS records from the cloud too.  

Can anyone see the problem with this?  The architecture of the intrinsic scalability of the Amazon cloud does certainly allow you to create lots of small servers for things, so you've got a webserver basket, containing a half dozen server-eggs; and another basket for database-eggs.  There's a massive problem here.  Enormous problem.  All of your baskets are inside one enormous basket.  One incredibly big basket called "the Amazon cloud".  

What appears to be happening to Amazon's cloud at the moment is one of two things:

1) People have built crap websites, or have only one egg.  If you've only got one server, and it goes down, you're screwed again.  You might as well have a dedicated server from anywhere else.  You've still got a massive Single Point of Failure, and when the worst case scenario happens, you're fucked.

2) People have lots of inter-cloud redundancy, but no intra-cloud redundancy.  This is akin to having lots of small baskets of eggs, in one big picnic hamper. 

This is actually very common.  It's trivially easy to construct a pretty big network on the Amazon Cloud, you add more EC2 compute nodes, then add some S3 storage, EBS block stores, Cloudfront CDN, oh, maybe Route 53 DNS, how about Simple Payments Service for micropayment, maybe Simple Message Queue, and that's before I get onto their database offerings.

Amazon have gone a long way to making sure that everything you could ever need for this kind of system building architecture is there, at one place.  
They're like Home Depot, only there's a greater chance of Amazon having what you want. 

**ERRR.**

There's a problem here.  I feel the same way about people who buy a 50 disk SATA array, and fill it with disks with the same batch number.  It's no surprise that if one fails, you're probably going to get another failure, caused by the same bug or hardware problem.  

If you're going for true redundancy in the face of real adversity, then you need to start putting your eggs in many separate baskets.  Globally distributed baskets. Baskets held by many different people.  

I generally approve the use of S3 for system backups, because by and large, it's fast, cheap, and pretty secure (especially if you encrypt it).  It's *really* fast if you're uploading from inside Amazon's network.  There is an epic problem though.  Say you take nightly snapshots, and upload them to S3.  One day, your server goes down, either Amazon's fault or one of a number of other reasons.  

I can see 2 enormous problems here.  Primarily, if it's a fault on the Amazon network, it may affect your snapshot storage, and the ability to access them in a timely fashion, so while your Disaster Recovery Plan may say "Download the disk image and redeploy", you may not be able to download the disk image.  Then you're screwed.

It's also possible that a disk error on the Amazon side corrupts your snapshot images, in which case, again, you're screwed.  In a subtly different way. 

Secondly, and this is a far more "doh!" problem, you may be able to locate and download your disk images, but not decrypt them, because the encryption key is stored on the primary server (also inside the backup image, encrypted).  This is easily solved.  Copy the key, print it out, and store it in an envelope in the company safe / bank deposit box / other secure location.

The biggest problem with all of this, is that there doesn't seem to be a straightforward way to share data and server instances across diverse cloud providers.  I'd like to build an universal image, and then deploy it to the Rackspace Cloud, Amazon EC2, Flexiscale, and so on, and be able to 

a) interchange data between them easily (not too hard, but would require some API glue)

b) have a global system for GSLB between them, so that if EC2 is offline, then all traffic is mopped up by the other two clouds.

c) Have a sensible "in-one-place" billing system (more API glue)

Physically, and from an engineering point of view, the biggest challenge of that lot is b.  You'd need true global redundancy, and that don't come cheap.   However, I think that's the topic for another blogpost.  

In the meantime, perhaps you should evaluate where your eggs are, and how many baskets you have.

You should worry somewhat when all of your eggs are in one superbasket.  

Then I think it's time for an ovum redistribution exercise.


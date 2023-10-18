---
title: "Why Do Corners Get Cut?"
date: "2013-03-04T19:59:58.000Z"
description: "If it ain't broke, but isn't perfect, should it be fixed?"
warning: ancient
---
How many times have you found something at work that’s not *quite* how it should be?  Perhaps you’ve got a server with “Green” drives in?  Or a cheap unmanaged switch somewhere.  Or something with a self-signed SSL certificate.  Or a linux box instead of a router.  Or a desk fan propped up behind a server, because otherwise it overheats.  Or something with a big label above that states in large, unfriendly letters **“Do not unplug.  Ever”. **

I’ve been to many different companies over the last few years, and I’ve seen some absolutely terrifying things lurking in dark corners of server rooms and infrastructure.  I’ve seen AD servers booting off desktop-grade NAS devices, or backups being stored on USB disks that didn’t pass S.M.A.R.T tests 3 years ago, and unsurprisingly, still don’t. 

Everyone who works with these types of system knows that it’s not perfect, and it’s the best they can do under the circumstances.  And this, unsurprisingly, leads to stress of one form or another, either wondering what’ll happen if/when it breaks, or the stress of what you do when it actually breaks.

It all basically stems from working within a company with unrealistic expectations of what’s feasible within the limited budget provided by senior management.  In an ideal world, we’d all have datacentres with A/B feeds, and big-ass UPSes, and fabulous glycol-chillers.  But we don’t.  Hardly anyone does.  I think Google probably do, but well, they’re *fucking Google*. 

Because it’s so much easier to reuse an old disk here, or say “*we’ll change it when we have next year’s budget*” - Here’s a dirty little secret.  It’ll never happen.  Ever.  Next year’s budget will be zapped by next year’s problems, and you know the old adage, if it ain’t broke, don’t fix it. 

It’s really a case of "*well, actually it’s not broken, but it’s not perfect, but we still can’t afford to fix it and make it perfect, so y’know, we won’t.*"

This kind of thing filters down into software architecture too.  A surprising amount. Whenever you’ve found a database table being used as a message queue, or an eval() call that’s only ever used by an internal service (!).  It’ll have been put in because of deadlines, and the struggle to meet them means that something has to be bodged, somehow. 

It’s also quite apparent in Agile teams when the project has been over-specced, or under budgeted for time (or other resources), and something falls through the cracks.  In my experience, the first to go is code reviewing.  Actually, testing is usually the first to go, but those two are pretty closely connected.

It’s similar to the BDD vs TDD mindset, where unless testing is an integral part of software development, ie, performed either before or in parallel with, application coding, then it’ll probably never happen.  Documentation too, to some extent.

Interestingly enough, I don’t think the developers or engineers are to blame for these problems.  They’re usually more than willing to do whatever is necessary to maintain the service/software/thing.  The responsibility ought to lie with whomever holds the purse strings.  

It’s all very well saying “*Our developers get the best possible software, and the best possible tools*”, but if the servers are old, running on tired disks, in an overheating room, fanned by a desktop fan, then I’m afraid that you’re not actually providing the best possible tools.  

It’s really just an illusion.
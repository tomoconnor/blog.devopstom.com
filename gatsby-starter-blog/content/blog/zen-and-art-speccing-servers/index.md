---
title: Zen and the Art of Speccing Servers
date: "2010-06-11T23:46:37.121Z"
description: 
warning: ancient
---

Say for example you want to build a new Virtualization cluster.  You've chosen the CPUs you want, and know you want 32 GB of fast shiny RAM.  

The next thing to decide on is how the hell you're gonna store your VMDK (or otherwise) images, and then store the backups and snapshots too.
So.  A typical VM Host server might be one of three choices.
For sake of argument, i'm using Dell as a vendor.

Option 1:
===========

> Dell R805, Dual AMD 2425HE, 6 cores per CPU, 2 CPUs.
> 32G of fast DDR2 ECC RAM.  

Ah. Hard disks. Bugger.

You can have only 2 disks, in the R805 chassis.  Bugger.

I'll have 2 fast SAS 300GB 6Gbit 15K 2.5" drives, in RAID 1.

Bugger.  Only 300GB of storage.  That's about enough for 3 small servers, or one big one.

Bugger.

Approx Cost: £3100


So, If i want to use the R805, i'm gonna need some kind of backend storage, be it NAS, or SAN, or an Unified Storage Device, providing NFS and iSCSI.  

Option 2:
===========

Dell R815
Dual or Quad CPU, also AMD, 8 or 12 cores per CPU.
> 32 G of RAM, again
> More disks!
> Split volumes, R1 / R5 (shame it's not R6, but there we go.)
> 2x300GB SAS + 4 x 500GB SATA
> Giving 300GB + 1.3TB
A bit better, but prohibitively expensive.
> Dual 8 Core CPU = £6208
> Quad 8 core CPU = £6698
> Dual 12 Core  = £7408
> Quad 12 Core = £8608

Bugger.

 

Option 3:
==========

> Dell 2970
> Dual 2425HE, again
> 32 GB RAM

> Option A (8x2.5" disks)
> 2 x 300GB SAS + 6x500GB SATA
> = 300GB + 2.3TB

> Total Cost : £ 5125

 

> Option B (6x3.5" disks)
> 2 x 300GB SAS + 4 x 2TB SATA
> = 300GB + 5.7TB
> Total Cost : £4705

OR
> 2 x 300GB SAS + 4 x 1TB SATA
> = 300GB + 2.8TB
> Total Cost : £4145

Right.  Now.  The interesting part is that this last server, the cost of storage alone, is only £191/TB.

One of the biggest problems associated with having large disk storage on the actual VM host itself, is the problem of not being particularly able to free up pockets of unused disk space.

Alternatively, a separate storage node would effectively allow better distribution of the storage, and exporting disks across the network.  

So let's price that up.

From Broadberry.co.uk 

(Because I like their up-front pricing, and shiny configurator)

> Supermicro chassis, Intel server mobo, Intel Xeon E5504, dual CPU, 24GB RAM
> 6x300GB 15K SAS = 1.3TB
> 6x 2TB SATA = 9.5TB
> Total Storage: 10.8TB
> Total Cost:     £6528

That's about £605 per TB.  Not ideal. 

 

But there's no real doubt that using iSCSI (or NFS) would provide masses more flexibility for the provisioning of storage for this project.  Because the initial plan involved high-availability, using IP-based network storage protocols would also allow the disk-traffic to be routed across the public internet, using some kind of VPN technology.

My gut feeling is that the best solution is a cheap(-er) server, backed onto a more expensive disk storage unit.  

I did consider pricing up a DAS array, and connecting it to one or other of the VM Hosts directly, either by FC or SAS, but then in the remote case of the failure, the disks aren't easily exportable to another server. Especially as SAS traffic can't be directly routed over the network.
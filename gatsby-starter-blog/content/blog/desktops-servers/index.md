---
title: Desktops As Servers
date: "2011-05-01T17:00:00.000Z"
description: "Hardware is cheap, downtime is expensive."
warning: ancient
---

Personally, I hate the idea of using a desktop as a server in a production environment.  I'm going to define the term "production environment" first. If you've got an environment, any environment where the service provided is relied on by anybody, for any reason, then that's a production environment.  If it's just for you, and you don't mind when it all goes wrong and the shit hits the fan, then that's fine.

**Case in point:** I've got 2 re-appropriated desktops as a pair of Domain Controllers for testing a domain deployment.  Each desktop is running Windows 2008 R2 server, and provides Active Directory, DHCP, DNS and Windows Deployment Services.  This was fine for testing, and playing around with building workstations, but the problem comes when people find out about this, and want to rely on it.  For about a week, I was experimenting with using Windows' DHCP and DNS servers for the entire office.  This was fine and dandy until there was a powercut, and neither of the desktops came back on automatically.  This is because, unlike most servers, the default ACPI configuration is to start "off", and not "last setting" or "on".

So the desktops didn't boot up, and nobody could get a new DHCP lease.  Bit of a bugger that, but easily fixed.

In the event that I ever do get this kind of scenario in the office in production, where people are reliant on the availability of the Domain Controllers for login and file sharing, then I've already got some HP Proliant servers specced up and ready to order. 

There's other problems too.  Desktop hard disks aren't designed for 24/7/365 operation, and aren't designed for a high duty cycle like that of a server.  What disk manufacturers call "Enterprise Disks" are much more sturdily built than "Desktop Disks", they're designed to work harder, at higher temperatures, with higher duty cycles, and are generally designed to be always on.  

There's also a running trend amongst high capacity desktop hard disks, where they're "Green" or "Energy Efficient".  One of the ways that manufactures implement this, is having the disk stop spinning when it's not in use, or send the entire unit to sleep.  If you have a RAID set built out of Green Disks, then you'll probably find at some point that the array ends up degraded - "broken" in layman's terms.  There's probably nothing wrong with the disk, but the disk firmware has shut it down, or put it to sleep because it's not immediately being used.  The RAID controller, software or hardware, sees this as a disk failure, and all hell breaks loose.  Especially if you have 2 of them that go to sleep, in a RAID 5 array, then you're really screwed.

Desktop motherboards are also a different breed, they're generally designed with Athlon or Intel Core processors in mind, which have a very different fetch-execute cycle to a Server-grade Opteron or Xeon.  They're kinda not really designed with server operation in mind, and are sorta "slower" or less performant than an equivalent speed server processor.

On the topic of Desktop Motherboards, they're also less built for high memory configurations, typically with 2 or 4 DDR3 slots, and their capability to accept ECC (Error Correcting Code) RAM is very variable.  Some do, some don't.

I like build-in redundancy, and defence-in-depth, especially when building server solutions.  I like having ECC RAM, it's more expensive, but does protect against bit-flip scenarios, those which could cause kernel oops, panics and blue screens of death.  I also like having more than one of things, like multiple disks, and so on.  I visibly squirm when I find SMEs using desktops as servers, in production, and then find that the "server" (or desktop) only has one hard disk.

Server motherboards also often have neat features built in, like more PCI slots (and 64 bit width ones -- handy for RAID cards).  There's also iLO/DRAC/IPMI for remote management built in, but remember, if you have remote management, make sure it's configured before it's too late. 

They also tend to have better BIOSes, which are designed for headless operation, no more "Keyboard not found - Please press F1 to continue" messages, which prevent your headless server from booting.

Servers that are built as servers, on server hardware, cost more than a desktop, but last far longer.  You get a much greater Return On Investment by not having to replace disks and memory that have failed in the first year, because they've simply worn out. 

As with any electronic equipment, the bathtub curve of failure rates applies, but the entire graph length is much shorter for consumer-grade hardware. 

If you look at the cost of a server, along side the cost of a desktop, then the cost of a server really is quite a lot higher.  The rub is that the cost of downtime can be enormous, especially if the services provided by the server is core to the business, or it's core to the operations, such as logins, and file sharing (in the case of an office domain).

Hardware is cheap, Downtime is damn expensive. 

Perhaps, along side everything else, the old adage is truer than ever:

You really do get what you pay for.
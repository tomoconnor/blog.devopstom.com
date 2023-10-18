---
title: "Retrospective: Unlock London Hackathon"
date: "2012-05-22T20:00:00.000Z"
description: "Event notes: an Astound Wireless job"
warning: ancient
---

Lessons learnt from the Unlock London Hackathon.

I had an email on May 16th,  asking for some assistance in setting up the wifi network for another hackathon.  After my impromptu assistance at LondonRealtime went down so well, and "saved the network"; apparently I was a natural choice for the next one.  

At least I knew (sorta) the network layout at White Bear Yard.  

The main difference between this one and the last, was a bit more warning on the side of "We're gonna need wifi", but there was still a clause that it's gotta be "*flawless*" - Their words, not mine.

Given the budget for the network was about £400, I recommended a stack of cheap Access Points (which are the ones I used at LondonRealtime, and they're actually insanely good, considering they're less than 20 quid each), a couple of 24 port switches, and a smallish ethernet router.

In this scenario, the router doesn't actually do a lot other than NAT, and DHCP.

Last time around, we used my Cisco 2621XM, this time, they bought a Cisco RVS4000.  Actually a solid bit of kit.  I'm pretty impressed with that part of the network, aside from the fact that it's DHCP server doesn't seem to be able to handle anything larger than a /24 network address for the DHCP pool.  Anyway.

The switches do almost nothing, so unmanaged switches are fine.  There's no need for anything too heavy here, because there's no need for a VLAN, and nothing intelligent going on at all.

All the Access Points are configured identically.  Static IP address, WPA2-PSK, 802.11bgn, and the same SSID.  

The advantage of this is that you get some form of client-side roaming, a bunch of different APs, with different ESSIDs, but the same SSID, and most OSes are intelligent enough to figure out how to move you around across channels and APs.  

We had 10 Access Points, 5 on each floor, about 10-15 metres apart.

By and large, it worked pretty well.  The problem came a lot later on, when you get 150+ people, each with >1 devices.  Interestingly, we never exhausted the DHCP pool, not on the first day, and the peak throughput was touching 55Mbit.  That's quite impressive alone.  

At about 2 PM, the organiser of the event called me, as apparently some people were experiencing pretty heavy packetloss connecting to the network.  I did some investigation, and discovered 2 interesting things.

1.  The wifi itself was still rock solid, and I could ping any device from anywhere else on the network.  That's one potential problem ruled out.
2. The packetloss was occurring between the 3rd hop and the rest of the world.

Connectivity between the two floors was also fine.  Each floor has it's own switch, and the connection between the two was also fine.

Basically, it looked like this.
```
192.168.32.1 - 5 - 10 ms (Our router)
10.10.10.1 - 10 - 20 ms (Their first edge switch)
10.20.30.40 - 6501ms (Their core switch)
193.100.xxx.xxx - 8ms (Their CPE)
```

So there's a pretty enormous jump and a lot of packetloss associated with the connection between `10.10.10.1` and `10.20.30.40`.  It turns out, that those 2 are layer 3 switches for the building.

Here's my theory: 

On an average day, there's 100-200 people in the building, spread across 3-4 floors, across 3-4 switches (depending on who you talk to).  That amount of traffic is fine for the upstream provider, who supply a 1Gbit pipe into the building.

What we did, however is take 150 people, and 150+ devices, and shove it all down one port on a HP Procurve switch, instead of spreading it out across a bunch of ports.

At some point, the switch reached it's port buffer capacity, and started dropping traffic.  I can't blame it, really. 

The reason for only using one port is that it would have probably been a lot more work to configure a split network with two  (or more) routers, and still have a sensible amount of management.

Interesting so far.

At about 3PM on Saturday afternoon, I turned on the firewall on the edge router, and started blocking P2P and bittorrent traffic (as best it was supported, anyhow).  This had the almost instant effect of cutting the outbound traffic from ~25Mbit to about 5Mbit.  

We're providing a free wifi service for the hackathon.  **We're not providing free wireless so you can download movies.**  

One of the annoying things about the Cisco RVS4000 is that there's no intrinsic way to see who's using what data, i.e. there's no support for Netflow, or similar.  There's also no sensible builtin traffic graph, which is more annoying.  There is however SNMP data, which I only started to collect *after* disabling BitTorrent and P2P, sadly. .  I need to collect the RRD files from my impromptu "laptopserver".. 

Here's an interesting side-note:  I took my Netbook along on Saturday and Sunday to connect up as a SNMP data receiver.  Basically just running Ubuntu 12.04 server (with icewm, for firefox), and munin.  Nothing fancy.  I would've installed Logstash, but I realised I only have a GB of RAM on that laptop, so it's less than ideal.

I'd had the forethought to shove my 60GB OCZ SSD in there on friday night, so that I knew it'd work on saturday.  First problem.   Laptops make crap servers.  Even if you disable the ACPI features in the bios, the whole thing still tends to go to sleep.  In the end, I used xdotool's mousemove feature to move the mousepointer about a bit so that the OS didn't see it as having gone to sleep.  

From my point of view, given I was actually *working* at this event, rather than volunteering, the whole thing had a totally different feeling.  I felt pretty good on Friday night after setting everything up.  We tested the speed and throughput, and it was pretty solid.  You could roam between access points and across floors without any problem. 

Come Saturday afternoon, I was feeling pretty stressed.  The wifi was solid, but the network problems were effectively out of my control.  There wasn't any sensible steps we could take to increase the speed (and decrease the packet loss), and maintain a level of segregation between the guests and the White Bear Yard network.  This was always a pretty high priority, as the potential for cyber-espionage and general badness is quite high when you let 100+ perfect strangers onto your premises and onto your network.

 

Conclusions:

48 hours and £400 isn't really enough time and budget to provide a bulletproof wireless solution.  That's not to say I didn't regret having a crack at it.  I think you'd be hard pushed to find a wireless solutions vendor who'd even consider that project given that timescale and that budget.

The Cisco RVS4000 router is nice, but doesn't provide enough management tools to make it a truly great platform.  On contrast, I think I slightly prefer the Cisco 2621XM router, in spite of it being 10 years older, it feels like a more robust platform by an order of magnitude.

Taking the traffic for 150 people and shoving it down one Fast Ethernet port is going to cause problems no matter how you look at it.
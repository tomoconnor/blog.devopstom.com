---
title: "Things That Concern Me: Unified Threat Management"
date: "2013-11-24T19:59:58.000Z"
description: "Is it all it's cracked up to be?"

---

We live in a dangerous world.  

It should come as no surprise to anyone who is a Citizen of the Internet, that the risks of interacting with others on the 'net is a somewhat dangerous business.

Riskier still, is operating a server, or entire network with direct connection to the internet.

The number of denial of service and code execution exploits has risen dramatically in the last decade, unsurprisingly.  The number of black-hat hacking attempts (to use "hacking" from the vernacular of the media - rather than it's true, nobler meaning) has also risen. 

The thing is, whilst this kind of attack used to be perpetrated by lone, troubled individuals, access to exploits and malware is now far simpler, and easier than ever before.

The upshot of this is, that every internet-connected system is at risk.  There's a requirement for strong firewalls and access mechanisms, powerful packet filters and intrusion detection systems.

Looking solely at the edge here, it's easy to see why the measures need to be in place to protect systems.  You have a house, but you keep the door shut, right?

I've noticed recently that there's a somewhat worrying trend in network security, towards 'all-in-one' devices, comprising a Router, Firewall, VPN Endpoint, Intrusion Detection/Prevention Services (IDS/IPS), Anti-virus, Anti-malware, Anti-spam, Web Proxy, Data Loss Prevention (DLP), VoIP Gateway (often referred to as ALG, or sometimes Session Border Controller (SBC)).

These devices are generally referred to as Unified Threat Management (UTM), or sometimes Next-Generation Firewalls (NGFW).

In a traditional (non-UTM) network, there'd be perhaps, 3 or 4 (or more!) individual boxes, providing a different feature set from the above list.

 

For example: 

![Simple network without UTM](./Simple-Network-Non-UTM.png)

 

Simply put, there are a number of physical devices, each performing its own specific role.  

The IDS sits between the Edge Router and the inside of the Firewall, comparing traffic, so that if traffic that's detected on the outside, is visible on the inside, it shows that the firewall might not be as effective as it could be.

The users' HTTP traffic can be filtered through the Web Proxy, if desired.  VPN sessions can be terminated inside the firewall on the VPN Endpoint.

There's a lot of different devices here, and there's a reasonable management overhead, which is why the managers aren't smiling.

This is kinda network design 101.

In a UTM deployment scenario, all of those services (plus a few more, for good measure) are deployed (and probably enabled out-of-the-box), on a single device.

For example:
![Simple network with UTM](./Simple-Network-UTM.png)


The UTM device in the example, has a Router, Firewall, VPN, Proxy, Anti-virus, Anti-spam, Anti-malware, Data Loss Prevention, Wireless LAN controller, IDS/IPS, and an ethernet switch all bundled up together.  (I'm not referring to a specific product here, but I've seen all of those features in some form on a variety of UTM devices.)

There's also a Centralised Management System, with single sign-on, to make for 'easier' management.

This makes the managers happy.  They've got a "Single Pane of Glasss" management interface, all their eggs in one basket, and so on.

There are many things about UTM that make me generally uneasy.  

1. All your eggs are in one basket.

I'm not saying here that you couldn't have multiple UTM devices in a High Availability (HA) pair, of course you could, and probably should.. However, what I am saying is that they'll both have to be identical, and the same vendor, probably - I've not found a UTM device that exhibits full interoperability with another one of a different vendor, in a HA arrangement.

This also introduces your UTM appliance as a Single Point of Failure.  If the device is overwhelmed, which is entirely possible (see point 4), and stops responding to traffic, then your network is going down.  Hard.

 

2. Single Sign On. 

Generally regarded as a Good Thing™.  The staggering downside is that if an attacker gains access to one password, they have access to the entire box, and I'm pretty sure they'll start turning things off.  Probably starting with the IDS/IPS, and/or logging/audit trail.

I'd much rather have a number of secure passwords to a range of services, than one password [to rule them all and in the darkness bind them].

 

3. Defence in Depth. 

The primary tenet of running any kind of secure system.  Basically, have lots of different layers of security, each with tight access controls between the layers.

UTM effectively destroys this, on two fronts.  Firstly, everything is on one device, implying that if the device is compromised, so are all the services.

Secondly, it's not usually obvious, or visible what happens to the traffic as it passes between services.  Is it all on one big fat pipe, or is there some level of separation?

Conversely, in a network with lots of devices, it's possible to choose different devices from different vendors, and as a result, you'll have some greater level of assurance that you'll be able to block and incoming attack, because exploits for one platform won't be effective on one from a different vendor (hopefully!).

 

4. Performance

Looking at the non-UTM network map, there's 5 different services, all on its own device.  Let's pretend that we've got a 1Gbit internet connection, and a 1Gbit LAN.

Each of those devices should have the processing power to handle 1Gbit of traffic, without breaking a sweat (futureproofing, etc..)

Now, putting that onto a single UTM box, we've got the requirement to handle 5Gbit of traffic internally (although, it probably needs to be higher still.). Secondly, the CPU and memory requirements are higher.  It'll need *at least* 5 cores, because you *really* don't want CPU time  contention when there's packets to be handled.. - Otherwise, everything will slow down. 

Imagine you've got a UTM device running linux, on a 2 core system.  One core will forever be doing the general OS stuff, leaving you with one free one.  Not great for performance.

These services do need CPU and memory resources just like anything else would.

 

Now consider all the services on the UTM device, and recall that they'll probably all be enabled on the box, by default.  Every packet entering the device, will be handled by each service.  As you can imagine, this places a lot of load on the CPU and internal interconnects.  Each service will have to allocate RAM for buffers, and so on.  

As you enable multiple services, performance will take a hit.

 

Now, I'm not saying that UTM is *always* a bad thing.  All I'm saying is that I'd have to have a bloody good reason to deploy a UTM device in a network.  

Probably the best reason I can think of comes from analysing the potential risks and impacts of an attack, and subsequent breach.  

Budget also plays an important part in this, because UTM devices to tend to be less expensive than a full array of differing devices, from different vendors, each performing it's own task.

 

I'd rather have a higher management overhead, and greater security, defence in depth and so on, than a single signon, single password, single device. 

Some of the management overhead can be mitigated by the inclusion of a log management platform, or a Security Information and Event Management (SIEM) device, which will aggregate logs, events, alarms, and so on from a number of different devices and services, presenting the output in a single management view.  This is significantly different from having all of the management onboard the UTM device, as it still grants you a level of isolation between management traffic and dirty, internet-facing traffic.

 

**TL;DR**

Unified Threat Management is not a silver bullet.  It might fit for some deployment scenarios.  It's probably suitable for small office / home office usage.

If you do deploy a UTM appliance, bear in mind the following:

* You potentially sacrifice defence in depth security for management simplicity.

* If it's performance you want, either turn off some of the features (and potentially sacrifice security even further), or have dedicated security appliances.

* UTM concerns me a bit, and it should probably raise concerns with you too.


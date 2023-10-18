---
title: Seriously, What?!
date: "2011-04-23T17:00:00.000Z"
description: "Scary infrastructure decisions"
warning: ancient
---

Sometimes you read something on the internet and think "Huh? Really?".  

When I read this, I swear, you could almost hear my brain go *boggle*.  

When I first started using Java, I remember reading something in the [EULA](https://www.oracle.com/technetwork/java/javase/downloads/jdk-6u21-license-159167.txt) (yes, I read it), about not using it for mission-critical or life-critical circumstances.  Something about avionics and nuclear power stations. 
Specifically 
> "You acknowledge that Licensed Software is not designed or intended for use in the design, construction, operation or maintenance of any nuclear facility."

 The thing is, we all click-through these, because we all suspect that nobody would actually use Java for a nuclear power station, or say, host a mission-critical service on the cloud. 

However, tonight, that is exactly what it appears someone has done.  I've also archived the page as a PDF, should it get deleted from sheer terror.

[Scary infrastructure decisions ahead.](./eeeeek.pdf) (147.7 KB PDF)

I am honest-to-FSM scared by the concept that there could be no built-in redundancy to that system.  (Part of me wants the CTO from them to contact me WRT systems consultancy, the other part wants me to run around screaming)

I think the commenters say it best, but I'll still add my $0.02 here.

While Amazon EC2 may be compliant to a number of standards, and have previously had no major issues, this latest incident should serve as a reminder to all users of cloud infrastructure.  

It's no different to any other system.  It can go down, you can lose your data, and shit can hit the fan.

Have lots of redundancy built-in from day one.  Have lots of different layers of security and redundancy, like Defense-in-depth for nuclear reactors.  

Plan for the **worst case scenarios**, because in systems engineering, we deal with the **when**, not the **what if**.


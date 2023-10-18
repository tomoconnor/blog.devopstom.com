---
title: "Technology Empires"
date: "2012-05-05T20:00:00.000Z"
description: "War stories from dealing with massive companies"
warning: ancient
---

As time goes on, and I find myself dealing with more and more large companies, it's pretty obvious that there's a recurring pattern.  

**Many large companies are awful to work with.**

Dell are pretty bad.  
On one recent occasion, we had the need at work to call Dell Support for a mysterious problem with our shiny new blade servers.  
The biggest problem when dealing with Dell in particular is their scripted support technicians on 1st line.  Most other companies seem to hire people with the capability of abstract thought.  
Dell hires chimpanzees to read from scripts.  Woe befalls you if you attempt at any point to deviate from the script, and heaven forbid you ask a question that they don't have the answer to.  If that happens, there's a 1 in 3 chance of any of the following outcomes.

1) "Let me transfer you to another member of the team" - You get put on hold.  Sometimes indefinitely, sometimes the call just drops out.  Generally, you have to go through the same question-answering process all over again.

2) They make something up.  This is the worst trait by far.  I'd rather be told "Sorry, I don't know" rather than having them blame a) our choice of Operating System [Yes, it's not windows, or RHEL, but our problem is with a Layer 2 switch..]. or b) Lie blatantly about the problem.

3) They just hang up on you.  I suspect this is a fight-or-flight response, but I've been hung up on many a time when asking for support.

So just pretend for a moment that you've made it past the first line.  You've got hold of someone who's prepared to acknowledge there's a problem.  Well done.  That's a big step out of the way.  You've already spent *3* hours on the phone, being bounced between 1st and 2nd line; you've dialled and redialled after the phone-line has mysteriously gone dead.  Now you've found a smart guy who knows what he's talking about, so you take his name.  Just a formality really, as they never seem to be addressable by this means.  You can also forget about asking for his Direct Dial number, because they don't exist either.

You can also bet heavily that if you call back at any point in the future, that they're not working that day/hour/week, they've moved departments/countries/employer, or they're just AWOL.

So you call your account manager.  You ask them why the support is *so* inept, when you're paying the cost of a sports car for hardware.  When you know that the problems you're having could likely be resolved with the assistance of an on-site engineer, or a Skype/WebEx chat and screen-share with someone suitably wise and with the time and patience to sit down, do some diagnostics and actually solve the problem.  

Your account manager will no doubt explain that on-site visits aren't covered under your support agreement, despite you swearing blind that you ticked that box when you set up the damn contract, and that nobody is currently available for your one-on-one session over the internets. 

So you make an appointment, for them to call you, and dial-in to a screen-sharing session.  A session which might, unsurprisingly, never happen. You've scheduled 3pm on a Thursday afternoon, and that time rolls around, and it's half past 3, no call.  It's now 4pm, no call.  You call your account manager, only to discover that due to Flux Repolarisation issues, or some such garbage, your conference call has now been cancelled, and you'll have to re-book another slot. 

*sigh*

That's just one technology company.  One market leader.  You'd think that they've become the market leader for a reason.  They have, but that reason sure as hell ain't customer service expertise.

--

**Let's now look at a different field entirely.**

Imagine that you're running the technology and engineering team for a company, and you want to grow the size of your physical infrastructure.  For argument's sake, imagine you want to grown your server room from one rack to 5 racks.  You're gonna need more power to do that sensibly, and safely.  To get more power, you're gonna need a new feed from the building's intake room to your floor.  You're gonna need a new meter, and for that, you're gonna have to call your current supplier.  

EDF.

So you call the number on your bill.  You talk to the Business Accounts team.  Or someone from it, anyway.  They tell you to talk to Metering Services, and give you a number.  You call the number.  It's engaged, so you sit, on hold, in a queue of an inconceivable length, listening to Zero 7 from now until eternity.  Someone might answer your call, but more likely, you'll get cut off.

So you call back and repeat the process.  Eventually you get through to someone, who tells you that they can't help you without you having a project number.  To get a project number, you have to call a different team, Project Acquisitions, or some irritatingly redundantly named team of paper-pushing drivel-monkeys.  To get a Project number, you later discover, you need a MPAN number.  The MPAN number is, generally, like a serial number for your meter.  If you're having a new meter, you might not have a MPAN number.  To get a new meter, you seemingly need one, but to get one, you need a Meter.  So round and round you go.

Don't forget that every time you call in, you'll get a different person, again.  You'll go through the same questions every time, probably in a slightly different order.  You'll be passed between departments like a hot potato, and have your call dropped more frequently than a wet fish.

If by sheer luck and persistence you manage to request a new project/MPAN/Metering ID/other electricity jargon number, then you'll probably be told that you'll have to call back at a later date, in order for the number to be in the system, and available for use.  This time period can be anything from 24 hours, to 30 working days.

Don't forget that your infrastructure build-out project is being delayed by this insanity, and every day that you're waiting for them, you're not making money by having your servers working their little ball grid arrays off.

Jumping forwards in time, you've got hold of the required numbers, you've booked an appointment with one of their engineers to discuss the plan for installing a new 3-phase metered circuit to your premises.  Their engineer gives you a list of requirements and specifications that you will pass to your electrical contractors so that they can do the hard work, the wrangling armoured 3-phase cable, the fuses, the breakers, the consumer units and testing.  That's all covered by your electricians.  

So you've got the spec sheet.  You've got the electricians in, and they've run cable between the intake room and your premises.  They've done all the work to the latest standards, everything is *perfect*.  It matches what the electricity board have said to the letter of the spec sheet.  It's not only perfect, but it's also beautiful.  Their cabling quality is second to none.  There's not even a hidden rats nest of insanity in a dark corner.

It's time to play the phone game again.  This time, you need an electricity board engineer to come out and install a meter.  This requires turning the power off to the floor, because of the temporary supply you've been using in the interim.  So you've arranged some time when the work can be carried out.  As I'm sure you're aware, in a busy, profitable company; finding this kind of time for outages is both tricky and expensive.

It's 6:30AM on the day of the meter installation.  The electricians are here to oversee the installation, and you're just waiting on the man from EDF.  You're clutching onto your letter from the Metering Administration Team, clearly stating that you've got the appointment reserved.  You made this way in advance, and followed the instructions of the phone-jockey to the letter for this process.  You're totally sick of the sound of Zero 7, and struggle not to wheeze and rend your hair when you hear that damn song on the radio.

EDF man turns up, and states in no uncertain terms, that the wiring he can see isn't up to "code".  What he actually means is that they'll only certify work that's been done to a standard produced in the 1980s....

**Digression:** New wiring legislation allows for more leniency in the layout of the wiring.  Specifically the 17th edition allows the armoured sheath of high-current cable to be used as the earth loop connection.  If the cable's certified to the 17th edition, it's all kosher.  
The EDF wiring manual states that they have to have a separate earth feed *as well*.  
-- This alone makes less sense, because it's easier to accidentally damage a separate earth cable, rather than a bundle of armoured cable strands, but there we go.  Since when has legislation made the blindest bit of sense?

So that's the first strike against the master plan.   Your electrician friends tell you that EDF (and other electricity boards) are famous for doing this, moving the goal posts, and generally being obtuse and obstructive.  It's almost like they don't want your money.  In a sense, it's pretty clear that they don't.

The second strike comes when Mr EDF tells you that he's only got you down for one appointment, and that 2 are needed for a meter installation.  The reasoning behind this isn't clear, or isn't made clear at least.  This is the first time you've heard such a thing, and the people on the Metering Administrations Team sure as hell didn't tell you this when you made the appointment for the visit.  It's pretty clear at this point that all the problems are down to an epic number of failures in inter-team communication, at a number of levels, throughout a massive company.

Mr EDF apparently doesn't have any free time for the rest of the day.  Apparently he finishes work at 16:30.  He's not prepared to come back after that, because he doesn't get overtime, and he's "not coming back in his own bloody time".  Smooth.  It's like he doesn't want to be a nice, friendly man, and help out some people.  

I'll bear that in mind if you ever need a favour from me.

It could be 2-3 weeks before you can get another engineer in, with a double appointment, to get the meter installed.  Another 2-3 weeks of wasted time.  Another expensive and difficult scheduled downtime.  Another tongue-lashing from your managing director, who expected the whole project to be finished months ago.

 

That's enough war stories for now.  I hope you can see the general theme here.  

Here's a few other companies I can't stand dealing with, for their awful customer service, or their absolutely astounding failure to communicate efficiently:

* BT
* IBM.
* Oracle.
* Insight. 
* Misco/Wstore (Used to be good... Turned into box-shifters, and went rapidly down hill)
* T-Mobile. 
* Thames Water.

-- 

But there is a revelation in all of this.  There still exist some small companies who do things well.  They're not market leaders in their fields, but the do have the agility to provide excellent customer service.  My home phone line is provided by Gradwell, they just get lines wholesale through BT.  Whilst it's fractionally more expensive, it does have the massive benefit that I don't have to waste my time dealing with peons at BT.  I don't spend hours on hold, in a queue, listening to irritating 8-bit renditions of Bach.  I call my account manager, or ask them on twitter, and the problems are resolved quickly and easily.

My home electricity and gas are provided by a smaller company than EDF, called Ecotricity.  I like them immensely.  Even as a home customer, I have a single point of contact for enquiries about billing and so on.  This is a massive step forward from the traditional sales team, contact team, and so on.  I have their direct-dial in telephone number, their email address and should I need it, their manager's email and phone numbers.  I've never needed it.

I'd rather pay a small percentage more for decent customer service, and the actual feeling that I'm being treated like a person, with feelings; and not just as Yet Another Customer. 

I forever fear the days when these small, light, efficient companies become too big, or get bought out and borged by the powers of the larger enterprises.  Too frequently when this happens, the quality of service goes down the pan.  Middle managers from up high insert themselves like a playing card in the spokes of a bicycle.  Interfering and making lots of noise in the process. 

 

There's a handy parallel to draw between the problems of communication within a large company, and the fall of the Roman Empire.

One of the often mentioned possible causes for the fall of the Roman Empire, was the lack of communication involved with maintaining a large entity.  More accurately, the problem isn't with a lack of communication, but the latency involved with the type of message passing. 

In [Peter Heather's book "The Fall Of The Roman Empire: A New History Of Rome And The Barbarians"](https://books.google.co.uk/books?id=wCOJfTB7HtgC&pg=PA107&lpg=PA107&dq=fall+of+the+roman+empire+due+to+communications&source=bl&ots=JNBfY4sXlr&sig=YIulN34Kn-eoCk7flrkOx_hkYzs&hl=en&sa=X&ei=xhGlT43kBoX80QWbr_WXBA#v=onepage&q=fall%20of%20the%20roman%20empire%20due%20to%20communications&f=false), he stated that even at a daily rate of 50km, it could easily take 3 months to travel the 4000km from the edge of the empire to Rome.  

>"Furthermore, measuring [the size of the empire] in the real currency of how long it took human beings to cover the distances involved, you could say it was five times larger than it appears on the map.  To put it another way, running the Roman Empire with the communications available then was akin to running, in the modern day, and entity somewhere between five and ten times the size of the European Union."

When dealing with the large companies mentioned above, it seems clear to me that there's a definite problem with communications between teams and departments.  

Frequently, there's also a problem within teams and departments, and as a result, the quality of service provided to clients, customers, and often those working for the company declines rapidly as the company grows.
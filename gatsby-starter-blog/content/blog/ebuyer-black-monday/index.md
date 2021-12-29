---
title: eBuyer's Black Monday
date: "2011-11-28T17:00:00.000Z"
description: "Tell your operations staff when you're running a promo."
warning: ancient
---

It's only a few days after Black Friday, and eBuyer have experienced their very own Black Monday.  

I'll set the scene.  This morning, I had an email mailshot advertising a £1 sale of clearance range items on eBuyer.  Sounded like an ideal plan, especially if there were any hard disks up for grabs. 

I never actually got that far though.  I followed the "sneak preview" instructions, duly "liked" eBuyer on facebook.  10:30 rolled by, and oh look.  Error 500 from eBuyer.  Refresh.  Try again.  Same error.  Connection terminated.  Session reset.  Page returned no data. 

Let's have a look at the Facebook page for eBuyer.  Seems to be some problem here.. Reports coming in from all over the web that the site's down.  *gasp* Surely.. No? Oh my god.. They didn't anticipate the extra load on their servers that this clearance sale would cause?  

What a surprise.  Another company suffering *seriously* bad PR. There's enough bile and vitriol about the entire fiasco split between their Facebook page, and #ebuyer on twitter .

So what actually happened?  

 Well, eBuyer effectively started a DDoS against themselves at about 10:30 this morning.  It's fairly safe to assume that there's two main problems.  

1) Their site is almost entirely dynamic content that has to be generated on every page view, especially as the clearance prices are only visible if you're logged in.  So there's cookies involved, so the content can't be cached.  This means that every page has to be "built" from scratch by the web-server(s), it's gotta make requests to the backend databases for prices and stock levels.  Imagine this happening for every user.. Then every user's click, then every user's click in multiple tabs.  No wonder the site's fucked.

2) More importantly, the majority of their connectivity has been saturated by people trying to access the site.   There's commenters on Facebook and Twitter both saying if you chain-refresh the site, basically keeping your finger held down on F5, then you'll get to the site quicker.  Probably not guys.  

The problem is that you've got the audience of the email, the people from twitter advertising, anyone who's seen the Retweet and anyone from Facebook all piling down to go and have a look at the eBuyer deal.  

Then there's the others.. The rubberneckers.  Those are the types who slow you down on the highway by leaning out of the window of their car watching the ambulances wheel off the bodies.  They're down there too.  Looking for the charred remains of eBuyer and the scorchmark of their PR agency.  

So how did Amazon survive their Black Friday sales? 

Well, that's fairly straightforward.  Amazon are *vast* and incredibly intelligent when it comes to business analysis.  Amazon have been running sales for a lot longer, on a massive scale, but they've learnt from their mistakes.  I can remember the Amazon sites falling over at peak time over Christmas.  It's happened.  It used to happen quite often, but they scaled up, and they scaled out.  That's the only way they got to be one of the top eCommerce sites active on the internet.  

I'm not saying that eBuyer need the same level of architecture as Amazon have, but the key point is the Business Intelligence that's missing.  This is how the conversation should've gone:

> **Date**: 01/11/11
> 
> **Place**: Business Intelligence Dept, Ebuyer HQ.
> 
> **Alice**: "Hey Bob.. On our mailing list, we've got a reader coverage of, hey, let's say 750,000 people, right?"
>
> **Bob** : "Sure Alice, Why?"
>
> **Alice**: "When we launch the £1 deals on the 28th of November, I bet they're all gonna visit at 10:20, so they can sit there and refresh until we launch the deals."
>
> **Bob**: "Oh Alice, you're so wise.  We need to tell the Operations team so that they can get some extra server power in for that day."
>
> **Alice** : "Correctamundo, Bob.  Imagine if we launched these deals and our site went down.  Boy would our faces be red!".

I can speak with experience when I say that I've worked in places where they fail to plan for scalability during sale season, or before a product launch.  It's impossibly stressful to be given less than a week's notice of this kind of event, and have to be in the position to ensure the site's stability and continued uptime.  In some cases, it is *actually impossible*, especially without a massive amount of planning, scaling up the number of servers, increasing the bandwidth to the routing core.  Sorting out load balancing, particularly eCommerce aware load balancing.. It's tricky.


This is where "cloud" services and infrastructure comes into it's own.  Numerous cloud IaaS providers are offering scalable bandwidth, pay as you go, scalable servers, add as many as you like to their scalable cloud load balancers.  eBuyer would only have had to pay for a couple of weeks usage, probably.  A few days for testing, then 2 days either side of today to iron out bugs.  I'm sure that it could have been handled more gracefully. 

Somewhere inside eBuyer HQ, someone forgot to tell the Operations team.  Perhaps one day, advertising and marketing teams will come down from their high horses, and realise that without full support from the entire company, an advertising campaign such as this is actually seriously detrimental to the company.  

Live situation update: It's **11:44 on the 28th**, and eBuyer's site is still down.  @ebuyer on Twitter are attempting to make up for their technical faux-pas by promising better servers for future sales.  Yes, well done.. That's called "Locking the door after the horse has bolted".


I last wrote about a [similar problem in 2009](/cost-forward-thinking), when Derren Brown advertised his website on his TV show, and it was down for 2-3 days.  Since then, I'm still seeing the same problems, companies failing to anticipate server load caused by advertising.  Nothing's changed.  It's still embarrasing.  It's still unacceptable, and in this case, for eBuyer, it's going to prove very costly.  Not only have they lost the potential of more sales from their £1 sale, but they've also lost the regular traffic buying stuff for their day-to-day needs.

Black Monday for eBuyer, Stunningly good day for their competitors.
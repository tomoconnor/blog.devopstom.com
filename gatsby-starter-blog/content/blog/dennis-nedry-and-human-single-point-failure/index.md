---
title: "Dennis Nedry and the Human Single Point of Failure"
date: "2012-12-25T20:00:00.000Z"
description: "It's all fun and games until the raptor fences go down"
warning: ancient
---

> **"John, I can't get Jurassic Park back on line without Dennis Nedry."**

Words you never want to hear uttered.  Unless you work for *InGen*, it's highly unlikely.  

Although there is the remaining problem of the Human Single Point of Failure (HSPOF).  After you've spent the last year or two eliminating the single points of failure from your computational infrastructure, you realise that you're the only one who knows which cronjobs run when, and on which servers.  You're the only one who knows how to [kickstart the postgresql streaming replication](/postgres-replication-91), and `pg_basebackup` isn't documented in the wiki.  

You don't want to travel on the same bus, train, or plane as your colleagues, for the fear that if you both died, or were significantly incapacitated in a vehicular (or otherwise) accident, then there'd be a significant lack of knowledge within the rest of the team to carry out the business duties.

Ah.  The human single point of failure.  Is there a way to get around this problem?  Yep, probably.  But only if you spend a near equal amount of time in documenting your system as you did building it.  

Having been in the situation of starting work on a system that's entirely (or nearly entirely) undocumented, then you wonder what'll happen to the systems when the lead architect leaves.  The guy who's got the secrets to the systems locked in his head.How do you even start the process of that knowledge transfer? 

I still maintain that a better personnel structure of Jurassic Park would have lead to a better (yet far less exciting) conclusion to the film.  If Ray Arnold and Dennis Nedry had worked as part of an agile team, with a complete company wiki documenting the systems and infrastructure, then the outcome would've been far less gory.  

As far as the documentation of new (and existing) systems is concerned, then the only real way forward is documenting as you go.

The concept of writing a full swathe of documenting an entire system in one go, at the end of the project, is a staggering undertaking, even for the most committed systems engineers. 

Experience, and some common sense tells me that the best way to ensure knowledge transfer between team members (to eliminate the HSPOF), is to have another member of staff shadow the person with the knowledge.  See what they do day-to-day, when they edit the code, ask what they're doing, why they're doing it, whether something else would work as well or better.  Learn from them, from their experience, and hopefully gain some insight that will allow a better system to be built.  

And another thing.  
The idea of designing a park with no out-of-danger access paths, that is to say, if you have to turn off the fences to get to the dock, then you've seriously fucked up the design of your park.  
There should be a secure path between all locations that is "out of band", for want of a better word.  And what's the deal with the circuit breakers being on the other end of the compound.  

That's pretty dumb.  Especially if you've gotta go through the park.  

*Outside* where the dinosaurs are in order to get to the other end of the compound.  

I mean.. Isn't there a better route,  a protected route? 

Tsk.

Oh, and Happy Holidays to all my readers, across the globe, from Russia to London, and South America to Australia.
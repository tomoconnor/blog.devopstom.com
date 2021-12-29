---
title: "Interesting Thing Of The Day: Network Motifs"
date: "2012-12-01T20:00:00.000Z"
description: "Crossing ancient streams"
warning: ancient
---

Interesting thing of the day:

>Milo, Ron, et al. "Network motifs: simple building blocks of complex networks." Science Signalling 298.5594 (2002): 824.
> Fulltext available from Google Scholar: - http://bit.ly/YAstgD

 

It occurs to me that in scalable systems engineering (the sort of thing I do for a living), you only tend to see Bi-fan networks and Bi-parallel ones.  

Bi-fan is rougly equivalent to a cross-connected core switch whereas Bi-parallel is a good representation of a Virtual IP with Load balancer.

There are some Fully Connected Triads, often in the form of multi-master database replication clusters and in fullly-meshed networks which could theoretically scale up to N-ads, where N is probably no more than 10 or 15.  With many connected mesh members, the complexity grows quadratically, where the number of connections is given by (n^2 -n )/2.  

A well-designed job dispatch system / queue exhibits a lot of the same functionality of a combination of bi-parallel with the feed-forward loop, where the message queue and job consumer/worker is represented by the feed-forward loop, and the system's resilience/redundancy is built in with the bi-parallel motif, of course sometimes more than n+1 redundancy is needed, and there are some tri-parallel motifs. N+M complexity is sometimes seen too, in systems where extreme levels of redundancy is required.  

I shall have to ponder over some other Network Motifs seen in this field.  There's definitely more than those I've mentioned, but the interesting thing is that along with software engineering, these design patterns are also quite well rooted in the design of a scalable network.  

It seems evident that most evolved systems have eliminated the single points of failure, at least in the Motifs demonstrated in the article (with the exception of the Three-chain food web, which as an isolated unit is still dependent on the availability (or hunger) of node Y).  
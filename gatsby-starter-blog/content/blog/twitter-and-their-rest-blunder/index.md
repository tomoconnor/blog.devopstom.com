---
title: "Twitter and their REST blunder"
date: "2011-12-31T20:00:00.000Z"
description: "If the service isn't OK, why return 200?"
warning: ancient
---

Hah.  So it's New Year's Eve.   Twitter is down, and has been for about 3-4 hours.  That's because the NYE celebrations have already started.  Somewhere on the other side of the world.

I include Twitter on my personal website.  The one you're reading.  There's a template tag that displays my five latest tweets. 

About 2-3 hours ago, I got some error reports about XML parse errors on that template tag.  I use api.twitter.com and pull in the XML feed for parsing.  No problems there, it's always worked.

It relies on testing that the status from Twitter was sensibly formed.  In order for that to happen, the request has to return HTTP 200 OK.  That's cool.  That's easy, and it's one of the principle tenets of RESTful APIs.  

Until tonight, it seems. 
```
curl -I 'http://api.twitter.com/1/statuses/user_timeline.xml?screen_name=metacheetr&count=5'
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 4968
Set-Cookie: k=217.146.95.120.e7f9484aa1ec178d; path=/; expires=Sat, 07-Jan-2012 16:25:06 UTC; domain=.api.twitter.com; httponly
Date: Sat, 31 Dec 2011 16:25:06 UTC
Server: tfe
Twitter is currently down for maintenance.
We expect to be back soon. For more information, check out Twitter Status »
Thanks for your patience!
```

So what we find is that Twitter are returning HTTP 200 OK, for a status which is blatently not what I asked for.

This is **BAD**.  This is bad for two reasons, mostly.

1. Web developers and engineers rely on HTTP Status codes to actively represent the output of the service, and also to provide a machine-readable representation of the webpage returned.
1. One of the principle tenets of RESTful APIs is that the status returned should accurately represent the state of the service. 

There's nothing to stop you returning a HTML response with a HTTP 5xx status code.  It's better to do that, in fact.  The user sees something pretty, and the machine readable representation says "Er, this service is fucked.

But if you return 200, and encapsulate a non-standard error message, inside an arbitrary block of HTML, it makes machine parsing ***incredibly*** difficult.

So I'm turning off the latest_tweets template tag for a bit.  At least until Twitter read this, apologise, and start returning decent HTTP status codes.

Come on guys, you're one of the flagship startups of Web 2.0, if we can't look up to you to set a good example, what hope does everyone else have? 
---
title: "The 'Change One Thing' Rule"
date: "2016-08-26T19:59:58.000Z"
description: "A rule to live by"
---

Whenever we have planned (and sometimes unplanned) downtime, at work, I'm usually asked the question "*While we've got the entire system down to do X, shall we do Y also?*"

Typically **X** is planned, and we're doing major maintenance - There's one coming up when there's grid circuit maintenance, where we're hoping it'll be fine on UPS and emergency generator - with an at-risk period.

Occasionally, **X** is unplanned, like the time that the air conditioning failed, and everything shut down to save itself.

I always decline the option to do **Y** at the same time, because it violates the **"Change One Thing"** rule.

If I'm declaring a system outage to, say, upgrade the firmware on the core switch stack, I don't want to also take that opportunity to rewire a cabinet, or simultaneously upgrade VMware Hypervisors.  I'll declare another outage for those individually.  

The problem with breaking the **Change One Thing** rule is that if you change two things, and something doesn't work quite right afterwards, you can't be 100% certain which to roll back, and it'll typically take N times longer to fix (where N is the number of things you changed).

So I'm a bit of a stickler for this.  I don't really enjoy giving up a weekend to work on a system when nobody else is using it but I'd rather do one task at a time, and get it right, and be able to make the most of the rest of the weekend; as opposed to having to pick through the permutations of the things that've changed, to try to restore service before 9AM on a Monday morning.

Fortunately, I've got my team quite well trained not to get distracted from the One Thing we're doing when Outage Time comes.  I can almost guarantee though, that someone will ask "Can we do 'this other thing' at the same time?".  

 

To which, my answer will always be:
## No.

 

That can wait for another day, and we shall do that, and only that, then.
---
title: "One Size Does Not Fit All"
date: "2013-03-23T19:59:58.000Z"
description: "Interviews are not like hats"
warning: ancient
---

The tech interview process is broken.

**Fundamentally.**

About a month ago, I wrote about how I've had some terrible interview experiences over the last 6-odd weeks or so. 

I also just read this, and agree with everything said there.  I think there's more to say.

I'm disheartened to find that these aren't the exceptions, they're the rule. 

The thing is, companies seem to have one type of interview, **The Developer Challenge**.

That works fine, as a whole, if you're looking for **Developers**.  

But I'm not a developer.  Not really, anyway.  I can write code in a variety of different languages, I can pick up most sane languages without too much trouble.  I still struggle with functional things like OCaml, F#, Haskell (and for different reasons, Erlang).

But Python, Ruby, Java, C#, C++ and that ilk, I'm generally fine with.

I know lots and lots about how python datastructures are neat, and why dictionaries are awesome, and how you can play with a string as if it were a list.

I know why polynomial-time algorithms are bad.

I know things like how you find DNA sequence alignments (I used to be a bioinformaticist).

I was explaining to a friend of mine (an engineer at Google), that if I was asked to make a search engine as part of an interview question, then I'd do the following:

1. `wget --mirror`
2. load that data into Elasticsearch
3. build a noddy webapplication with Flask and PyES to search the Elasticsearch index

Done.

Because for me, a very pragmatic individual, I'd rather be building on other people's working code, functioning libraries, assembling building blocks to make one large system out of the component parts.  

If I need systems-glue, I'll typically break out Python, or in extreme cases, Java.

If I want to search a list of words in better than O(n) time, I'll use a Trie.  This I know from having read books, and wikipedia, and StackOverflow and so on.  

What I won't waste my time doing is trying to implement my own Trie library, because I know of at least 2 perfectly good, generic(ish) libraries, in a variety of languages, for doing what I want to do.

Doing things this way means that someone else (someone far better at datastructures than me) has tested it, found edge cases, written documentation, fixed bugs and maintained the damn thing.

**Interesting side note:** If I’ve had to resort to using a library from PyPi (or Github) for a Trie, it’s fairly indicative, to me, that Python needs one in it’s standard library.  That’ll piss off the interviewers for sure.  Next thing you know, we’ll have to do everything in x86 Assembly.

I don't see my approach as anything other than realistic.  It's how I'd solve the problem for work.  I'm not aversed to reading through the library (and looking for insanity), or fixing bugs and sending them upstream, but in a real-life situation, I'd rather not spend my life whittling a log into a wheel.  I'd rather use an existing template for a wheel. 

It occurs to me that there aren't that many problems that people come across which aren't a copy, or subset of an existing, solved problem.  If you wanted to clone a sheep, you wouldn't work everything out from first principles, you'd read the research from the team who cloned Dolly, and work from there.

And another thing.. We need to stop with the whole "Here's a challenge, come back in X amount of time with a solution."  That approach is LAME.  

I like to discuss things with people.  I don't like waiting for a day to get an answer, by the time my question has gone through the recruiter/HR/tech-lead.  
Just do the thing in a skype session with me, or TeamViewer, or something. 

Whilst I was discussing a particular problem with my Google friend, he basically suggested that even without getting a decent answer to the challenge, but by explaining my thought processes, it's as good as a win from the hiring point of view.  You just don't get that kind of interaction with a "go code this and come back" challenge.  

Sure, it's more time consuming for the interviewer, but surely they're already at the point where they want to spend time with you.  I mean, having passed the phone screen, it's  time for something a little more in depth.  I get the whole, "we don't know you, come to us with this challenge" spiel, that Facebook used to do (remember the challenges page?).

Actually, why not just check out my github/bitbucket repositories?  There's loads of stuff I've written there.  

In a similar vein, I'm brought back to interview questions (and scenarios) where I've been told that searching the web is out of the question.  

I'm always left wondering whether searching the web for an answer to a problem is also disallowed for the other developers (who do work there).  If it is, I'm 99% sure I don't want to work there anyway.

It's effectively saying: "We don't want you to learn from other people's mistakes.  You have to make them yourself.".  That's not a realistic expectation, IMO.

Some of the best interviews I've ever had have taken place in a pub, in a coffee shop, in a cafe, over lunch.  

During one of the best technical challenges I've done, I said to the interviewer: "It's been a while since I did this, I'm gonna see what the ServerFault community says".  In this case, this was an acceptable answer, and worked perfectly.

**TL;DR [1]: STOP ASKING INTERVIEW CANDIDATES TO REINVENT THE WHEEL. ASK THEM QUESTIONS ABOUT WHY THEY'VE CHOSEN THE TOOLS THEY HAVE.**

Moving on:

Interview challenges should be closer tailored to the actual job title.  This "one true developer challenge" thing works fine for hiring developers.  God help the candidate if they get sent the same challenge as a software engineer, if they're a front-end UX engineer.  

**The challenge set should probably be related to a real-life problem that fits into their job remit.**

For **developers**, fine, ask them to write algorithms.

For **QA engineers**, ask them about Selenium, or Cucumber.

For **Sysadmins/DevOps/SRE/Systems Engineers/Systems Architects**, then ask questions like the ElasticSearch one, or "How would you aggregate logfiles from a cluster of N servers", or "How would you allow a cluster of N servers to share an assets directory"? or "How would you check database replication lag?", or "Why are 2 switches at the core better than one?".

For **Designers/UX/JS/CSS engineers**, ask them something like: "Design an interface for an ATM to make it more user-friendly.  Bonus question: Make it more user-friendly to disabled users." or "How would you make this site look awesome on a mobile device?"

Hopefully you can see the difference between the different class of question, and the different class of job.  
It's no good asking a greengrocer which is the leanest cut of beef, just like it's not terribly conducive to ask a sysadmin to solve a problem that might require the use of a Red-Black Tree.  

It's possible they'll know the answer, but I sure as hell wouldn't fail them for not knowing. 

**TL;DR [2]: TAILOR INTERVIEW QUESTIONS TO MATCH WHAT YOU EXPECT OF THE CANDIDATE IN REAL LIFE.**

Interviews are not baseball caps.  

**One size does not fit all.**
---
title: The True Age Test
date: "2009-03-23T23:46:37.121Z"
description: Early criticism of facebook posts gathering osint.
warning: ancient
---

A few weeks ago, I wrote about this facebook meme, “The Name Game” and I hypothesised that this wasn’t a meme, but actually a data gathering exercise, possibly started by scammers.
I’ve found another one.  One of my friends took the “True Age Test”, and came out younger than their actual age.  I’ve just had a brief flick through the questions.

Starting off with fairly harmless, questions which are related to the app, “What is your actual age, what race are you, how much exercise do you get” etc…

Rapidly progresses into “Have you ever had any heart conditions, did anyone in your family die before the age of 60 from coronary related illnesses”

Later, “Do you have diabetes. Do you have any Digestive problems, Do you use drugs, How depressed do you feel, What is your relationship status” and so on.

Now, not only are these questions a bit personal, but there is no obvious information on how your data will be stored, or used, or archived.  Given that facebook already shares a good proportion of your personal data with these applications, what is the probability that you’ve just answered enough data to build up a probability report of how much a risk you would be to a) a future employer, b) a bank, building society, etc or c) an insurance salesman.

It also doesn’t state (nowhere that I saw, anyway) what they’re gonna do with the data, Is it transient, or stored in a file somewhere.  How long is it stored for? Do they plan to sell the data? Domestically, or overseas?

Also, without a comprehensive code review, it’s not very easy for people to see whether the data is going to be exported through a backdoor in the code, so even if they say “Oh no, the data isn’t stored, or identifiable”, there doesn’t seem to be any easy way to prove that.

IIRC, Facebook don’t ask to see your sourcecode to the application, so it might be quite easy for an individual with malevolent intent to gather a vast amount of potentially sensitive information quite easily.

The motivation for people to participate in this application is simple “I want to prove that my ‘real age’ is younger than my biological age, therefore I feel good about myself”.

We all want to feel good, don’t we?

But at what cost?
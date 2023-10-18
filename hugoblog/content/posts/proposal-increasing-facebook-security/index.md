---
title: "Proposal: Increasing Facebook Security"
date: "2011-03-06T23:46:37.121Z"
description: 2 factor authentication is good
warning: ancient
---

As I proved in my last blogpost, it's actually trivial to compromise a facebook account given a very small amount of personal information.  After talking to a number of other geeks on Friday night, two things became quite apparent. 

1. Facebook security is poor, at best, and the ability to change the user's contact email address is shocking.
1. Security questions and secret answers are easily exposed by social engineering, thus, these questions only work effectively if you have a completely different identity which you only use for secret questions and answers.

I don't approve entirely of having secret questions that aren't related to you directly.. I mean, if you had a secret question which was "What is your mother's maiden name?", and you gave an answer which wasn't true, you'd have to do two things. a) remember that you lied, and b) always use the same one, or you'd be forever confused.

Anyway.  The real point to tonight's blogpost is that Facebook Security is gash.  Seriously, even I was surprised that I was able to change my friend's contact email address, and sucessfully change his password.  

The only good thing about all of this, is that Facebook lock the account for 24 hours, and email the other email accounts,  This was the only way that my friend was able to regain control of his account.

I propose that facebook implement two-factor authentication for password resets, and possibly logins too.  Given that Facebook already has and retains your phone numbers, it would be trivial, both in cost and implementation to produce a mechanism of 2-factor authentication for advanced profile control.

User story:

1. Alice wants to reset her facebook password.  
1. She clicks the Forget Password link, and correctly identifies her profile.
1. She selects one of her registered phone numbers for 2-factor authentication.
1. She then selects whether she is to recieve a voice call, or SMS message.
1. Facebook send a validation code to the number, either as a SMS, or a short voice call, reading out the code.
1. Alice enters the validation code, confirming her identity.

This system would only work if you couldn't change the numbers that Facebook could contact you on (like you can currently change your contact email address), and you had already confirmed your phone numbers with Facebook in advance (on registration, perhaps, it could authenticate your phone number)

I don't suppose anyone who works for Facebook reads this, do they?

Interesting sidenote:

It appears that it is not possible to change a Facebook Security Question, for "security reasons".  

"To protect account security, it is not possible to update your account’s security question once you have added one. "

Why the buggery not?  This seems unusual.  Surely these kinds of events (twitter memes, facebook notes for these Name Game things) expose users' security questions and answers, and most important thing to do after a data breach, is to change the credentials in question.  

Most peculiar... 

## Update:
Facebook did implement 2 factor authentication for account lockouts.  I doubt it was because of anything I said though. 
---
title: "Analysis and Comment: Why Point of Sale is a POS"
date: "2011-12-04T17:00:00.000Z"
description: "Why are all cash register systems awful?"
warning: ancient
---

A little background, perhaps:
Last night I attended a Winter Party at a bar called The Sterling, in the ground floor of the Gherkin. 
I have absolutely no problem with the organisation of the Party itself, but more complaints about the Venue. 

I drink in various bars quite a lot.  I've worked in a few bars.  I observe how bar staff operate, and how their tills work.  There's a few longstanding massive problems with pretty much every till/billing system I've ever seen in a bar.

Some are downright terrible, and who knows what the designers/integrators were thinking.  These typically have a **full 103 button QWERTY keyboard**, and you've gotta type shit in, to get stuff to come up on the tab.  It's slow, poorly designed for purpose, and not ideal to use a keyboard in a wet/food preparation environment, so the keyboard gets filthy and broken, so the staff hammer on the keys harder and harder.. Yeah. You can see where this is going.

There's the **generic Touchscreen POS system**, that's been adapted for bar use. Generally a better lot, but the biggest problem is traditional POS relies on having barcodes on everything, and a scanner on the till.  Doesn't work very well for bar things, where things aren't quite as fixed-format as a traditional shop.

One of my favourite haunts in Brighton gave all the staff a **very small barcode reader**, and everything was barcoded.

You ask for a double G&T, the barman swipes the Gordons shot twice, and a bottle of tonic, and you're done.  It worked *perfectly*.  This is the right kind of idea.   Sadly, it's about the only place I've ever seen it.

While we're vaguely on the topic of Brighton,  there was a particular favourite cocktail bar down there which had the most remarkably unfriendly till system. [@PoBK](https://twitter.com/pobk) and I persuaded the barman to let us have a look at the interface after it took him nearly 10 minutes to enter the contents of a custom cocktail.  
The conclusion we came to was that some idiot designer had melded the keyboard entry system with the touchscreen system, but failed to recognise the actual *modus operandi* of a bar.  Every time you wanted a new ingredient, you had to re-search the database, then enter the quantity, in **Millilitres** on the keyboard.

**So here's an idea.  Cocktail bars need a till system that's designed for their use.** Frequently used ingredients in frequently used sizes have more accessible buttons than other, less frequently used things.  Ergo, Gordons Gin has a big button, Anchovy paste, a smaller one, at the end of a list.

Also enter a bunch of cocktails (matching the menu) into the database, so you can ring up a mojito with one click, rather than say, entering 2 shots of rum, mint, lime, etc etc etc.

**I'm going to change theme here, momentarily.**

One of the biggest complaints I've heard of (mostly) last night's venue, is the concept of tab inflation.

A number of people have reported on Twitter and IRC that their tab receipt contains/contained drink items which were not their own. 

I can come up with a few reasons for this.  All of them are avoidable, and yet were not avoided by design/implementation.

Having observed the staff using the bar tab feature, this is the typical pattern:

1. Customer orders drinks. 
1. Customer presents tab card, (a business card sized piece of paper with a number on)
1. Barstaff hit "Add to Tab" button, which displays a screen of small (1 square cm) touchscreen buttons with numbers 1 to 500, or so. 
1. Barstaff select matching button to tab card, and receipt is printed.
1. Customer gets drunk.
1. GOTO: 1

**There's a number of problems with this.**

Let's start with Authorisation and Authentication, or *"How does the bar know you are who you say you are"*.  Well, under the above system, they don't.  
I could grab/steal/replicate/forge anyone's card, and rack up a massive amount of money on their tab, because there's no checking mechanism in the system to be sure that the tab belongs to me.  

In the past, I've seen systems (often in hotels) where you sign the paper against your room number, the bar staff keep the paper, then when you check out, you can see the list of receipts. 

My local pub asks for the name on the payment card that is securing the tab.

There's a pub in Hammersmith that uses a slightly more upmarket solution where the tab card is actually a key to a lockbox that holds your debit/credit card. A bit better, but that also leaves much to be desired in the whole card security theatre.

Authentication is a bit of a bugger. Many of the ideas that work well, aren't ideally suited to the fast paced realm of bar service.  Many of the ideas that are currently used are massively insecure.

From what I've heard, the above problem isn't the actual vector for tab inflation, or at least, if it is, it's of considerably smaller incidence and volume.

The biggest problem is threefold. Primarily, the bar staff have to manually read the number from the card, and secondly, match it with a list of small numbers from 1 to 500 (ish), thirdly, and from an **User Experience (UX)** point of view, most importantly, the buttons are small, closely spaced, and there doesn't appear to be a molly guard (something to stop you from incrementing the wrong tab, in short, "Are you sure you mean tab 123?").

On more than one occasion last night, I heard the admission "I might've put that on the wrong tab, I'm not so good with numbers." - Fair enough.  I'm not brilliant with numbers myself, but I can design that point of failure out of your system.

Here's the simplest solution.  It's so simple you're gonna kick yourself.  Print some computer-readable representation of the tab card number, on the card itself. 

A few options, in ascending price might be: Barcode, QR Code, Mag-stripe, Punched Card, RFID contactless technology, SmartButton, Fingerprint recognition (This'd be cool.)

Yep, that's it.  It's slightly more expensive, but you'd soon find that you'd be losing less revenue through unclaimed drinks.  You'd get more money from repeat business, because you've not alienated customers by saying "you lied, and you did actually buy this drink" when they, blatently, haven't.

Add a simple short authorisation code that the customer sets when they start the tab, and you've got primitive Authorisation as well as Authentication.


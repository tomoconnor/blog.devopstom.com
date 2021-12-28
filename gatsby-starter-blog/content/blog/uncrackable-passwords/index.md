---
title: Uncrackable Passwords
date: "2009-02-09T22:40:32.169Z"
description: A brief discussion on the fallacy of an uncrackable password
warning: "ancient"
---

I got an email today from some software company.. Trying to sell me a password management tool.  I used to use KeePass which was pretty effective.  This one is considerably more expensive.  Among its features, it boasts:

* Generate uncrackable passwords using the integrated Password Formulator
* Maximum protection of your sensitive data thanks to the security algorithm Rijndael 256-Bit!
* Instead of passwords like “toothbrush” or “Rover”, which can both be cracked in a few minutes, you now use passwords like `g\/:1bmV5″£$p’}=8>,,/2¬%CN?\A:y:Cwe-k)mUpHiJu:0md7p@<i` (with a 1-GHz-Pentium-PC, it takes approx. 307 years to guess this password!).
* Password lists on the internet: Place your encrypted password lists on the Internet and enjoy access to all of them, no matter where you are!
* Protection from keylogging (intercepting of keystrokes) – All password fields are internally protected from keylogging.

I’ve got issues with all three five points above.

1. That’s a pretty bold statement to say that your passwords are uncrackable.. I suspect they really mean that they haven’t been able to crack them, or somebody  hasn’t been able to crack them YET.
1. Another word for  Rijndael…  Yep, AES.  Really nothing that sophisticated.  Under closer inspection they’re really no better than the free alternatives.
1. While `g\/:1bmV5T$x_sb}8T4@CN?\A:y:Cwe-k)mUpHiJu:0md7p@<i` may be long, secure, mixed cases, characters, alphanumeric, and symbols, it’s certainly not memorable.  So what happens if you generate this password for XYZ internet banking service, and then you go on holiday and forget to pay a bill, or need to move some money about.. You don’t have your password safe with you.  Bugger.
1. Does anyone else think this is potentially asking for trouble? Assuming XYZ company is hosting them, “securely”, how can you prove they don’t have a backdoor to decrypt the files.  Do you trust them? Considering you’ve paid €30 for this package, it’s not really as binding as a really expensive legal SLA.

The other thing that’s at the front of my mind now, is what password do you use to lock the password safe? Do you use a long, complex, difficult to break one, which you’ll probably never remember, and will need to write it down (therefore making it totally pointless anyway), or a simple short password like your first pet’s name, and some thoughtful numbers after it.

Sidenote to point 3.  307 years on a 1GHz Pentium.. What about a dual-quad core Pentium Xeon.  Or a distributed attempt across 256 nodes of dual-quad core Xeons.  Still, it’s reaching a bit far, but it doesn’t mean that this password is unbreakable.  Not by a long way.

Uh, right.. So this software is going to prevent me from putting a PS2/USB hardware keylogger between the PC and the keyboard? I think not. And if it claims to protect against software keylogging, how could you prove that it wasnt a keylogger itself.  It would be a pretty ingenious way to harvest credentials, make the user believe they’ve just bought a security enhancement, really they’re buying a back door.  (I’m not saying that’s what they’re doing, but it’s certainly enough to make me want further verification of the publisher’s honesty.)

I really don’t like the sound of this software, actually, I’m not keen on this “credentials management” type thing at all.  There’s too many unanswered questions.  And that’s before we get onto the rather open question of the use of biometrics for passwords. There seems to be a growing trend at the moment where biometric data (fingerprints, webcam images, iris scans) provide the password data, as opposed to the identity data that is then confirmed with a password.

Private keys and passwords are easy to change when compromised, but how do you change your fingerprint, facial shape, or iris detail when your credentials are compromised?

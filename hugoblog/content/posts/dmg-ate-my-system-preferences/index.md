---
title: "That DMG ate my System Preferences"
date: "2012-02-22T20:00:00.000Z"
description: "Adventures in naughty Bash"
warning: ancient
---

Well, that's certainly another strange problem.

We have a tendency to build our own DMG images for certain bits of software we roll out here.  Sometimes we'll incorporate our own patches, other times it's just to make the application structure more FHS compliant, and stop it *"shitting all over the filesystem"*, as we so charmingly term it.

In the past, we've used `InstallEase` to build a DMG and PKG installer for OSX.  It's been pretty good up until last week, when one of our engineers built a PKG that had a nasty side effect of destroying the System Preferences once installed.  We initially pegged this as "another weird thing about Lion", and rebuilt the image, and so on.  

We tested the installation again on a different computer, all the time using Munki for package deployment.  

These are the important test findings: 

1. If you install the package interactively, it's fine.
2. If you install the older version, it's fine.
3. If you install the new version with munki, it breaks everything.
4. If you install anything else with munki, it's also fine.

That points to a clear difference between installing Interactively (with a person doing it) and an automated deployment with Munki.

I actually had a look at the package today, and gave it a good hard poking.  I found 2 really rather worrying things.

1. Inside the Resources directory, there's an Universal Binary called DeleteObjectHelper (Not off to a good start here..), and a file called `DeleteObjectList.plist`

DeleteObjectList contains:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>objectList</key>
	<array>
		<dict>
			<key>filePath</key>
			<string>Library/Preferences</string>
		</dict>
		<dict>
			<key>filePath</key>
			<string>Library/Receipts</string>
		</dict>
	</array>
</dict>
</plist>
```
2. Inside the postflight (think postinst for debian) file was the following:
```bash
#!/bin/bash
"$1/Contents/Resources/DeleteObjectHelper" "$1/Contents/Resources/DeleteObjectList.plist" "$HOME" "$3"
```
So here's what happens if you install it by hand.  Postflight runs after installation, and replaces `$HOME` with `/Users/YourName`.  `DeleteObjectHelper` goes off and deletes the files in `~/Library`.  I assume this is to delete older versions, or something similar.

I suspect that if you run it non-interactively, with munki, then it might run as root.  It might also not have the environment that you'd expect as an interactive user. That means that if `$HOME` was `/`, or nonexistant, it might default to `/`.  

**Very Bad Indeed.**

The pointer to all of this was something system.log complaining about Preferences files being missing, when trying to authenticate to our wireless.  I've worked with *nix systems long enough to pretty much start looking in `/var/log/(syslog|messages|system.log)` as a matter of principal.

So, for whatever reason, the process that builds the package is incorporating a thing for cleaning up after itself.  Not a bad thing, but it does have a fantastic bug in it that seems to make it delete the System Preferences if it can't find $HOME.  As it's running as root (or whomever powerful), it just goes ahead and deletes everything it can find.

The questions that remain on my mind are these:

1. Why did it put that in there anyway, as none of our other hand-rolled packages have it.
2. Can we build PKG and DMG files with some kind of GNU toolchain on our Jenkins environment to stop this nonsense from happening again (We know exactly how the build process works, etc..)
3. What on *earth* went through those developer's minds when they wrote that Evil Little Binary?

 
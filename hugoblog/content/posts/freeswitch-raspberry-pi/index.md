---
title: "Freeswitch on a Raspberry Pi"
date: "2013-06-13T19:59:58.000Z"
description: "Build your own PBX for under $40"
warning: ancient
---

I've had a [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi) for ages now.. I got one free courtesy of [Paypal at their Charity Hack in late 2012](https://www.flickr.com/photos/martin_88/8535321648/in/set-72157632935902976), and our team (see photo, I'm there!) went on to use it to create the (World's First?) [Raspberry Pi based Wifi Hotspot](https://github.com/tomoconnor/captive-backend). 

I've wanted to do something potentially useful, definitely interesting, and probably rewarding with it for a while.  

I've also recently acquired an Arduino with Ethernet Shield, so that's also been on my mind for another hack platform.  

That aside for now, I've recently moved back from London, and into the basement flat at my parents' place.  

We had that flat built originally for my late grandfather, but after his death, it lay empty for a while, and as I'm now living here again, it's pretty much perfect for me. 

The only minor problem being, that until recently, it was effectively isolated from the rest of the house.. There was a dual pair of BT phone cable, originally to power a simple 9v intercom system, which whilst providing electrical connectivity for an intercom, wouldn't be suitable for FastEthernet, let alone Gigabit.

So the other weekend, we attached a length of CAT5e cable, and used the existing phone cable to re-thread the new CAT5 cable... So now I've got ethernet down here.

I've got an old CiscoLinksys SPA942 phone that I've had for at least 5 years, now.. And whilst that's all very well for dialing out, it's a little inconvenient for my folks, at least until I get a Malvern number on my SIP account, otherwise it's a national rate to dial my 0203 number..

So I thought to myself, "Well, it's a 4-line phone..".  Originally, I was going to get a 2nd hand retro Trimphone or something similar, rip out the guts, and use it to house the Raspberry Pi, plug in a USB soundcard, and USB Wifi Dongle, and then use that as a SIP handset, dialing out to my AQL VoIP service.

But then I had a better idea.

Why not compile [FreeSWITCH](https://wiki.freeswitch.org/wiki/Installation_Guide#Introduction) onto the Raspberry Pi, then use my VoIP Phone down here to register to it, and use a VoIP Softphone on my parents' other devices.  

As it turns out, it only takes about *6* hours to compile FreeSWITCH (I did prune out [modules.conf](https://gist.github.com/tomoconnor/5776461), disabling IVR, `mod_flite` and some other stuff I thought I wouldn't want/need.).

I once compiled a kernel and Gnome 3.0 on an old 300MHz Via embedded PC, and that took 5 days.. I'm quite pleased by the speed of the Raspberry Pi's compile time.

I suppose if I'd been in a rush, I could've cross-compiled it. 

I'm no stranger to VoIP, but in the past, when I've been configuring PBXes, I've always used Asterisk, and whilst there *is* an Asterisk binaries package in Raspbian, I don't *actually* like Asterisk, and their Dialplan config format makes my eyes bleed. 

My good friend Richard ([@pobk](https://twitter.com/pobk)) is a big fan of FreeSWITCH, so I figured it's about time I saw what all the fuss is about.  If ever there's a good test for an application, it's running on a massively restricted system, in both CPU power, Disk space and memory.. 

My only minor complaint is the amount of libraries that are required to build/run FreeSWITCH.. 

Here's what I ran to build:
```bash
sudo apt-get install build-essential
sudo apt-get install git-core build-essential autoconf automake libtool libncurses5 libncurses5-dev make libjpeg-dev pkg-config unixodbc unixodbc-dev zlib1g-dev
sudo apt-get install libcurl4-openssl-dev libexpat1-dev libssl-dev screen
screen -S compile
#inside a Screen
cd /usr/local/src
git clone git://git.freeswitch.org/freeswitch.git
cd freeswitch
./bootstrap.sh
<edit modules.conf>
./configure
make && make install && make all install cd-sounds-install cd-moh-install
```

Total installed size is 550MB.  I'm sure I could get that down, as 500MB of that is the sounds/ directory

I think if I were having a 2nd attempt, I'd only bother with the 8Khz sample rate audio. (replacing cd-sounds-install with sounds-install)

I also ran through this http://wiki.freeswitch.org/wiki/Freeswitch_init#Debian.2FUbuntu

Then configured an init.d script based on the one on that page.. Don't forget to change the FS_GROUP variable to "daemon"


A brief (very) test with sipp, and after about a minute, it's handled 700 connections, and the load average is about 190 ;).

It's still accepting calls, and the voice lag is about 10 seconds.

As this'll only ever handle one or two calls concurrently, I think that's a pretty good result.

I configured 5 devices, all using the default, preconfigured extensions, 1000-1004, and then set up a ring group.

I'm using Telephone  on my mac, http://www.zoiper.com/ on my Android(s), and Arkphone on my mum's iPad.

Seems to work pretty well.
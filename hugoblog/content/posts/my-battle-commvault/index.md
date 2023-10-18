---
title: "My Battle With Commvault"
date: "2012-09-03T20:00:00.000Z"
description: "A bit long-winded and wordy, but interesting."
warning: ancient
---


This is a bit long-winded, and wordy.  If you've come here looking for tips on improving commvault backup performance and/or throughput, then you should click here for the good stuff. 

It's been a long time since I blogged.  It's been a really long time since I blogged about anything we've been doing at $Dayjob. 

I've spent the better part of the year working on sorting out the backup solution here. 

Initally, we had a storage server, with 30-odd TB of SATA storage, using some bit of LSI technology with a battery-backed write cache.. Pretty good for scheduled rsnapshot backups. However, in May, we decided to sort out off-site backups, and build up some kind of disaster recovery strategy. 

Our storage reseller sent us a bunch of quotes for a number of hardware and supported software solutions.  We're kinda limited by budget, and as a result also limited to the technologies we could use for backup.

We have a Hitachi/Bluearc NAS filer, which is comprised of 2 tiers, one high-speed SAS pool, and one lower-speed, but huge SATA pool.  The storage is all connected across the NAS backplane with 4Gbit FibreChannel, and the 2 NAS heads are cross-connected and interconnected to our core switch with 4 (2 per head) 10Gbit Fibre Ethernet links. 

Given the cost of media, and the ease of transporting offsite, a tape backup system was chosen.  It's far cheaper in terms of offsite/offline storage to have tape media that sits in a box, rather than boxes of spinning rust that have to be maintained in a cool room, with power and maintenance costs included.

The first solution would involve directly connecting the tape drives to the filer.  Ideally with FibreChannel, but as we've already used all the FC ports on the filer for storage, we'd have to invest in a pair of FC switches.  This is not a small outlay, and makes that solution prohibitively expensive.

Luckily, an alternative exists, where we have a backup server, use NFS to mount the exported filesystems, connect that to the core with 10Gbit Ethernet, and then connect the tape drives to that server. 

We ordered a Spectra Logic T50e autochanger, Commvault Backup software and a server to run it all on, from our storage vendor.  This is where the problems started.

Predictably, there was at least one problem with the hardware.  Our 10Gbit core is entirely fibre, specifically MMF, but that's beside the point here.  The new backup server that had been ordered turned up with an X540-T2 NIC, which is 10Gbit over Copper. 

What we needed was one with SFPs, like this, the ![X520-SR2](./1013714974.jpg).  So we had to order one of those, and have that delivered, and postpone the whole project until that card arrived.  Three.. Days .. Later, the NIC arrived.  Without any optics.  Apparently when ordering from Dell, these are two separate line items.  This is not the case when ordering from Intel or anyone else.

So, a week later, we got the whole NIC and made it work.  About 2 weeks later, the reseller/distributor of the commvault software was on-site to install the software onto our server.  Problem.. We'd been told that the entire software suite works on Linux.  The actual scenario is that the backup controller (Commcell GUI) only works on Windows (despite just being a Java application).  Not only does it only work on Windows, but it only works on Windows Server 2008 (probably works on 2003, but who the hell wants to install that in a new system).  So we had to get a windows server license from somewhere.  Luckily, one of the things I'd downloaded was the Windows 2008 Evaluation version ISO, and shoved it in the central ISO store.  So on the day that the Commvault installer guy turned up, we had assumed that the plan would be something like: 

1. Install the software onto a linux box.
2. Start backing stuff up.
3. Pub?

Instead, we had to try and install windows 2008 R2 onto a virtual machine (which was slower than cold toffee).  In the end, I just stole my desk mate's PC, and installed windows server on there, then moved it from under our desk, to the store room.  At some point, we're gonna have to P2V it and free up a workstation (or install it somewhere more permanent).

So.. a good proportion of the day was taken up with fucking about with Windows Servers, before actually getting to any of the configuration stuff.

I think it's good that for the most part, the process of installing the Unix/Linux Commvault agent is pretty straight forward, as once you've got the server-side set up, the client installation goes off, talks to the server, and pretty much installs itself.

More installations should happen like this, incidentally.

Anyway.. We eventually got the Linux client installed, and the "Media Agent" - This is the bit that actually talks to the NFS, and also talks to the Fibrechannel attached tape devices, and manages the autochanger.  We had to define the directories to be backed up in the Windows Server controler, then configure what goes where, and so on.  (That's definitely a blog for another day).  We kicked off a backup of *everything* - which at the time was about 20TB, and buggered off home.  

We came up with figures at some point for the amount of data we'd be backing up when working at full capacity.. This works out at about 30TB in total, and we wanted to be able to do it over a weekend, so in under 48 hours. So we'd need a backup system that could perform at at least 30TB/48Hrs, which works out at 173MB/s sustained throughput. -- This is important, and we'll come back to this figure a number of times.

The first best-effort backup we tried, we were getting a total combined throughput of 290GB/hr (Commvault chooses GB/hour as it's unit of choice, a pretty weird one, to be honest..) 290GB/hr is 80.5MB/s.. At this speed, a full backup will take 30TB/(80.5MB/s), which is about 4.4 days.  So, we'd have overshot our 48 hour backup window by more than double. 

This is about the time that we decided that we could have two problems, and decided to break into a very long and deep warren of rabbit holes, surrounding to objectives, benchmarking both the tape performance (including CommVault), and also benchmarking the Bluearc NAS.

It's actually a lot more complex than all of that, because there's no single point of this system that isn't somehow connected to a bunch of other systems.  We'd need to look at the Bluearc, the 10Gbit core, the brand new backup server, the fibre channel cards, the tape drives, the tape library, and all the software in between.

Luckily, benchmarking NFS is fairly straight forward, and I devised a thing to run a bunch of IOZone tests overnight, so that the next day, I'd have between 1 and 1000 datapoints of IOZone performance to have a look at.

We had to figure out the optimum parameters for reading NFS files at speed, given a range of possible block and chunk sizes, as well as a number of options regarding being single-threaded, or multi-threaded.

This is the script I threw together to generate benchmarking runs using IOZone.
```bash
THREADS="1 2 4 8 16 24"
RS="8k 256k 512k 1M 4M 8M 16M"
FS="1M 2M 8M 16M 32M 256M"
echo "cd /mnt/shows/benchmarktest/`hostname`/" > `hostname`-fullbenchmark.sh
for f in $FS;
	do for r in $RS; 
		do for t in $THREADS; 
			do echo "iozone -R -b autobench-${t}T-${r}-${f}-`date +%s`.asc -l${t} -u${t} -i0 -i1 -F $(seq -w -s ' ' 1 ${t}) -r ${r} -s ${f}"; 
		done; 
	done; 
done >> `hostname`-fullbenchmark.sh
```

`/mnt/shows` is a location that's on our Fast SAS storage, so at least by this benchmark, we'd be looking at close to the maximum throughput we could ever hope to get out of our disk arrays. As the majority of our data is also stored on that faster pool, it's likely to be similarly representative of the real data.

Anyway. 

After the first benchmark run, we're seeing a roughly direct correlation between number of threads and throughput, up to a point where there are more threads than cores, and the throughput slows down.  This is to be expected, because when you have more running threads than CPU cores, you get more and more context switches, and so on. 

The other conclusion that's easy to draw is that re-read performance is crazyhigh.  Basically,  the Bluearc has a certain quantity of high-speed cache for data that's being read and re-read, so the first time you get it, you pull it from the disk.. Subsequent reads come out of memory, and so are blindingly fast, but only for small file sizes.  Big files can't be stored in the onboard cache, which is where the BlueArc's option for a SSD tier becomes phenomenally cool.. It also comes at an insane price, but there we go.

The overall best performance was from a 4k block/record size, from a 1MB file, using 20 reader threads (on a box with 32 cores, 20 threads is pretty much the sweetest spot), and this gave us 1.4TB/hour read performance. 

We reported these results to BlueArc, who believed that we may have hit a bug in the hardware optimised NFS server that they provide.  The two choices we had were, a) upgrade to the latest patch release, and/or b) try turning off hardware optimisation, and see what happens.

We ran another overnight batch of IOZone tests with the hardware optimisation turned OFF, and found that for some read sizes/thread counts, the performance was greatly improved, primarily small chunks from a big file, with only 2 reader threads, whilst performance for operations with multiple readers was absolutely destroyed.  

The knock-on effect of this is that if we were only servicing 2-4 clients, or 2-4 reader threads, we'd be fine to stick with software NFS processing, but for multiple user performance (like having a farm of 100 rendernodes reading and writing to the disks), we'd be screwed if we left hardware NFS mode off.

So we turned that back on, and set about tuning NFS read block sizes to closely match the best performance we've seen.

We got some Bluearc engineers in at some point in the last 100 days (it really does get hard to keep track of when actual events happened), and they upgraded our RAID arrays and also the NAS heads to the latest version which seemed to improve NFS performance too. 

That really is the very quick review of tuning NFS performance.  There's probably a lot more to say, if I can remember it!

The interesting thing to note about 10Gbit Ethernet performance is that in the scenario we're using it, we're probably having 1-4 threads at a time utilising the connection.  This makes it incredibly difficult to saturate the link, because the majority of applications written in the last 10 years have been designed with 1Gbit Ethernet in mind, so they either don't scale terribly well, or are hard-coded to use a small number of threads.  This includes the Linux NFS client, funnily enough, and if you compare the model of the Linux NFS client to the underlying model of the one used in Solaris, which uses SunRPC, it's actually possible to multiplex over a single connection by setting the number of network threads. 

If you're booted into Solaris/OpenSolaris/OpenIndiana, you can edit this setting with the following little gem.

```bash
$ mdb -kw
clnt_max_conns/W 8
q
```
`mdb` is a bit like sysctl.conf on Linux, and `clnt_max_conns` is the number of threads to use for the NFS client.  We found that if we booted into OpenIndiana, and mounted the NFS mounts, and set the `clnt_max_conns` quite high, we'd get blistering performance out of the Bluearc, until OpenIndiana locked up and fell over.  We didn't really put much effort into figuring this one out, it was more just a proof of concept thing to see if it worked.  I suppose I'd quite like to go back to tinkering with OpenIndiana on 10Gbit Ethernet, but there really isn't time.

Sadly.

I did a bunch of stuff to the NIC driver options on Linux when trying to get decent performance out of these 10G NICs, and came, somewhat surprisingly, to the conclusion that if you leave the driver module defaults as they are when you install the thing, you get far better performance than you do when you start dicking about with things like TCP Window Scaling, TCP Checksumming and Selective ACK.  The less you fiddle with, the better your performance is, and if for whatever reason, you get slightly better performance through tinkering, it'll be in the 0.5-1% range, rather than a 10-20% increase in speed.

This is something else that I'd quite like some more time to benchmark and test things with, but again, we neither have the time to revisit this in any great depth or detail, nor the available hardware for tinkering.  10Gbit NICs are still pretty expensive. I suppose when they come down in price, I'll probably buy some of my own and have a good old fiddle with them, and the settings, and write that up.. One day.  ;) I suppose if someone wanted to buy me the kit, I'd have to evaluate it and write up some kind of benchmark about what kind of performance you get by altering all the different settings, but again, as with everything the performance is all very well and good, but it'll only ever be a benchmark, as your true performance and throughput do tend to vary greatly depending on what it is you're doing with the hardware.

With the NFS benchmarking out of the way (for now), it was time to have a good old poke at the tape drives.  

Luckily, it's pretty straightforward to benchmark a tape drive.. There's three options for doing it. 

1. Commvault "Validate Drive" - You load a tape, it writes stuff to it, reads the stuff back and gives you a speed. At least this mechanism just generates data from /dev/zero or wherever, and writes it directly to the tape, so there's no disks involved.
2. IBM TapeTool - These are IBM HH-5 Ultrium drives, so there's an IBM Tape Test utility, I suspect this does something very similar to the Validate Drive tool.
3. Gnu tar. Despite being clever FC-attached tape drives, they're still just magnetic character devices, so we can write directly to them with mt and tar.  

Results:

1. Commvault reported ~120MB/s write, and ~135MB/s read speeds for both drives.  
2. The IBM Tape Tool utility reported ~130MB/s write speeds for both drives: http://imgur.com/AYH3x http://imgur.com/IOUiP
3. When it came to using Gnu Tar for benchmarking, it was easiest to make it a two step process.

The `pv` tool is incredibly useful for this kind of thing, as you can visualise the amount of data flowing down a FIFO pipe. 

With tar, just run something similar to the following.

`tar cvf - /path/to/thing/to/backup | pv > /dev/st0`

So tar will read stuff from the path, and write it to the file -, which is STDOUT, which is piped into pv to see the throughput, and then written out to `/dev/st0` (testing this by writing to /dev/null is also acceptable for testing NFS performance only, just grab the data with tar, and dump it in the bitbucket)..

If we can get the performance with tar anywhere near the benchmarked write speeds from TapeTool or Commvault, we'll have proved 2 things.  1) that the tape drives can handle data at this rate, and also that the NFS servers can push it at this rate.

From the SAS tier, we get this kind of performance.

`2GB 0:00:44 [ 253MB/s] [                   <=>                                           ]`
From the SATA tier, we get: 

`2.43GB 0:01:10 [ 116MB/s] [                                                      <=>     ]`
Which is to be expected.  We always expected the SATA disks to be slower, by about this margin.

So, with the SAS tier able to push a single stream at 253MB/s, that's 126MB/s per drive, which is pretty close (slightly over) the speed of the Commvault validation, and a little under the IBM TapeTool benchmark.

So we've effectively proved, using real data, that the SAS disks can push at the right speed to keep the tapes happy at full speed.  We've also proved that the tape drives can write at this speed.

So why the bloody hell are we still seeing only 300-400GB/hour performance through Commvault?

Breaking it apart, there's a *lot* of different parameters in CommVault that are tunable regarding performance tweaking. 

I'm gonna come back to this in a more in-depth way at some point, but it breaks down to:

In Subclient Properties:

1. Number of Data Readers: set this to 2x number of tape drives (even though CommVault support tell you not to!)
2. Allow multiple data readers within a drive or mountpoint - Tick this.
3. Storage Device -> Data Transfer Option -> Resource Tuning -> Network Agents: Set this to 2 or 4 (see which works better for you).

 

In Storage Policy Properties (specifically, the properties of your storage policies):

1. Device Streams (set this to 2x your tape drives too)
2. Enable Stream Randomization: Tick this.
3. Select an Incremental Storage Policy (and make the same changes to that one as this one).

 

In the Copy Properties (of all given Storage Policies):

1. Media tab -> Enable Multiplexing
2. Set Multiplexing Factor to 2x the number of tape drives too.
3. Enable "Use Device Streams rather than multiplexing if possible".

In the Global Commvault Control panel:

In Media Management

1. Set Chunk Size for Linux FileSystem to 16384 MB.

Media Agent Properties:

1. "Maximum number of parallel data transfer operations"

Set "Restrict to" to 200 (the highest it'll go).  -- Why there's any kind of restriction here is a mystery to me.

In the Control tab, Data Transfer box

2. Enable "Optimize for concurrent LAN backups".

With the above settings, we're now getting performance between 700 and 900GB/hour depending on what we're backing up, but luckily, for now, this is within our backup window.  I fear that if we plan to backup more within the same time period, we'll need more tape slots, and more tape drives.

And that's it, I'm afraid.  There's no more details I can give you about tuning the hell out of Commvault. I hope, that if some poor bastard is trying to configure CommVault to give decent backup performance speeds that this will be of some use. 


---
title: "Puppet, apt, and a Thundering Herd"
date: "2012-02-21T20:00:00.000Z"
description: "They're flocking this way!"
warning: ancient
---

Puppet really is great.  Don't ever get me wrong there.  It's saved me masses of time and money over the last few years, and allowed me to do my job quickly and efficiently.  

That said, it really does have issues with scalability.  After about 20-30 clients using WEBBrick, everything kinda falls over a bit.

We had this problem at Baseblack.  We've now got ~60 workstations and rendernodes all using Puppet for configuration management and software deployment.  It's great.  It far simplifies the process of rolling out updates and upgrades to new build machines.  

The problem we had was most clearly shown by the frequency with which the PSON error comes up.

`"err: Could not retrieve catalog from remote server: Could not intern from pson: Could not convert from pson:"`

And so on.  

This was always a transient error, and would go away if you ran puppet 2-3, or more times, and then it'd work fine.  This isn't really a valid long-term solution.  It's alright now and again, but it brings up a problem of reliability, and "how do you know when it's last run, if it can't be guaranteed to run every time".

I wrote a dirty wrapperscript that would re-run it if it failed, and so on.  This kinda worked a bit better, but sometimes, stuff would just not run anyway.  

So today, I'd had enough of this problem, and decided to do the Apache2/Mod_passenger thing.  Back in the days of Puppet 0.24/0.25x this was a bit more of a pain in the arse than it seems to be now.  Back then, the server of choice was Mongrel, now it's Passenger/mod_rack.

Just follow this guide: http://docs.puppetlabs.com/guides/passenger.html

It's actually a pretty good explanation of the steps, and I don't see any need to replicate the information here.  I made a couple of slight modifications.  

The `config.ru` file is horribly outdated in the given form, and I used this one instead: https://github.com/puppetlabs/puppet/blob/master/ext/rack/files/config.ru
I changed the apache2 defaults for mpm_worker so that it could handle a shitload more requests than the default. 

Incidentally, [this thing is cool](https://www.linode.com/community/questions/6354/my-mpm-worker-calculator).  Some guy's written an insanely simple "calculator" spreadsheet for OpenOffice and Excel that allows you to calculate decent settings for `MaxClients`.

```
<IfModule mpm_worker_module>
ServerLimit 150
StartServers       5
MinSpareThreads    5
MaxSpareThreads    10
    ThreadLimit         5 
    ThreadsPerChild      5
MaxClients        750
    MaxRequestsPerChild   0
</IfModule>
```

I wanted to be able to handle our own thundering herd of workstations and rendernodes, so that meant that the default `ServerLimit` had to go, and that it had to be able to handle *many* more threads than the default.

I also moved the puppetmasterd "application" from `/usr/share/puppet` to `/srv/puppet` because in my mind (and the FHS), it makes more sense.

There's a bit of a caveat in the process of moving that directory, in that wherever it is, it must be chowned `puppet:puppet`.  After that, it's all fine.

The problems really started for us after that.  It worked fine with one workstation testing it, but throw 2+ at passenger, and apache tended to kill off the ruby children. 

The big hint was in `/var/log/apache2/error.log`: 
```
[Tue Feb 21 15:50:12 2012] [alert] (11)Resource temporarily unavailable: apr_thread_create: unable to create worker thread

[Tue Feb 21 15:50:15 2012] [error] (12)Cannot allocate memory: fork: Unable to fork new process
```

So, our puppetmaster runs on Proxmox as a VM.  Proxmox is an OpenVZ virtualisation host, and as a result, it has hardlimits based around the content of `/proc/user_beancounters`.  This is the thing that's basically stopping Apache from spawning threads to handle the requests.

There's a [page here](https://web.archive.org/web/20120123041303/http://blog.eukhost.com/webhosting/how-to-remove-openvz-limits-on-a-vps/) about how to remove OpenVZ's limits: 

```
clear; cat /proc/user_beancounters
vzctl set 101 –tcpsndbuf 999999999:999999999 –save
vzctl set 101 –tcprcvbuf 999999999:999999999 –save
vzctl set 101 –numtcpsock 999999999:999999999 –save
vzctl set 101 –numflock 999999999:999999999 –save
vzctl set 101 –othersockbuf 999999999:999999999 –save
vzctl set 101 –numothersock 999999999:999999999 –save
vzctl set 101 –numfile 999999999:999999999 –save
vzctl restart 101
```
Where 101 is the #id of your VZ container.  

I also had to bump the allocated memory up from 512M to 2GB, and push the swap (and why not) up to 1GB.  

After a quick restart of the Puppet container, and restarting apache one last time, I sucessfully ran 56 puppetd.  One on each of the nodes, all at once, without a single error.

Sounds like success to me.

--

The next problem we've got that's currently holding back speedy software deployments, and `apt-update`s, is our `apt-cacher-ng` server.  That too is a Proxmox VM, and I initially thought that the same problems might be true, with having a connection limit on OpenVZ, which would be preventing stuff from getting a connection.  

If I run apt-get update on 50+ nodes simultaneously, the probability that some of them will error, something about connection to `http://apt` failing, is pretty close to P(1).

```
W: Failed to fetch http://ppa.launchpad.net/ubuntu-x-swat/x-updates/ubuntu/dists/lucid/main/binary-amd64/Packages.gz  Unable to connect to apt:3142:
W: Failed to fetch http://ppa.launchpad.net/webupd8team/java/ubuntu/dists/lucid/main/binary-amd64/Packages.gz  Unable to connect to apt:3142:
E: Some index files failed to download, they have been ignored, or old ones used instead.
```
The 3142 bit is because we've got apt-cacher-ng running on port 3142, and a line in `/etc/apt/apt.conf.d/` containing `Acquire::http { Proxy "http://apt:3142"; };`

This is the most fool-proof way to make sure that *everything* gets cached, PPAs, and the whole kitchen sink.  This is what we want to do, because having a full debmirror is a) wasteful of disk space, something we're always at a premium with, working in VFX, and b) disk space is expensive; especially after the Thailand floods.

So, we're using `apt-cacher-ng`, and I don't personally see that changing anytime soon. 

Right.  So I bumped up the limits in openVZ's configuration, and there's still a problem that means that the apt requests aren't getting handled.

I suspect that because Apt's protocol is just HTTP, that it might be possible to use something like Varnish or HAProxy and a bunch of `apt-cacher-ng` backends.  

It doesn't appear that `apt-cacher-ng` can run with multiple threads for handling lots more requests/second. 

```
root@apt:/# netstat -anp|grep 3142|wc -l
2964
```
Yah.. That could be a problem.

That said, I've just tested hammering it with ab as follows:

```
tom.oconnor@charcoal-black:~$ ab -n25000 -c550 -X apt:3142  http://ppa.launchpad.net/mozillateam/firefox-stable/ubuntu/dists/lucid/main/binary-amd64/Packages.gz
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
Benchmarking ppa.launchpad.net [through apt:3142] (be patient)
Completed 2500 requests
...
Finished 25000 requests

Server Software:        Debian
Server Hostname:        ppa.launchpad.net
Server Port:            80
Document Path:          /mozillateam/firefox-stable/ubuntu/dists/lucid/main/binary-amd64/Packages.gz
Document Length:        420 bytes
Concurrency Level:      550
Time taken for tests:   3.127 seconds
Complete requests:      25000
Failed requests:        0
Write errors:           0
Non-2xx responses:      25018
Total transferred:      20990102 bytes
HTML transferred:       10507560 bytes
Requests per second:    7994.07 [#/sec] (mean)
Time per request:       68.801 [ms] (mean)
Time per request:       0.125 [ms] (mean, across all concurrent requests)
Transfer rate:          6554.54 [Kbytes/sec] received
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   16 208.9      1    3002
Processing:     1   19  97.9      9    1478
Waiting:        1   19  97.9      9    1477
Total:          4   35 230.9     10    3023
Percentage of the requests served within a certain time (ms)
  50%     10
  66%     12
  75%     13
  80%     13
  90%     16
  95%     20
  98%    128
  99%   1056
 100%   3023 (longest request)
 ```


25k requests, at 550 concurrency, and I still can't make it return errors.

So it looks like the problem isn't with serving files from the cache, it's downloading new stuff at the same time, and serving simultaneously.  

So it's blocking.  Well that's an epic stack o' fail.

Here's some [more evidence](https://awaseconfigurations.wordpress.com/2011/11/13/problems-with-apt-cacher-ng-and-parallel-fabric-execution-or-the-crash-of-the-cacher/) to back up my findings.  They're using Squid. 

They're also using Fabric for orchestration.  Intriguing.
 
More here on this one later on.. When I've actually figured it out.
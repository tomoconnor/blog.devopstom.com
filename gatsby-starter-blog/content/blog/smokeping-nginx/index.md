---
title: "Smokeping on Nginx"
date: "2012-04-23T20:00:00.000Z"
description: "In praise of a simple tool"
warning: ancient
---

Smokeping is one of my favourite diagnostic tools for tracking down sporadic network issues.

You install it, configure it with a list of hosts, and it pings them regularly, and keeps track of the round-trip times, latency, packetloss, and so on.

The web frontend is a Perl CGI script, and as a result, it's a bit of a bugger to make it work on Nginx.

I wasn't gonna install Apache just for this one thing...

Firstly, my server I'm doing this on is ancient, so I installed Smokeping from source.  If you're running a more modern OS, and one where apt-get doesn't return 404 for the package files, I suggest you use vendor provided packages (or community provided PPAs).

Let's get to it.

I downloaded Smokeping from [here](https://oss.oetiker.ch/smokeping/pub/).  These [installation instructions](https://oss.oetiker.ch/smokeping/doc/smokeping_install.en.html) are great.  I already have rrdtool installed as it's a dependency of Munin (another firm favourite of mine) too.

Fping I downloaded from here , and shamefully built from source too.

The recommended webserver is Apache, but as I'm using Nginx already, and prefer it over Apache for performance and scalability, I decided it couldn't be that hard to do it without Apache.

I had to install a bunch of prerequesite Perl modules.  Fortunately, once you've extracted the smokeping distribution archive, there's a script "setup/build-perl-modules.sh" that does all the hard work for you.

So all I basically did. 
```
mkdir smokeping_install
cd smokeping_install
wget http://oss.oetiker.ch/smokeping/pub/smokeping-2.6.8.tar.gz
wget http://oss.oetiker.ch/smokeping/pub/fping-2.4b2_to4-ipv6.tar.gz
tar xzvf smokeping-2.6.8.tar.gz
tar xzvf fping-2.4b2_to4-ipv6.tar.gz
cd fping-2.4b2_to4-ipv6
./configure
make
sudo make install
cd ~/smokeping_install/smokeping-2.6.8
./configure
make
sudo make install
```
Which puts the fping binary in `/usr/local/sbin/fping`

and smokeping itself in 

`/opt/smokeping-2.6.8`
Things I had to do by hand:

```mkdir /opt/smokeping-2.6.8/cache
chmod a+w /opt/smokeping-2.6.8/cache
```
and of course, the Nginx config.

I wanted to tack smokeping onto my Munin vhost, so I just added a couple of sections to the bottom of that vhost configuration
```nginx
        location /smokeping {
            include proxy.conf;
            proxy_pass http://127.0.0.1:10000/smokeping.cgi;
        }
        location /smokeping/ {
            alias /opt/smokeping-2.6.8/htdocs;
        }
```
Nginx can't serve CGI scripts by itself, so it requires a CGI server bound to localhost in order to make those accessible.  I'm using Thttpd, as suggested here.

I downloaded `thttpd` from [here](http://www.acme.com/software/thttpd/).

It's insanely easy to build, same old combo of `./configure && make && make install`.

The Nginx wiki article about Thttpd CGI serving suggests a patch to thttpd for adding the X-Forwarded-For header. 

Patching the file is *easy*.  Just save the patch file, and drop into the thttpd-2.25 source directory, and run

`patch < thttpd.patch`
Then make and install as per usual.

Here's my thttpd.conf file (in `/etc/thttpd.conf`)
```
host=127.0.0.1
port=10000
user=www-data
logfile=/var/log/thttpd.log
pidfile=/var/run/thttpd.pid
dir=/opt/smokeping-2.6.8/htdocs/
cgipat=**.cgi|**.pl
```

Once smokeping is running, it will generate rrd files that can be examined by the CGI scripts to produce html output. 

Ta da!
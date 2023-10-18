---
title: "Appearance Can Be Deceptive"
date: "2012-03-16T20:00:00.000Z"
description: "Don't make legit things that look like malware."
warning: ancient
---

## Does this look suspicious to you?

```
85.25.184.184 - - [13/Mar/2012:00:41:09 +0000] "POST /3293b4da67abffca2460244619d9a8bca3f4c401.php HTTP/1.1" 200 1096 "-" "Mozilla/5.0 (Windows; U; MSIE 9.0; WIndows NT 9.0; en-US))"
85.25.184.184 - - [13/Mar/2012:00:42:57 +0000] "POST /3293b4da67abffca2460244619d9a8bca3f4c401.php HTTP/1.1" 200 359 "-" "Mozilla/5.0 (Windows; U; MSIE 9.0; WIndows NT 9.0; en-US))"
85.25.184.184 - - [13/Mar/2012:00:42:57 +0000] "POST /3293b4da67abffca2460244619d9a8bca3f4c401.php HTTP/1.1" 200 450 "-" "Mozilla/5.0 (Windows; U; MSIE 9.0; WIndows NT 9.0; en-US))"
85.25.184.184 - - [13/Mar/2012:01:14:12 +0000] "POST /3293b4da67abffca2460244619d9a8bca3f4c401.php HTTP/1.1" 200 354 "-" "Mozilla/5.0 (Windows; U; MSIE 9.0; WIndows NT 9.0; en-US))"
```


Sure as hell looks dubious to me.  Looks like something doing something that it oughtn't.  

My first instinct on seeing this is to look at the  `3293b4da67abffca2460244619d9a8bca3f4c401.php` file.

Let's have a quick look inside that file.

This looks dodgy:
```php
<?php
define('PAS_RES', 'atwentycharacterhash');
define('PAS_REQ', 'another20characters!');
define('RSA_LEN', '256');
define('RSA_PUB', '65537');
define('RSA_MOD', '1223197119012299291923139142398545904360596535434245223132131312321381297978456412347');
```

That's definitely dubious.  It doesn't get any better.

`$version=2;$requestId='0';$jsonRPCVer='2.0';`

Yuk. JSON RPC.  All on one line.  Yuk.

`function senzorErrorHandler($errno, $errstr, $errfile, $errline)`

This looks really dodgy, and it's where the story really unfolds.

Google for "`senzorErrorHandler`".  Top hit is http://stackoverflow.com/q/8250319/167299 from StackOverflow, with the particularly scary title "My wordpress has been hacked, but what did the hacker do and how can I prevent it/ fix damage done".  

I'm frequently advising people over on Serverfault that if your server's been hacked, you can pretty much forget about doing anything other than [Nuking It From Orbit](https://knowyourmeme.com/memes/nuke-it-from-orbit) and rebuilding from the last known good backup.  

Anyway.  Let's have a look at that IP address that it's coming from. `85.25.184.184`.  Who owns that?
```
inetnum:         85.25.184.0 - 85.25.185.255
netname:         BSB-Service-1
descr:           BSB-Service GmbH
route:          85.25.0.0/16
descr:          PlusServer AG
```

Well that's a bit non-descript.  That could be any server or botnet.  I don't like the way this is going.  Let's try opening a HTTP connection to that site.  Just drop it into your web browser.  Oh Look... Nothing there.  That's a bit dubious.

You'd be in the majority if you said "Screw that, IPTables it out, and delete that weird hex-named file in the webroot."  

You might even go as far as saying "Nuke it from orbit, and restore it from backups". 

If you did that, however, you'd destroy that site's profile with `WebsiteDefender`.

`WebsiteDefender` have an agent that sits in your site's root directory.  Instead of having a sensible name, like "website-defender-agent.php", they go with a 32char hex string, that looks dodgy.

Hidden away in the `WebsiteDefender` website, there's this page about [their scanning IP addresses](https://web.archive.org/web/20120501212133/http://www.websitedefender.com/faq/websitedefender-ip-addresses/)

Apparently you should allow access to scanners.websitedefender.com to allow their traffic to hit your server.  
```
;; ANSWER SECTION:
scanners.websitedefender.com. 3600 IN	A	188.138.0.168
scanners.websitedefender.com. 3600 IN	A	188.138.24.27
scanners.websitedefender.com. 3600 IN	A	188.138.56.125
scanners.websitedefender.com. 3600 IN	A	85.25.184.184
```

And there's that mysterious 85.25 IP address.  I came very close to destroying that agent file, then did some closer digging.

If you read the non-accepted answer on that StackOverflow question, then you'll see that someone else has drawn the same conclusion as me. Actually, this is how I figured it out, but I think it bears repeating.

So there's a few lessons to be drawn from here. 

I've no particular exception to website agents, and having files on your website for them. 

Here's how I'd improve the WebsiteDefender one.

1. Rename the agent file from something hexadecimal and weird, to "website-defender-agent.php"
2. Set up better whois information for your IP addresses.
3. Have your scanning IPs redirect inbound HTTP requests to your FAQ. (This alone would have helped instantly).
4. Don't hit the server so damn frequently.
5. Hit the server from different IP addresses now and again.
6. Oh, and put a block of comments at the top of your Agent code saying What it is, Where it's from, What it does, and Why it's there.

That would make life a lot easier for any sysadmin who finds the same file and panics a little.
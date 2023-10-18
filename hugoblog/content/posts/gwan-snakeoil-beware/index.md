---
title: "GWAN: Snakeoil Beware"
date: "2012-11-12T20:00:00.000Z"
description: "Sometimes some things just don't add up..."
warning: ancient
---

I’ve heard quite a bit about the “G-WAN Application Server” over the past few weeks.  Initially it was a Serverfault question that left me thinking “WTF” (http://serverfault.com/questions/445835/virtual-host-on-g-wan)

I took a look at their website and thought: “Those are pretty insane claims”.  They’re also the kind of crap you tend to see where the intended audience is somebody who has absolutely no clue about scalability, or production-readiness. Y’know, Managers.

 > - Quite well summarised by this comment:   GWAN isn't designed to be a robust webserver, it's designed to perform exceptionally well in contrived and outlandish benchmarks, so PHBs will demand the IT team use it and buy support... – Chris S♦ 19 hours ago

Interestingly enough, the only person who answered that question was a fellow called Gil who apparently works for G-WAN..  I don’t normally take much offense to product owners on Serverfault et al, but the vast majority of his answers do seem to be a bit spammy. 

A lot of the pages on the website refer back to benchmarking the server.  I’m really not interested in that, not here, anyway.  You’ll see why in a little while.

Moving on, somewhat, I decided that I should at least *download* the server, and have a poke about.  

So, I downloaded the tar.bz2 file containing the server (Bzip2? I suppose they’d be interested in making it *appear* as small as possible.)

This is what I unbzipped. 

https://pastebin.com/E2t0vbwi

 

I’m a little terrified,  to be quite honest.

One of the things I *love* about Apache is being able to download the source code and  take a good old nose about in it.  I rarely *need* to, but it’s nice to have the option.  The biggest problem I have with closed-source applications is that this isn’t possible.  We just have to trust that they’ve not left any big-ass buffer exploits in there, or that the code doesn’t also contain a ssh daemon, or there’s not a backdoor that’s gonna send all my keystrokes to $unfriendly_nation. 

That doesn’t appear to be possible, because of this gwan: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, stripped

So, back to that directory tree.. 

There’s a mysterious “0.0.0.0_8080” directory, containing *more* stuff. This appears to be some kind of virtualhost configuration. 

Interesting way to do it, but it’s a bit esoteric. 

Inside that directory, there’s the even more bizarre “#0.0.0.0” directory, which sounds entirely redundant, because didn’t the next level up already define the IP address? 

-- Apparently the 2nd level directory is for defining virtual hosts on the listener.  Well, that *almost* makes sense, but why not just use a config file like Apache, Nginx, Lighty, or well, any other goddamn server, Ever. 

Oh well, it’s weird. 

Inside that, there’s ./csp, ./www, ./handlers and ./logs.

From their README.txt:
```
/csp ........ G-WAN C/C++ and Objective-C/C++ examples
/www ........ G-WAN web server's 'root' directory
/handlers ... G-WAN web server's handler sample
/logs ....... G-WAN web server's access/error log files
```
OK.  So what’s the difference between a csp and a handler? 

Well, inside csp, there’s a file called hello.c, which appears to be a *very* simple hello world example.  Frankly, I’m a bit bored of Hello World examples, so let’s see if I can make my own directory structure, and make a FizzBuzz page?  That way I’ll have some idea how ass this idea is, and whether I’d ever *ever* see a use for it.

"`handlers`" is an odd one.  I get that .csp contains files like hello.c which seem to be the closest thing to a MVC’s Controllers. Maybe. 

Handlers is worrying, actually.  the file main_hello.c__ appears to contain some kind of HTTP handler, assigning c functions to the processes required to build a HTTP request. Why do I care about building a HTTP request?  Do I care? Do I *need* to write a special handler just to pass off to fizzbuzz.c?

Of course, there’s bugger all documentation to tell me what to do.  There is however, commercial support, which starts at 149 Generic Currency Units (their page shows Swiss Francs, but I can’t seem to change it).

So a “Hobbyist” pays 149 GCU. Seems steep, most other Hobbyist programmes are free. 

Consultants pay 1499 GCU. 

Enterprises pay 10x that amount. 

If you want 24x7 support, you’re looking to pay 149,999.00 GCU. 

God help you if you also want to white label it, as they tack on an extra 199,999.00 GCU for that. Blimey.  

For an extra 7,999 GCU you can enter into a code escrow agreement, meaning that you get an encrypted version of their source code, and if they go bust, they *hopefully* give you the key.

I think it’d be a far better use of the money to pay for the Code Escrow, and then use the 250k you’ve saved on Amazon GPU instances to bruteforce the key.  But there we go.  You’d probably find that they’ve used Ultra super duper mega encryption that was *also* written in house.   

Back on topic.  Well, almost.

I just found this rather bold statement hidden in their FAQ (http://gwan.com/faq#license):

> “G-WAN never had a security breach since day one in June 2009 (other servers can't sustain the same claim).”

I suspect there’s one reason for them to make that statement. Nobody’s using it.

Alternatively, it could be that they do exist, but nobody’s found them because we can’t look at the source code and find them.

G-WAN was written to port Desktop apps to the Web because there was no Web application server able to do that job.

Eugh. 

There’s two ways to make Desktop applications more widely available.  

1) Rewrite the fucking thing, don’t port it.

2) Use something like Citrix XenApp to publish it.  

I don’t approve of the concept of writing a server to “port” a desktop application.  There’s something deeply odd about that concept. 

Despite its small footprint, G-WAN is an all-in-one solution because communicating with other servers (FastCGI, SCGI, etc.) takes time (enlarging latency), and wastes CPU and RAM resources. Remember that our goal here is to use the ultimate low-latency and resource-saving solution. This is why G-WAN is a:

> * Web server
> * App. server
> * Cache server
> * Key-Value store server
> * Reverse-proxy and elastic load-balancer server

What ever happened to the UNIX philosophies  of doing one thing, and doing it well, and then having a bunch of loosely coupled servers performing different tasks. Evidently not considered here.  

I’d rather have a separate web server, a separate application server, connect the two over proxied HTTP, then connect to the cache with a non-proprietary protocol, and the key/value server over http or a similar mechanism. 

No, I don’t like the idea of having all in one box.  It’s asking for mischief.

Oh, and their examples are weird.   I found this snippet earlier on, and it made me go “Huh?”
```c
// but we don't want to display "%20" for each space character
   {
      char *s = szName, *d = s;
      while(*s)
      {
         if(s[0] == '%' && s[1] == '2' && s[2] == '0') // escaped space?
         {
            s += 3;     // pass escaped space
            *d++ = ' '; // translate it into the real thing
            continue;   // loop
         }
         
         *d++ = *s++; // copy other characters
      }
      *d = 0; // close zero-terminated string
   }
```

I dunno about you, but I’d probably just call urldecode() and be done with it.

Oh, and this.

http://pastebin.com/zVnPKzuq

Templating. Heard of it? MVC? Heard of it? Evidently not.

Right.  Back to trying to make a Fizzbuzz application?

I had a brief attempt, and when I restarted the server I got this: 
```
vagrant@precise64:~/gwan_linux64-bit$ ./gwan
Floating point exception
```
No logs, nothing on stderr, nothing.  Smooth, guys.  I half expected a `HTTP 402 - Payment Required` ;)

 

-- The 2nd time I tried, I got this equally cryptic error message
```
Signal        : 11:Address not mapped to object
Signal src    : 1:SEGV_MAPERR
errno         : 0
Thread        : 0
Code   Pointer: 000000445659 (module:./gwan, function:??, line:0)
Access Address: 000000000001
Registers     : EAX=000004113418 CS=00000033 EIP=000000445659 EFLGS=000000010246
                EBX=000004113418 SS=08201c24 ESP=000008801a00 EBP=000000000000
                ECX=0000004457e0 DS=08201c24 ESI=000000000001 FS=00000033
                EDX=000000000000 ES=08201c24 EDI=000004113418 CS=00000033
Module         :Function        :Line # PgrmCntr(EIP)  RetAddress  FramePtr(EBP)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

This bit is epic. “`function ?? line: 0`”  - You’d think it’d know where it threw a wobbly. 

I’ll remind you that when I fuck up in ruby or python, the interpreter tells me where it failed. I like that, it makes debugging a huge application far easier. 

I really can’t be arsed to make a contrived example work on a server that’s just weird. 

I’m gonna go back to writing in *real* languages, designed with web application development in mind. 

I just can’t take application development seriously if there’s no kind of built-in MVC.  

My conclusions are these: 

**G-WAN is the Gentoo of the Application Server field.**
Designed for those who are more concerned with made up benchmarks and the belief that they’ve tuned *every* little feature to the max.

It’s also weird in all the ways I mentioned above.  

Oh, and don’t even consider using it in a production environment.  If you’ve *really* got to the point that you can’t tune Apache, or Nginx, or Lighty, then there are other solutions, but G-Wan isn’t one of them.

There’s Varnish, and there’s HAProxy, and there’s Pound and there’s other ways to accelerate your application, but I can’t see any benefit from rewriting into C and using a somewhat shonky API with a bizzare lack of debug output. 

I mean, by all means, if you’ve got the cash to drop on the support, do so, but I’d rather spend that cash on hiring a skilled engineer to make your application scale, and your servers hum happily.

Oh, and I suspect that if you ran G-WAN on Gentoo you’d find one of three things.

1. You’d create a supermassive black hole.
2. Everything would slow down (relativistically speaking).
3. Nobody would give a fuck. 

Probably the latter.
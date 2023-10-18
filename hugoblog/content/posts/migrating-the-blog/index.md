---
title: Migrating this Blog
date: "2021-12-29T22:56:01.123Z"
description: "Moving this blog to the CLOUD!"
---

I've been maintaining this blog for a *very* long time.  I actually know exactly how long (since 2008), because I've been through every single article in the last few days, and re-formatted everything in Markdown. 

Originally, this blog started off as a Wordpress site, then became something in Perl, then became something I wrote around Django 0.9. 

The Something based on Django lasted unti fairly recently, when I basically became bored of maintaining it, and didn't particularly want to try and migrate it to a later version of Django. 

And I was recently reminded that sometimes when linking to it, it'd randomly redirect to hxxp://127.0.0.1:8000/ -- Which *was* the Proxy address, but I have *absolutely* no idea why it did this (and spent an hour re-confirming that I had no idea why it did this).

So I started on 26th December, manually reformatting every page into Markdown, replacing images and links from cache / archive.org / other sources, and reformatting it so that old pages now have a warning like: 

**This page is ancient and has linkrot**

or similar depending on what `warning` tag I configured for that page. 

Linkrot is stunning.  So many sites that you expect are going to be around forever, apparently aren't. 

So, starting off with [Gatsby](https://www.gatsbyjs.com/), and the [Gatsby Starter Blog](https://github.com/gatsbyjs/gatsby-starter-blog) - I started migrating everything into plain markdown, in a fairly sane structure. 
I thought about the following, but it wasn't actually worth it in the amount of time it'd take to script, vs the amount of time to do it manually. 

1. Dump the SQL from the database, and script a thing to rebuild all the blogposts as Markdown. - This was a non-starter, because the django-cms database layout is a mess.
2. Spider the site, and convert HTML to Markdown - The downside to this is that there's a lot of missing links and broken images.
3. Manually go through each post and copy to `something.md` - This is what I did, and it gave me an opportunity to drop a bunch of no-longer-relevant crap, and update links/images.

It took me about 3 days to reformat everything into Markdown, then I thought about uploading the static files to AWS/S3 and putting Cloudfront in front of it, but because I  was tired by the end, I decided that I'd give Gatsby Cloud a shot, given that Gatsby is the static site generator I'm using, and for a single site, it's free. 

So with the site now in Github, and connected to Gatsby Cloud, and deployed, the next question is around the old site, and what to do about people who might have the old site bookmarked  and search engines that've got the old site indexed. 

So I wrote some [quick and dirty Go](https://github.com/tomoconnor/redirectify/blob/main/main.go), that basically sits as a HTTP server, pretending to be the old webserver, and responds with HTTP 301 "Moved Permanently" and the new URL. 

So now the blog lives here at https://blog.devopstom.com/, and hopefully I should be able to fully decommission the old server in 3-6 months. 








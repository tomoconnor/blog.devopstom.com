---
title: mod_rewrite is killing social media
date: "2011-04-24T17:00:00.000Z"
description: "A rant about image hotlinking not being all that bad"
warning: ancient
---

This is a little ranty, but it's really pissed me off lately. 

That’s right. It’s you. The ones with image hotlink protection, and the ones who rewrite URLs to do strange and special SEO things, but who don’t actually think about what happens when you send someone a link to something.

(For the uninformed, hotlink protection is that thing where you get sent a link to an image, but the site owner is being draconian, and redirects you to google, because your referer wasn’t their own site, so the image must have been stolen, and put on another webpage (!))

Here’s what happens.  Someone makes a blog, and posts a funny image of a kitten.   We all like kittens, so I copypasta the link, and send it to my friend. 

Problem is, the site owner is being a twat. They think that we’re still in the 1990s, and bandwidth is expensive.  They set cookies when I visit the site, and then they look for those when I look at their images. 

I post a tweet like “Hey, check out this cute kitty! hxxp://you.are.killing.the.inter.net/kitty.jpeg”

I have the cookies, so it looks fine. My friends, however, do not, so they redirect to google, or something equally stupid.

Here’s the result. Either I look stupid, or they look stupid, or both.  Neither of these are particularly good things. 

I can’t save the image, and host it somewhere else, because that would be stealing it from the site owner / copyright holder, adding a dose of further legal problems, and also a massive layer of effort on top. 

Here’s what site owners should do.  Stop being a twat.  If you’re concerned about bandwidth usage from your assets, host them on Amazon’s S3 cloud, and shovel it all through Cloudfront.  Set up a CNAME to your Cloudfront Distribution point, like “media.killing.the.inter.net”, and serve your static assets through there.  You’ve got enough bundled bandwidth in the Free Tier to last more than long enough, and you also leave a sensible system by which I can share your media files on the social web.

The first time I found this today, was on some guy’s site where he claimed to be “the self appointed curator of the internet”.  Hell,  I think I’m better for the health and wellbeing of the internet, personally.  I don’t protect against hotlinking, because it’s stupid. It’s like anti-right-click scripts on websites. Those are fucking dumb too. 

By “protecting” your images with some mod_rewrite trickery, you’re actually diminishing the traffic to your website. I’m never going to link to you again, because you’ve got crap policies.  You’ve also lost the inquisitive organic traffic sources, the people who go “ I wonder what else is on that site”, because you bounce them through to google, instead of your homepage, or the page that the image was originally on.  That would be smart, that would mean you’d get more traffic in general, more adword hits, etc etc.  

But no, you’re all still living in the past, back in the days when bandwidth was an expensive commodity.  Wake up and smell the megabits.  We’re not in that world any more.  If your host is threatening to cut you off for costing them a fortune in bandwidth, tell them to fuck off, and find somewhere else.  There’s no shortage.

Hell, go it alone on an EC2 micro instance on the free tier.  I’ll even tell you how to do it. 

Secondly, if I’m visiting webpages on your site, I’d like you to do 2 things. They’re really simple, and you should have been doing them for years. 

1) When I click a link, I’d like the Address bar to change accordingly.  Or you can show a permalink link.  One or the other.   I’d like to be able to share your website with my friends on twitter, or IRC, or Facebook.  I can’t do this if you don’t give me the links to share.  All I end up sharing is an invalid link that then bounces them to a HTTP Err 500 page, or a 302 Redirect to google.  Yeah. Smart move there.   NOT.

2) This is the biggie.  I’d really like it if once you’ve generated an URL, then it doesn’t change.  Ever. You could make my life immeasurably easier if I can keep a bookmark to your site for 10 years, and never have to wonder “where’d that page go? I’m sure that URL is right...”

Oh, and read this: http://www.w3.org/Provider/Style/URI
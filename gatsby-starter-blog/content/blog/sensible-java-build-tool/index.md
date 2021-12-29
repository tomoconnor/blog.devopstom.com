---
title: A Sensible Java Build Tool
date: "2011-07-22T17:00:00.000Z"
description: "TL;DR: Maven."
warning: ancient
---

I've been writing Java in one sense or another for a few years now.  I learnt stuff at university, then used it in a few jobs.  I've written Beans and Applets, and various bits of stuff in between. 

It's fairly safe to say, that I like Java.  

One thing however, that's been a pretty consistent bugbear in all the time I've been writing Java, has been the classpath, and dependency resolution.  Luckily, all that can now change.  

I used to think that Ant was a pretty neat build tool.  All the IDEs supported it, it kinda worked most of the time, but sometimes, building was a bit of a ballache - Some stuff had to be in your lib/ folder, sometimes in Ant's lib/ too.  

Lately though, and this week in particular, I've been playing with Maven.  

Maven is a pretty fucking cool build tool for Java applications.  I suspect it probably works with other languages, but it's "designed" for Java. 

I don't think I really have the expertise or knowledge to explain how Maven works, partly because I haven't studied the inner workings that deeply, but also, because it's far better explained [here](https://maven.apache.org/what-is-maven.html)

Instead, I'm going to dive right in, and explain what I've been working on this week.  

The company I work for currently, is making a pretty radical shift away from using PHP for everything.  Instead, we've been investigating Java for creating a middleware layer that everything can talk to.

I'm pretty chuffed with this, but I do wish that it had come a lot earlier on.  If it had, I might not have been so decisive to leave when offered a better job.

Basically, when we came up with this project, I insisted that we do it properly, for a change.  

I suggested that a good workflow would be something like: 
> Netbeans IDE -> Maven Project -> Git SCM -> Jenkins CI -> Maven Repository 
(We chose Artifactory, but I did test Sonatype Nexus too, but didn't like it).

This is a good pattern for the Joel Test's *"Can you make a build in one step?"*

I basically wanted to create a demo project that can be used as the basis for all future java projects, I do the R&D to make the initial POM work, then everyone else can clone this, or inherit from it.. 

This decision was twofold, I also wanted to figure out JPA/Hibernate and have some clue how that works for reverse engineering the classes from an existing database, the answer to that is: Pretty well, actually. - But that's another story.

My IDE of choice is Netbeans.  I've been using it since I was at university, except for a small android-related foray into Eclipse, and an experimental nosing around IntelliJ IDEA. 

Stuff I did:

1. Created a new Netbeans Maven project from the quickstart archetype.
1. Added the Dependencies on Hibernate (all the sub-dependencies get resolved, and added)
1. Added the `<scm></scm>` and` <ciManagement></ciManagement>` lines to the POM
1. Added maven-shade-plugin to allow us to build a fat JAR, which makes the jar bigger - it includes all the dependency JARs, but does make deployment a damnsight easier.
1. Configured `<distributionManagement></distributionManagement>` to contain the url of the repository we're using.

That's pretty much it.  Here's the finished POM, with various bits of secret removed. 

When I edit something in Netbeans, and commit a change, there's a post-commit hook (post-receive) that calls the Jenkins API, and builds the project.  Jenkins then deploys the artifacts (a fat JAR and a POM) to the Artifactory.

Epic.

 
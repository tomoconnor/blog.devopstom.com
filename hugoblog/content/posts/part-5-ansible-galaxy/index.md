---
title: "Part 5: Ansible Galaxy"
date: "2014-10-30T19:59:58.000Z"
description: "Part 5 of 5"
warning: ancient
---

## This article suffers from severe linkrot.

It's been a while since I wrote Parts 1,2,3,4 on my Ansible Tutorial series, but I've recently changed my approach somewhat when using Ansible, and certainly when I build on Parallax.  

I've started using more and more from Ansible Galaxy.  For those of you who don't know, Galaxy is a community "app store"-like thing for sharing reusable Ansible Roles.  

https://galaxy.ansible.com/

Let's pretend we want to deploy a staging server for a Python/Django application, using Postgres as the backend database all on a single server running Ubuntu 14.04 Trusty.

I've recently done something similar, so I know roughly what roles I need to include.  YMMV.  

Starting with the basic stuff.  Let's find a role to install/configure Postgres.  

https://galaxy.ansible.com/explore#/

Click the "database" category.  

I tend to like to sort by Average Score, in Reverse order, so you get the highly rated ones at the top.

The top-rated Postgres role is from ANXS https://galaxy.ansible.com/list#/roles/512

There's a bunch of useful links on that page, one to the role's github source, and the issue tracker.

Below, there's a list of the supported platforms (taken from the role's metadata yml file).

Just check that your target OS is listed there, and everything will probably work fine.

It's also worth checking that your ansible installed version is at least as new as the role's Minimum Ansible Version.

Starting with a base-point of Parallax (because it still has some really useful bits and bobs in it - like 'common').. 

cd ./playbooks/part5_galaxy (or whatever you've called your playbook set).

If you want to directly install the role into the roles/ directory, you'll need to append the -p flag, and the path (relative or absolute) to your project's roles directory.  Otherwise they tend to get installed in a global location (which is a bit of a pain if you're not root).

So when you run:
```
ansible-galaxy install -p roles ANXS.postgresql
 downloading role 'postgresql', owned by ANXS
 no version specified, installing v1.0.3
 - downloading role from https://github.com/ANXS/postgresql/archive/v1.0.3.tar.gz
 - extracting ANXS.postgresql to roles/ANXS.postgresql
ANXS.postgresql was installed successfully
```

You should have output that resembles that, or something vaguely similar.

The next thing to do, is to integrate that role into our playbook.  

In tutorial.yml, you can see that there's a vars: section in the play definition, as well as some variables included when a role is listed.

This also introduces a slightly different way of specifying a role within the playbook, where you can follow each role up with the required options.

There's an option within ANXS.postgresql to use monit to ensure that postgresql server is always running.  If you want to enable this, you will also need to install the ANXS.monit role.

In a way not entirely different to pip freeze, and the requirements file, you can run

`ansible-galaxy list -p roles/ >> galaxy-roles.txt` 
and then be able to reimport the whole bunch of useful roles with a single command:

`ansible-galaxy install -r galaxy-roles.txt -p roles`

I've determined from past experience that the following Galaxy roles tend to play nicely together, and will proceed to install them in the tutorial playbook so you get some idea of how a full deployment workflow might look for a simple application.

These are the roles I've used.. 
```
 ANXS.apt, v1.0.2
 ANXS.build-essential, v1.0.1
 ANXS.fail2ban, v1.0.1
 ANXS.hostname, v1.0.4
 ANXS.monit, v1.0.1
 ANXS.nginx, v1.0.2
 ANXS.perl, v1.0.2
 ANXS.postgresql, v1.0.3
 ANXS.python, v1.0.1
 brisho.sSMTP, master
 EDITD.supervisor_task, v0.8
 EDITD.virtualenv, v0.0.2
 f500.project_deploy, v1.0.0
 joshualund.ufw, master
```

ANXS provide a great many roles which all play nicely.  Some of those are included as they are dependencies of other roles.  I tend to use sSMTP to forward local mail to Sendgrid, because I hate running email servers. 

`f500.project_deploy` is a capistrano-like deployment role for Ansible which supports the creation of symlinks to the current deployed version (which subsequently allows rollbacks).

I don't want to go into the process of explaining how to modify this to deploy a Django application, I'm going to assume you've got enough information to figure that out for yourself.  

I've also added the ufw role, which configures Ubuntu's ufw package, a neat interface to IPTables.  

Basically, it should be quite easy to see how it is possible to build a playbook without having to write quite so much in the way of new ansible tasks/modules.


OTHER USEFUL COMMANDS:
`ansible-galaxy init `
This will create a role in a format ready for submission to the Galaxy community.

`ansible-galaxy list`
Show currently installed roles.

`ansible-galaxy remove [role name] `
Removes a currently installed role.

ENDNOTE
When you look at the list of available roles, it's quite staggering what you could possible integrate, without having to do too much coding yourself.

It's fantastic.  At the time I wrote this article, there were 7880 users, and 1392 roles in total.  It's also growing rapidly day on day. 

There's plenty more information on the Galaxy intro page, which covers how to share your own roles.
---
title: "Part 1: Getting Started With Ansible"
date: "2014-01-26T19:59:58.000Z"
description: "Part 1 of 5 all about Ansible"
---
An introduction to Ansible Configuration Management

 

A brief history of Configuration Management

===========================================
* CFEngine - Released 1993. Written in C
* Puppet - Released 2005 - Written in Ruby. Domain Specific Language (DSL. SSL Nightmare.
* Chef - Released 2009 - Written in Ruby, also a DSL, more like pure Ruby
* Juju - Released 2010, Python, Very ubuntu.
* Salt - Released 2011, Python, Never got it working right
* Ansible - Released 2012, Python.  Awesome. 

Why Ansible?
============

It’s agentless.  Unlike Puppet, Chef, Salt, etc.. Ansible operates only over SSH (or optionally ZeroMQ), so there’s none of that crap PKI that you have to deal with using Puppet.

It’s Python. I like Python.  I’ve been using it far longer than any other language. 

It’s self-documenting,  Simple YAML files describing the playbooks and roles.

It’s feature-rich.  Some call this batteries included, but there’s over 150 modules provided out of the box, and new ones are pretty easy to write.

 

Installing Ansible
==================

 

You can get it from the Python Package Index (PyPI):

`pip install ansible`
 

You can get it from your OS package index

`sudo apt-get install ansible`
 

You can download the source from Github and run setup.py yourself.

`git clone https://github.com/ansible/ansible.git`
 

My preferred way of installing it is inside a virtualenv, then using pip to install it. 


Ansible Modes
=============

Playbook Mode

 - This executes a series of commands in order, according to a playbook.

 

Non-playbook mode

 - This executes an ansible module command on a target host. 

 

I'll primarily be focussing on Playbook Mode, and hopefully giving an insight on what a playbook consists of, and how to use Ansible to deploy an application.

Parallax
========

I've put together a collection of Ansible bits I've used in the past to give a quick-start of what a Playbook might look like for an example service.  

I'll be referring back to this in the rest of this article, so you'll probably want to grab a copy from Github to play with:

`git clone https://github.com/tomoconnor/parallax.git`
 

First Steps
===========

 

1. Install Ansible (see above)
2. Clone Parallax

 

From a first look at the source tree of Parallax, you should see a config file, and a directory called "playbooks".

The config file (ansible.cfg) contains the ansible global configuration.  Lots more information about it, and its directives can be found here: 

http://docs.ansible.com/intro_configuration.html#the-ansible-configuration-file



Playbooks
---------

Playbooks are the bread and butter of Ansible.  They represent collections of 'plays', configuration policies which get applied to defined groups of hosts.

In Parallax, there's a "playbooks" directory, containing an example playbook to give you an idea of what an Ansible Playbook looks like.

 

Anatomy of a Playbook
=====================

If you take a look inside the Parallax example playbook, you'll notice there's the following file structure:
```
.
├── example_servers.yml
├── group_vars
│   ├── all
│   └── example_servers
├── host_vars
│   └── example-repository
├── hosts
├── repository_server.yml
├── roles
│   ├── __template__
│   ├── common
│   ├── gridfs
│   ├── memcached
│   ├── mongodb
│   ├── nginx
│   ├── nodejs
│   ├── redis
│   ├── repository
│   ├── service_example
│   └── zeromq
└── site.yml
```

Looking at that tree, there's some YAML files, and some directories.  

There's also a file called "hosts".  This is the Ansible Inventory file, and it stores the hosts, and their mappings to the host groups.

The hosts file looks like this:
```ini
[example_servers]
192.168.100.1 set_hostname=vm-ex01
# example of setting a host inventory by IP address.
# also demonstrates how to set per-host variables.

[repository_servers]
example-repository
#example of setting a host by hostname.  Requires local lookup in /etc/hosts
# or DNS.
[webservers]
web01
[dbservers]
db01
```

It's standard INI-like file format, hostgroups are defined in [square brackets], one host per line.  Per-host variables can follow the hostname or IP address.  If you declare a host in the inventory by hostname, it must be resolvable either in your /etc/hosts file, or by a DNS lookup.

The playbook definitions are in the .yml files.  There's 3 in the Parallax example.  Two which are separate YAML files, and one that's a kind of, catchall in 'site.yml'.

site.yml is the default name for a playbook, and you'll likely see it crop up when you look at other ansible examples (https://github.com/ansible/ansible-examples/).

You'll also see lots of files called 'main.yml'.  This is the default filename for a file containing Ansible Tasks, or Handlers.  More on that later.

So, site.yml, consists of 3 named blocks.  If you look closely, you'll see that the blocks all have a name, they all have a hosts: line, and they all have roles.

The hosts: line sets which host group (from the Inventory file 'hosts') to apply the following roles to.

The roles: line, and subsequent role entries define the roles to apply to that hostgroup.   The roles currently defined in parallax can be seen in the above tree structure.

You can either put multiple named blocks in one site.yml file, or split them up, in the manner of 'example_servers.yml' and 'repository_server.yml'

Other stuff in 'site.yml':

'user:' - This sets the name of the user to connect to the target as.  Sometimes shown as remote_user in newer ansible configurations.  

'sudo:' - This tells Ansible whether it should run sudo on the target when it connects.  You'll probably want to set this as "sudo: yes" most often, unless you plan to connect as root.  In which case, this (ಠ.ಠ) is for you.

 



Roles
=====

A role should encapsulate all the things that have to happen to make a thing work.  If that sounds vague, it's because it is. 

The parallax example has a role called common, which installs and configures the things that I've found are useful as prerequisites for other things.  You should go through and decide which bits you want to put into your 'common' role, if you decide to have one.

Roles can have dependencies, which will require that another role be applied first.  This is good for things like handling the dependencies before you deploy code.



Inside A Role
-------------

Let's take a look at one of the pre-defined roles in Parallax: 
```
├── redis
│   ├── files
│   ├── handlers
│   ├── meta
│   ├── tasks
│   └── templates
```

This, unsurprisingly is a quick role I threw together that'll install Redis from an Ubuntu PPA, and start the service.

In general, a role consists of the following subdirectories, "files", "handlers", "meta", "tasks" and "templates".

files/ contains files that will be copied to the target with the copy: module.

handlers/ contains YAML files which contain 'handlers' little bits of config that can be triggered with the notify: action inside a task. Usually just handlers/main.yml - See http://docs.ansible.com/playbooks_intro.html#handlers-running-operations-on-change for more information on what handlers are for.

meta/ contains YAML files containing role dependencies.  Usually just meta/main.yml

tasks/ contains YAML files containing a list of named steps which Ansible will execute in order on a target.  Usually tasks/main.yml

templates/ contains Jinja2 template files, which can be used in a task with the template: module to interpolate variables in the template, then copy the template to a location on the target.  Files in this directory often end .j2 by convention.



Example Role: Redis
-------------------

 

Path: `parallax/playbooks/example/roles/redis`

Structure: 
```
.
├── files
├── handlers
├── meta
├── tasks
│   └── main.yml
└── templates
```

All there is in this one, is a task file, unsurprisingly called 'main.yml' - Told you that name would crop up again.
- Actually, there's a .empty file under files, handlers, meta, and templates.  This is just so that if you commit it to git, the empty directories won't vanish.  

 

Let's have a look at the redis role's tasks:

```yml
$ cat tasks/main.yml
---
 - name: Add the Redis PPA
   apt_repository: repo='ppa:rwky/redis' update_cache=yes
 - name: Install Redis from PPA
   apt: pkg=redis-server state=installed
 - name: Start Redis
   service: name=redis state=started
```

Each named block has an action below it.  Each action refers to an Ansible Module. There's an index of all available modules and their documentation here: http://docs.ansible.com/list_of_all_modules.html

 

Basically explained:

`apt_repository`: module configures a new apt repository for the system.  It can take a ppa name, or a URL for a repository.  update_cache tells ansible to run apt-get update after it's added the new repository.

`apt`: module tells Ansible to run apt-get install $pkg using  whatever value has been defined for pkg. 

`service`: tells Ansible to execute "`sudo service $name start`" on the target.

 

I recommend you have a trawl through the roles as configured in Parallax, and see if you can make sense of how they work.  If you open the Ansible Module Index, you'll be able to use that as a quick reference guide for the modules in the roles.

 

One of the most useful features of Ansible, in my opinion is the "with_items:" action that some modules support.  If you want to install multiple packages with apt at the same time, the easiest way to do it is like this: 

(example from roles/common/tasks/main.yml)

 
```yml
 - name: install default packages
    apt: pkg={{ item }} state=installed
    with_items:
      - aptitude
      - vim
      - supervisor
      - python-dev
      - htop
      - screen
 ```

Running Ansible
===============

 

Once you've got your Host Inventory defined, and at least one play for Ansible to execute, it'll be able to do stuff for you,

 

I've just spun up a new Ubuntu 13.10 Virtual Machine.  It has the IP Address 192.168.1.96

 

I'm going to create a new hostgroup called [demoboxes] and put that in:

```
[demoboxes]
192.168.1.96 access_user=user
```

The variable access_user is required *somewhere* by the common role, to create the ssh authorised keys stuff, under that user's home directory. 

and in site.yml:

```yml
- name: Install all the packages and stuff required for a demobox
  hosts: demoboxes
  user: user
  sudo: yes
  roles:
    - redis
    - nginx
    - nodejs
    - zeromq
```

I've included a few other roles from parallax for the halibut. 

I'm going to run ansible-playbook -i hosts site.yml and see what happens. 

For the first run, we'll need to tell ansible the SSH and Sudo passwords, because one of the thing that the common role does is to configure passwordless sudo, and deploy a SSH key. 

In order to use Ansible with SSH passwords (pretty much required for the first run of normal machines (unless you deploy keys with something far lower level, like kickstart), you'll need the sshpass program. 

On ubuntu, you can install that as follows:

`sudo apt-get install sshpass`
When you use Parallax as a starting point, one thing you'll want to do is edit

 `roles/common/files/authorized_keys`
and put your keys in it. 

 

So, for a first run, it's:

 `ansible-playbook -i hosts -k -K site.yml`
 

You'll get the following prompts for the ssh password, and the sudo password:

SSH password:
sudo password [defaults to SSH password]:
 

Enter whatever password you gave Ubuntu at install time. 

 

Once the following tasks have completed, you can remove -k -K from the ansible command line

```
TASK: [common | deploy access ssh-key to user's authorized keys file] *********
changed: [192.168.1.96]
TASK: [common | Deploy Sudoers file] ******************************************
changed: [192.168.1.96]
```

Because at that point, you'll be able to use your ssh key, and passwordless sudo.

 

At the end of the run, you'll get a Play Recap, as follows:
```
PLAY RECAP ********************************************************************
192.168.1.96               : ok=19   changed=8    unreachable=0    failed=0
```

You should now be able to open http://192.168.1.96/ (or whatever your server's IP address is) in a browser.
```
Toms-iMac-2:example tomoconnor$ curl -i http://192.168.1.96/
HTTP/1.1 200 OK
Server: nginx/1.4.1 (Ubuntu)
Date: Sun, 26 Jan 2014 17:48:47 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Mon, 06 May 2013 10:26:49 GMT
Connection: keep-alive
ETag: "51878569-264"
Accept-Ranges: bytes
```

Hurrah. 
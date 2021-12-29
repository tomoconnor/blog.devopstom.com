---
title: "Part 2: Deploying Applications with Ansible"
date: "2014-01-27T19:59:58.000Z"
description: "Part 2 of 5"
---
You should by now have worked your way through Part 1: Getting Started with Ansible.  If you haven't, go and do that now. 

In this article, I'll be demonstrating a very simple application deployment workflow, deploying an insanely simple node.js application from a github repository, and configuring it to start with supervisord, and be reverse-proxied with Nginx.

As with last time, we'll be using Parallax as the starting point for this.  I've actually gone through and put the config in there already (if you don't feel like doing it yourself ;)

 
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
#    - deploy_thingy
```

In the `9c818d0b8f` version, you'll be able to see that I've created a new role, inventively called "`deploy_thingy`".  

**Updated**

I've been recommended that my `__template__` role be based on the output of 

`ansible-galaxy init $rolename`

So I've recreated the `__template__` role to be based on an ansible-galaxy role template.

There's not that many changes, but it does include a new directory 'default/' containing the Galaxy metadata required if you wish to push back to the public galaxy role index.

 

In an attempt to make creating new roles easier, I put a `__template__` role into the file tree when I first created Parallax, so that all you do to create a new role is execute:

`cp -R __template__ new_role_name`
in the roles/ directory.
```
.
├── files
│   ├── .empty
│   ├── thingy.nginx.conf
│   └── thingy.super.conf
├── handlers
│   ├── .empty
│   └── main.yml
├── meta
│   ├── .empty
│   └── main.yml
├── tasks
│   └── main.yml
└── templates
    └── .empty
```

In this role, we define some dependencies in `meta/main.yml`, there's two files in the files/ directory, and there's a set of tasks defined in `tasks/main.yml`.  There's also some handlers defined in `handlers/main.yml`.

 

Let's have a quick glance at the `meta/main.yml` file. 
```yml
---
dependencies:
  - { role: nodejs }
  - { role: nginx }
```

This basically sets the requirement that this role, deploy_thingy depends on services installed by the roles: nginx and nodejs.

Although these roles are explicitly stated to be installed in site.yml, this gives us a level of belt-and-braces configuration, in case the deploy_thingy role were ever included without the other two roles being explicitly stated, or if it were configured to run before its dependencies had explicitly been set to run.

tasks/main.yml is simple. 
```yml
---
 - name: Create directory under /srv for thingy
   file: path=/srv/thingy state=directory mode=755
 - name: Git checkout from github
   git: repo=https://github.com/tomoconnor/shiny-octo-computing-machine.git
        dest=/srv/thingy
 - name: Drop Config for supervisord into the conf.d directory
   copy: src=thingy.super.conf dest=/etc/supervisor/conf.d/thingy.conf
   notify: reread supervisord
 - name: Drop Reverse Proxy Config for Nginx
   copy: src=thingy.nginx.conf dest=/etc/nginx/sites-enabled/thingy.conf
   notify: restart nginx
```

We'll create somewhere for it to live, check the awful code out of my git repository, Then drop two config files in place, one to configure supervisor(d), and one to configure Nginx.

Because the command to configure supervisor(d) and nginx change the configuration of those services, there are notify: handlers to reload the configuration, or restart the service.

 

Let's have a quick peek at those handlers now:
```yml
---
  - name: reread supervisord
    shell: /usr/bin/supervisorctl reread && /usr/bin/supervisorctl update
  - name: restart nginx
    service: name=nginx state=restarted
```

When the supervisor config changes (and we add something to `/etc/supervisor/conf.d`), we need to tell supervisord to re-read it's configuration files, at which point, it will see the new services, and then run supervisorctl update, which will set the state of the newly added items from 'available' to 'started'.

When we change the nginx configuration, we'll hit nginx with a restart.  It's possible to do softer actions, like reload here, but I've chosen service restart for simplicity.

 

I've also changed the basic Ansible config, and configuration of `roles/common/files/insecure_sudoers` so that it will still ask you for a sudo password in light of some minor criticism.

I've found that if you're developing Ansible playbooks on an isolated system, then there's no great harm in disabling SSH Host Key Checking (in ansible.cfg), similarly how there's no great problems in disabling sudo authentication, so it's effectively like NOPASSWD use.  

However, Micheil made a very good point that in live environments it's a bit dodgy to say the least.  So I've commented those lines out of the playbook in Parallax, so that it should give users a reasonable level of basic security.  At the end of the day, it's up to you how you use Parallax, and if you find that disabling security works for you, then fine.  It's not like you haven't been warned. 

But I digress.

The next thing to do is to edit site.yml, and ensure that the new role we've created gets mapped to a hostgroup in the play configuration.

In the latest version of Parallax this is already done for you, but as long as the role name in the list matches the directory in roles/, it should be ready to go.

Now if we run:

`ansible-playbook -k -K -i playbooks/example/hosts playbooks/example/site.yml`
 

It should go through the playbook, installing stuff, then finally do the git clone from github, deploy the configuration files, and trigger a reread of supervisord, and a restart of nginx.

If I now test that it's working, with: 
```
curl -i http://192.168.20.151/
HTTP/1.1 200 OK
Server: nginx/1.4.1 (Ubuntu)
Date: Mon, 27 Jan 2014 14:51:29 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 170
Connection: keep-alive
X-Powered-By: Express
ETag: "1827834703"
```

That **X-Powered-By: Express** line shows that Nginx is indeed working, and that the node.js application is running too. 

You can get more information about stuff that supervisord is controlling by running: 

`sudo supervisorctl status`
on the target host.

```$ sudo supervisorctl status
thingy                           RUNNING    pid 19756, uptime 0:00:06
```

If the Nginx side is configured, but the node.js application isn't running, you'd get a HTTP 502 error, as follows: 
```
curl -i http://192.168.20.151/
HTTP/1.1 502 Bad Gateway
Server: nginx/1.4.1 (Ubuntu)
Date: Mon, 27 Jan 2014 14:59:34 GMT
Content-Type: text/html
Content-Length: 181
Connection: keep-alive
```

So, that's it.  

A very simple guide to deploying a very simple application with Ansible.  Of course, it should be obvious that you can deploy *anything* from a git repository, it really boils down to the configuration of supervisord.  For that matter, it doesn't have to be supervisord.

I consider configuring supervisord for process controlling to be outside of the scope of this article, but I might touch on it in future in more detail. 

Next up, Part 3: Ansible and Amazon Web Services.

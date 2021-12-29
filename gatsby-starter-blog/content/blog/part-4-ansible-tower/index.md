---
title: "Part 4: Ansible Tower"
date: "2014-07-21T19:59:58.000Z"
description: "Part 4 of 5"
warning: ancient
---

## This article suffers from severe linkrot.

You may remember that in January, I wrote a trilogy of blogposts surrounding the use of Ansible, as a handy guide to help y’all get started.  I’ve decided to revisit this now, and write another part, about Ansible Tower.

In the 6-odd months since I wrote Parts 1, 2 and 3 of my Getting Started with Ansible guide, it’s had over 10,000 unique visitors.  I’m quite impressed with that alone.  I’ve built the ansible-based provisioning and deployment pipelines for two more companies, both based off my Parallax starting point I’ve been working on since January.  That alone has been gathering Stars and Forks on Github.

And so, to part four: Ansible Tower.


Ansible Tower is the Web-based User Interface for Ansible, developed by the company behind the Ansible project.

It provides an easy-to-use dashboard, and role-based access control, so that it’s easier to allow individual teams access to use Ansible for their deployments, without having to rely on dedicated build engineers / DevOps teams to do it for them.

There’s also a REST API built into Tower, which aids automation tasks (we’ll come back to this in Part 5).

In this tutorial, I’m going to configure a server running Ansible Tower, and connect it to an Active Directory system.  You can use any LDAP directory, but Active Directory is probably the most  commonly found in Enterprise deployments.

PREREQUISITES:
Ansible Tower server (I’m using a VMware environment, so both my servers are VMs)

> 1 Core, 1GB RAM Ubuntu 12.04 LTS Server, 64-bit

Active Directory Server (I’m using Windows Server 2012 R2)

> 2 Cores, 4GB RAM

Officially, Tower supports CentOS 6, RedHat Enterprise Linux 6, Ubuntu Server 12.04 LTS, and Ubuntu Server 14.04 LTS.

Installing Tower requires Internet connectivity, because it downloads from their repo servers.

I have managed to perform an offline installation, but you have to set up some kind of system to mirror their repositories, and change some settings in the Ansible Installer file.  

I *highly* recommend you dedicate a server (vm or otherwise) to Ansible Tower, because the installer will rewrite pg_hba.conf and supervisord.conf to suit its needs.  Everything is easier if you give it it’s own environment to run in.  

You *might* be able to do it in Docker, although I haven’t tried, and I’m willing to bet you’re asking for trouble.

I’m going to assume you already know about installing Windows Server 2012 and building a domain controller. (If there's significant call for it, I might write a separate blog post about this...)

 

**INSTALLATION STEPS:**
SSH into the Tower Server, and upload the ansible-tower-setup-latest.gz file to your ~directory.

Extract it

Download and open http://releases.ansible.com/ansible-tower/docs/tower_user_guide-latest.pdf in a browser tab for perusal and reference.

Install dependencies:

```sudo apt-get install python-dev python-yaml python-paramiko python-jinja2 python-pip sshpass
sudo pip install ansible
cd ansible-tower-setup-$VERSION
```
(where $VERSION is the version of Ansible it untarred.  Mine’s 1.4.11.)

It should come as no surprise that the Ansible Tower installer is actually an Ansible Playbook (hosts includes 127.0.0.1, and it’s all defined in group_vars/all and site.yml) - Neat, huh? 

Edit group_vars/all to set some sane defaults - basically changing passwords away from what they ship with.

```pg_password: AWsecret
admin_password: password
rabbitmq_password: "AWXbunnies"
```
**Important** - You really need to change these default values, otherwise it’d be theoretically possible that you could expose your secrets to the world!

The documentation says if you’re doing to do LDAP integration, you should configure that now. 

I'm actually going to do LDAP integration at a later stage.

`sudo ./setup.sh`
With any luck, you should get the following message. 

The setup process completed successfully.
 

With Ansible Tower now installed, you can open a web browser, and go to hxxp://whateveryourserveris

You’ll probably get presented with an unsigned certificate error, but we can change that later.

SIDENOTE ON SSL.  
It’s all done via Apache2, so the file you’d want to edit is:

`/etc/apache2/conf.d/awx-httpd-443.conf`
 

and edit:

```
SSLCertificateFile /etc/awx/awx.cert
SSLCertificateKeyFile /etc/awx/awx.key
```
 

You can now log into Tower, with the username: admin, and whatever password you specified in group_vars/all at setup time.

In terms of actually getting started with Ansible Tower, I highly recommend you work your way through the PDF User guide I linked earlier on.  There’s a good example of a quickstart, and it’s really easy to import your standalone playbooks.

When you import a playbook, either manually or with some kind of source control mechanism, it’s important to remember that in the playbook YAML file, you set hosts: all, because the host definition will now be controlled by Tower, so if you forget to do that, you’ll probably find nothing happens when you run a job.

Now for the interesting part…(and let’s face it, it’s the bit you’ve all been waiting for)

**INTEGRATING ANSIBLE TOWER WITH LDAP / ACTIVE DIRECTORY**

Firstly, make sure that you can a) ping the AD server and b) make a LDAP connection to it.

ping is easy.. Just ping it by hostname (if you’ve set up DNS or a hosts file)

LDAP is pretty straight forward too, just telnet into it on port 389.  If you get Connection Refused, you’ll want to check Windows Firewall settings.

On the Tower server, open up:

 /etc/awx/settings.py
After line 80 (or thereabouts) there’s a section on LDAP settings.

Settings you’ll want to change (and some sane examples):

`AUTH_LDAP_SERVER_URI = ''`
set this to the ldap connection string for your server:

`AUTH_LDAP_SERVER_URI = 'ldap://ad0.wibblesplat.com:389'`
On the AD Server, open Users and Computers, and create a user in Managed Service Accounts called something like “Ansible Tower” and assign it a suitably obscure password.  Mark it as “Password never expires”.

We’ll use this user to form the Bind DN for LDAP authentication.

I’ve also created another account in AD->Users, as “Bobby Tables” - with the `sAMAccountName` of `bobby.tables`, and a simple password.  We’ll use this to test that the integration is working later on.

We’ll need the full DN path for the config file, so open Powershell, and run

`dsquery user`
In the list that's returned, look for the LDAP DN of your newly created user:

 `“CN=Ansible Tower,CN=Managed Service Accounts,DC=wibblesplat,DC=com”`
Back in /etc/awx/settings.py, set:

`AUTH_LDAP_BIND_DN = 'CN=Ansible Tower,CN=Managed Service Accounts,DC=wibblesplat,DC=com'`

Password using to bind above user account.
```py
AUTH_LDAP_BIND_PASSWORD = 'P4ssW0Rd%!'
AUTH_LDAP_USER_SEARCH = LDAPSearch(
    ‘CN=Users,DC=wibblesplat,DC=com',   # Base DN
    ldap.SCOPE_SUBTREE,             # SCOPE_BASE, SCOPE_ONELEVEL, SCOPE_SUBTREE
    '(sAMAccountName=%(user)s)',    # Query
)
```

You’ll want to edit the `AUTH_LDAP_USER_SEARCH`  attribute to set your site’s Base DN correctly.  If you store your Users in an OU, you can specify that here.
```py
AUTH_LDAP_GROUP_SEARCH = LDAPSearch(
    'CN=Users,DC=wibblesplat,DC=com',    # Base DN
    ldap.SCOPE_SUBTREE,     # SCOPE_BASE, SCOPE_ONELEVEL, SCOPE_SUBTREE
    '(objectClass=group)',  # Query
)
```
Again, you’ll want to specify your site’s Base DN for Groups here, and again, if you store your groups in an OU, you can specify that.

This is an interesting setting:
```
# Group DN required to login. If specified, user must be a member of this
# group to login via LDAP.  If not set, everyone in LDAP that matches the
# user search defined above will be able to login via AWX.  Only one
# require group is supported.
#AUTH_LDAP_REQUIRE_GROUP = 'CN=ansible-tower-users,CN=Users,DC=wibblesplat,DC=com'
# Group DN denied from login. If specified, user will not be allowed to login
# if a member of this group.  Only one deny group is supported.
#AUTH_LDAP_DENY_GROUP = 'CN=ansible-tower-denied,CN=Users,DC=wibblesplat,DC=com'
```

Basically, you can choose a group, and if the user’s not in that group, they ain’t getting in. 

Both of these are specified as Group DNs:

It’s easy to discover Group DNs with

`dsquery group`
from Powershell on your AD server.

Another clever setting.  It’s possible to give users the Tower “is_superuser” flag, based on AD/LDAP group membership:
```py
AUTH_LDAP_USER_FLAGS_BY_GROUP = {
    'is_superuser': 'CN=Domain Admins,CN=Users,DC=wibblesplat,DC=com',
}
```
Finally, the last setting allows you to map Tower Organisations (Organizations) to AD/LDAP groups:
```py
AUTH_LDAP_ORGANIZATION_MAP = {
    'Test Org': {
        'admins': 'CN=Domain Admins,CN=Users,DC=wibblesplat,DC=com',
        'users': ['CN=ansible-tower-users,CN=Users,DC=wibblesplat,DC=com'],
        'remove_users' : False,
        'remove_admins' : False,
    },
    #'Test Org 2': {
    #    'admins': ['CN=Administrators,CN=Builtin,DC=example,DC=com'],
    #    'users': True,
    #    'remove_users' : False,
    #    'remove_admins' : False,
    #},
}
```
Committing the changes is as easy as restarting Apache, and the AWX Services.

Restart the AWX Services first, with

`supervisorctl restart all`
Now restart Apache, with:

`service apache2 restart`
I created two groups in

`CN=Users,DC=wibblesplat,DC=com`
Called “ansible-tower-denied” and “ansible-tower-users”.

I created two users, “Bobby Tables (bobby.tables)” - in ansible-tower-users, and “Evil Emily (evil.emily)” - in ansible-tower-denied.  

When I restarted Ansible’s services, and tried to log in with bobby.tables, I get in.  

When I view Organizations, I can see Test Org (according to the mapping), and Bobby Tables in that organisation.

When I try to log in as evil.emily, I get “Unable to login with provided credentials.” - Which is what we expect, as this user is in the deny access group.

**USING ANSIBLE TOWER**

As far as how to use Tower is concerned, I don't really want to re-hash what Ansible have already said in their User Manual PDF. 

I will, however walk through the steps to getting Parallax imported, and deployed on a test server.

For this purpose, I've built a Test VM in my development environment, running Ubuntu 14.04.  I'm going to configure Tower to manage this VM, download Parallax playbooks from Github, and create a job template to run them against the test server.

In this example, I'm logged in as the 'admin' superuser account, although with the correct permissions configured within Tower, using Active Directory, or manual permission assignment, it's possible to do this on an individual, and a team level.

**A FEW QUICK DEFINITIONS: **

**Organizations** :- This is the top-level unit of hierarchical organisation in Tower.  An Organization contains Users, Teams, Projects and Inventories.  Multiple Organizations can be used to create multi-tenancy on a Tower server.

**Users** : - These are the logins to Tower.  They're either manually created, or mapped in from LDAP.  Users have Properties (name, email, username, etc..), Credentials (used to connect to services and servers), Permissions (to give them Role-based access control to Inventories and Deployments), Organizations (organizations they're members of), and Teams (useful for subdividing Organizations into groups of users, projects, credentials and permissions).

**Teams** : - A team is a sub-division of an organisation.  Imagine you have a Networks team, who have their own servers.  You might also have a Development team, who need their development environment.  Creating Teams means that Networks manage theirs, and Development manage their own, without knowledge of each others' configurations. 

**Permissions** : - These tie users and teams to inventories and jobs.  You have Inventory permissions, and Deployment permissions. 

Inventory permissions give users and teams the ability to modify inventories, groups and hosts.

Deployment permissions gives users and teams the ability to launch jobs that make changes "Run Jobs", or launch jobs that check state "Check Jobs"

**Credentials** : - These are the passwords and access keys that Tower needs to be able to ssh (or use other protocols) to connect to the nodes it's managing.

There are a few types of Credentials that Tower can manage and utilise:

*SSH Password* - plain old password-based SSH login.

*SSH Private Key* - used for key-based SSH Authentication.

*SSH Private Key w/ Passphrase* - Used to protect the private key with a passphrase.  The passphrase may be optionally stored in the database.  If it's not, Tower will ask you for the password when it needs to use the Credential.

*Sudo Password* - Required if sudo has to run, and needs a password to auth.

*AWS Credentials* - Stores AWS Access Key and Secret Key securely in the Tower Database.

*Rackspace credentials* - Stores Rackspace username and Secret Key. 

*SCM Credentials* - Stores credentials used for accessing source code repositories for the storage and retrieval of Projects.

**Projects** : - These are where your playbooks live.  You can either add them manually, by cloning into 

/var/lib/awx/projects
or by using Git, SVN, or Mercurial and having Tower do the clone automatically before each job run (or on a schedule).

**Inventories** : - These effectively replace the grouping within the Playbook directory hosts file.  You can define groups of hosts, and then configure individual hosts within these groups.  It's possible to assign host-specific variables, or Inventory specific variables from this.

**Groups** : - These live in Inventories, and allow you to collect groups of similar hosts, to which you can apply a playbook.

**Hosts** : - These live in Groups, and define the IP address / Hostname of the node, plus some host variables.

**Job Templates** : - This is basically a definition of an Ansible job, that ties together the Inventory (and its hosts/groups), a Project (and its Playbooks), Credentials, and some extra variables.  You can also specify tags here (like --tags on the ansible-playbook command line).

Job Templates can also accept HTTP Callbacks, which is a way that a newly provisioned host can contact the Tower server, and ask to be provisioned.  We'll come back to this concept in Part 5.

**Jobs** : - These are what happens when a Job Template gets instantiated, and runs a playbook against a set of hosts from the relevant Inventory.

**Running Parallax with Tower**

The first thing we need to do (unless you've already done this / or had one automatically created by LDAP mapping), is to create an Organization. - Again, it's best to refer to the extant Ansible Tower documentation linked above for the best way to do this. 

I've actually mapped my Test Org in via the LDAP interface, so the next step is to create a Team.

I've called my Team "DevOps"

I'm going to assign them a Credential now. 

Navigate to Teams / DevOps

Under "Credentials", click the `[+]`

Select type "Machine"

On a server somewhere, run ssh-keygen, and generate a RSA key.  Copy the private key to the clipboard, and paste it into the SSH Private Key box.  

Scroll down, and click Save.

From the tabbed menu at the top, click Projects and then click the `[+]`

Give the Project a meaningful name and description.  Enter the SCM Type as Git

Under SCM URL, give the public Github address for Parallax, and under SCM Branch set "tower"

Set SCM Update Options to "Update on Launch" - this will do a git update before running a job, so you'll always get the latest version from Git.

Click Save.

This will trigger a job, which will clone the latest version from Git, and save it into the Projects directory.  If this fails, you might need to run: 

chown -R awx /var/lib/awx/projects


Next, create an Inventory.

Pretty straightforward - name, description, organisation.

Select that Inventory, and create a Group - It's possible to import Groups from EC2, by selecting the Source tab when you create a new group.

Select that group you just created, and create a host under it, with the IP Address / hostname of your test server.

At this point, you can assign per-host variables.

Nearly there!

Click "Job Templates", and create a new job template.  As I said before, these really tie it all together.

Give it a name, then select your Inventory, Project, Playbook and Credential.

Click Save.

To launch it, click the Rocketship from the Job Templates Listing.

You'll get redirected to the Jobs page, showing your latest job in Queued.  

Unless you have a very busy Tower server, it won't stay Queued for long.  Click the refresh button on the Queued section to reload, and you should see it's moved to Active.

You can click on the job for an update on its status, or just patiently wait for it to complete.

When the job's done, you'll either have a red dot, or a green dot indicating the status of the job.

That's it.  You've installed Ansible Tower, integrated it with Active Directory, and created your first deployment job of Parallax with Tower.


OTHER RESOURCES: 

Coming Soon: Part 5. Automation with Ansible.
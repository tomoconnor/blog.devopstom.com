---
title: "Postgres Replication on 9.1"
date: "2011-12-20T17:00:00.000Z"
description: "postgresql magic runes"
warning: ancient
---

Our new PowerDNS cluster (of 2 nodes, so far).. Is backed by Postgresql.  

In the past, I’ve found that Postgres performs far better for a PowerDNS backend, than MySQL, and certainly better than the BIND, LDAP or SQLite backends.

Until version 9.x, Postgres replication was a pretty sorry state of affairs.  There were a few options for replication. 

Slony was commonly used, if not very good.. You’d tend to get a horrific SPoF around the single master.  In total, there were 9 or 10 different third party solutions for Postgres replication and clustering.  They all had their pros and cons, and some were great, and some were downright awful.  

In 2008, the Postgres core team started to bring replication and clustering into the fold with the rest of the features of Postgres, and now, in 9.x, the option of hot and warm standby are both available, and stable.

There’s a comprehensive writeup of the history of Postgres replication here: http://wiki.postgresql.org/wiki/Replication,_Clustering,_and_Connection_Pooling

One of the things I adore about the hot-standby replication mode is that the basic configuration (/etc/postgresql/9.1/main/postgresql.conf) is identical between master and standby.

This makes puppeting insanely easier than it would’ve been if the master and standby had to have largely different configuration files.

I changed about 5 config settings in the main config file.

```
listen_addresses = '*'
wal_level = hot_standby 
max_wal_senders = 5
wal_keep_segments = 32
log_destination = 'syslog'
```

^^ I only changed the log_destination to make centralised logging easier in future.

There’s a limited change to pg_hba.conf to allow host-based authentication of the standby to the master.  

Add a line like:

`host    all     all     $SLAVE_IP/32      trust`
I actually did this as a Puppet template file.

On the standby server, you drop a file called “recovery.conf”, into `/var/lib/postgresql/9.1/main`

Yes, the **DATA** directory.  Yes, that makes no sense. Yes, It should by rights be `/etc/postgres`.... but it isn’t.

In that file, you have 2 lines.
```
standby_mode = 'on'
primary_conninfo = 'host=<%= psql_master -%> user=<%=replication_user -%> password=<%=replication_password -%> '
```

That’s copypasta’d from a puppet template.  

The interpolated lines are more like:

`standby_mode = 'on'`
`primary_conninfo = 'host=192.168.100.100 user=replicant password=wibblewibblewibble '`

Then all you’ve gotta do is instantiate the standby with pg_basebackup, and then restart the master, and the standby, and they should come up, connect to each other, and start streaming replication updates.

It’s pretty magical.

`pgbasebackup` lives in `/usr/lib/postgresql/9.1/bin/pg_basebackup`

and should be run (as postgres user)

`/usr/lib/postgresql/9.1/bin/pg_basebackup -D /var/lib/postgresql/9.1/main/ -x -h $Master_Hostname -U postgres`

You should start the standby first, so that the master doesn’t have a chance to get out of sync.

The standby will start accepting read-only connections as soon as it’s up to date with the master.
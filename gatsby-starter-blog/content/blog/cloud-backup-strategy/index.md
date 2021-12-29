---
title: Cloud Backup Strategy
date: "2011-05-30T17:00:00.000Z"
description: "Backups are hard."
warning: ancient
---
It has recently been brought to my attention that a number of users of cloud-based hosting services tend to use an "integrated" backup solution provided by the cloud host.  This is probably some form of snapshot-based backup of a server's state. 

I quite like the idea of doing this, especially if there's no impact to the server being backed up whilst the snapshot is taken. 

However, I can immediately see one big problem with it.  

At least one scenario I can see that would require me to restore a backup is failure of the server host.  Under this circumstance, it might be possible that a) you will be unable to get hold of the backup, which is probably stored somewhere on their storage cloud. or b) You can get access to the storage, but the backup is a proprietary format, either a raw snapshot, or a VMDK disk image which might be difficult/impossible to transfer to a different host.  

I'd be especially scared of using snapshot-backups for a database server, because in the unlikely event that the restore target is different to the backed up server, you might have some compatability problems, especially if you're using x86 MySQL and go to a x86-64 host.  

For this reason, I think it's probably best to have a couple of different backup strategies.  

I suggest having a snapshot backup is a good thing, and will allow a very fast restore process, but is only useful while your server's host is online.  

In the event that your host has gone down, it's important to have an offline/offsite backup.  This alternative backup should also be as platform agnostic as possible.

In other words, any databases should be exported as SQL files, and as far as possible, the system state should be backed up.  I tend to keep track of what packages have been installed by storing `dpkg --get-selections > /var/lib/backup/dpkg-state` or similar.  This means that if I have to rebuild a server, i can just use that file, and restore the state of package installations really quickly and easily.  

That, and a copy of `/etc`, and restoration should be pretty easy.

On the other hand, the concept of trying to restore a VMware, KVM or Xen snapshot (which might be inaccessible, or otherwise unavailable for export/download) onto a different system entirely, frankly fills me with a little bit of fear.

Given the choice, a snapshot restore is almost certainly preferable, but it'd be prudent to have a backup strategy for your backup strategy. ;)
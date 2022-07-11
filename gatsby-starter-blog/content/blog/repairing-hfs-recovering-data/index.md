---
title: Repairing HFS+, Recovering Data
date: "2022-07-11T22:10:01.123Z"
description: "The process I went through to recover data off a dead macbook"
---

So my downstairs neighbour's macbook died the other day. 

I wasn't surprised it died, it was a 2010 MBP.  Considering it lasted 12 years before suffering a terminal logic board failure, I'll write up as "pretty good going". 

Mind you, I did replace the CPU fan a couple of years ago, and swapped the original HDD for a samsung SSD  about 4 years ago.  

Funnily enough, neither of those parts were what failed this time.  This time the GPU just up and quit, and instead of buying a replacement logic board, my neighbour bought a 2nd hand 2017 MBP. 

Which meant that the only thing I had to do was get her data off the old SSD and load it up onto the new one.

I had hoped that this would be as simple as plugging it in with a dock, and copying the data off within Finder. 

Unfortunately, something in the process of the old MBP dying (and being rebooted uncleanly, many many times) had effectively made the old SSD unreadable. 

But HFS+ is a Journaled File System, I hear you cry.  So how do we recover the journal, and recover the data? 

```
[4230540.946280] usb-storage 2-1.1:1.0: USB Mass Storage device detected
[4230540.946621] usb-storage 2-1.1:1.0: Quirks match for vid 2537 pid 1068: 800000
[4230540.946689] scsi host8: usb-storage 2-1.1:1.0
[4230545.248786] scsi 8:0:0:0: Direct-Access     XDISK    X9               BB6Q PQ: 0 ANSI: 6
[4230545.249591] sd 8:0:0:0: Attached scsi generic sg4 type 0
[4230545.250407] sd 8:0:0:0: [sde] 488397168 512-byte logical blocks: (250 GB/233 GiB)
[4230545.251506] sd 8:0:0:0: [sde] Write Protect is off
[4230545.251510] sd 8:0:0:0: [sde] Mode Sense: 43 00 00 00
[4230545.252616] sd 8:0:0:0: [sde] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
[4230545.335804] sd 8:0:0:0: [sde] Attached SCSI disk
```

Dmesg sees it as /dev/sde, but for some reason, neither fdisk or gparted can see any partition data.  

**First step.** Make a bit-wise copy, so we're working on the copy for the recovery operations. 
Not only is this faster, because it doesn't have to go thru the USB dock, but if we fuck it up, we've only fucked up a copy. 

`dd if=/dev/sde of=/recovery/recovery.img`

This takes a while.  I went off to have dinner in the meantime.  When I came back I had a 250GB disk image file. 

This first step is **REALLY** important in any kind of data recovery / forensic examination process.  The last thing you want to do is compromise any data you may have remaining by running the wrong command in the wrong shell window. 

Next, I installed `hfsprogs` which contains the `mount.hfsplus` and `fsck.hfsplus` utilities. 

I tried straight up `fdisk /recovery/recovery.img` which saw that it was a 250G 'volume' but didn't see any partitions within. 
I tried `gparted`, which also saw the correct size, but failed to identify any volumes. 

I then wanted to check that the data was *actually* within the disk image, so ran `binwalk` on it, which identified a bunch of EFI filesystem space, and then a load of other files - This was a good sign, worst case scenario, I could have used `binwalk -e` and extracted all the things that binwalk could find.  

However, working with a filesystem is always easier when you can interact with it as a filesystem, rather than a bucket of file. 

I tried `kpartx` and `losetup --partscan --find --show recovery.img`, but got some interesting errors: 

```
Warning: Disk has a valid GPT signature but invalid PMBR.
  Assuming this disk is *not* a GPT disk anymore.
  Use gpt kernel option to override.  Use GNU Parted to correct disk.
```

I've already tried `gparted` / GNU Parted -- but this gave me a hint about what might be going on.  What if the disk has a GPT signature, but has lost the rest of the MBR.  We know the data's on there, so all we need now is to repair the GPT / MBR, and have another shot with `kpartx` or `losetup`. 

Enter `gdisk` - it's like `fdisk` but for GPT. 

Documentation I followed loosely: https://www.rodsbooks.com/gdisk/repairing.html

```
sudo gdisk recovery.img
GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with corrupt MBR; using GPT and will write new
protective MBR on save.

Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to recovery.img.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.
```

I didn't actually do anything other than just write back the GPT. 

But now, `fdisk` was able to list the partitions!

```
sudo fdisk -l recovery
Disk recovery: 232.91 GiB, 250059350016 bytes, 488397168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9C882443-6F2B-497E-8F1A-EF375C4241DB

Device     Start       End   Sectors   Size Type
recovery1          40    409639    409600   200M EFI System
recovery2      409640 487127591 486717952 232.1G Apple Core storage
recovery3   487127592 488397127   1269536 619.9M Apple boot
```

Which means we can mount them!

```
sudo losetup --partscan --find --show recovery.img
/dev/loop46
```

`losetup` basically creates a loopback device to the image file, and makes mounting partitions significantly easier. 

Now running fdisk against `/dev/loop46` shows mountable partitions. 

```
(base) user@sugarloaf:/media/user/music/recovery$ sudo fdisk /dev/loop46

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/loop46: 232.91 GiB, 250059350016 bytes, 488397168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9C882443-6F2B-497E-8F1A-EF375C4241DB

Device            Start       End   Sectors   Size Type
/dev/loop46p1        40    409639    409600   200M EFI System
/dev/loop46p2    409640 487127591 486717952 232.1G Apple Core storage
/dev/loop46p3 487127592 488397127   1269536 619.9M Apple boot

Command (m for help): q
```

Except... 

```
(base) user@sugarloaf:/media/user/music/recovery$ sudo mount /dev/loop46p2 /mnt/hfs/
mount: /mnt/hfs: wrong fs type, bad option, bad superblock on /dev/loop46p2, missing codepage or helper program, or other error.
```
Meanwhile, in `dmesg`

```
[4242985.734827] hfsplus: invalid secondary volume header
[4242985.734829] hfsplus: unable to find HFS+ superblock
```

So there's still something not quite right.  But a while ago, we installed hfsprogs, which has `fsck.hfsplus`

```
sudo fsck.hfsplus /dev/loop46p2 
** /dev/loop46p2
** Checking HFS Plus volume.
fsck_hfs: Volume is journaled.  No checking performed.
fsck_hfs: Use the -f option to force checking.

(base) user@sugarloaf:/media/user/music/recovery$ sudo fsck.hfsplus -f /dev/loop46p2 
** /dev/loop46p2
** Checking HFS Plus volume.
** Checking Extents Overflow file.
** Checking Catalog file.
** Checking multi-linked files.
   Orphaned indirect node iNode12107050
** Checking Catalog hierarchy.
   Invalid directory item count
   (It should be 108391 instead of 108392)
   Invalid directory item count
   (It should be 37 instead of 36)
** Checking Extended Attributes file.
** Checking volume bitmap.
   Volume Bit Map needs minor repair
** Checking volume information.
   Volume Header needs minor repair
** Repairing volume.
** Rechecking volume.
** Checking HFS Plus volume.
** Checking Extents Overflow file.
** Checking Catalog file.
** Checking multi-linked files.
** Checking Catalog hierarchy.
** Checking Extended Attributes file.
** Checking volume bitmap.
** Checking volume information.
   Invalid volume file count
   (It should be 1375422 instead of 1375423)
   Invalid volume free block count
   (It should be 3815940 instead of 2279930)
** Repairing volume.
free(): invalid pointer
Aborted
```
I don't know why it aborted. Maybe it had just finished.


Once more for luck. 
```
(base) user@sugarloaf:/media/user/music/recovery$ sudo fsck.hfsplus -f /dev/loop46p2 
** /dev/loop46p2
** Checking HFS Plus volume.
** Checking Extents Overflow file.
** Checking Catalog file.
** Checking multi-linked files.
** Checking Catalog hierarchy.
** Checking Extended Attributes file.
** Checking volume bitmap.
** Checking volume information.
** The volume sshdd appears to be OK.
```

So let's have another shot at mounting it. 
```
(base) user@sugarloaf:/media/user/music/recovery$ sudo mount /dev/loop46p2 /mnt/hfs
```

Look ma! No errors!

```
(base) user@sugarloaf:/media/user/music/recovery$ cd /mnt/hfs/
(base) user@sugarloaf:/mnt/hfs$ ls
 Applications   cores          dev   home                     installer.failurerequests   lost+found   Network   sbin     tmp     usr   Volumes
 bin            DamagedFiles   etc  'Incompatible Software'   Library                     net          private   System   Users   var
```
That looks exceptionally promising. 

So all I did next was enable ssh on the new macbook (Sharing -> Remote Login). 
Dropped root@sugarloaf's SSH Public Key in `~/.ssh/authorized_keys`, and rsynced data across from the mounted hfs loopback device over to `~/Recovered/` on the new macbook pro!

All done!
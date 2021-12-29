---
title: "Hacking initrd.gz on Ubuntu's Netboot Installer"
date: "2012-03-01T10:00:00.000Z"
description: "Life on the bleeding edge"
warning: ancient
---

This morning, I did something unquestionably naughty, and totally got away with it.

A little background.  We just had some *brand* new workstations delivered.. They turned up yesterday afternoon.  They're high performance 3d workstations with an Intel DX79TO mainboard.  This mainboard has the intel 87529 Gigabit Ethernet controller.  I wouldn't normally pay so much attention to the controllers and so on that are actually on a board, but in future, I will.

This controller is not supported by the `e1000` driver that comes in the PXE installer on the netboot CD for Ubuntu 10.04.  

We plugged them in, fired them up, and watched the PXE installer fail as it couldn't find a supported kernel module for that hardware.  Bugger.

Our primary plan was to buy some cheap 1GE NICs, install with those, update the driver then carry on.

My personal plan was to update the initramfs of the PXE installer, give it a newer `e1000` kernel module, and *pray* that it works.

Things you need.

1. e1000 [source from intel](https://www.intel.com/content/www/us/en/download-center/home.html?agr=Y&DwnldID=15817&ProdId=3299&lang=eng&OSVersion=Linux*&DownloadType=Drivers).
2. The initrd.gz and linux files from the pxe installer.
3. The linux-headers for the kernel of the pxe boot installer

So go ahead and grab those initrd.gz and 'linux' files, then run `file` on 'linux' and grab the kernel version. Now you can download the headers for that version.

If you have a look inside the Makefile for the e1000e drivers, there's this block

```ifeq (,$(BUILD_KERNEL))
BUILD_KERNEL=$(shell uname -r)
endif
```

which I read, and thought "Ha! I can provide `BUILD_KERNEL` as an environment variable and build against that instead."

`BUILD_KERNEL=2.6.32-21-generic make`

If you run make install, then it'll insert it into your own kernel.  If you just run make, you can look in the `src` directory, and find the .ko file we need.

Then comes the harder part.  

Make some directories like 

`kernelhacking/{initrd,e1000e}`

then copy the `initrd.gz` into `initrd/`

and run

`zcat initrd.gz | (while true; do cpio -i -d -H newc --no-absolute-filenames || exit; done)`

then `mv` the `initrd.gz` up a level.

Now you've got the rootfs that the installer uses.

```
tom.oconnor@charcoal-black:~/kernelhacking/initrdhacking$ ls -lah
total 544K
drwxrwxr-x 15 tom.oconnor dialout 2.0K 2012-03-01 11:35 .
drwxrwxr-x  5 tom.oconnor dialout 2.0K 2012-03-01 13:40 ..
drwxr-xr-x  2 root        root    8.0K 2012-03-01 11:35 bin
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 dev
drwxr-xr-x 13 root        root    4.0K 2012-03-01 11:50 etc
-rwxr-xr-x  1 root        root     376 2012-03-01 11:35 init
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 initrd
drwxr-xr-x 12 root        root    4.0K 2012-03-01 11:35 lib
lrwxrwxrwx  1 root        root       4 2012-03-01 11:35 lib64 -> /lib
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 media
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 mnt
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 proc
drwxr-xr-x  2 root        root    4.0K 2012-03-01 11:35 sbin
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 sys
drwxr-xr-x  2 root        root    2.0K 2012-03-01 11:35 tmp
drwxr-xr-x  6 root        root    2.0K 2012-03-01 11:35 usr
drwxr-xr-x  8 root        root    2.0K 2012-03-01 11:35 var
tom.oconnor@charcoal-black:~/kernelhacking/initrdhacking$ 
``` 

The driver we want to replace is the e1000e.ko, deep within

`lib/modules//kernel/drivers/net/e1000e`

I'm going to switch to root now, to save on sudo keystrokes:
```
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/lib/modules/2.6.32-21-generic/kernel/drivers/net/e1000e# ls
e1000e.ko
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/lib/modules/2.6.32-21-generic/kernel/drivers/net/e1000e# mv e1000e.ko /home/tom.oconnor/kernelhacking/old-e1000e.ko
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/lib/modules/2.6.32-21-generic/kernel/drivers/net/e1000e# cp /home/tom.oconnor/kernelhacking/src/e1000e-1.9.5/src/e1000e.ko .
```

I also found a `pci.ids` update file in the e1000 package.  Let's track that down, or at least figure out where it gets installed to.  

```
tom.oconnor@charcoal-black:~$ locate pci.ids
/usr/share/misc/pci.ids
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking# cd usr/share/misc/
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# ls
pci.ids.gz
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# zless pci.ids.gz 
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# gunzip pci.ids.gz 
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# ls
pci.ids
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# vim pci.ids 
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# ls -lah
total 448K
drwxr-xr-x  2 root root 2.0K 2012-03-01 11:42 .
drwxr-xr-x 10 root root 2.0K 2012-03-01 11:35 ..
-rw-r--r--  1 root root 358K 2012-03-01 11:35 pci.ids
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# pwd
/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# ls
pci.ids
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# rm pci.ids 
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# wget http://pciids.sourceforge.net/v2.2/pci.ids.gz
--2012-03-01 10:49:50--  http://pciids.sourceforge.net/v2.2/pci.ids.gz
Resolving pciids.sourceforge.net... 216.34.181.96
Connecting to pciids.sourceforge.net|216.34.181.96|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 190157 (186K) [application/x-gzip]
Saving to: `pci.ids.gz'
100%[=======================================>] 190,157      242K/s   in 0.8s    
2012-03-01 10:49:51 (242 KB/s) - `pci.ids.gz' saved [190157/190157]
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking/usr/share/misc# ls -lah
total 256K
drwxr-xr-x  2 root root 2.0K 2012-03-01 11:49 .
drwxr-xr-x 10 root root 2.0K 2012-03-01 11:35 ..
-rw-rw-r--  1 root root 186K 2012-02-27 02:15 pci.ids.gz
```

I couldn't be arsed patching the existing pci.ids, so I thought it would be just as good to replace it with the latest one from http://pciids.sourceforge.net/

Turns out, it is!

The next thing to do is to rebuild the initramfs back into a cpio.gz file.

I grabbed most of my initrd fu from here. It's a little out of date, perhaps, but seemed to work for me.

Let's say we've got this
```
root@charcoal-black:/home/tom.oconnor/kernelhacking/initrdhacking# ls
bin  dev  etc  init  initrd  lib  lib64  media  mnt  proc  sbin  sys  tmp  usr  var
```
First things first

`$ touch initrdhacking/etc/mdev.conf`
That's actually the only bit I had to do, and I only did it for compatibility reasons.  The rest is all there, as we took it from a working initrd.gz file.

The compress bit was the bit i didn't quite know about.

```cd initrdhacking
find . | cpio -H newc -o > ../initrd.cpio
cd ..
gzip initrd.cpio
```
Then we'll copy the new initrd.gz in place of the old one on the tftp server.  You should always make a backup of the old one, in case this all goes horribly wrong, etc.
```
cd /tftpboot/ubuntu-1004-installer/amd64/
mv initrd.gz old.initrd.gz
cp /home/tom.oconnor/kernelhacking/initramfs.cpio.gz initrd.gz
```
Done.

I went over to the new workstations, and kicked them into a PXE netboot/preseed session.

:D.  They're installing.  Best feeling ever.

I stuck my hands in an initramfs and fiddled with it's private parts.

Epic Win.

See Part Two for the continuing saga of the e1000e kernel module.
---
title: "Building a DKMS package of the latest Intel e1000e driver"
date: "2012-03-01T20:00:00.000Z"
description: "Life on the bleeding edge Part 2"
warning: ancient
---

This is a continuation to my [earlier blogpost about hacking initrd.gz](/hacking-initrdgz-ubuntu-netboot-installer).

After it's installed with the modified installer kernel, and finished building, the installed kernel doesn't have the e1000e driver.  This is because the netboot installer pulls in a kernel and it's wherewithall from apt.  It also gets a new initrd.

As a result, I've decided to build an e1000e-dkms module, and we'll specify the preseed installer to install that, along with linux-headers-generic, linux-headers-2.6.32 and build-essential.

Building DKMS modules is fiddly, and requires some work and testing to make it work

There's a few ways to build DKMS modules, according to the Ubuntu Wiki on the topic. I've found another way to do it, albeit a slightly sneaky, round-about way, but it is a lot quicker than rolling one from scratch.

**Here's what I did.**

Knowing that we use the Wacom kernel drivers elsewhere in the organisation, and they're installed by DKMS from source too, I started off by grabbing the wacom-dkms package from the PPA. 

Deb files are really quite a simple archive format, that you can extract with

`ar xv $PACKAGE_NAME`

```
tom.oconnor@charcoal-black:~$ mkdir dkmsroll  
tom.oconnor@charcoal-black:~$ cd dkmsroll/
tom.oconnor@charcoal-black:~/dkmsroll$ ls
tom.oconnor@charcoal-black:~/dkmsroll$ wget http://ppa.launchpad.net/doctormo/wacom-plus/ubuntu/pool/main/w/wacom-source/wacom-dkms_0.8.8-0ubuntu4_all.deb--2012-03-01 14:57:43--  http://ppa.launchpad.net/doctormo/wacom-plus/ubuntu/pool/main/w/wacom-source/wacom-dkms_0.8.8-0ubuntu4_all.deb
Resolving ppa.launchpad.net... 91.189.90.217
Connecting to ppa.launchpad.net|91.189.90.217|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 624404 (610K) [application/x-debian-package]
Saving to: `wacom-dkms_0.8.8-0ubuntu4_all.deb'
100%[===================================================================================================================================================================================================>] 624,404      428K/s   in 1.4s    
2012-03-01 14:57:44 (428 KB/s) - `wacom-dkms_0.8.8-0ubuntu4_all.deb' saved [624404/624404]
tom.oconnor@charcoal-black:~/dkmsroll$ ar xv wacom-dkms_0.8.8-0ubuntu4_all.deb 
x - debian-binary
x - control.tar.gz
x - data.tar.gz
```

`debian-binary` is a file that states the version of the dpkg file.  You can ignore this safely for this exercise.

`control.tar.gz` is the archive that contains the metadata for the package.  Let's extract that. 


```
tom.oconnor@charcoal-black:~/dkmsroll$ tar zxvf control.tar.gz 
./
./prerm
./postinst
./md5sums
./control
``` 

We're going to re-roll the package later on, slightly differently, but we'll need to keep the prerm and postinst files.  These are plain old shellscript files with special names that will get run by dpkg in the installation phase.

 

There's 4 you really tend to find, `prerm`, `preinst`, `postrm` and `postinst`.

`control` contains the important metadata, like `Depends:` and `Description:` and `Package:`, which is how apt indexes packages, and dpkg knows what to install in what order, and so on.

The control file above looks like this:
```
Package: wacom-dkms
Source: wacom-source
Version: 0.8.8-0ubuntu4
Architecture: all
Maintainer: Taylor LeMasurier-Wren <ripps818@gmail.com>
Installed-Size: 2964
Depends: dkms (>= 1.95)
Section: misc
Priority: optional
Description: Wacom kernel driver in DKMS format
```

There's one thing left, and that's data.tar.gz.  This is where the package data is actually stored, and when you uncompress it, you get a bunch of directories that when extracted by dpkg, will be reproduced in /

```
tom.oconnor@charcoal-black:~/dkmsroll$ tar xzvf data.tar.gz 
./
./usr/
./usr/src/
./usr/src/wacom-0.8.8/
./usr/src/wacom-0.8.8/dkms.conf
./usr/src/wacom-0.8.8/GPL
./usr/src/wacom-0.8.8/config.guess
./usr/src/wacom-0.8.8/AUTHORS
./usr/src/wacom-0.8.8/config.guess.cdbs-orig
```

... And so on.

What we've now got is the following:
```
tom.oconnor@charcoal-black:~/dkmsroll$ ls 
control  control.tar.gz  data.tar.gz  debian-binary  md5sums  postinst  prerm  usr/  wacom-dkms_0.8.8-0ubuntu4_all.deb
```
Let's have a look at that usr/ directory (which when installed, becomes /usr/):

We only really care about the directory structure here.. 
```
tom.oconnor@charcoal-black:~/dkmsroll$ tree -d usr/
usr/
|-- share
|   |-- doc
|   |   `-- wacom-dkms
|   `-- man
|       `-- man8
`-- src
    `-- wacom-0.8.8
        `-- src
            |-- 2.6.16
            |-- 2.6.18
            |-- 2.6.24
            |-- 2.6.30
            |-- include
            |-- util
            |-- wacomxi
            `-- xdrv
16 directories
```

So what we will need to do, to create an e1000e DKMS package, is create a tree structure that looks a bit like this.

So let's create a directory to contain all this package building nonsense.  
```
tom.oconnor@charcoal-black:~/dkmsroll$ mkdir e1000e-dkms
tom.oconnor@charcoal-black:~/dkmsroll$ cd e1000e-dkms
tom.oconnor@charcoal-black:~/dkmsroll/e1000e-dkms$ 
```
Now we're here, let's create 3 directories, 'DOWNLOAD', 'info', and 'src'.

'DOWNLOAD' is where we'll leave the source package file that we downloaded from Intel, and any other associated downloaded stuff.
```
tom.oconnor@charcoal-black:~/dkmsroll/e1000e-dkms/DOWNLOAD$ ls
e1000e-1.9.5  e1000e-1.9.5.tar.gz
```
You can go ahead and extract the downloaded tarball.  We need to pick and choose bits out of it, and generally have a good ol' reorganise.

'info' is a directory of my own creation.  It's where I tend to stash the metadata files, some of which will be built into the 'control' file, and some are (post|pre)(inst|rm) scripts.. 

Best thing to do, copy the postinst/prerm files from the extracted wacom DKMS control files into info/, and edit them in your favourite text editor.

You're looking for anything in the file that says "wacom", basically. 

In the postinst, in this case, it looks like this.
```
NAME=wacom
PACKAGE_NAME=$NAME-dkms
CVERSION=`dpkg-query -W -f='${Version}' $PACKAGE_NAME | awk -F "-" '{print $1}' | cut -d\: -f2`
ARCH=`dpkg --print-architecture`
```

All you need to do is change wacom to e1000e wherever it's mentioned, and update the version variable in the prerm file.  Just go through the files and be sensible about where stuff needs changing.

Next thing we need to do is set up where the package will be installed to.

Earlier on, you made a 'src' directory.  As I said before, this will effectively be dropped into / by dpkg, so `usr/` becomes `/usr/` and so on.  Within reason, you can drop anything into the right path here, and dpkg will deploy it.  Dropping stuff into `/sys`, `/proc`, or `/dev` might land you in hot water.

I've set up the directory structure below usr/ to be very similar to the wacom one, except without usr/share/* (for simplicity).
```
tom.oconnor@charcoal-black:~/dkmsroll/e1000e-dkms/src$ tree -a
.
`-- usr
    `-- src
        `-- e1000e-1.9.5
            |-- dkms.conf
            `-- src
                |-- 80003es2lan.c
                |-- 80003es2lan.h
                |-- 82571.c
                |-- 82571.h
                |-- defines.h
                |-- e1000.h
                |-- ethtool.c
                |-- hw.h
                |-- ich8lan.c
                |-- ich8lan.h
                |-- kcompat.c
                |-- kcompat_ethtool.c
                |-- kcompat.h
                |-- mac.c
                |-- mac.h
                |-- Makefile
                |-- manage.c
                |-- manage.h
                |-- Module.supported
                |-- netdev.c
                |-- nvm.c
                |-- nvm.h
                |-- param.c
                |-- phy.c
                |-- phy.h
                `-- regs.h
4 directories, 27 files
```

The innermost src/ directory (actually `/usr/src/e1000e-1.9.5/src`) contains the contents of the src/ directory from the `e1000e-1.9.5.tar.gz` file we downloaded from Intel.

Above that, we've got a `dkms.conf` file, which is basically like a pre-make file which controls the DKMS builder.

We need to write that `dkms.conf` file, and specify how the DKMS builder should actually run make.

Here's the contents of the `dkms.conf` file I used to match the source tree above.
```
MAKE="cd src/ && BUILD_KERNEL=${kernelver} make"
CLEAN="cd src/ && make clean"
BUILT_MODULE_NAME="e1000e"
BUILT_MODULE_LOCATION="src/"
DEST_MODULE_LOCATION="/kernel/../updates/"
PACKAGE_NAME="e1000e"
PACKAGE_VERSION="1.9.5"
REMAKE_INITRD="yes"
AUTOINSTALL="yes"
```

`$kernelver `is a variable provided for use inside dkms.conf files. It's basically the contents of `uname -r`.

We can pass `BUILD_KERNEL` as an environment variable to make to allow us to specify the location of the linux headers for the kernel we're using, or to specify an alternate kernel to build against.

`DEST_MODULE_LOCATION="/kernel/../updates/"`

is where DKMS will shove the created .ko file, and `BUILD_MODULE_NAME` is the name of the kernel module, without the .ko extension.  Simple really (at least in this case).

We're ready to build the deb file now.  We've got the metadata in order, we've got the contents of the package in place, where DKMS is expecting it to be, and we've got the dkms.conf file written.

So change back to the root of the package build environment (contains the DOWNLOAD and info, and src directories).

You'll need jordansissel's `fpm` package builder for this next bit.
```
fpm -n "e1000e-dkms" \
    -v "1.9.5" \
    -a "all" \
    -s dir \
    -C src \
    -t deb \
    --pre-uninstall info/e1000e-dkms.prerm \
    --post-install info/e1000e-dkms.postinst \
    --url "http://example.org/techblog" \
    --description "DKMS Intel e1000e driver" \
    --iteration "custombuild-r1" \
    --depends "dkms (>= 1.95)" \
    --replaces "e1000e-dkms (<< 1.9.5)"
Created /home/tom.oconnor/dkmsroll/e1000e-dkms/e1000e-dkms_1.9.5-custombuild-r1_all.deb
```

Ta-da!  You've now got a dkms package, which if you like, you can inspect the contents of, just as we did before.

Here's the contents of the control file, for example:
```
Package: e1000e-dkms
Version: 1.9.5-custombuild-r1
License: unknown
Vendor: none
Architecture: all
Maintainer: <tom.oconnor@charcoal-black>
Depends: dkms (>= 1.95)
Replaces: e1000e-dkms (<< 1.9.5)
Standards-Version: 3.9.1
Section: default
Priority: extra
Homepage: http://example.org/techblog
Description: DKMS Intel e1000e driver
```

Clever, huh? And not a single bit of dh_make in sight.  I love fpm for this exact reason.

We'll just test that installation process.  Mine looks slightly different here because I've previously installed a few of these packages, but your should look something like this:
```
tom.oconnor@charcoal-black:~/dkmsroll/e1000e-dkms$ sudo dpkg -i e1000e-dkms_1.9.5-custombuild-r1_all.deb
[sudo] password for tom.oconnor: 
(Reading database ... 385051 files and directories currently installed.)
Preparing to replace e1000e-dkms 1.9.5-baseblack-r6 (using e1000e-dkms_1.9.5-custombuild-r1_all.deb) ...
-------- Uninstall Beginning --------
Module:  e1000e
Version: 1.9.5
Kernel:  2.6.32-38-generic (x86_64)
-------------------------------------
Status: Before uninstall, this module version was ACTIVE on this kernel.
e1000e.ko:
 - Uninstallation
   - Deleting from: /lib/modules/2.6.32-38-generic/updates/dkms/
 - Original module
   - No original module was found for this module on this kernel.
   - Use the dkms install command to reinstall any previous module version.
depmod....
Updating initrd
Making new initrd as /boot/initrd.img-2.6.32-38-generic
(If next boot fails, revert to the .bak initrd image)
update-initramfs....
DKMS: uninstall Completed.
------------------------------
Deleting module version: 1.9.5
completely from the DKMS tree.
------------------------------
Done.
Unpacking replacement e1000e-dkms ...
Setting up e1000e-dkms (1.9.5-custombuild-r1) ...
Loading new e1000e-1.9.5 DKMS files...
First Installation: checking all kernels...
Building only for 2.6.32-38-generic
Building for architecture x86_64
Building initial module for 2.6.32-38-generic
Done.
e1000e.ko:
Running module version sanity check.
 - Original module
 - Installation
   - Installing to /lib/modules/2.6.32-38-generic/updates/dkms/
depmod....
Updating initrd
Making new initrd as /boot/initrd.img-2.6.32-38-generic
(If next boot fails, revert to the .bak initrd image)
update-initramfs....
DKMS: install Completed.
Processing triggers for initramfs-tools ...
update-initramfs: Generating /boot/initrd.img-2.6.32-38-generic
```

So let's recap.  We took an existing dkms package from a PPA, took it apart, figured out how it's been put together.  
Made our own directory structure to look like that, modified the postinst and prerm files to fit our DKMS module instead, then dropped the source directory into the source tree, and built the new debian package with fpm.

In order to make this installable at system build time, I simply published the deb to our internal apt repository, then in our preseed file, we've got a pkgsel\include line that now looks like this.

`d-i pkgsel/include string puppet puppet-common facter ssh zsh curl dkms linux-headers-generic build-essential linux-headers-2.6.32-38 e1000e-dkms
2.6.32-38` is the default lucid kernel that has been installed initially, so we can specify that explicitly here.  

When the preseeder runs, it installs `dkms`, `linux-headers` and `build-essential`, then grabs the `e1000e-dkms` package and installs that too.

The DKMS builder triggers a rebuild of the initramfs that lives in `/boot`, so that next time we boot, the kernel loads the new `e1000e.ko` module, and the system can then access the network.

**Job's a good 'un.**
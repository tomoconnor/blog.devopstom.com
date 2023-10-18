---
title: "Building Updated Packages for Sun-Java6 on Ubuntu"
date: "2012-01-05T20:00:00.000Z"
description: "The bare basics of making .deb files"
warning: ancient
---

Firstly, welcome back.

It's now 2012, and there's lots more to write about.


Recently, Oracle withdrew the ability for Linux distributions to repackage Java and distribute their own packages.  This has been widely regarded as a bad idea.  I tend to agree.

So, let's re-roll an old sun-java6 deb file, with a new content to contain the latest 6u30 java release.

You will need: 

1. A set of build packages (I've got a set for lucid, so if this goes away, I'll find some way to host them.) from http://archive.canonical.com/ubuntu/pool/partner/s/sun-java6/
1. The latest Java packages: http://download.oracle.com/otn-pub/java/jdk/6u30-b12/jdk-6u30-linux-i586.bin and http://download.oracle.com/otn-pub/java/jdk/6u30-b12/jdk-6u30-linux-x64.bin
1. `dch`.  just install `devscripts` package to get this. 
1. Some idea of how packaging on debian/ubuntu works.
 

Let's get started.

```
mkdir package-build
cd package-build
wget http://archive.canonical.com/ubuntu/pool/partner/s/sun-java6/sun-java6_6.26-2lucid1.dsc
wget http://archive.canonical.com/ubuntu/pool/partner/s/sun-java6/sun-java6_6.26-2lucid1.debian.tar.gz
wget http://archive.canonical.com/ubuntu/pool/partner/s/sun-java6/sun-java6_6.26.orig.tar.gz
wget http://download.oracle.com/otn-pub/java/jdk/6u30-b12/jdk-6u30-linux-i586.bin
wget http://download.oracle.com/otn-pub/java/jdk/6u30-b12/jdk-6u30-linux-x64.bin
tom.oconnor@charcoal-black:~/package-build$ ls -1
jdk-6u30-linux-x64.bin
jdk-6u30-linux-i586.bin
sun-java6_6.26-2lucid1.debian.tar.gz
sun-java6_6.26-2lucid1.dsc
sun-java6_6.26.orig.tar.gz
tom.oconnor@charcoal-black:~/package-build$ dpkg-source -x *.dsc
gpgv: Signature made Tue 13 Dec 2011 22:31:53 GMT using RSA key ID CC559573
gpgv: Can't check signature: public key not found
dpkg-source: warning: failed to verify signature on ./sun-java6_6.26-2lucid1.dsc
dpkg-source: info: extracting sun-java6 in sun-java6-6.26
dpkg-source: info: unpacking sun-java6_6.26.orig.tar.gz
dpkg-source: info: unpacking sun-java6_6.26-2lucid1.debian.tar.gz
tom.oconnor@charcoal-black:~/package-build$ cd sun-java6-6.26/
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.26$ ls
debian  jdk-6u26-dlj-linux-amd64.bin  jdk-6u26-dlj-linux-i586.bin
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.26$ rm *.bin
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.26$ ../jdk-6u30-linux-i586.bin jdk-6u30-dlj-linux-i586.bin
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.26$ ../jdk-6u30-linux-x64.bin jdk-6u30-dlj-linux-amd64.bin
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.26$ vim debian/rules
```

Head down to the block "# check if the sources are the "same"

Then find and comment the block following it, out, so you get this.
```
#       : # check if the sources are the "same"
#       set -e; set -- $(all_archs); a1=$$1; shift; \
#       unzip -q -d tmp-$$a1/src $$a1-jdk/src.zip; \
#       for a2; do \
#         unzip -q -d tmp-$$a2/src $$a2-jdk/src.zip; \
#         echo "Comparing sources: tmp-$$a1/src tmp-$$a2/src ..."; \
#         echo "    diff -ur $(diff_ignore)"; \
#         diff -ur $(diff_ignore) tmp-$$a1/src tmp-$$a2/src; \
#       done
```

Save that file, and then run:

`dch -v 6.30`
This will create a changelog entry for version 6.30, and open `$EDITOR` to edit the changelog entry. 

Enter a stub entry.. 

I put something like 
```
* Updating internal contents to 6u30
```

.. There's some output, but you can ignore this.
```
dch warning: New package version is Debian native whilst previous version was not
dch warning: your current directory has been renamed to:
../sun-java6-6.30
dch warning: no orig tarball found for the new version.
```

```
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.26$ cd ..
tom.oconnor@charcoal-black:~/package-build$ cd sun-java6-6.30/
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.30$ dpkg-buildpackage -b -uc
```
... LOTS OF STUFF ...
```
tom.oconnor@charcoal-black:~/package-build/sun-java6-6.30$ cd ..
tom.oconnor@charcoal-black:~/package-build$ ls
ia32-sun-java6-bin_6.30_amd64.deb     sun-java6_6.26-2lucid1.dsc  sun-java6-6.30                sun-java6-bin_6.30_amd64.deb   sun-java6-fonts_6.30_all.deb   sun-java6-jdk_6.30_amd64.deb  sun-java6-plugin_6.30_amd64.deb
sun-java6_6.26-2lucid1.debian.tar.gz  sun-java6_6.26.orig.tar.gz  sun-java6_6.30_amd64.changes  sun-java6-demo_6.30_amd64.deb  sun-java6-javadb_6.30_all.deb  sun-java6-jre_6.30_all.deb    sun-java6-source_6.30_all.deb
```

Woo. Debs.

What you want to do with them now is up to you.  Next blogpost, I'm going to go over creating a package repository with reprepro.

 

Thanks  to [@mibus](https://twitter.com/mibus) for his [similar](http://www.mibus.org/2011/12/31/oracle-java-6-ubuntu/) article, which this is based partially on. 
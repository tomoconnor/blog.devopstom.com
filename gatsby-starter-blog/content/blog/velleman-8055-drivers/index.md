---
title: Velleman 8055 Drivers
date: "2010-12-04T23:46:37.121Z"
description: Details on some drivers I wrote once.
warning: ancient
---

`Download the Qcontrol application source. (32.6 KB)`

About 5 years ago, I wrote some "drivers" to interface a Velleman K8055 USB interface card .  After a recent request, I have decided that they should be brushed up a little, and reinstated with a download link.

I'm open to bug reports, but I can't promise that I'll be able to fix them quickly. 

Instructions to compile/install

You'll need the following dependencies:
libusb-dev libqt4-dev qt4-qmake (ubuntu deps)

1. run `qmake -o Makefile qcontrol.pro`
2. `cd lib`
3. `make all; make install`
4. `cd ..`
5. `make`
6. plug in your 8055 usb board
7. `sudo ./qcontrol`
8. enjoy!


I'm reconsidering writing some python bindings for the `libueib.so` driver, stay tuned!  (Never Happened.)


---
author: "Ripx80"
title: "Archlinux kernel configuration and compiling"
linktitle: "Archlinux kernel configuration"
description: "Step by Step kernel configuration and compiling for your system"
date: 2017-09-22T13:27:26+02:00
draft: false
tags: ["arch", "optimization", "security"]
categories: ["linux"]
weight: 0
---

## Introduction

This is a short discurse about self kernel compilation on a arch system. Gentoo users are familiar with this kind of instructions or use the genkernel programm.
You can find an article on https://wiki.archlinux.org/index.php/Kernels/Traditional_compilation

### Install kernel sources and headers

```
$ pacman -S base-devel # only gcc and make
$ mkdir /usr/src/linux
$ cd /usr/src/linux
```
download the newest kernel release, at this moment it is 4.13.0
```
$ wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.13.2.tar.xz
$ wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.13.2.tar.sign
$ wget https://www.kernel.org/pub/linux/kernel/v4.x/sha256sums.asc
```

check the sha256 checksum and decompress the archive and check the sign. you can see the offical key from Greg Kroah-Hartman on the linux website.
```
$ grep 'linux-4.13.2.tar.xz' sha256sums.asc
064adc177a384a7aee6b18ef5d47c1cea3a43fae1aaa6aa95fdc97eb137ffcd1  linux-4.13.2.tar.xz
$ sha256sum linux-4.13.2.tar.xz
064adc177a384a7aee6b18ef5d47c1cea3a43fae1aaa6aa95fdc97eb137ffcd1  linux-4.13.2.tar.xz

$ unxz linux-4.13.2.tar.xz
$ gpg2 --verify linux-4.13.2.tar.sign
$ gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 647F28654894E3BD457199BE38DBBDC86092693E
```
when everything looks good continue with this steps
```
$ tar -xpf linux-4.13.2.tar
$ ln -s linux-4.13.2 linux;cd linux
```
### Kernel configuration and compiling

Now we are in the kernel source tree. clean up the tree is a good thing to begin :-)
if you have a kernel .config copy it to a save place.
```
$ make clean && make mrproper
```
Now the work begin to configure your specific kernel config. this will be saved in .config file in your kernel base dir. A good begin is to use your previously hand made config or use the current running:
```
$ make localmodconfig
# or the running arch config
$ zcat /proc/config.gz > .config

```
you will be asked about the new features. if you unsure use the default.
after this a .config will be created in your kernel base dir. Now its time for you!
use the old way with
```
$ cp .config ../config-default
$ make menuconfig
```
or the new interface
```
$ make nconfig
```
After edit your configuration (super-slim config of my computer will to be download at the end of this article) buid your kernel. But before we install the modules we change the version of our Kernel in the Makefile. We will increase this number if we build a new kernel. So we have no conflicts with the modules... (-j NUMBER, is for the number of cores in your computer)
```
$ nano -w Makefile

EXTRAVERSION = -acr-1

$ make -j3; make modules_install
```


### Setting up your boot environment

When your compile process was successful you will be copy your kernel to your /boot folder (64-bit system). While I have a full encrypted system I need a ramdisk to open my partitions before I boot the initial system. So lets copy and make a ramdisk with the correct modules.

```
$ cp arch/x86_64/boot/bzImage /boot/kernel-4.13.2-acr-1
$ mkinitcpio -k 4.13.2-acr-1-ARCH -g /boot/initramfs-linux-4.13.img
```

And change your boot menu. I used the syslinux boot-manager. So my menu config looks like this.
```
$ nano -w /boot/syslinux/syslinux.cfg#

LABEL arch-4.13.2-atr-1
    LINUX ../kernel-4.13.2-atr-1
    APPEND root=/dev/mapper/crypt cryptdevice=/dev/sda3:crypt rootflags=subvol=__active/root rw intel_iommu=on amd_iommu=on
    INITRD ../initramfs-linux-4.13.img

$ syslinux-install_update -iam
```

And now reboot. If all is correct you will be get the password prompt of the crypt device and you systems boot correctly. But when you look of the size of your kernel...

### Optimize your kernel settings

```
$ du -sh kernel-4.13.2-acr-1-ARCH initramfs-linux-4.13.img
5.2M	kernel-4.13.2-acr-1-ARCH
4.8M	initramfs-linux-4.13.img
```

This is not the optimization you want to. Its time to optimize your kernel needs.
```
$ cd /usr/src/linux/linux
$ make mrproper
$ cp ../config-default ./.config
$ nano -w Makefile
EXTRAVERSION = -acr-2
$ make nconfig
```
Now you deselect all the options you doesn't need for you computer and your setup.
Sometimes its a difficult thing but when you look in the description of a kernel feature you will get an recommendation what you should do:
```
 If you don't know what this means you don't need it.
# or
This is generally a good idea, so say Y.
 ```
Get you a smaller kernel size like 1MB? Its you challange...

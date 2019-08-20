---
layout: post
title: "Initializing and Cache Mechanism in Linux Kernel"
date: 2017-03-12 10:51:47 +0530
tag:
- Linux
- Processes of OS
category: technical
author: B Krishnan Iyer
color: 1976D2
---
I got updates on for my Debian-based Linux operating system and in between system got crashed. It
automatically rebooted, after that all I was able to see is a black screen and had a command line interface
and written a word, “initramfs”.

Way come out of the initramfs boot menu

I booted a live CD of and opened the terminal without installing the OS. Then I executed few commands which
is as follows:
```sh
sudo fdisk -l|grep Linux|grep -Ev 'swap'  #for find out the partition drive in which the crashed OS is
installed.

sudo dumpe2fs /dev/sda2 | grep superblock  #to list all superblock in the partition.

sudo fsck -b 32768 /dev/sda2 -y  #this command will repair the OS by alternate superblock and ‘y’ flag is
used to allow the terminal to fix.
```
It got fixed but this error left with a question.

So question is- What is initramfs?

The Mechanism

In RAM some space is reserved for the system files and also to manage the caches. Ramfs is one of the file
system which exports Linux's disk caching mechanisms as a dynamically resizable RAM-based filesystem. These
types of mechanisms are used to speed the access to directories. When the disk is mounted the cached files
will also be mounted as the system files. Benefit of ramfs is it requires very less amount of memory in the
disk. With the normal mechanisms cached files are deleted when VM tries to reallocate the memory but with
ramfs memory can’t be freed by VM.

Rootfs

Rootfs is a special instance of ramfs. You can't unmount rootfs because you there are signal handlers
running parallel which assure that system is not brought down. It made to check for and handle an empty list
then it's easier for the kernel to just make sure certain lists can't become empty.

Linux kernels contains a gzipped cpio(it is a general file archiver utility) format archive, which is
extracted into rootfs when the kernel boots up.  After extracting, the kernel checks to see if rootfs
contains a file "init", and if so it executes it as PID(it is a unique number that identifies each of the
running processes in an operating system ).

Now there are two possibilities,

Case #1:  If found, this init process is responsible for bringing the system the rest of the way up,
including locating and mounting the real root device (if any).

Case #2: If rootfs does not contain an init program after the embedded cpio archive is extracted into it,
the kernel will fall through to the older code to locate and mount a root partition, then exec some variant
of /sbin/init out of that.

Relation with cpio

The config option CONFIG_INITRAMFS_SOURCE (in General Setup in menuconfig and living in usr/Kconfig) can be
used to specify a source for the initramfs archive, which will automatically be incorporated into the
resulting binary.  This option can point to an existing gzipped cpio archive, a directory containing files
to be archived, or a text file specification. One advantage of the configuration file is that root access is
not required to set permissions or create device nodes in the new archive.

The kernel does not depend on external cpio tools.  If you specify a directory instead of a configuration
file, the kernel's build infrastructure creates a configuration file from that directory (usr/Makefilecalls
scripts/gen_initramfs_list.sh), and proceeds to package up that directory using the config file (by feeding
it to usr/gen_init_cpio, which is created from usr/gen_init_cpio.c).  The kernel's build-time cpio
creation code is entirely self-contained, and the kernel's boot-time extractor is also self-contained.


Instructing  Initramfs

  Create a hello.c file with the code:
```sh
  #include <stdio.h>
  #include <unistd.h>

  int main(int argc, char *argv[])
  {
    printf("Hello world!\n");
    sleep(999999999);
  }
```
Now give the commands:
```sh
 gcc -static hello.c -o init
 echo init | cpio -o -H newc | gzip > test.cpio.gz

 qemu -kernel /boot/vmlinuz -initrd test.cpio.gz /dev/zero

```
When debugging a normal root file system, it's nice to be able to boot with "init=/bin/sh".  The initramfs
equivalent is "rdinit=/bin/sh".

My Note

I was totally annoyed because I was about to loose the whole data but that’s not the case when I understood
why is needed. Open source culture give you the freedom to learn anything, then it is your responsibility
to help in developing it not to ignore it. Open source contributions is the best way you can do for the
society even if you are not highlighted, enjoying the satisfaction when your code or contribution
benefitted lacks of people.

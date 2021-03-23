---
layout: post
title: Installing Gentoo on an existing FreeBSD ZFS install
tags: zfs zol freebsd gentoo
---

After hitting some walls with hardware support for virtualization in FreeBSD, I decided
to move things over to Linux, specifically Gentoo. ZFS on Linux has matured and is now
converging with the FreeBSD codebase. In light of this I figured I would go through
the steps to transition without having to recreate the zpool.

## Installing Linux

### Make backups

Before attempting this - make backups. Make very sure they can be restored, there 
are several points where this can render the system unbootable.

### Install bootloader, kernel, and initramfs

I use [rEFInd](https://www.rodsbooks.com/refind/) as a Linux bootloader. It is 
simple, lightweight, and allows great recovery when things go sideways. And it
looks nice.

After pulling the rEFInd zip, we need to install it. The EFI partition can
be mounted using mount_msdos. Create a directory in EFI/FreeBSD
```shell
mount_msdosfs EFI-Boot-Partition /mnt
mkdir /mnt/EFI/FreeBSD
mv /mnt/EFI/boot/BOOTx64.efi /mnt/EFI/FreeBSD/loader.efi
```

Next install rEFInd in the EFI/boot directory. This way we can copy 
everything over to other disks and remain bootable in case of failure.

Finally, create and EFI/linux directory. Add an appropriate kernel and initramfs
as well as create a refind_linux.conf file to control the kernel command line. This
will vary depending on which initramfs you use. I used a custom one based loosely on
the one with genkernel so the instructions on how to do this will definitely vary.

### Boot to Linux and install a stage3 tarball

I was never able to unpack the stage3 tarball and end up with a usable system
from FreeBSD, even if I used gtar. The busybox based tar in my initramfs did
the trick but a ZFS enabled boot disk will probably be more comfortable for 
pretty much everyone.

Import the pool and create a dataset. Unpack stage the stage3 tarball an
chroot in (possibly after setting up networking) and set it up as per
the Gentoo Handbook. Obviously the kernel and bootloader are taken care
of in my case but your setup may vary. Make sure to set the root password
or else it won't be possible to get into the new install.

If you need to go back to FreeBSD, it seems like loader.efi can still pull
things up *until the bootfs property is changed.* The FreeBSD will not
let you boot if bootfs is pointing at a Linux root dataset. If your initrd 
allows booting an arbitrary dataset, dual booting is possible here.

## Quirks and issues

### Overlay mounts

Once booted into Linux, another issue cropped up. I have 2 pools, one entirely 
SSDs for things where IOPS are a priority and another made of spinning disks
for bulk storage. The mount points for these are interleaved through the 
directory hierarchy as needed. For example my home directory is on the SSD pool
but there is a disk backed archive directory under it.

By default, FreeBSD handled this use case very well and mounted things up at 
boot as expected. ZFS on Linux did not like doing this since it appeared to just
be mounting datasets as they were discovered. It turns out that overlay needed 
to be set to on for these datasets. This is supposed to be the default on both 
sides but somehow the setting got dropped.

### sharenfs issue

This led to the next issue: NFS flags are different between Linux and FreeBSD. 
This means that any datasets with *sharenfs* set to anything other than *on* 
needs to be translated. FreeBSD's handling is straightforward but there is a 
quirk on Linux, there is a legacy translation layer in 
*zfs/lib/libshare/os/linux/nfs.c* that attempts to be compatible with pools 
from the Solaris version of ZFS.

### Upgrading to openZFS 2.0 - can't mount NFS

Finally, another wrinkle arose later when upgrading to openZFS 2.0. Everything 
appeared to be working at first however some datasets caused the client to 
hang when trying to mount them over NFS. When using *zfs send* to copy the 
dataset, the copy suffered the same issue.  Newly created datasets on the 
same pool did not suffer this problem so I used rsync to recreate my 
snapshots and move the data to a new dataset to get things moving.

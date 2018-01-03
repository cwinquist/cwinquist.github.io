---
layout: post
title: Moving a ZFS boot environment to a new zpool on FreeBSD 11.1
---

I recently found myself needing to move my FreeBSD boot environment to a new SSD
based pool within the same server. [This guide from Dan Langille](https://dan.langille.org/2017/09/29/moving-boot-from-one-zpool-to-another/)
seemed the closest to what I was trying to do but didn't quite fit. There were 
a few key differences to account for:

* My boot environments *were* a complete dataset - no need for rsync
* I wanted to keep the old snapshots of the boot environment which means sending a replication stream
* I already had a FreeBSD 11.1 installer ISO that plays nicely with the IPMI 
virtual CDROM on the server so I'd prefer that over mfsBSD

### Set up partition and new pool

I already had the pool set up for other uses but this obviously needs to happen 
first. If at all possible I recomment UEFI booting instead of using the zfs 
BIOS bootcode. The EFI bootloader can be copied to the EFI partitions on each 
of the new pool disks before starting.

### Boot from the 11.1 install media to a livecd

It certainly does not seem safe or wise to try and do this with everything 
mounted and without a boot disk handy. It may be possible but I was not so 
willing to tempt fate. The existing boot environment is on **zroot** and I'm
moving it to **zssd** here.

### Import old and new roots without mounting anything and altroots

First make directories to set as altroots for each pool since if we mount 
anything on the filesystem root on the liveCD it will make a mess of things.

```shell
mkdir /tmp/newzfs /tmp/oldzfs
```

Import both zpools (the liveCD is a different "host" so it will not import
them auto matically) with altroots and the -N flag to tell it not to mount 
any datasets yet.

```shell
zpool import -N -R /tmp/oldzfs zroot 
zpool import -N -R /tmp/newzfs zssd
```
### Send over the stream from the old pool to the new pool

Since I want to keep the existing snapshots, I sent a replication stream of the 
boot environment root to the new pool. This is allowed since we aren't
mounted and will send over all the preexisiting snapshots.

```shell
zfs snapshot -r zroot/ROOT@pre-move
zfs send -R zroot/ROOT@pre-move | zfs recv -d -u zssd
```

### Disable original pool boot environment

Now that the data is copied over, I want to disable booting the original version
on the old pool without deleting it so it's still there in case something went 
wrong. First, I turn off automounting of the active boot environment on the
original pool.

```shell
zfs set canmount=noauto zroot/ROOT/11.1
```

Also note that the canmount property was copied with the replication stream so
we don't need to explicitly set it to on for the new pool.

Next, I set the bootfs property on the new pool to the correct value and empty 
it on the old pool so that the bootloader will do the right thing.

```shell
zpool set bootfs=zssd/ROOT/11.1 zssd
zpool set bootfs= zroot
```

### Export pools and reboot

Finally, I had to export both pools so they would not be considered mounted
by the liveCD on the next boot.

```shell
zpool export zroot
zpool export zssd
```

After rebooting, I had to import the old pool to get the other mounts active 
again since it will not automatically import.

```shell
zpool import zroot
```

After this a quick *zfs list* can verify that the correct version of the boot
environment is mounted. The version on the old pool can now be safely deleted.

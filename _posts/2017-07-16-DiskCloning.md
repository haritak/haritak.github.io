---
layout: post
title:  "ZFS and Disk Cloning"
date:   2017-07-16 21:47:14 +0300
categories: linux
---

Recently I swapped my primary disk for another one.
(on a Thinkpad X230 running Arch Linux)

Since I am using an uncommon setup, I thought it would
be nice to document the process.

So this article is about cloning an ecrypted zfs partition.

Maybe I could just dd one to the other, but I am moving
to a smalled disk (hopefully with better reliability).

# Setup

I use one disk with two primary partions :

1. One 4GB as /boot
2. The rest as an encrypted partiotion which contains ZFS.

# Process overview

1. Create the corresponding new partitions on the new disk.
2. Cloning the ZFS using `zfs send` and `zfs recv`.
3. Installing grub on the new disk.
4. Swaping the old with the new disk.

# The process detailed

Initialy, on my old disk I took a snapshot.

```
zfs snapshot -r zrt/r@201707_16
```

Now, on to the new disk:

After I created two partitions (using fdisk, the first marked as boot)
I went on to :

1. Create an encrypted partiotion using cryptsetup
2. Create a new pool using the encryped partiotion
3. Copy the old snapshot (`zfsz2/flash_2017@201707_16`)
 to the new filesystem (`sanzrt/r -F`)

{% highlight bash %}
cryptsetup luksFormat /dev/sdj2 
cryptsetup luksOpen /dev/sdj2 sanzrt       
zpool create sanzrt /dev/dm-0      
zfs send -Rv zfsz2/flash_2017@201707_16 | zfs recv -v  sanzrt/r -F
{% endhighlight %}

Continuing on the new disk:

1. Create the boot partiotion
2. Copy all files of the old boot partiotion to the new one

{% highlight bash %}
mkfs.ext4 /dev/sdb1
cp -Rva /boot/* /mnt/external/
{% endhighlight %}

After having copied the zfs filesystem, I import the new filesystem under
a different directory (/mnt/external instead of /)

{% highlight bash %}
cryptsetup luksOpen /dev/sdb2 sanzrt
#Enter passphrase for /dev/sdb2: 
zpool import sanzrt -R /mnt/external
{% endhighlight %}

I mount the new boot under the /mnt/external/boot

{% highlight bash %}
mount /dev/sdb1 /mnt/external/boot
arch-chroot /mnt/external/
grub-install /dev/disk/by-id/usb-SanDisk_SDSSDXPS480G_M6116A016V20-0\:0
#Installing for i386-pc platform.
#Installation finished. No error reported.
#At this point I had to edit the fstab (/boot was pointing to old device)
sync
exit
umount /mnt/external/boot/
zpool export sanzrt
{% endhighlight %}

Shutdown, replace the old with the new disk and PowerOn!

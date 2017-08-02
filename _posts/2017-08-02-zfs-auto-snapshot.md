---
layout: post
title: zfs-auto-snapshot
categories: dev linux
---

Just installed [zfs-auto-snapshot][1]. Simply followed the instructions. I will update this post sometime in the (hopefully) near future.

Meanwhile:

ZFS can check your data for data corruption. (zfs scrub). In cases with redudant disks, it can correct any detected error. 
However, even on laptops,
where usually there is only one disk, it can correct the error if _copies_ has been set to two for the filesystem. 
Since this is a huge waste of space on laptops (halves the useful disk capacity), it's a really good idea for sensitive
data to have a separate zfs with two copies enabled.

For those new to zfs, all zfs filesystems that share the same pool (i.e. are  on the same physical disk), have no fixed size. They grow
as long as there is free space.

[1]:https://github.com/zfsonlinux/zfs-auto-snapshot

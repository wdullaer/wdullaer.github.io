---
layout: post
title: "Comparison of RAID-like Systems for a Home NAS"
quote: "Do you really need RAID for a home NAS?"
image: /media/2015-04-05-Hack-WD-Greens/cover.jpg
video: false
excerpt: "There are a lot of ways to configure a server with multiple disks as a redundant network attached storage device. Most rely on RAID-like systems, but there are alternatives. In this post I compare a few of those systems and explain why I chose to use AUFS combined with snapraid."
comments: true
categories:
- Linux
- Tutorial
tags:
- Linux
- Snapraid
- AUFS
- NAS
- JBOD
---
> This post is part of a short series on NAS technology:
>
> - [Make your WD Greens NAS ready]({% post_url 2015-04-05-Hack-WD-Greens %})
- A comparison of different RAID and Union file systems
- [How I configured Snapraid and AUFS]({% post_url 2016-03-19-snapraid-and-aufs-configuration %})
- Incremental backups to object storage using rclone and zbackup

Like a lot of IT people I have a small server at home which serves as the centralised storage for my music, movies, pictures and other files. While buying an appliance, like a qnap or synology, is a perfectly viable option, I would have felt like a carpenter buying his furniture at Ikea. So I've built my own solution using a HP Proliant Microserver (because it was great value for money at the time), 16 gigs of RAM (because this too was cheap back then) and 3 WD Green harddrives (you can guess the theme here). While not massively fast, it is a system fit for purpose: low power, expendable, with lot's of storage. This post will go into a bit more detail on the software I used to configure the NAS.

## Requirements
In order to compare the different options when configuring a server as a NAS, it is worthwhile to list the features I was looking for.

1. **Expose disks as a single volume**  
  I don't want applications using the NAS to have to worry about how the NAS works, in much the same way that you don't worry where dropbox saves its files. I want to merge all the disks into a single volume

2. **Fault Tolerance**  
  When one of the disks eventually dies, I don't want to lose any of my data. I should be able to replace the disk and carry on.

3. **Expandability**  
  I want to be able to add storage at a later date without much hassle. I don't want to have to buy all my storage up front, or have to copy over all my data just to be able to increase my storage space

4. **Graceful Failure**  
  In case of multiple disk failures, I don't want to lose all my data. I should be able to recover the data on the still working disks.

5. **[Bitrot](http://en.wikipedia.org/bitrot) protection**  
  If data on a magnetic disk is not refreshed every now and then it risks to "flip" (a 1 becomes a 0 or vica versa) after some time. Most filesystems are blind to this phenomenon, but I don't want to see the pictures of my son's first birthday evaporate over time.

It is worth noting that some features are explicitly not part of my requirements:

* **Enhanced disk IO performance**  
  I don't intend to serve files to hundreds of users with this. Standard disk read/write performance will do just fine, even in small office scenario's
* **High Availability**  
  While I want my data to be durable (I do not want to lose any), I can live with the fact that some of my data will be temporarily unavailable in the unlikely, but inevitable, case of a disk failure. I will not be running an order management system of this, where every minute of downtime is costing me money.

There are quite a bit of technologies around in the Linux world that meet one or more of these requirements, such as (in no particular order):

* [JBOD](#jbod-with-aufs-/-mhddfs)
* [RAID](#raid-features-and-drawbacks)
* [ZFS](#zfs-features-and-drawbacks)
* [BTRFS](#btrfs)
* [AUFS / MMDHFS](#jbod-with-aufs-/-mhddfs)
* unRaid
* [Snapraid](#snapraid-features-and-drawbacks)


## Solution Proposals
In the following sections I will discuss the pro's and cons of a few of them. I didn't discuss  [unRaid](http://lime-technology.com) in depth, because it is a proprietary solution and I prefer my solutions open and free.

> As this is a constantly evolving domain, I've probably forgotten a few and I'm happy to hear your suggestions in the comments.

### JBOD with AUFS / MHDDFS
[JBOD](http://en.wikipedia.com/JBOD) stands for "Just A Bunch Of Disks" and is not really a technology as such. It is the "naive" way of adding capacity to your server, by just adding disks, each with their own file system. This is very easy to setup and has very little overhead: you can use the full capacity of all your disks. The drawback of this is of course that you do not get any redundancy, error recovery or speed improvements. You are also exposing the fact that your server storage is made up out of a lot of disks. Every application will need to handle this fact.

It would be convenient if we could expose all our disks as a single virtual volume to the outside world. This is where union file systems, like [aufs](https://en.wikipedia.org/wiki/Aufs) or [mhddfs](https://romanrm.net/mhddfs), come into play. They allow you to expose any number of mount points as a single moint point, giving you a unified view of the underlying data.

Depending on how you configure them, you can do things like exposing multiple folders as a single folder, copy-on-write (record changes to data on a different volume than the actual data), tree balanced writing (spread new data over a number of different volumes), etc.

Union file systems allow us to hide the fact that the data is living on multiple hard drives from the outside world.

#### Pro's

* All disks exposed as a single volume
* All disk space is available for storage
* If a disk fails, only data on that disk is impacted
* Storage can be expanded by just adding a disk and tweaking the union file system config
* No need to format the disk when adding it to the storage volume

#### Cons

* No read or write speed improvements
* No redundancy: a disk failure will always result in data loss
* No bitrot protection

#### Conclusion
Apart from improved durability, this solution ticks most of the boxes we are looking for.

### RAID features and drawbacks
[RAID](https://en.wikipedia.org/wiki/RAID) stands for "Reduntant Array of Independent Disks". There are various schemes, also called levels, of RAID configurations, each of which offers a different tradeoff of the following goals:

* Reliability
* Availability
* Performance
* Capacity

RAID was initially proposed to improve the performance of disks. Most RAID levels (except for RAID 1) do this by "striping" the data across all the disks in the array: the data is chopped in blocks which are made available on separate disks. If you are accessing a file you can read from multiple disks in parallel, which will improve performance.

Most RAID levels also provide some redundancy and will continu working even if one (or more, depending on the configuration) of the drives in the array die.

Unfortunately the performance boost of striping has quite a few costs. The size of the array is fixed when it is created. You can't just easily add a new disk to it. Also, if more disks die than the parity allows, all your data is lost, because it is striped across all of the disks. Finally all your disks must have the same size, which means you will buy them together making them much more likely to die at the same time. Striping has other drawbacks as well, but they would lead us to far.

#### Pro's

* All disks are exposed as a single volume
* When using RAID 1 or higher, data is saved redundant and can survive a disk failure
* The RAID array will have better IO performance than a single disk

#### Cons

* The array cannot be easily expanded
* Disks must be all the same size
* If more disks die than the redundancy allows all data is lost
* Disks must be formatted to be added to the array (migrating data in becomes costly)
* No bitrot protection

#### Conclusion
RAID systems can be configured to fulfill the requirements, but it's search for performance improvements (which I don't quite care about) brings quite a few drawbacks compared to JBOD systems.

### ZFS features and drawbacks
[ZFS](https://en.wikipedia.org/wiki/zfs) is a file system originally made by SUN microsystems to overcome some of the drawbacks and limitations of traditional RAID arrays. It focusses hard on data integrity, such as bitrot protection. ZFS has gained quite a bit of popularity with hobbyist and tweakers through the [FreeNAS](https://www.freenas.org) server distribution.

ZFS provides data integrity by saving checksum of each block of data. When data is read the checksum of the current data is compared with the saved checksum. If they do not match, ZFS will try to heal the data using a redundant copy of the data. This way ZFS will guarantee that only correct data is read. By saving the checksum in a different place than the actual data, the system effectively checks itself.

ZFS can also be configured to work as a RAID array, using striping to provide IO performance improvements and redundancy. However, ZFS has been designed so that some of the drawbacks of striping are not applicable.

Compared to RAID arrays, ZFS has less restrictions on the drives: you can group together disks of different sizes, but the capacity of each drive will be limited to the capacity of the smallest drive in the group. ZFS pools can be extended in size, but this effectively boils down to adding an additional (Z-)RAID array to the pool, making it far from ideal for small pools like the ones we are trying to create.

For good performance, ZFS can also be quite demanding on the system resources, especially RAM. It is recommended to have 1 GB of RAM per TB of storage. This is not a big issue for me personally: I equiped my server with apply RAM, but not every home NAS will be in this situation.

ZFS has a bunch of other nice features, such as copy-on-write support and snapshotting, which can be very useful in more critical environments but are probably a bit overkill for a home NAS.

#### Pro's

* All disks are exposed as a single volume
* Automatic protection against bitrot and other silent data corruptions
* Better IO performance than a single disk
* Good data redundancy features
* Capacity can be increased

#### Cons

* Cumbersome to increase capacity
* Disks must be formatted to added to the pool
* Resource intensive
* If more disks die than the redundancy allow, all data can still be lost

#### Conclusion
Compared to RAID, ZFS offers a lot of advantages. Especially its focus on data integrity is a very big advantage. If I were to create a competitor to Amazons AWS or need to serve a big enterprise with files, I would definitely consider ZFS. Unfortunately it's focus on big deployments makes it unwieldy in the home NAS case.

### Btrfs
[Btrfs](https://en.wikipedia.org/wiki/Btrfs) or better-fs looks like a very promising option. It has a lot of the same goals as ZFS and aims to provide a robust filesystem with good data integrity, RAID features, snapshotting and much more. At this point in time however, it is still quite experimental and I wouldn't trust any data on it that I can't afford to lose. In the not so distant future however, this looks like it could become your one stop shop for all your filesystem needs on Linux.

### Snapraid features and drawbacks
Unlike the previous entries, [snapraid](http://snapraid.sourceforge.net) is not a filesystem. It is a program that adds redundancy and data integrity features to your current filesystem setup. It does not provide any union file system features.

Snapraid works by checksumming the data contained on certain drives and saving this checksum information on a parity drive. For each parity drive your disk pool can survive 1 disk failure. The only requirement on the disks is that the parity disk is at least as large as the largest data disk. Additionally, like ZFS, snapraid can scrub the datadisk and check if the data still matches the checksums. This allows it to detect silent data corruption like bitrot.

The biggest drawback to snapraid is that it needs to be scheduled: it does not provide its features online when the system is in use. This means that in between two runs snapraid, any changes made to the data are essentially unprotected.

When combined with a JBOD/AUFS system like we described earlier, extending the disk pool with snapraid is very easy: you add it to the system, point snapraid at it and run a sync. Provided the new disk is not bigger than the current parity disk(s), your new disk along with its data is now protected by snapraid.

#### Pro's

* Little requirements on the disks, their size or filesystem
* Protects against disk failures
* If more disks die than the parity allows you to recover, the remaining data is still OK
* Protects against silent data corruption such as bitrot

#### Cons

* No IO performance improvement
* Batch program: there is a window in which new data is unprotected

#### Conclusion
Snapraid nicely complements the JBOD / AUFS system described earlier, by provided redundancy and bitrot protection. It poses little restrictions on the drives you add to the system and does not modify the data of the underlying file systems. It is not perfect however: being a batch program it leaves a window in which new data is unprotected.

## Conclusion
Given my requirements, a JBOD / AUFS setup protected by snapraid was the logical choice. This was mostly driven by the fact that I am not interested in the IO performance improvements a RAID-like system gives you. Given the fact that the price of storage decrease tremendously over time, I was also very keen on easily growing my storage as I needed it. RAID-like systems typically require you to buy all the storage you need up front.

If the small window wherein data loss is possible is not appropriate for your application I would very much recommend giving ZFS a closer look.

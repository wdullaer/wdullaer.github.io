---
layout: post
title: "Create a NAS with redundancy using Snapraid"
quote: "Because sometimes RAID and ZFS are overkill"
image: /media/06-02-2015-Grim-Fandango-With-Linux-Radeon-Drivers/cover.png
video: false
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

> - Make your WD Greens NAS ready
- A comparison of different RAID and Union file systems
- How I configuration of Snapraid and AUFS

Like a lot of IT people I have a small server at home which serves as the centralised storage for my music, movies, pictures and other files. While buying an appliance, like a qnap or synology, is a perfectly viable option, I would have felt like a carpenter buying his furniture in Ikea. So I've built my own solution using a HP Proliant Microserver (because it was great value for money at the time), 16 gigs of RAM (because this too was cheap back then) and 3 WD Green harddrives (you can guess the theme here). While not massively fast, it is a system fit for purpose: low power, expendable, with lot's of storage. This post will go into a bit more detail on the software I used to configure the NAS.

## Requirements
In order to compare the different options when configuring a server as a NAS, it is worthwhile to list the features I was looking for.

1. Expose disks as a single volume
  I don't want applications using the NAS to have to worry about how the NAS works, in much the same way that you don't worry where dropbox saves its files. I want to merge all the disks into a single volume

2. Fault Tolerance
  When one of the disks eventually dies, I don't want to lose any of my data. I should be able to replace the disk and carry on.

3. Expandability
  I want to be able to add storage at a later date without much hassle. I don't want to have to buy all my storage up front, or have to copy over all my data just to be able to increase my storage space

4. Graceful Failure
  In case of multiple disk failures, I don't want to lose all my data. I should be able to recover the data on the still working disks.

5. [Bitrot](http://en.wikipedia.org/bitrot) protection
  If data on a magnetic disk is not refreshed every now and then it risks to "flip" (a 1 becomes a 0 or vica versa) after some time. Most filesystems are blind to this phenomenon, but I don't want to see the pictures of my son's first birthday evaporate over time.

There are quite a bit of technologies around in the Linux world that meet one or more of these requirements, such as (in no particular order):

* JBOD
* Raid
* ZFS
* BTRFS
* AUFS / MMDHFS
* unRaid
* Snapraid

In the following sections I will discuss the pro's and cons of a few of them.

> As this is a constantly evolving domain, I've probably forgotten a few and I'm happy to hear your suggestions in the comments.

### JBOD with AUFS / MMDHFS
[JBOD](http://en.wikipedia.com/JBOD) stands for "Just A Bunch Of Disks" and is not really a technology as such. It is the "naive" way of adding capacity to your server, by just adding disks, each with their own file system. This is very easy to setup and has very little overhead: you can use the full capacity of all your disks. The drawback of this is of course that you do not get any redundancy, error recovery or speed improvements. You are also exposing the fact that your server storage is made up out of a lot of disks. Every application will need to handle this fact.

It would be convenient if we could expose all our disks as a single virtual volume to the outside world. This is where union file systems come into view. The allow you to take existing

Pro's:

* All disks exposed as a single volume
* All disk space is available for storage
* If a disk fails, only data on that disk is impacted
* Storage can be expanded by just adding a disk and tweaking the union file system config
* No need to format the disk when adding it to the storage volume

Cons:

* No read or write speed improvements
* No redundancy: a disk failure will always result in data loss
*

### RAID features and drawbacks

### ZFS features and drawbacks

### AUFS features and drawbacks

## AUFS Configuration

## Snapraid Configuration


    mount
    /dev/sdc1 on / type ext4 (rw,noatime,errors=remount-ro)
    proc on /proc type proc (rw,noexec,nosuid,nodev)
    sysfs on /sys type sysfs (rw,noexec,nosuid,nodev)
    none on /sys/fs/cgroup type tmpfs (rw)
    none on /sys/fs/fuse/connections type fusectl (rw)
    none on /sys/kernel/debug type debugfs (rw)
    none on /sys/kernel/security type securityfs (rw)
    udev on /dev type devtmpfs (rw,mode=0755)
    devpts on /dev/pts type devpts (rw,noexec,nosuid,gid=5,mode=0620)
    tmpfs on /tmp type tmpfs (rw,noatime)
    tmpfs on /run type tmpfs (rw,noexec,nosuid,size=10%,mode=0755)
    none on /run/lock type tmpfs (rw,noexec,nosuid,nodev,size=5242880)
    none on /run/shm type tmpfs (rw,nosuid,nodev)
    none on /run/user type tmpfs (rw,noexec,nosuid,nodev,size=104857600,mode=0755)
    none on /sys/fs/pstore type pstore (rw)
    tmpfs on /var/spool type tmpfs (rw,noatime)
    tmpfs on /var/cache type tmpfs (rw,noatime)
    tmpfs on /var/log type tmpfs (rw,noatime)
    /dev/sdb1 on /mnt/disk2 type ext4 (rw,noatime)
    /dev/sda1 on /mnt/disk1 type ext4 (rw,noatime)
    none on /mnt/data type aufs (rw,br=/mnt/disk1=rw:/mnt/disk2=rw,create=pmfs)
    systemd on /sys/fs/cgroup/systemd type cgroup (rw,noexec,nosuid,nodev,none,name=systemd)
    rpc_pipefs on /run/rpc_pipefs type rpc_pipefs (rw)
    nfsd on /proc/fs/nfsd type nfsd (rw)

Add anacron and postfix folders to rc.local when using tmpfs: /var/spool/anacron /var/spool/postfix



Sync script: http://zackreed.me/articles/83-updated-snapraid-sync-script

Configure postscript: https://rtcamp.com/tutorials/linux/ubuntu-postfix-gmail-smtp/

---
layout: post
title: "Create a NAS with Redundancy Using Snapraid"
quote: "Because sometimes RAID and ZFS are overkill"
image: /media/2015-04-05-Hack-WD-Greens/cover.jpg
video: false
excerpt: "Like a lot of techie people, I run a home server to centralize access to music, video's, photos and other files. Because I have built my own server using a HP Proliant Microserver running Ubuntu, I need to do a bit more work myself in order to group my disks together in a redundant pool. This post will focus on how I've configured AUFS to expose my disks as a single volume and configured Snapraid to add redundancy and bitrot protection to the data."
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
- [A comparison of different RAID and Union file systems]({% post_url 2015-04-28-Comparison-Of-Raid-Like-Systems %})
- How I configured Snapraid and AUFS
- Incremental backups to object storage using rclone and zbackup

Like a lot of tech people, I run a home server to centralize access to music, video's, photos and other files. Because I have built my own server using a HP Proliant Microserver running Ubuntu, I need to do a bit more work myself in order to group my disks together in a redundant pool. This post will focus on how I've configured AUFS to expose my disks as a single volume and configured Snapraid to add redundancy and bitrot protection to the data.

## AUFS Configuration
The purpose of AUFS is to expose the various disks in the system as one folder. Other programs and systems, such as NFS, Samba or Plex, can interact with this folder without having to worry about distributing the data across the various disks.

The first step is to mount all the individual drives somewhere on the filesystem. In my case all drives use the ext4 filesystem, resulting in the following entries in `/etc/fstab`

```bash
# Mount the data storage
UUID=ac0a0d55-8263-4e1b-8912-cc7942762141       /mnt/disk4      ext4    noatime,defaults        0       0
UUID=f01ded31-2132-495f-9900-45eda4980e60       /mnt/disk3      ext4    noatime,defaults        0       0
UUID=ba85c14c-173a-4f71-8618-8733a46af67b       /mnt/disk2      ext4    noatime,defaults        0       0
UUID=8f3b7486-7529-4552-820c-284232c8354d       /mnt/disk1      ext4    noatime,defaults        0       0
```

The next step is to mount all those individual mount points on top of the folder we want to expose using AUFS

```bash
none    /mnt/data       aufs    br=/mnt/disk1=rw:/mnt/disk2=rw:/mnt/disk3=rw,create=mfs,auto    0       0
```

AUFS is configured using the options string of the `/etc/fstab` entry:

* `br=` specifies the folders (branches in AUFS speak) which should be layered on the mount point and whether they are read-only, write-only or read-write. In this setup the branches are disks 1 to 3. I'm reserving disk 4 as the parity disk for snapraid.
* `create=` specifies how writes are distributed among the various write enabled components

AUFS supports a number of create strategies, which fall roughly into the following 3 categories:

* **top-down**  
  selects the highest writable branch where the parent directory exists. This strategy allows you to keep certain folders on certain disks.
* **round-robin**  
  round robins between the writable branches. This can be useful if you want to maximize read and write throughput.
* **most-free-space**  
  selects the branch with the most free space.

I have chosen a most-free-space (mfs) strategy, but all of them have their value depending on the use case.


## Snapraid
[Snapraid](http://snapraid.sourceforge.net) is a system that adds redundancy and bitrot protection to a set of disks by running periodic checksums over the data. It is available for Linux and Windows. Its batch nature means that there is always a window in which new writes are unprotected. Its major selling point is that you are not tied into a specific filesystem. For slow moving data, like the data often found on a home NAS, I think this is an acceptable trade-off. Recent versions of Snapraid have also added the ability to pool disks together. This might be useful if you're running a Windows server, but AUFS is still the better tool for the job on Linux.

### Install Snapraid
Snapraid is not available by default in the Ubuntu repositories, but the author maintains a [PPA](https://launchpad.net/~tikhonov/+archive/ubuntu/snapraid) with up to date binaries. This makes installing Snapraid as simple as:

```bash
sudo add-apt-repository ppa:tikhonov/snapraid
sudo apt-get update
sudo apt-get install snapraid
```

### Configure Snapraid
Snapraid looks for its configuration in `/etc/snapraid.conf`. The Ubuntu package already comes with a default configuration, which still requires the following additions:

* **disk**: the mountpoint of the disks you wish to protect
* **parity**: the absolute path of the file where the parity information will be stored. This can **not** be on a data disk
* **content**: The absolute path of the file where snapraid keeps the disk content. You should have multiple copies for some redundancy.
* **exclude**: Paths and regexes that should be excluded from the parity. This can be useful to exclude fast changing files (Snapraid doesn't like it when a file changes during parity calculation).

In order to save the parity of disk1, disk2 and disk3 on disk4, I ended up adding the following lines to the default snapraid configuration:

```bash
parity /mnt/disk4/snapraid.parity

content /var/snapraid/snapraid.content
content /mnt/disk1/snapraid.content
content /mnt/disk2/snapraid.content
content /mnt/disk3/snapraid.content

disk d1 /mnt/disk1/
disk d2 /mnt/disk2/
disk d3 /mnt/disk3/

exclude *.unrecoverable
exclude /tmp/
exclude /lost+found/
exclude *.bak
exclude .DS_Store
exclude .Thumbs.db
```

### Schedule Snapraid
Because snapraid does not protect the data in real time we will need to schedule it. This could be as simple as putting a `snapraid sync` entry into your crontab, but I stumbled upon [a great script](http://zackreed.me/articles/83-updated-snapraid-sync-script) which makes the setup much more refined:

1. Run `snapraid diff` to find out how many files have changed
2. If the number of deleted files exceeds a threshold (50 by default), cancel the sync and send an email. This allows you to review the changes and potentially restore an accidental deletion. You can tweak the threshold to suit your work pattern.
3. Run `snapraid sync` to update the parity file.
4. Every so few days (7 by default) run a partial scrub of the contents. `snapraid scrub` verifies files against their parity allowing it to detect bitrot.

Rather than configuring a crontab entry, I pasted the script into `/etc/cron.daily/snapraidsync` to schedule it.

The script uses `mutt` to send emails, which you might have to install

```bash
sudo apt-get install mutt
```

In order for `mutt` (or `sendmail`) to work, you will also need to configure postfix. More specifically, you will need to give it an SMTP server to use. If you have a gmail address you can use gmails SMTP server. Setting this up is slightly more tricky than it should be, but here's [a very good tutorial](https://rtcamp.com/tutorials/linux/ubuntu-postfix-gmail-smtp/) that walks you through it.

## Further Reading

* [AUFS Manual: branch write strategies](http://aufs.sourceforge.net/aufs.html#Policies%20to%20Select%20One%20among%20Multiple%20Writable%20Branches)
* [Snapraid Sync script](http://zackreed.me/articles/83-updated-snapraid-sync-script)
* [Configure postscript with gmail](https://rtcamp.com/tutorials/linux/ubuntu-postfix-gmail-smtp/)

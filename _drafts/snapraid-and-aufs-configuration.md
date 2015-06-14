---
layout: post
title: "Create a NAS with redundancy using Snapraid"
quote: "Because sometimes RAID and ZFS are overkill"
image: /media/2015-04-05-Hack-WD-Greens/cover.jpg
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

> - [Make your WD Greens NAS ready]({% post_url 2015-04-05-Hack-WD-Greens %})
- [A comparison of different RAID and Union file systems]({% post_url 2015-04-28-Comparison-Of-Raid-Like-Systems %})
- How I configured Snapraid and AUFS


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
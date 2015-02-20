---
author: Wouter Dullaert
comments: true
date: 2010-02-26 15:19:10+00:00
layout: post
slug: boot-iso-files-from-usb-with-grub4dos
title: "Boot ISO files from USB with grub4dos"
quote: "Create the ultimate recovery stick"
excerpt: "The goal is to make a universal bootable usb device with a small boot partition and a data partition on which we'll store the iso files. This means you can just download almost any bootable iso and boot it without having to burn a cd or unpack the iso. We'll install grub4dos as boot loader, using the 'triple mbr' feature to increase the compatibility with different mainboard and BIOS configurations. We'll be using command line linux applications to reach our goal, any distro will do."
wordpress_id: 6
categories:
- Tutorial
tags:
- boot
- bootable
- grub4dos
- iso
- linux
- triple mbr
- usb
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

The goal is to make a universal bootable usb device with a small boot partition and a data partition on which we'll store the iso files. This means you can just download almost any bootable iso and boot it without having to burn a cd or unpack the iso. We'll install [grub4dos](https://gna.org/projects/grub4dos/) as boot loader, using the 'triple mbr' feature to increase the compatibility with different mainboard and BIOS configurations. We'll be using command line linux applications to reach our goal, any distro will do.

1. Find the path to your usb device, here we'll use `/dev/sdb`

2. Optionally, to erase the usb device, issue the following commands as root:

    ```bash  
    shred -v -n0 -z /dev/sdb
    ```

3. Note down the size of the device in bytes:

    ```bash
    fdisk -lu /dev/sdb

    Disk /dev/sdb: 1031 MB, 1031798272 bytes
    ```

4. To make `fdisk` happy, partitions must end on cylinder boundaries. Here we'll use the standard 63 sectors/track and 255 heads, so to calculate the number of cylinders and the last sector of the last cylinder, do the following:

    ```
    num_cyl = floor( 1031798272 / 512 / 63 / 255 ) = 125
    last_sector = num_cyl*255*63 - 1 = 2008124
    ```

5. We'll make two partitions: a small boot partition with a ext2 filesystem and a second fat32 (or ntfs) partition to hold the iso files. Choosing ext2 as the boot filesystem hides it from windows systems, and makes the second fat32 partition visible. The boot partition only has to hold the 'grldr' and the 'menu.lst' file, so we set its size to 1 cylinder (or a little less if we take the first 95 boot sectors into account). The last sector of the first partition is `1*255*63-1 = 16064`.

    We use `fdisk` to set up the geometry of the device and make the first partition. Replace `$num_cyl` and `$last_sector` with the appropriate values.

    ```
    fdisk /dev/sdb << EOF
    x
    s
    63
    h
    255
    c
    $num_cyl
    r
    u
    n
    p
    1
    95
    16064
    n
    p
    2
    16065
    $last_sector
    t
    2
    c
    a
    1
    w
    EOF
    ```

6. At this point, remove the usb device and plug it back in. The final layout should look like this:

    ```bash
    fdisk -lu /dev/sdb

    Disk /dev/sdb: 1031 MB, 1031798272 bytes
    255 heads, 63 sectors/track, 125 cylinders, total 2015231 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Disk identifier: 0x8d7751c5

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1   *          95       16064        7985  83  Linux
    /dev/sdb2           16065     2008124      996030    c  W95 FAT32 (LBA)
    ```

7. Create the filesystems:

    ```bash
    mke2fs -L boot /dev/sdb1
    mkdosfs -F 32 -n data /dev/sdb2
    ```

8. As root, make a temporary working folder. Download the latest grub4dos and install boot code on the first partition:

    ```bash
    cd
    mkdir usbtemp
    cd usbtemp
    wget http://download.gna.org/grub4dos/grub4dos-0.4.4-2009-06-20.zip
    unzip grub4dos-0.4.4-2009-06-20.zip
    cd grub4dos-0.4.4
    ./bootlace.com --floppy=0 /dev/sdb1
    ```

9. Extract the first 96 sectors and create the triple MBR:

    ```bash
    dd if=/dev/sdb of=boot.img bs=512 count=96
    ./bootlace.com boot.img
    dd of=/dev/sdb if=boot.img bs=512 count=96
    ```

10. Mount the boot partition and copy the necessary grub4dos files:

    ```bash
    mkdir /mnt/usbboot
    mount /dev/sdb1 /mnt/usbboot/
    cp grldr /mnt/usbboot/
    ```

11. As a test, we'll boot the [SliTaz LiveCD](http://www.slitaz.org/) from the usb device. The iso will be saved to the second partition, and an entry in the `menu.lst` file on the boot partition will make the iso available for booting.

    ```bash
    mkdir /mnt/usbdata
    mount /dev/sdb2 /mnt/usbdata
    mkdir /mnt/usbdata/images
    cd /mnt/usbdata/images
    wget http://mirror.switch.ch/ftp/mirror/slitaz/iso/2.0/slitaz-2.0.iso
    cd /mnt/usbboot/
    umount /dev/sdb2
    cat > menu.lst << EOF
    title Boot SliTaz 2.0 LiveCD
    find --set-root /images/slitaz-2.0.iso
    map --mem /images/slitaz-2.0.iso (0xff)
    map --hook
    chainloader (0xff)
    EOF
    cd
    umount /dev/sdb1
    ```

12. This should do it...  
To clean up just remove the 'usbtemp' folder.

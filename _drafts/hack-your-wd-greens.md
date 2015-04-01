---
layout: post
title: "Hack your WD Greens"
quote: "A small firmware tweak will turn them into Reds"
image: /media/2015-04-07-Hack-WD-Greens/cover.jpg
video: false
excerpt: "WD Green and Red disks are physically identical, with only a firmware setting distinguishing them. The default settings on a WD Green will cause them to die prematurely when used in a file server environment. Fortunately you can change this setting very easily. Here's how."
comments: true
categories:
- Tutorial
tags:
- Western Digital
- Firmware
- wdidle3.exe
- idle3-tools
---
> This post is part of a short series on NAS technology:

> - Make your WD Greens NAS ready
- A comparison of different RAID and Union file systems
- How I configuration of Snapraid and AUFS

I recently needed more storage space in my home NAS and decided to add a few disk. Because my NAS is mostly used as a media server, I don't need any high RPM drives: 5400 RPM drives will suffice. After some googling I settled on buying Western Digital drives. They have good and bad reviews like any other brand, but they never seem to score below average in any of them.

There is now just one choice left to be made: the cheaper WD Greens of "server ready" WD Reds?
It turns out that these drives are physically identical, with just a firmware parameter and warranty period setting them apart.

## Difference between WD Green and Red harddisks
In order to minimize energy usage, WD Green disks are configured to very aggressively park the head of the disk: this happens after 3 seconds. That setting is OK if you're using this as your OS disk, because disk usage tends to be grouped in peaks.

This setting is fatal for a disk if it gets used in a server. Disk usage on file servers is much wider spread, causing the disk heads to move *a lot*. So much so that some bits of plastic will wear out in the first year.

WD Reds wait much longer to park the disk head, eliminating this issue.

Fortunately for us, there is a tool that allows you to set this value yourself. On windows this a tool released by WD themselves because they shipped some Red branded disks, with the Green firmware: wdidle3.exe

On Linux some folks reversed engineered this and released it as idle3-tools.

## Change the idle time on Windows
The first thing you need to is get the wdidle3.exe program. You can get it from [Western Digital](http://support.wdc.com/product/download.asp?groupid=609&sid=113) or via a simple google search.

Once you have it on your machine, you'll need the command prompt to use it. The following command will read the current value of the parking speed:

```bash
wdidle3.exe blablabla
```

Using the tool you can set the parking speed to a maximum of 300 seconds or disable it completely.

```bash
wdidle3.exe blablabla
```

And that's it, your green drives are now perfectly safe to use in a server.

## Change the idle time on Linux

WD doesn't offer a tool to change the parking speed on linux, but a few folks reverse engineered it. It is available on [sourceforge](http://idle3-tools.sourceforge.net) and in the idle3-tools package on ubuntu

```bash
sudo apt-get install idle3-tools
```

You can check the current setting for `/dev/sda` with this command:

```bash
idle3ctl -g /dev/sda
```

You can change the setting to 300 seconds as follows:

```bash
idle3ctl -s 300 /dev/sda
```

If you want to check how much the head of your disk has already been parked, you can check the `Load_Cycle_Count` outputted by smartctl. Asuming your drive is `/dev/sda`, the command looks as follows:

```bash
smartctl -A /dev/sda | grep Load_Cycle_Count
```
WD drives are rated for up to 250 000 parking cycles. The `Load_Cycle_Count` should be well below that number (a couple 1000 maximum if the drive has been in operation for some time).

## Further reading

* [Interesting post on the FreeNAS forum explaining the issue in more detail](http://forums.freenas.org/index.php?threads/hacking-wd-greens-and-reds-with-wdidle3-exe.18171/)
* [Homepage and documentation of the idle3-tools](http://idle3-tool.sourceforge.net)

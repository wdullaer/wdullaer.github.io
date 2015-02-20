---
layout: post
title: "Hack your WD Greens"
quote: "A small firmware tweak will turn them into Reds"
image: /media/06-02-2015-Grim-Fandango-With-Linux-Radeon-Drivers/cover.png
video: false
comments: true
categories:
- Tutorial
tags:
- Western Digital
- Firmware
- wdidle3.exe
- idle3-tools
---
I recently needed more storage space in my home NAS and decided to add a few disk. Because my NAS is mostly used as a media server, I don't need any high RPM drives: 5400 RPM drives will suffice. After some googling I settled on buying Western Digital drives. They have good and bad reviews like any other brand, but they never seem to score below average in any of them.

There is now just one choice left to be made: the cheaper WD Greens of "server ready" WD Reds?
It turns out that these drives are physically identical, with just a firmware parameter and warranty period setting them apart.

## Difference between Green and Red
In order to minimize energy usage, WD Green disks are configured to very aggressively park the head of the disk: this happens after 3 seconds. That setting is OK if you're using this as your OS disk, because disk usage tends to be grouped in peaks.

This setting is fatal for disk if it gets used in a server. Disk usage on file servers is much wider spread, causing the disk heads to move *a lot*. So much so that some bits of plastic will wear out in the first year.

WD Reds wait much longer to park the disk head, eliminating this issue.

Fortunately for us, there is a tool that allows you to set this value yourself. On windows this a tool released by WD themselves because they shipped some Red branded disks, with the Green firmware: wdidle3.exe

On Linux some folks reversed engineered this and released it as idle3-tools.

## Change the idle time on Windows
Hack your WD green: http://forums.freenas.org/index.php?threads/hacking-wd-greens-and-reds-with-wdidle3-exe.18171/

## Change the idle time on Linux
sudo apt-get install idle3-tools

---
layout: post
title: "Play Grim Fandango on Linux with the Open Source Radeon Drivers"
quote: "The Nineth Underworld is just a tweak away, jefe"
image: /media/06-02-2015-Grim-Fandango-With-Linux-Radeon-Drivers/cover.png
video: false
comments: true
categories:
- Linux
- Tips
tags:
- Grim Fandango
- Linux
- ATI
- Radeon
- Ubuntu
- Open Source
---
Even though [the official line](http://steamcommunity.com/app/316790/discussions/0/620703493334148904) is that the open source AMD radeon drivers are not supported, you can get them to work very easily.

If you try to run the game unmodified you will run into the following error

```
libGL error: unable to load driver: r600_dri.so
libGL error: driver pointer missing
libGL error: failed to load driver: r600
libGL error: unable to load driver: swrast_dri.so
libGL error: failed to load driver: swrast
```

The solution is actually very simple: just remove or rename the included `libstdc++.so.6` (be sure to rename the 32 bit version, the game doesn't ship 64 bit binaries).

```bash
mv $GRIM_FANDANGO_ROOT/game/bin/i386/usr/lib/i386-linux-gnu/libstdc++.so.6 $GRIM_FANDANGO_ROOT/game/bin/i386/usr/lib/i386-linux-gnu/libstdc++.so.6.bak
```
This will force the game to use the local installed version which has no issues loading the OpenGL libraries.

I'm not exactly sure why this works, but at least I get to enjoy one of my childhood favourite games.

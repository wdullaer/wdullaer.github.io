---
layout: post
title: "Disable Right Click Zone In Linux On A Synaptics Touchpad"
quote: "Use that beautiful touchpad the way god intended"
excerpt: "By default the Synaptics driver on Linux will treat the bottom right corner of your touchpad as a right click button, even if you have a macbook like trackpad or Logitech T650 touchpad. I have become used to clicking with 2 fingers for a right click, so the that zone is more annoying than useful. Here's how to disable it."
image: /media/2015-10-08-Touchegg/cover.jpg
video: false
comments: true
categories:
- Tutorial
tags:
- touchegg
- logitech t650 touchpad
- apple magic trackpad
- multitouch gestures
- linux
- rightclick
---
Support for modern touchpads in Linux has been steadily improving in the last few years, but the default settings are still tailored towards yesteryears awkward touchpads rather the glass beauties you find in a macbook or high end dell. In a [previous post]({% post_url 2015-10-08-touchegg %}) I explained how you can enable multitouch gestures. This post explains how you can disable the right click zone, and just rely on a two finger click for the "right click" menu.

The right click zone is configured in the Synaptics driver. You can disable it using the `synclient` utility on a per session basis, or by tweaking `xorg.conf` if you want to save the settings.

## Disable for current session
Using `synclient`, you can disable the right click zone in the current session by running the following commands:

```bash
synclient RightButtonAreaLeft=0
synclient RightButtonAreaTop=0
```

You can run `synclient -l` to see if the settings have been properly applied.

## Disable for all sessions
To disable the right click zone permanently follow these steps:

1. Move the Synaptics configuration to a new file to prevent a package update from overwriting your changes

    ```bash
    mv /usr/share/X11/xorg.conf.d/50-synaptics    /usr/share/X11/xorg.conf.d/40-synaptics
    ```

2. Comment out the `Default clickpad buttons` block in `/usr/share/X11/xorg.conf.d/40-synaptics` to disable the right click zone:

    ```bash
    # Section "InputClass"
    #        Identifier "Default clickpad buttons"
    #        MatchDriver "synaptics"
    #        Option "SoftButtonAreas" "50% 0 82% 0 0 0 0 0"
    #        Option "SecondarySoftButtonAreas" "58% 0 0 15% 42% 58% 0 15%"
    # EndSection
    ```

    Alternatively, if you don't have the default ubuntu config, you can add the following block to the file:

    ```bash
    Section "InputClass"
        Identifier "Default clickpad buttons"
        MatchDriver "synaptics"
        Option "RightButtonAreaLeft" "0"
        Option "RightButtonAreaTop" "0"
    EndSection
    ```

## Further Reading

* [Disable Right Click Synaptics](http://kernpanik.com/geekstuff/2015/01/12/disable-rightclick-synaptics.html)

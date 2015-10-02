---
layout: post
title: "Multitouch trackpad gestures in Kubuntu 15.04 with Touchegg"
quote: "Out hands have 5 fingers, why only use 2?"
excerpt: "Support for multitouch gestures on Linux has been steadily increasing in the past years. I recently bought a Logitech T650 touchpad and all the basics worked right of the box: moving the cursor, two finger click for right click, two finger swipes for scrolling through documents. Unfortunately it wasn't immediately obvious how I could configure anything more (like pinching gestures, or 3 finger gestures)."
image: /media/2015-10-10-Multitouch-Gestures-With-Touchegg/cover.jpg
video: false
comments: true
categories:
- Tutorial
tags:
- touchegg
- logitech touchpad
- apple magic trackpad
- multitouch gestures
- linux
- kde
- kubuntu
---
Support for multitouch gestures on Linux has been steadily increasing in the past years. I recently bought a [Logitech blabla touchpad](http://support.logitech.com/product/touchpad-t650) and all the basics worked right of the box: moving the cursor, two finger click for right click, two finger swipes for scrolling through documents. Unfortunately it wasn't immediately obvious how I could configure anything more (like pinching gestures, or 3 finger gestures).

Googling the problem mostly turned up links from the technology stoneage, when these type of devices were new and support was still being added to the various pieces of the Linux stack. Most of this information was no longer relevant by this point in time (just like this post won't be relevant 5 years from now), but somewhere on page 5 in google, I found the solution: Touchegg.

## Installing Touchegg
Touchegg is a program which detects gestures from compatible touch devices (such as trackpads or touchscreens) and maps them to a variety of actions. [Some documentation](http://askubuntu.com/questions/206267/how-to-install-touchegg) states that you'll need to compile it from source, but there is a perfectly working version included in the Ubuntu repositories. This makes installing it as simple as

```bash
sudo apt-get install touchegg
```

Touchegg itself runs in userland, meaning you can start it by typing `touchegg` in a terminal without requesting root permissions. You will most likely want to run Touchegg when the session starts, so you can add it in Configuration panel.

![Screenshot goes here](blablabla)

Alternatively you can put a symbolic link to the program into the `.kde/Autostart` folder

```bash
ln -s `which touchegg` ~/.kde/Autostart/touchegg
```

## Configuring Touchegg
You can configure Touchegg via tweaking the `~/.config/touchegg.conf` xml file. By default it comes with a very extensive setup. I decided to add 2 tweaks, which should give you an idea on how to customize the configuration.

1. Three finger swipe left and right changes virtual desktops (like on a Mac)

```xml
<application name="All">
        <gesture type="DRAG" fingers="3" direction="RIGHT">
                <action type="CHANGE_DESKTOP">PREVIOUS</action>
        </gesture>
        <gesture type="DRAG" fingers="3" direction="LEFT">
                <action type="CHANGE_DESKTOP">NEXT</action>
        </gesture>
</application>
```

2. Two finger swipe left and right in Konsole and Yakuake changes terminal tab

```xml
<application name="Konsole, Yakuake">
        <gesture type="DRAG" fingers="2" direction="RIGHT">
                <action type="SEND_KEYS">Shift+Right</action>
        </gesture>
        <gesture type="DRAG" fingers="2" direction="LEFT">
                <action type="SEND_KEYS">Shift+Left</action>
        </gesture>
</application>
```

You can find out application names by launching `touchegg` in a terminal, performing the gesture over the application, and see how it's called in the debug output in the terminal window.

It is worth noting that gestures defined in the synaptics driver take precedence over your Touchegg setup. You can chose to rely on those for 2 finger gestures, or disable them in the touchpad configuration dialog and handle them in Touchegg.

You can find on overview of all available [gestures](https://code.google.com/p/touchegg/wiki/AllGestures) and [actions](https://code.google.com/p/touchegg/wiki/AllActions) on the Touchegg site.

## Touchegg on Ubuntu with Unity
Unfortunately using Touchegg with the Unity desktop manager is not as straightforward. Unity assigns actions to multitouch gestures itself. If you like this default setup, great, you won't need Touchegg. If you'd like to reassign the actions linked to the gestures, too bad, you can't change them. You will need to change some code in the Unity source and recompile before Touchegg will be able to do it's thing. There is a [really nice blog post](http://ineed.coffee/1068/os-x-like-multitouch-gestures-for-macbook-pro-running-ubuntu-12-10/) explaining the process.

## Further Reading

* [Disable Unity Default Gestures](http://ineed.coffee/1068/os-x-like-multitouch-gestures-for-macbook-pro-running-ubuntu-12-10/)
* [Touchegg Documentation](https://code.google.com/p/touchegg/wiki/Main)

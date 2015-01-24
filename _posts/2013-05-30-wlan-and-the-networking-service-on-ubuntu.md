---
author: Wouter Dullaert
comments: true
date: 2013-05-30 21:52:06+00:00
layout: post
slug: wlan-and-the-networking-service-on-ubuntu
title: "Wlan and the Networking service on Ubuntu"
quote: "In Soviet Russia, you don't wait for network, network waits for you"
image: /media/2013-05-30-wlan-and-the-networking-service-on-ubuntu/cover.png
wordpress_id: 37
categories:
- Linux
- Networking
- Tips
tags:
- /etc/network/interfaces
- boot
- network-manager
- release upgrade
- Ubuntu
- wlan
---
Because I have a mix of GUI and CLI systems, I configure all my machines through the command line. For networking this means I edit the `/etc/network/interfaces` file. Most of the time, this works just fine, except that after upgrading to the newest Ubuntu release on my laptop, all of a sudden my wireless network wouldn't come up at boot.

In a previous version of this post, I wrongly suggested that removing network-manager would solve the issue. While network-manager is part of the problem, it is also part of the solution.

## The Problem
Using `/etc/network/interfaces` is one of the oldest ways to configure networking on linux. It's straightforward, easy to read and perfectly capable of setting a fix IP on your server. However moving from your home wireless network to the hotspot in your local fancy coffee bar is not so easy. This is one of the reasons network-manager was created. Ubuntu supports network-manager very well.

When I first installed my laptop, the wireless network was working perfectly using `/etc/network/interfaces`. After a release upgrade I started to have occasional problems: the laptop would wait for an IP address that never came. Once the boot completed, running

```bash
sudo ifdown wlan0
sudo ifup wlan0
```

brought up the wireless network and got me an IP address.

Because the problem didn't happen every time I booted, I figured I had a race condition in my boot scripts. While this could potentially be fixed by messing with the boot scripts, I figured that this would cause me more problems when doing future upgrades. I decided it was time to get with the times and configure network-manager.

## The Solution
If you start from a standard Ubuntu installation, you should already have all required network-manager packages installed. In fact, you should have never run into this problem. I started from the minimal install however and installed KDE instead of Unity, so I needed to install some packages by running the following commands:

```bash
sudo apt-get update
sudo apt-get install network-manager-kde
```

This will install Network-Manager and the components to configure it from a KDE desktop.

Next I needed to tell the networking service that Network-Manager will manage my wireless interface. This is as easy as removing or commenting the configuration in `/etc/network/interfaces`. My file looks like this now:

```bash
# The loopback network interface
auto lo
iface lo inet loopback

# The wired network interface
# auto eth0
# iface eth0 inet dhcp

# The primary network interface
# auto wlan0
# iface wlan0 inet dhcp
#    wpa-ssid my-network-ssid
#    wpa-psk my-network-password
#    dns-nameservers 192.168.1.0
```

You can now configure wireless network in the settings of the KDE desktop.

[![New Wireless Connection](/media/2013-05-30-wlan-and-the-networking-service-on-ubuntu/Network-Manager.png)](/media/2013-05-30-wlan-and-the-networking-service-on-ubuntu/Network-Manager.png)

In true linux fashion, it is also possible to [configure Network-Manager using the command-line](http://arstechnica.com/civis/viewtopic.php?t=1163023)

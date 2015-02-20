---
author: Wouter Dullaert
comments: true
date: 2010-04-30 15:54:23+00:00
layout: post
slug: mendeley-and-kde
title: "Mendeley and KDE"
excerpt: "Some builds of Mendeley had a bug on KDE systems though: you couldn't open a pdf in an external viewer, which is rather annoying because the internal one is rather limited."
wordpress_id: 82
categories:
- Tutorial
tags:
- bug
- external
- KDE
- linux
- Mendeley
- pdf
- Qt
- reference software
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

[Mendeley](http://www.mendeley.com) is a very convenient bibliography tool. It keeps track of all your references, you can easily import from tons of sites and it backs up your collection on their servers. It also works on all major operating systems thanks to the Qt library.

Their latest builds had a bug on KDE systems though: you can't open a pdf in an external viewer, which is rather annoying because the internal one is rather limited.

Now it appears that you can solve this bug by remove the Qt library files that come with the mendeley installation. Just add .bak to all files that have "Qt" in their name in the mendeley install folder.
Mendeley will now use your systems Qt libraries and "open in external viewer" will work. As an added benefit Mendeley will now also better integrate in your desktop visualy.

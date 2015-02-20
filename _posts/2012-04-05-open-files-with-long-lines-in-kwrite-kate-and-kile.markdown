---
author: Wouter Dullaert
comments: true
date: 2012-04-05 13:18:10+00:00
layout: post
slug: open-files-with-long-lines-in-kwrite-kate-and-kile
title: "Open Files With Long Lines in KWrite, Kate and Kile"
excerpt: "In a recent KDE upgrade, the katepart text editor, which powers, kwrite, kile and kate, has been modified to open files with long lines as read only.
Sometimes it will tell you about this in a warning, sometimes you'll just notice that you cannot modify the file.
It also turns out that the maximum line length is actually very short: 1024 characters. You will have no trouble finding log files with lines longer than this."
wordpress_id: 106
categories:
- Tutorial
tags:
- fix
- kate
- KDE
- kile
- kwrite
- linux
- long lines
- regression
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

In a recent KDE upgrade, the katepart text editor, which powers, kwrite, kile and kate, has been modified to open files with long lines as read only.
Sometimes it will tell you about this in a warning, sometimes you'll just notice that you cannot modify the file.
It also turns out that the maximum line length is actually very short: 1024 characters. You will have no trouble finding log files with lines longer than this.

After some poking around we've figured out that the maximum line length is configurable:
Go to
`settings` > `configure editor` > `open/save`

There you can change the `line length limit`

The value 0 means that there is no maximum.
Hit `Ok` and you're good to go again.

As a side note: this is a very annoying change. Kate was working fine for us. This new change might fix some bug somewhere, but made the software unusable for our scenario, with absolutely no documentation or hint on how restore the previous behavior. Sometimes you don't even get a warning that the file was loaded read only (many keyboards have suffered because of this)

Edit: Apparently Kile remembers the read only state of a document. It will keep a document read only if it was previously read only, even after you made the change in the settings. You can turn off read only mode for the active mode by deselecting it in the `Tools` menu. Kate and KWrite do have this issue.

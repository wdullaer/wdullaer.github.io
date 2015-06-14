---
layout: post
title: "Firefox Desktop Reader Mode"
quote: "Make the web readable again"
excerpt: "The reader mode in the Android version of Firefox was released a few years ago with a lot of fanfare. In a recent update, the desktop version received the same functionality, but it doesn't seem to be enabled by default on all installations."
image: /media/2015-01-25-Release-An-Android-Library-On-Maven-Central/cover.jpg
video: false
comments: true
categories:
- Tutorial
tags:
- Firefox
- Reader mode
---
The reader mode in Firefox for android was [released](http://www.cnet.com/how-to/how-to-enable-reader-mode-in-firefox-for-android/) to [much](http://techcrunch.com/2012/10/09/firefox-16-new-developer-toolbar-reader-mode/) [attention](http://www.engadget.com/2012/08/30/firefox-16-beta-arrives-with-web-app-hooks-reader-mode-for-android/) in 2012. After 3 years, a similar functionality is being added in Firefox 38 to the desktop version of the browser. Whether the feature is enabled after you upgrade to seems to be a bit random. Fortunately enabling (or disabling if you really want to) the feature is really easy.

In the `about:config`, change the `reader.parse-on-load.enabled` setting to `true`.

![reader.parse-on-load.enabled Setting](/media/2015-06-14-Firefox-Desktop-Reader-Mode/reader.parse-on-load.enabled.png)

On sites which have large blocks of text you will now see a reader mode icon in the address bar.

![Reader Mode Enabled](/media/2015-06-14-Firefox-Desktop-Reader-Mode/reader-mode-enabled.png)

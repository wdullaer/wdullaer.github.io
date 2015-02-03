---
author: zniclomq
comments: true
date: 2013-06-24 20:09:42+00:00
layout: post
slug: mx-player-and-dts-sound
title: MX Player and DTS sound
wordpress_id: 49
categories:
- Android
- Tips
tags:
- Android
- Asus
- Audio
- Codec
- DTS
- ffmeg
- MX Player
- Software Decoding
- Tegra 3
- tf300
- Transformer Pad
---


**MX Player is one of the better video players available for android, but on my Tegra 3 powered tablet it required a bit of coaching before I could get the sound on most of my video files to work, particularly movies encoded with DTS. Fortunately, if you know where to look you can get it all working in a few minutes.**






By default MX Player will try to use hardware decoding for as much files as possible. Theoretically this is good, because it leaves your cpu free to do other things. Unfortunately the tf300 (and I think most Tegra 3 tablets) does not support a lot of audio codecs in audio (no AAC for example). So in the end it is better to use software decoding for audio. The CPU in the Tegra 3 is more than caple to handle audio decoding. You can change to software decoding by clicking the note icon when playing a video. You can also go into _Settings -> Decoder_ and tell MX Player to use the software decoder by default for audio.

[![MX Player S/W Audio](http://www.soundhacker.be/blog/wp-content/uploads/2013/06/Screenshot_2013-06-24-21-34-27-1024x341.jpg)](http://www.soundhacker.be/blog/wp-content/uploads/2013/06/Screenshot_2013-06-24-21-34-27.jpg)
  









The software decoder of MX Player, which is based on the open source ffmpeg, will decode almost all known audio codecs. However it will not decode DTS, which is strange because ffmpeg does indeed support DTS. It turns out that MX Player used to have support for DTS, but was forced to remove it due to licensing issues. The developer of MX Player now gives you the option to use your own custom compiled ffmpeg binaries, with DTS support. Fortunately someone on the internet already did the dirty work for you.





In the decoder section of the MX Player settings you can click a link to **[this thread](http://forum.xda-developers.com/showthread.php?t=2156254)** on xda-developers where you can download a zip with custom ffmpeg binaries for your device (MX Player will tell you which ones you need). You should download the appropriate binaries, for my tf300 this was the Arm v7 with Neon, unzip them and then point MX Player to this folder.
After a restart MX Player will use the custom binaries and your movies with DTS audio will play back as they are supposed to.



[![MX Player Custom Codec](http://www.soundhacker.be/blog/wp-content/uploads/2013/06/Screenshot_2013-06-24-21-34-40.jpg)](http://www.soundhacker.be/blog/wp-content/uploads/2013/06/Screenshot_2013-06-24-21-34-40.jpg)


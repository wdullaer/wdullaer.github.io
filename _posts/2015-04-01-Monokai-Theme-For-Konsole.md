---
layout: post
title: "A Monokai Theme for Konsole"
quote: "Because code should look pretty"
excerpt: "Through the Github Atom editor, I recently discovered the Monokai colour theme. This theme was originally created for the TextMate editor and then converted to Atom.
Because I find it quite pleasing to look at for long periods of time, I decided to give my terminal a similar look."
image: /media/2015-04-01-Monokai-Theme-For-Konsole/cover.jpg
video: false
comments: true
categories:
- Linux
tags:
- Terminal
- Konsole
- KDE
- Monokai
- Theme
---
Through the Github Atom editor, I recently discovered the Monokai colour theme. This theme was [originally](http://www.monokai.nl/blog/2006/07/15/textmate-color-theme/) created for the TextMate editor and then [converted to Atom](https://atom.io/packages/monokai).

Because I find it quite pleasing to look at for long periods of time, I decided to give my terminal a similar look. Here's a colourscheme file for my weapon of choice KDE Konsole:

{% gist wdullaer/c1851e113bf21ccdfeff Monokai.colorscheme %}

You should put this file into `~/.kde/share/apps/konsole` and then select it in the appearance tab of your profile settings.

![Konsole Settings](/media/2015-04-01-Monokai-Theme-For-Konsole/konsole_anim.gif)

Together with the [Adobe Source Code Pro](https://github.com/adobe/Source-Code-Pro) font, Konsole will then look like this:

![Konsole](/media/2015-04-01-Monokai-Theme-For-Konsole/konsole_monokai.png)

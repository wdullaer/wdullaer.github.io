---
author: olezfdtd
comments: true
date: 2010-03-16 14:29:05+00:00
layout: post
slug: extending-latex-memory
title: "Extending latex memory"
excerpt: "Latex is a very powerful typesetting application. A powerful package to be used with this is pgfplots, which allows you to render your plots using the latex engine. Unfortunately the default latex settings assume you're running latex on an ancient 20 MHz 486 with less ram than your average mobile phone. This is ok if you're just asking it to process text, but if you're using pgfplots it will run out of memory trying to draw your images."
wordpress_id: 33
categories:
- Tutorial
tags:
- latex
- linux
- livetex
- memory
- miktex
- pgfplots
- texmf.cnf
- windows
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

Latex is a very powerful typesetting application. A powerful package to be used with this is pgfplots, which allows you to render your plots using the latex engine. Unfortunately the default latex settings assume you're running latex on an ancient 20 MHz 486 with less ram than your average mobile phone. This is ok if you're just asking it to process text, but if you're using pgfplots it will run out of memory trying to draw your images.

The solution is to increase the amount of memory latex uses.

## Windows - Miktex
We're doing this in the windows command line. You can get to this by opening the start menu and do `run ...`. Type `cmd` to start the terminal. On vista and up, you can just type `cmd` in the search field.


1. Go the folder where Miktex is installed. Most likely this is `C:\Program Files\Miktex 2.7\`

2. Go to the subfolder `Miktex`

3. Go to the subfolder `bin`

4. run

    ```bash
    initexmf --edit-config-file=pdflatex
  ```
  Replace `pdflatex` with `latex` or `xetex` if you're using that.

5. You'll get a notepad screen with a file called `pdflatex.ini`. Add the line

    ```
    main_memory=2000000
    ```
  Don't worry about the exact number, just make it big.

6. Save the file

7. run

    ```bash
    initexmf --dump=pdflatex
    ```

8. That's it. Enjoy your nice pgfplots figures

## Linux - Texlive
Do the following things in a terminal as root.

1. Find out where your `texmf.cnf` is located

    ```bash
    kpsewhich texmf.cnf
    ```
  This will most likely be `/usr/share/texmf/web2c/texmf.cnf`

2. Open it and search for main_memory line and modify it to

    ```bash
    main_memory=2000000
    ```

3. Save the file

4. run

    ```bash
    fmtutil-sys --all
    ```
  to load the new settings

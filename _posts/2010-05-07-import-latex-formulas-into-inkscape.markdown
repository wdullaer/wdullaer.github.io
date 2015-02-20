---
author: Wouter Dullaert
comments: true
date: 2010-05-07 16:42:01+00:00
layout: post
slug: import-latex-formulas-into-inkscape
title: "Import latex formulas into inkscape"
excerpt: "Getting nice looking formulas in your presentations / posters is always a problem. Latex is great at producing nice looking formulas and text, but for presentations and posters it's not really equipped. You could however write your formulas in a latex document, then import the pdf into inkscape."
wordpress_id: 85
categories:
- Tutorial
tags:
- acrobat professional
- import
- inkscape
- latex
- pdf
- text to path
- vector image
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

Getting nice looking formulas in your presentations / posters is always a problem. Latex is great at producing nice looking formulas and text, but for presentations and posters it's not really equipped.

You could however write your formulas in a latex document, then import the pdf into inkscape and cut out the formulas. Unfortunately, inkscape will insist on treating your formulas as text, and mess them up in the process.
With a minor sidestep to acrobat professional we can get around this in five easy steps:

1. Open the pdf in acrobat professional.
2. Add a watermark using Document -> Watermark -> add. Set its opacity to 0%
3. Open advanced -> Printing Production -> Flattener Preview
4. Check "convert text to paths" and hit apply
5. Save your pdf

The text has now been converted to a vector drawing, and can now be imported into inkscape without problems.
If you don't add the watermark in acrobat professional, it'll just ignore the "convert text to paths" option (don't ask why). This trick can be used to import any fancy formatted text without running into font problems. (You do lose the ability to edit the actual text though)

Enjoy your nicely formatted formulas in your presentations / posters

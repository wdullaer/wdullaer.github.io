---
author: Wouter Dullaert
comments: true
date: 2012-03-28 14:22:23+00:00
layout: post
slug: precompile-pgfplots-using-tikzexternalize
title: "Precompile pgfplots using tikzexternalize"
excerpt: "Using pgfplots allows you to quickly create very beautiful plots, that look like they belong to your paper, by automatically using the correct fonts and styles. It does this by using the latex compiler to make the figures. This approach is very powerful but has a major drawback: the latex compiler was not made to do this. As a result, latex tends to run out of memory very fast, and projects with a lot of pgfplots tend to take a long time to compile."
wordpress_id: 96
categories:
- Tutorial
tags:
- latex
- livetex
- memory
- miktex
- pdf
- pgf
- pgfplots
- tex
- tikz
- tikzexternalize
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

Using pgfplots allows you to quickly create very beautiful plots, that look like they belong to your paper, by automatically using the correct fonts and styles. It does this by using the latex compiler to make the figures. This approach is very powerful but has a major drawback: the latex compiler was not made to do this. As a result, latex tends to run out of memory very fast, and projects with a lot of pgfplots tend to take a long time to compile.

We've [previously](http://blog.wdullaer.com/2010/03/16/extending-latex-memory/) shown how to increase the amount of memory latex has available, but this only solves half the problem.

A proper way to fix this is using the tikzexternalize command. It causes latex to compile the figures in a separate run, generating pdfs. These pdfs are then included in the main document, reducing both the memory requirements and the compilation time.

Using tikzexternalize is very simple, two step process:

1. Call latex with the "shell-escape" option.
	This looks like this on linux:

	```bash
  pdflatex -shell-escape mydocument.tex
	```

	The easiest way to do this is to modify the build command of your latex-editor of choice.

2. Add the following lines of code to your latex preamble

	```latex
  \usepgfplotslibrary{external}
  \tikzexternalize[prefix=TikzPictures/]
	```

	The first line loads the tikzexternalize library, the second line activates it. The second line also contains an option that specifies the folder where the resulting pdfs and intermediary files should be stored.



Latex will now externalize any tikzpicture you make. You can optionally use

```latex
\tikzsetnextfilename{name_of_resulting_pdf}
```

to specify a name for the resulting pdf file. This is strongly recommended, because _latex will only recompile a figure if you remove the resulting pdf._

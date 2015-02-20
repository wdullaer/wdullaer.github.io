---
author: Wouter Dullaert
comments: true
date: 2010-05-11 12:17:32+00:00
layout: post
slug: fix-matlab-and-latex-file-associations-on-kde
title: "Fix matlab and latex file associations on KDE"
excerpt: "KDE (and linux) in general has a very elaborate system of guessing the file type of your file, that depends on both the file extension and the file contents. In 99% of the cases these rules work, unfortunately the other 1% has been annoying me for quite some time."
wordpress_id: 87
categories:
- Tutorial
tags:
- comment
- file association
- KDE
- latex
- linux
- matlab
- mime
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

KDE (and linux) in general has a very elaborate system of guessing the file type of your file, that depends on both the file extension and the file contents. In 99% of the cases these rules work, unfortunately the other 1% has been annoying me for quite some time.

If you create a matlab m-file that starts with comments (% some comment), KDE will think the file is a latex file and insist on opening it with a dedicated latex editor.
This behaviour can be changed by modifying the mime type definitions.

Open `/usr/share/mime/packages/freedesktop.org.xml` in your favourite text editor as root.
Find the node `<mime-type type="text/x-tex">`. This will have a subnode that looks like:

```xml
<magic priority="10">
   <match value="%" type="string" offset="0"/>
</magic>
```

Change the priority to something smaller than 10, so it has a lower priority than the matlab magic rule.
Now run

```bash
sudo update-mime-database /usr/share/mime
```

And that's it. KDE will now see all your m-files as matlab scripts even if they start with a comment.

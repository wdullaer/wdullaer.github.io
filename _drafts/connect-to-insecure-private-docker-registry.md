---
layout: post
title: "Connect to a Private Docker Repository over HTTP"
quote: "Docker requires a system level tweak to allow you to connect without SSL"
image: /media/2015-04-05-Hack-WD-Greens/cover.jpg
video: false
excerpt: "Docker only allows you to connect to private docker repositories over SSL. However if you quickly set up a registry for dev purposes, configuring SSL is more hassle than it's worth. There is a setting on the Docker daemon to allow unsecured connections to specific private repositories, but it is well hidden."
comments: true
categories:
- Tutorial
tags:
- Docker
- SSL
- Linux
---
In `/etc/default/docker`
```bash
DOCKER_OPTS="--insecure-registry localhost:5000"
```

## Additional reading
<http://docs.docker.com/installation/ubuntulinux>

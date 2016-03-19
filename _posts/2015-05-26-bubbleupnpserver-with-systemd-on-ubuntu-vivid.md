---
layout: post
title: "BubbleUPnP Server with Systemd on Ubuntu Vivid"
quote: "Convert the Service From an Upstart Job to a Systemd Unit"
image: /media/2015-05-26-bubbleupnpserver-with-systemd-on-ubuntu-vivid/cover.jpg
video: false
excerpt: "Starting with Ubuntu 15.04, Canonical replaced their own init system Upstart with the new Linux standard Systemd. Even though this is a big change on a technical level, it was entirely transparent for all packages in the official repositories. BubbleUPnP Server, a closed source third party application, was the only application I had any issues with. I'll show you how you can make it work with systemd."
comments: true
categories:
- Linux
- Tutorial
tags:
- Linux
- BubbleUPnP Server
- Systemd
- Upstart
- Ubuntu
- Ubuntu Vivid
---
For Ubuntu 15.04 Vivid, [Canonical replaced](http://www.omgubuntu.co.uk/2014/02/ubuntu-debian-switching-systemd) their own init system Upstart with the de facto standard in the linux world for this kind of job: Systemd. Even though this is a very big change under the hood, all my upgrades went extremely smooth. None of the official packages had any problems whatsoever. The only problem I had was with a 3rd party application, installed from a ppa: BubbleUPnP server.

[BubbleUPnP server](http://www.bubblesoftapps.com/bubbleupnpserver/) is a companion app to the [BubbleUPnP app on android](https://play.google.com/store/apps/details?id=com.bubblesoft.android.bubbleupnp). It is a completely optional component and provides features like persistent playlists accross various DLNA renderers or transcoding when casting to a chromecast. The authors provide a PPA with Ubuntu packages, but this only ships with an Upstart job. The package will therefore not start as a service or at boot anymore.

I've spent some time converting the upstart script into a systemd service.
Hopefully this will be useful to some people.

## Create a Systemd Service

1. Delete the old Upstart scripts `/etc/init/bubbleupnpserver.conf` and `/etc/init.d/bubbleupnpserver`. They will confuse Systemd

2. Create the file `/usr/share/bubbleupnpserver/startService.sh`
  This script will run before the server starts and make sure it picks up on the configuration specified in `/etc/default/bubbleupnpserver`

    ```bash
    #!/bin/bash

    if [ -f "$DEFAULTFILE" ]; then
            . "$DEFAULTFILE"
    else
            USER=root
            DATADIR=/root/.bubbleupnpserver
            HTTP_PORT=58050
            HTTPS_PORT=58051
    fi

    # Make sure daemon is started with system locale
    if [ -r /etc/default/locale ]; then
            . /etc/default/locale
            export LANG
    fi

    OPTS="-dataDir ${DATADIR} -httpPort ${HTTP_PORT} -httpsPort ${HTTPS_PORT} -nologstdout"
    ```
3. Create the file `/lib/systemd/system/bubbleupnpserver.service`
  This is the actual Systemd Unit describing the service.

    ```ini
    [Unit]
    Description=BubbleUPnP Server
    Requires=network-online.target
    After=network-online.target

    [Service]
    Type=simple
    Environment=DEFAULTFILE=/etc/default/bubbleupnpserver
    Restart=on-failure
    ExecStartPre=/usr/share/bubbleupnpserver/startService.sh
    ExecStart=/usr/share/bubbleupnpserver/launch.sh ${OPTS}

    [Install]
    WantedBy=multi-user.target
    ```

4. Start the server in your current session by running either of the following commands:

    ```bash
    service bubbleupnpserver start
    ```

    ```bash
    systemctl start bubbleupnpserver.service
    ```

5. Register the service to start at boot by running `systemctl enable bubbleupnpserver.service`.


## Improvements

This version ignores the USER variable you can set in `/etc/default/bubbleupnpserver`. You can add that in by changing the `ExecStart` line to:

```bash
ExecStart=/bin/su - ${USER} -c "/usr/share/bubbleupnpserver/launch.sh ${OPTS}"
```

I haven't tested that though.

## Further Reading

* [Ubuntu wiki - Systemd for Upstart users](https://wiki.ubuntu.com/SystemdForUpstartUsers)
* [Archlinux wiki - Systemd Overview](https://wiki.archlinux.org/index.php/Systemd)
* [How To Use Systemctl to Manage Systemd Services and Units](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)
* [Gist with the Systemd Unit files](https://gist.github.com/wdullaer/50418e82727eb08c0ddd)

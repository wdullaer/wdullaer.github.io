I've spent some time converting the upstart script into a systemd service.
Hopefully this will be useful to some people.

Create the file `/usr/share/bubbleupnpserver/startService.sh`

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

Create the file `/lib/systemd/system/bubbleupnpserver.service`

```bash
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
WantedBy=multi-user.install
```

You can now start the server with `service bubbleupnpserver start`. It should also start at boot.

This version ignores the USER variable you can set in `/etc/default/bubbleupnpserver`. You can add that in by changing the ExecStart line to:

```bash
ExecStart=/bin/su - ${USER} -c "/usr/share/bubbleupnpserver/launch.sh ${OPTS}"
```

I haven't tested that though.

I hope this can be merged quickly and easily into the vivid package.

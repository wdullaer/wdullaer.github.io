---
layout: post
title: "Incremental backups to object storage using zbackup"
quote: "Because immutable data make everything better"
image: /media/2015-04-05-Hack-WD-Greens/cover.jpg
video: false
excerpt: ""
comments: true
categories:
- Linux
- Tutorial
tags:
- Linux
- zbackup
- NAS
- S3
- AWS
- Glacier
- Google cloud platform
---
> This post is part of a short series on NAS technology:

> - [Make your WD Greens NAS ready]({% post_url 2015-04-05-Hack-WD-Greens %})
- [A comparison of different RAID and Union file systems]({% post_url 2015-04-28-Comparison-Of-Raid-Like-Systems %})
- How I configured of Snapraid and AUFS

Install zbackup from the repositories

Optionally build the tartool from the zbackup source

Put .backup and .no-backup files everywhere

Install and configure gsutil or aws cli

TODO: configure the S3 bucket or GCP storage bucket

Create zbackup repository by running zbackup init /path/to/my/repo --no-password (or with a password file)

Put following script into /etc/cron.montly (or different folder for different frequency)

```bash
#!/bin/bash

# Load the config file variables
source /etc/zbackup.conf

# Make sure all ${BACKUP_ROOT} and ${BACKUP_TEMP_TARGET} are a directory with trailing slash
if [ -d ${BACKUP_ROOT} ]; then
        echo 'BACKUP_ROOT must be an existing directory'
        exit 10
fi
if [ -d ${BACKUP_TEMP_TARGET} ]; then
        echo 'BACKUP_TEMP_TARGET must be an existing directory'
        exit 10
fi

# Check the backup root directory for content that needs to be included
tartool ${BACKUP_ROOT} ${BACKUP_INCLUDES} ${BACKUP_EXCLUDES}

# Tar everything up and pipe it into zbackup
tar c --files-from ${BACKUP_INCLUDES} --exclude-from ${BACKUP_EXCLUDES} | zbackup backup ${BACKUP_TEMP_TARGET%/}/backups/backup-`date '+%Y-%m-%d'`

# Copy the temporary backup into GCP (or S3 Glacier)
gsutil rsync -d -r ${BACKUP_TEMP_TARGET%/} gs://`hostname`-backup
```

Create proper configuration in /etc/zbackup.conf

Sleep safer at night

## Further Reading
* http://unix.stackexchange.com/questions/175648/use-config-file-for-my-shell-script
* http://zbackup.org

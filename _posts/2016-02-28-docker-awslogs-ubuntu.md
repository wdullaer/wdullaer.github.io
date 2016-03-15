---
layout: post
title: "Pass Credentials to the awslogs Docker Logging Driver on Ubuntu"
quote: "Let's ship some logs"
image: /media/2016-02-28-docker-awslogs-ubuntu/cover.jpg
video: false
excerpt: "Last year docker added support for multiple logging drivers. This makes it very easy to integrate your docker containers with a centralized log management system in a transparent way. If you want to use the AWS Cloudwatch driver you will need to supply the docker daemon with access keys, which proved to be trickier than expected. Here's how I managed to get it running."
comments: true
categories:
- Linux
- Tutorial
- AWS
tags:
- Linux
- Ubuntu
- Docker
- AWS
- Cloudwatch
- Logs
---
Last year docker [added support](https://blog.docker.com/2015/04/docker-release-1-6/) for multiple logging drivers. This makes it very easy to integrate your docker containers with a centralized log management system in a transparent way. For some of my projects I wanted to start moving the logs of my containers into AWS Cloudwatch. In order to do this you need to supply the docker daemon, rather than the docker client, with access keys so that it can write into your account. This proved to be slightly more tricky than I had anticipated, which is why I'm documenting it here.


## Introduction
Docker consists of 2 main components: the docker client that you directly interact with on the cli and the docker daemon or docker engine that actually performs the work. While this design is great and enables a lot of the docker magic behind docker-swarm and docker-machine, both are unfortunately, and confusingly, named docker. On a Ubuntu system the docker daemon runs as a background service.

There are 3 ways to pass AWS credentials to the docker daemon:

1. Via environment variables
2. Via the shared credentials in `~/.aws/credentials`
3. Via the EC2 instance policy if it is running on an AWS EC2 instance

While using the EC2 instance policy is probably the easiest, not all of my servers were running in EC2, so this was not an option for me.

Setting credentials in `~/.aws/credentials` looks easy enough, but for most background services the `$HOME` environment variable is not set. Without this variable the docker daemon won't be able to resolve `~/`. Because specifying a `$HOME` folder for machine accounts / services feels like the wrong thing to do, I'm going to set the AWS credentials as environment variables in the service configuration.

## AWS User Creation
The first thing we need to do is create a user and accompanying access keys for our docker daemon to use. This user should be assigned at least the following policy so it can write events to Cloudwatch.

```json
{
  "version": "2012-10-17",
  "statement": [
    {
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

Ensure the logging group you will be writing to already exists. The logging driver will not create it and the above policy doesn't grant those rights.

## Configure the Docker Daemon
Configuring and scheduling background services is handled by an init system. In the latest LTS release (14.04) Ubuntu is using a system they developed themselves called `upstart`. In later releases they have changed to the more commonly used `systemd`. While the principles behind both systems are the same, their configuration syntax are quite different. I will cover modifying both here.

One thing we don't want to do is modify the actual service definition. These are maintained by your systems package manager. If you allow the package manager to overwrite this file during a package upgrade you will lose your modifications. Alternatively if you decide to keep the file as is, you will not receive any changes and you risk that your version is incompatible with the new package version.

In order to accommodate for this both `upstart` and `systemd` have a concept of overrides. These override service definitions are not maintained by the package manager and are merged into the base service definition when they are run, allowing you to add or override options.

### Upstart (up to and Including Ubuntu 14.10)
Upstart jobs (as background services are called in upstart lingo) are located in `/etc/init/`. This is also the place where you need to create the override file.

```bash
touch /etc/init/docker.override
```

Add the following stanza's to this file:

```bash
env AWS_ACCESS_KEY_ID=<aws_access_key_id>
env AWS_SECRET_ACCESS_KEY=<aws_secret_access_key>
```

You can now restart the docker daemon to make the changes take effect.

```bash
sudo service docker restart
```

### Systemd (Ubuntu 15.04 and Above)
Systemd will look for additional configuration / overrides in `/etc/systemd/system/<servicename>.service.d`.
Create the folder and a configuration file with the following commands:

```bash
mkdir -p /etc/systemd/system/docker.service.d/
touch /etc/systemd/system/docker.service.d/aws-credentials.conf
```

Be aware that if you create the folder, but do not create a config file inside, systemd will disable the service.

We can now add the following lines to `/etc/systemd/system/docker.service.d/aws-credentials.conf`

```ini
[Service]
Environment="AWS_ACCESS_KEY_ID=<aws_access_key_id>"
Environment="AWS_SECRET_ACCESS_KEY=<aws_secret_access_key>"
```

To make systemd aware of the changes, we need to reload the docker service configuration.

```bash
sudo systemctl daemon-reload
```

Now restart the docker service for the changes to take effect.

```bash
sudo service docker restart
```

## Configure the awslogs Driver
Now that the docker daemon has the credentials to write to cloudwatch, we can start using the awslogs driver in our containers.

The easiest way to do this in my opinion is by using docker-compose.
Put this in your `docker-compose.yml` file:

```yaml
web:
  image: busybox
  cmd: echo test
  log_driver: "awslogs"
  log_opt:
    awslogs-region: "eu-west-1"
    awslogs-group: "my-group"
    awslogs-stream: "my-stream"
```

Run the following command to try it out:

```bash
docker-compose up
```

You can also use docker directly via command line parameters:

```bash
docker run --log-driver="awslogs" --log-opt awslogs-region="eu-west-1" --log-opt awslogs-group="my-group" --log-opt awslogs-stream="my-stream" busybox echo test
```

## Some Comments
For veteran Ubuntu sysadmins the contents of this post will most likely be very obvious.
While there is no real technical issue, there definitely is a documentation issue with docker. They should probably make the distinction between the docker client and docker server much more obvious (potentially renaming one of them). It's also not entirely clear how you then go and properly configure the docker server, unless you start it manually on the commandline, which you shouldn't be doing. Now that systemd is almost universally adopted among Linux distributions, it probably wouldn't hurt to have some documentation on how systemd and the docker server work together.

## Further Reading
* [Docker Reference: Logging Overview](https://docs.docker.com/engine/reference/logging/overview)
* [Docker Reference: awslogs driver](https://docs.docker.com/engine/reference/logging/awslogs)
* [Upstart manpage](https://manpages.ubuntu.com/manpages/precise/man5/init.5.html)
* [Serverfault: set environment variables in systemd](https://serverfault.com/questions/413397/how-to-set-environment-variable-in-systemd-service#413408)

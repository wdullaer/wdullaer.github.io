---
author: Wouter Dullaert
comments: true
date: 2010-04-22 13:51:38+00:00
layout: post
slug: install-gitorious-on-your-own-fedora-11-server
title: "Install gitorious on your own Fedora 11 server"
excerpt: "We wanted to have a code repository that can be browsed from the web. We have chosen to install open source gitorious: http://www.gitorious.org. Unfortunately there are no ready made packages, nor is there an extensive amount of documentation on how to get it running."
wordpress_id: 51
categories:
- Tutorial
tags:
- '11'
- activemq
- apache
- fedora
- git
- gitorious
- install
- linux
- local
- mysql
- private
- rails
- ruby
- subfolder
- suburi
---

> This article originally appeared on <http://olezfdtd.wordpress.com>
> I've copied it over to my current blog to consolidate all my blogging efforts over the years in one place.

<blockquote class="message" markdown="1">
At the time gitorious was the best open source github alternative available. Now I would recommend [gitlab](http://gitlab.org).
</blockquote>

We wanted to have a code repository that can be browsed from the web. We have chosen to install open source gitorious: [http://www.gitorious.org](http://www.gitorious.org). Unfortunately there are no ready made packages, nor is there an extensive amount of documentation on how to get it running.

This guide is heavily based on the excellent [tutorial by cjohansen for ubuntu/debian](http://cjohansen.no/en/ruby/setting_up_gitorious_on_your_own_server).
We have just modified it to work with the fedora package manager and the fedora way of configuring/running services.

This is by no means a definitive guide: we didn't start from a blank server and we didn't double check everything, so we're not responsible if your server blows up. These are just the steps we took to get it running on our fedora 11 machine. We already had a working apache and mysql server, so we're not covering those steps. There are plenty of tutorials available online. When in doubt check the cjohansen tutorial or perform some extra googling.

First a small overview of the bits and pieces that you need to get working:

1. A rails application gitorious that serves the website
2. A git daemon that allows people to pull (but not push) code from the repos using the git port
3. A messaging system that performs tasks that would take too long if they were handled by the webserver
4. A poller daemon that actually performs the tasks the messaging system serves
5. A search daemon

It's a very long guide, so here is the table of contents, listing all the steps:

1. [Installing required packages](#installing-required-packages)
2. [ActiveMQ Messaging Server](#activemq-messaging-server)
3. [Memcache](#memcache)
4. [Installing The Gitorious Code](#installing-the-gitorious-code)
5. [Configure the git-daemon and git-ultrasphinx services](#configure-the-git-daemon-and-git-ultrasphinx-services)
6. [Installing Gems And Working Folders](#installing-gems-and-working-folders)
7. [Configuring The Gitorious Script](#configuring-the-gitorious-script)
8. [Configuring The MySQL Database](#configuring-the-mysql-database)
9. [Start All The Services](#start-all-the-services)
10. [Test The Gitorious Server](#test-the-gitorious-server)
11. [Migrate The Gitorious Server To Apache](#migrate-the-gitorious-server-to-apache)
12. [Extra: Running Gitorious In A Sub Folder](#extra-running-gitorious-in-a-sub-folder)
13. [Hints](#hints)

Enough chatter, on to the real stuff. All commands are run as root, unless otherwise mentioned.


## Installing required packages
Begin with installing some required packages
Gitorious is a web based hosting service for git based projects, so obviously, we need git

```bash
yum install git git-svn
```

Some other useful packages. Gitorious reminds you of special events by mail, so you will need sendmail.

```bash
yum install apg pcre pcre-devel sendmail zlib zlib-devel
```

Fedora configures the sendmail service, but doesn't start it automatically, so let's do that now:

```bash
service sendmail start
```

Gitorious is written in ruby.

```bash
yum install ruby ruby-devel ruby-ri ruby-rdoc ruby-irb
```

Some extra dependencies:

```bash
yum install oniguruma-devel libyaml-devel GeoIP-devel
yum install ImageMagick-devel ruby-RMagick
```

Allows gitorious to access a mysql database:

```bash
yum install mysql-devel ruby-mysql
```

Sphinx is the search daemon gitorious uses. Gems is a way to install ruby "packages". It's also the reason why gitorious is so difficult to package for fedora or debian.

```bash
yum install rubygems sphinx
gem install ultrasphinx --no-ri --no-rdoc
```


## ActiveMQ Messaging Server
We need this to run the ActiveMQ messaging server

```bash
yum install java-1.6.0-openjdk
```

Download the ActiveMQ source from the apache website (you can get a newer version if you like by modifying the url). Then extract it into `/usr/local/` to install it.

```bash
cd /tmp
wget http://www.powertech.no/apache/dist/activemq/apache-activemq/5.2.0/apache-activemq-5.2.0-bin.tar.gz
tar xzvf apache-activemq-5.2.0-bin.tar.gz  -C /usr/local/
```

Make the user that will run the ActiveMQ messaging server. Give it ownership over the ActiveMQ data folder, so it can modify the required files.

```bash
adduser --system --no-create-home activemq
chown -R activemq /usr/local/apache-activemq-5.2.0/data
```

Now let us configure the ActiveMQ daemon.

```bash
nano -w /usr/local/apache-activemq-5.2.0/conf/activemq.xml
```

Find the the networkconnectors tag and change it into:

```xml
<!-- The store and forward broker networks ActiveMQ will listen to -->
<networkConnectors>
    <!-- by default just auto discover the other brokers -->
    <!--  -->
    <!-- Example of a static configuration: -->
    <networkConnector name="localhost" uri="static://tcp://127.0.0.1:61616"/>
</networkConnectors>
```

Then, change the management context tag to:

```xml
<!-- Use the following to configure how ActiveMQ is exposed in JMX -->
<managementContext>
    <managementContext createConnector="true"/>
</managementContext>
```

By default ActiveMQ tries to use SSL. This didn't work for us. Edit the `bin/activemq` macro to disable it:

```bash
if [ -z "$SUNJMX" ] ; then
  SUNJMX="-Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
  #SUNJMX="-Dcom.sun.management.jmxremote"
fi
```

Now that ActiveMQ is configured, we need to make it a service that runs in the background and starts up at boot.

Create the scripts that start and stop ActiveMQ:

```bash
cd /usr/local/apache-activemq-5.2.0
touch activemqstart.sh
touch activemqstop.sh
```

Add following to `activemqstart.sh`

```bash
#!/bin/bash
export JAVA_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64
/usr/local/apache-activemq-5.2.0/bin/activemq-admin start &
```

Add following to `activemqstop.sh`

```bash
#!/bin/bash
export JAVA_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64
/usr/local/apache-activemq-5.2.0/bin/activemq-admin stop
```

Make these files executable

```bash
chmod +x activemqstart.sh
chmod +x activemqstop.sh
```

Now create the ActiveMQ service file:

```bash
cd /etc/init.d
touch activemq
```

Add this to activemq:

```bash
#!/bin/bash
#
# activemq       Starts ActiveMQ.
#
#
# chkconfig: 345 88 12
# description: ActiveMQ is a JMS Messaging Queue Server.
### BEGIN INIT INFO
# Provides: $activemq
### END INIT INFO

# Source function library.
. /etc/init.d/functions

[ -f /usr/local/apache-activemq-5.2.0/activemqstart.sh ] || exit 0
[ -f /usr/local/apache-activemq-5.2.0/activemqstop.sh ] || exit 0

RETVAL=0

umask 077

start() {
       echo -n $"Starting ActiveMQ: "
       daemon /usr/local/apache-activemq-5.2.0/activemqstart.sh
       echo
       return $RETVAL
}
stop() {
       echo -n $"Shutting down ActiveMQ: "
       daemon su -c /usr/local/apache-activemq-5.2.0/activemqstop.sh activemq
       echo
       return $RETVAL
}
restart() {
       stop
       start
}
case "$1" in
 start)
       start
       ;;
 stop)
       stop
       ;;
 restart|reload)
       restart
       ;;
 *)
       echo $"Usage: $0 {start|stop|restart}"
       exit 1
esac

exit $?
```

Make this file executable. Add it to the system as a service and then configure it so it starts up at boot:

```bash
chmod +x activemq
chkconfig --add activemq
chkconfig --levels 2345 activemq on
```

ActiveMQ can now be started using:

```bash
service activemq start
```


## Memcache
Install, configure and start the memcache service for better performance:

```bash
yum install memcached
chkconfig --levels 2345 memcached on
service memcached start
```


## Installing The Gitorious Code
Now that the required services are installed and running we can get on with installing the actual gitorious code.

Make a user git and group gitorious. Add the user git to the group gitorious.

```bash
adduser git --disabled-password
groupadd gitorious
usermod -a -G gitorious git
```

Make the folders that will hold the gitorious installation:

```bash
mkdir /var/www/git.myserver.com
chown git:gitorious /var/www/git.myserver.com
chmod -R g+sw /var/www/git.myserver.com
```

Get the latest and greatest gitorious code as the git user:

```bash
su - git
cd /var/www/git.myserver.com
mkdir log
mkdir conf
git clone git://gitorious.org/gitorious/mainline.git gitorious
```

Still as the git user, remove the `.htaccess` file (we'll configure this later ourselves) and make some extra directories:

```bash
cd gitorious/
rm public/.htaccess
mkdir -p tmp/pids
```

Back as root, make sure the gitorious script (which allows users to push code into gitorious) is available as a system command, by symbolic linking it to `/usr/local/bin`
Then make all gitorious scripts executable and configure some extra permissions:

```bash
su root
ln -s /var/www/git.myserver.com/gitorious/script/gitorious /usr/local/bin/gitorious
chmod ug+x script/*
chmod -R g+w config/ log/ public/ tmp/
```


## Configure the git-daemon and git-ultrasphinx services
The next few steps need to be performed as the git user again.
Edit the git-daemon service in `/var/www/git.myserver.com/gitorious/doc/templates/ubuntu/git-daemon`:


```
GIT_DAEMON="/usr/bin/ruby /var/www/git.myserver.com/gitorious/script/git-daemon -d"
PID_FILE=/var/www/git.myserver.com/gitorious/log/git-daemon.pid
```

Then edit the git-daemon script `/var/www/git.myserver.com/gitorious/script/git-daemon`
Find the line that says `host = 0.0.0.0` and replace it with `git.myserver.com `(line 265 in our case). Do not use localhost or the daemon won't listen to external calls.

Now link the `git-daemon` service and the sphinx service to `init.d` (as root)

```bash
ln -s \
  /var/www/git.myserver.com/gitorious/doc/templates/ubuntu/git-ultrasphinx \
  /etc/init.d/git-ultrasphinx
ln -s \
  /var/www/git.myserver.com/gitorious/doc/templates/ubuntu/git-daemon \
  /etc/init.d/git-daemon
```

Make both files executable and configure them to start up on boot:

```bash
chmod +x /etc/init.d/git-ultrasphinx
chmod +x /etc/init.d/git-daemon
chkconfig --levels 2345 git-daemon on
chkconfig --levels 2345 git-ultrasphinx on
```


## Installing Gems And Working Folders
Gitorious uses a ton of gems, so lets install some more of them

```bash
gem install --no-ri --no-rdoc rails mongrel mime-types textpow chronic ruby-hmac daemons mime-types oniguruma BlueCloth ruby-yadis ruby-openid geoip rspec rspec-rails RedCloth echoe
```

Make some working directories the directory that will hold the git repositories hosted in gitorious. These need to be owned by the git user:

```bash
su git
mkdir /var/git
mkdir /var/git/repositories
mkdir /var/git/tarballs
mkdir /var/git/tarball-work
chown -R git:git /var/git
```

Gitorious uses the authorized_keys file from the git user to identify users trying to push code. We need to make sure this file exists and has the right permissions

```bash
su git
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
```


## Configuring The Gitorious Script
Still as the git user: create the configuration files from the supplied templates:

```bash
cd /var/www/git.myserver.com/gitorious
cp config/database.sample.yml config/database.yml
cp config/gitorious.sample.yml config/gitorious.yml
cp config/broker.yml.example config/broker.yml
```

Before we continue a small word about ruby/rails. This language works with the concept of environments. Basically this means that you can use different configurations for different purposes. Gitorious uses 3 configurations: development, testing and production. You can specify different settings for each of these environments. A different database for each environment will make sure you don't mess up your live production database when testing new code. Unfortunately not all gitorious scripts default to running in the same environment, some will run in development others in production, which will result in incomplete or conflicting settings and tons of error messages. It's not unlike user permission madness. We are not interested in hacking on gitorious, just running the system so we will be working on the production environment. To avoid problems always explicitly specify the production environment for each ruby command you run.

Now that the disclaimer is out of the way, we can start editing the config files.
In the `gitorious.yml` we need to store a secret, so lets create one first. Copy it, make sure it's all on one line and keep it somewhere ready to paste:

```
apg -m 64
```

Below we'll print an example `gitorious.yml` file. The original `gitorious.sample.yml` contains explanations about all the various entries. An important line to notice is the `gitorious_client_host` line. We're running this from apache so it says port 80. Later in the tutorial we'll run the production environment from mongrel for testing. This one always runs on port 3000. So this line needs to be 3000 then. Later on when you migrate to apache change it back to 80.

```yaml
production:
  cookie_secret: paste_the_cookie_secret_you_made_here
  repository_base_path: "/var/git/repositories"
  extra_html_head_data:
  system_message:
  gitorious_client_port: 80
  gitorious_client_host: git.myserver.com
  gitorious_host: git.myserver.com
  gitorious_user: git
  exception_notification_emails: youradmin AT yourserver
  mangle_email_addresses: true
  public_mode: false
  locale: en
  archive_cache_dir: "/var/git/tarballs"
  archive_work_dir: "/var/git/tarball-work"
  only_site_admins_can_create_projects: false
  hide_http_clone_urls: false
  is_gitorious_dot_org: false
```


## Configuring The MySQL Database
Edit `config/database.yml` by setting your favourite user names, passwords and database names. Afterwards create mysql stuff you just entered into `config/database.yml`:

```
mysql -uroot -p
> create database gitorious;
> create database gitorious_test;
> create database gitorious_dev;
> grant all privileges on gitorious.* to YOURUSER@localhost \
    identified by 'YOURPASSWORD';
> grant all privileges on gitorious_test.* to YOURUSER@localhost;
> grant all privileges on gitorious_dev.* to YOURUSER@localhost;
```

Once this is done, we can install some remaining gems (which I think autoconfigure themselves using some of the settings we've already made). You need to run this as root from the right folder to work!

```bash
su root
cd /var/www/git.myserver.com
rake gems:install
```

Configure the ruby scripts to work with your mysql configuration (mind your environment settings!):

```bash
su git
cd /var/www/git.myserver.com/gitorious
rake db:migrate RAILS_ENV=production
```

Create the admin user of the gitorious installation on the gitorious console (which is just a front for the mysql console)

```
RAILS_ENV=production script/console
> user = User.first
> user.login = "git" # Change to your desired username
> user.activate
> user.accept_terms
> user.save
```

Other fun commands for the console you might want to use later on are `user = User.find_by_login "user_login"` and `user.destroy` to remove users from the database.

The database is now more or less set up. It's time to configure the sphinx search daemon.
Edit `/var/www/git.myserver.com/gitorious/config/ultrasphinx/default.base` and replace 0.0.0.0 with localhost.

Next load these configurations into the ruby files

```bash
cd /var/www/git.myserver.com/gitorious
rake ultrasphinx:bootstrap RAILS_ENV=production
```

Set the permissions again, just to make sure:

```bash
su root
cd /var/www/git.myserver.com/gitorious
chown -R git:gitorious config/environment.rb script/poller log tmp
chmod -R g+w config/environment.rb script/poller log tmp
chmod ug+x script/poller
```


## Start All The Services
Time to start the activemq server:

```bash
service activemq start
```

Start the git-daemon:

```bash
env RAILS_ENV=production /etc/init.d/git-daemon start
```

Test the poller daemon:

```bash
su git -c "cd /var/www/git.myserver.com/gitorious && env RAILS_ENV=production script/poller run"
```

Let it run for a few minutes. If you see no errors, ctrl+c the command and daemonize the process by running:

```bash
env RAILS_ENV=production script/poller start
```

Gitorious has a habbit of running as much as possible over ssl. This gives us problems with the login, so we're going to disable this on our local install by adding the following to `config/environments/production.rb` (you might want to get this running on larger deployments)

```ruby
SslRequirement.disable_ssl_check = true
```


## Test The Gitorious Server
Everything is now configured. We're going to test our gitorious install by running it in mongrel, a ruby webserver before migrating it to apache. This command always seems to run gitorious on port 3000 no matter what you have specified elsewhere. Like we mentioned before, this means that you have to specify gitorious_client_port 3000 in `config/gitorious.yml` for now.

```bash
su git -c "cd /var/www/git.myserver.com/gitorious && script/server -e production"
```

Open up a browser and surf to http://git.myserver.com:3000 and login using the gitorious admin account we created earlier in the gitorious console.

Gitorious uses your public ssh key to link your push request to a gitorious account. Go to the computer that you'll be using to push code to gitorious. If you don't already have a public ssh key run:

```bash
ssh-keygen
```

to generate one. Now do

```bash
cat .ssh/id_rsa.pub
```

and copy the output. Go to http://git.myserver.com:3000/~adminusernamehere/keys/new (where you substitute adminusernamehere for the admin user you logged in as). Paste the public key into the box and hit ok. Wait a minute or two and then refresh. The key should say "ready". If it doesn't after 2 minutes, delete the current one and try again; for some reason this doesn't always work the first time.

The permissions on .ssh folder of the user uploading to gitorious are extremely important:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
```

Now try to pull a dumy repository to your workmachine and push some content to gitorious.


## Migrate The Gitorious Server To Apache
If all the the previous things work, we can migrate our gitorious install from mongrel to apache.
We need the passenger apache modules to run ruby/rails on apache

```bash
su root
gem install passenger
passenger-install-apache2-module
```

Follow the installers instructions. If some bits and pieces are missing it will tell you. In our case we needed to install httpd-devel first

```bash
yum install httpd-devel
```

The last part is to set up a virtual host in apache to show the gitorious server.

```bash
cd /etc/httpd/conf.d
nano -w gitorious.conf
```

gitorious.conf:

```
NameVirtualHost git.myserver.com

<VirtualHost git.myserver.com>
        ServerName git.myserver.com
        DocumentRoot /var/www/git.myserver.com/gitorious/public
        ErrorLog /var/www/git.myserver.com/log/error.log
        CustomLog /var/www/git.myserver.com/log/access.log combined
</VirtualHost>
```

Restart apache and you're done!

```bash
service httpd restart
```


## Extra: Running Gitorious In A Sub Folder
The tutorial assumes that you will run gitorious on it's own domain: git.myserver.com
In some cases you might want to run it in a subfolder, because you have other services running: myserver.com/git
This isn't officially supported, but with some minor hacks you can easily get it done.

First make a `/etc/httpd/conf.d/gitorious.conf` with just the following content:

```
RailsBaseURI /git
```

This tells passenger that the subfolder git contains a rails app. Otherwise you just get a directory listing.

Next put a symbolic link of the public folder of gitorious into the html folder:

```bash
ln -s /var/www/git.myserver.com/gitorious/public /var/www/html/git
```

Now restart apache

```bash
service httpd restart
```

Normally this should be enough, but since the gitorious supposes you run at the domain root, you need to manually change a few lines.
Open `/var/www/git.myserver.com/gitorious/public/stylesheets/base.css` and replace all `url(/images/whatever.jpg)` with `url("../images/whatever.jpg")` (there are not too many).
Next open `/var/www/git.myserver.com/gitorious/app/helpers/application_helper.rb`
and on line 173 put git/ before images/default.gif (where git is the subdomain you're running gitorious in)

And finally a very very ugly hack: in `gitorious/lib/gitorious/ssh/client.rb` change line 91 to:

```ruby
query_url = "git/#{@project_name}/#{@repository_name}/config"
```

That should make gitorious play nice in a subfolder. These hacks are not very pretty and your install will probably break on an upgrade, so make sure you have a backup ready.

## Hints
* You can restart the gitorious app by doing (so you don't need to restart the entire apache server)

    ```bash
    touch /var/www/git.myserver.com/gitorious/tmp/restart.txt
    ```

* To get the times right in your install, set the timezone in `config/environment.rb` by changing the line

    ```ruby
    config.time_zone = 'UTC'
    ```
  to

    ```ruby
    config.time_zone = 'Your_City_Here'
    ```
  find out the usable values by running

    ```bash
    rake time:zones:local
    ```
  in `/var/www/git.myserver.com/gitorious`

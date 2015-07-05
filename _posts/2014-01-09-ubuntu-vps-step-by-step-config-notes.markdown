---
layout: post
title: "Ubuntu VPS Step-By-Step Configuration Notes"
date: 2014-01-09 14:54
comments: true
categories: [Linux, SSH, Ubuntu, VPS, configuration, firewall]
keywords: Ubuntu, Linux, VPS, install, SSH, firewall, uwf, nginx, PostgreSQL, MySQL, ruby, PHP, node.js, software, redis, erlang, riak
description: UPDATED (June 30th, 2015) - Notes and step-by-step tutorial to configure and secure Ubuntu VPS covering SSH, ufw, sysctl, nginx, node.js, ruby (rbenv), PostgreSQL, MySQL, PHP, Github, erlang, riak....
---

**UPDATED** (June 30th, 2015)

**Changes**:

_June 30th, 2015_

- Covering _Ubuntu 14.04 LTS_.
- Adde extra _SSH_ security, using _Public Keys_ and some more tweaks in `sshd_config`.
- Added usual _PHP_ & _MySQL_ install and configure.
- Using `service <COMMAND> <ACTION>` seems _Ubuntu_ nowdays have problem with `/etc/init.d`, it is old school, but I've liked it. (Thanks to _Rapha_ for mention it in comments).
- Latest Ruby `2.2.2`

_January 9th, 2014_

- First version covering _Ubuntu 13_.


Here are notes and step-by-step tutorial to set _Ubuntu VPS_ secure and with _nginx_, _node.js_, _ruby_, _PostgreSQL_, ...

It should work for _Ubuntu_ 12 or 13 or 14 versions. My _VPS_ provider still doesn't have support _Ubuntu_ 15, but I don't think that there would be any problem using this for _Ubuntu_ 15.

It assumes knowledge of _SSH_ and connecting to your fresh VPS install.

Throughout the install process for editing I’m using _vi_, you can use what ever makes you happy: _emacs_, _nano_, ...

Those are just my notes, if someone has something better or can improve it please do comment!

## Security

### Update and installing essential software

	sudo apt-get -y update
	sudo apt-get -y install curl git-core python-software-properties software-properties-common

### Locale problem

Very annoying error in apt: _locale: Cannot set LC\_CTYPE_. It is probably due to your local machine, add this to your `.bashrc` or `.zshrc`:

    export LANGUAGE=en_US.UTF-8
    export LANG=en_US.UTF-8
    export LC_ALL=en_US.UTF-8

### User - Admin Group & SSH

	sudo groupadd admin
	sudo adduser <USERNAME>
	sudo usermod -a -G admin <USERNAME>
	sudo dpkg-statoverride --update --add root admin 4750 /bin/su

Test new user and test that has `sudo` privileges:

	su <USERNAME>

Secure SSH:

	sudo vi /etc/ssh/sshd_config

Make these changes:

* `Port` <DESIRED_PORT_NUMBER> (Don't use default 22)
* `PermitRootLogin` `no`
* `X11Forwarding` `no`
* `AllowTcpForwarding` `no`
* `AllowUsers` USERNAME USERNAME2

If your SSH is very slow setting `X11Forwarding` to `no` can really help in this case!

	sudo service ssh restart

Connect to _VPS SSH_:

    ssh <USERNAME>@<VPS_DOMAIN_OR_IP> -p <PORT>

### SSH Extra Security

The idea is to not permit passwords and use _SSH_ keys for authentication, and some other tweaks to _SSH_.

    mkdir ~/.ssh
    vi ~/.ssh/authorized_keys

And paste the content of your local machine _Public Key_ file: `.ssh/id_rsa.pub` and make it `rw` only for current user:

    chmod 600 ~/.ssh/authorized_keys

Or use `ssh-copy-id` from your local machine, which isn't on _Mac_, and can be installed with `brew` but I have no luck:

    brew install ssh-copy-id
    ssh-copy-id "<USERNAME>@<VPS_DOMAIN_OR_IP> -p <PORT>"

Try to connect to _SSH_ and it should not ask you for password, it should be instant login:

    ssh <USERNAME>@<VPS_DOMAIN_OR_IP> -p <PORT>

Open _SSH_ config file, uncomment `PasswordAuthentication` which is by default set to `yes`:

    sudo vi /etc/ssh/sshd_config

Change it to `no`:

    PasswordAuthentication no

Restart _SSH_ and try to connect again:

    sudo service ssh restart

Exit the VPS shell and try to connect to VPS _SSH_.

### Firewall

To have `ping` and `apt-get` working, add the following into _iptables_, as _ufw_ just interfaces with it.

    sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

    sudo apt-get -y install ufw
    sudo vi /etc/default/ufw

Make this change:

    IPV6=no

Allow some ports and services:

    sudo ufw allow <PORT_NUMBER_CHANGED_TO_SSH>
    sudo ufw limit ssh
    sudo ufw allow 80/tcp
    sudo ufw allow out 53
    sudo ufw logging on

Be sure that you allowed the changed port for _SSH_ because otherwise, you’ll be unable to get into VPS!!!

    sudo ufw enable

Check the Firewall status with:

    sudo ufw status verbose

### sysctl

    sudo mv /etc/sysctl.conf /etc/sysctl.conf.orig
    sudo vi /etc/sysctl.conf

Add this to a `sysctl.conf`:

    # IP Spoofing protection
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.conf.default.rp_filter = 1

    # Ignore ICMP broadcast requests
    net.ipv4.icmp_echo_ignore_broadcasts = 1

    # Disable source packet routing
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv6.conf.all.accept_source_route = 0
    net.ipv4.conf.default.accept_source_route = 0
    net.ipv6.conf.default.accept_source_route = 0

    # Ignore send redirects
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.default.send_redirects = 0

    # Block SYN attacks
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_max_syn_backlog = 2048
    net.ipv4.tcp_synack_retries = 2
    net.ipv4.tcp_syn_retries = 5

    # Log Martians
    net.ipv4.conf.all.log_martians = 1
    net.ipv4.icmp_ignore_bogus_error_responses = 1

    # Ignore ICMP redirects
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv6.conf.all.accept_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv6.conf.default.accept_redirects = 0

    # Ignore Directed pings
    net.ipv4.icmp_echo_ignore_all = 1

  Then run, there can be possible some errors, especially on _VPS_ with _OpenVZ_...

    sudo sysctl -p

### Denyhost or Fail2Ban

Before _Denyhost_ was uset but it seems it is no longer maintained and there is no Ubuntu package anymore so the _Fail2Ban_ is alternative, but since the SSH is set to permit only _SSH_ keys, both of those are redundant.

### Remove Samba File Sharing

If for some reason you have completely unnecessary _Samba_ installed by default on VPS you can remove it with:

    sudo apt-get -y remove --purge samba

### Remove Bind DNS Server

Same with Bind, if you don’t need it, remove it!

    sudo apt-get -y remove --purge bind9
    sudo rm -rf /var/cache/bind
    sudo rm -rf /usr/share/bind9
    sudo rm -rf /etc/bind

## SSH Key and GitHub

Check if `~/.ssh` has `*.pub` files, if not generate the _SSH_ key:

    ssh-keygen -t rsa -C "<YOUR_EMAIL>"

Use default place to store: `~/.ssh`

Do not use passphrase, unless you want to type every time you commit to _GitHub_. _(Or anyone knows better way?)._

    cat ~/.ssh/id_rsa.pub

Copy the contents to clipboard.
Go to _GitHub_ > Account Settings > SSH Keys.
Hit `Add SSH Key` give title and in `Key` paste public key from clipboard.

    ssh git@github.com
    eval `ssh-agent -s`
    ssh-add -k

## Software Install


### Instal nginx

    sudo add-apt-repository ppa:nginx/stable
    sudo apt-get -y update
    sudo apt-get -y install nginx
    sudo service start nginx

If error is `nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)` then probably _Apache_ is installed beforehand:

    sudo apt-get remove apache2-mpm-prefork
    sudo apt-get remove apache2-mpm-worker
    sudo apt-get remove apache2

You can check and purge installed _Apache_:

    sudo dpkg --get-selections | grep  apache
    # Purge others that are given as result from above command
    sudo dpkg --purge apache2
    sudo service nginx start

If something is wrong look for nginx logs:

    sudo vi /var/log/nginx/error.log

### Install and configure PostgreSQL, create DB and DB User

    sudo add-apt-repository ppa:pitti/postgresql
    sudo apt-get -y update
    sudo apt-get install -y postgresql libpq-dev

Log into `postgres` change main password, create db user with password and create new database with previously created user as owner:

    sudo -u postgres psql
    \password  "<PASSWORD>"
    create user <USERNAME> with password '<USER_PASSWORD>';
    create database <DB_NAME> owner <USERNAME>;
    \q

### Install MySQL, create DB and DB User

    sudo apt-get install mysql-server

The install will prompt you for `root` user password that needs to be re-entered, choose password wisely and try to remember it! After install log into _MySQL_:

    mysql -u root -p # After enter there is need to write password and hit enter again

Use mysql command line app for interacting with _MySQL_, lets create user and DB with user granted on this DB:

    mysql> CREATE DATABASE <DB_NAME>;
    mysql> SHOW DATABASES;  # The new db should be listed.
    mysql> CREATE USER '<DB_USERNAME>'@'localhost';
    mysql> SELECT User,Host FROM mysql.user; # The new user should be listed on host localhost.
    mysql> GRANT ALL ON <DB_NAME>.* to '<DB_USERNAME>'@'localhost' identified by '<DB_PASSWORD>'; # It will grant all on this DB by this user.
    mysql> exit # Or new command \q

If you need to run _SQL_ script for database, use this from command line:

    mysql -u <DB_USERNAME> -p <DB_NAME> < <SCRIPT_NAME>.sql


### Install Ruby with rbenv and rbenv-installer

    cd
    curl https://raw.github.com/fesplugas/rbenv-installer/master/bin/rbenv-installer | bash
    vi .bashrc

At top add:

    export RBENV_ROOT="${HOME}/.rbenv"

    if [ -d "${RBENV_ROOT}" ]; then
      export P
      ATH="${RBENV_ROOT}/bin:${PATH}"
      eval "$(rbenv init -)"
    fi

Reset `.bashrc`:

    . ~/.bashrc

Note that `bootstrap-ubuntu-12-04` works fine for _Ubuntu_ 13 & 14 (probably 15 too):

    rbenv bootstrap-ubuntu-12-04

Firstly list _rbenv_ available _Ruby_ versions and pick one to install:

    rbenv install -l
    rbenv install 2.2.2

Set chosen _Ruby_ version to be "global", and test it:

    rbenv global 2.2.2
    ruby -v

### Install PHP & friends

The long list of _PHP_ packages covering _Imagemagick_, _MySQL_, _bcrypt_,...:

    sudo apt-get install imagemagick php5-fpm php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl php5-xcache

Change usaul suspects of `php.ini` properties:

    sudo vi /etc/php5/fpm/php.ini

* `upload_max_filesize` = (SOME_NUMBER)M (Default 2Mb - safe to use 25M)
* `post_max_size` = (SOME_NUMBER)M (Default 8Mb - safe to use 25M)
* `memory_limit` = (SOME_NUMBER)M (Default 128Mb - safe to use 512M)

Restart _PHP-FPM_ and _Nginx_:

    sudo pkill php5-fpm; service php5-fpm start
    sudo service nginx restart

### Install node.js

    sudo add-apt-repository ppa:chris-lea/node.js
    sudo apt-get -y update
    sudo apt-get install -y nodejs


### Install and Configure redis

    sudo apt-get -y install redis-server
    sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.default

  * `pidfile /var/run/redis-server.pid`
  * `logfile /var/log/redis-server.log`


## Esoteric Installments

### Install erlang

    sudo apt-get -y install erlang-nox

### Install and Configure riak

    curl http://apt.basho.com/gpg/basho.apt.key | sudo apt-key add -
    sudo bash -c "echo deb http://apt.basho.com $(lsb_release -sc) main > /etc/apt/sources.list.d/basho.list"

If _Ubuntu_ is not _LTS_ instead of `lsb_release -sc` use name of _LTS_, eg. _Ubuntu_ 14.10 (_Utopic_) is not LTS use 14.04 codename _trusty_ instead. Use this [Wikipedia table](https://en.wikipedia.org/wiki/List_of_Ubuntu_releases#Table_of_versions) to help with names for particular _Ubuntu_ version(s).

    sudo apt-get -y update
    sudo apt-get -y install riak

#### Change PAM Based Limits for riak

    sudo vi /etc/pam.d/common-session

Add at the end of file:

    session required    pam_limits.so

Update `limits.conf`:

    sudo vi /etc/security/limits.conf

    # Add this to limits.conf
    *               soft     nofile          65536
    *               hard     nofile          65536

If accessing riak nodes via _SSH_:

    sudo vi /etc/ssh/sshd_config
    UseLogin yes

Reboot machine and test that limit of open files is `65536` with:

    ulimit -a
    > open files                      (-n) 65536

or with:

    ulimit -n


Hope it helps, please give feedback and comments with any missing/wrong parts to this
_Ubunt VPS_ install, because I'm far from _Linux_ guru, but appreciate secure and stable _Linux_ machine!
---
layout: post
title: "Installing nginx in Mac OS X Mountain Lion with homebrew"
date: 2012-10-10 03:45
comments: true
categories: [nginx, mac os x, homebrew]
---


## Install with brew

Use `brew` to install the `nginx` with command:

``` bash
$ brew install nginx
```

After install run:

``` bash
$ sudo nginx
```

## Testing

Test it by going to URL:

    http://localhost:8080

## Configuration

The default place of `nginx.conf` on Mac after installing with `brew` is:

    /usr/local/etc/nginx/nginx.conf

### Changing the default port

The nginx default port is 8080, we shall change it to 80. First stop the nginx server if it is running by:

``` bash
$ sudo nginx -s stop
```

Then open `nginx.conf` with:

``` bash
$ vim /usr/local/etc/nginx/nginx.conf
```

and change the:

``` nginx /usr/local/etc/nginx/nginx.conf
server {
    listen       8080;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
```

to:

``` nginx /usr/local/etc/nginx/nginx.conf
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
```

Save configuration and start nginx by

``` bash
$ sudo nginx
```

## Changing the path of defualt web location

The nginx html folder (brew install only) is by the defult in:

    /usr/local/Cellar/nginx/1.2.3/html

**Note**: change `1.2.3` to your nginx version.

The defualt path configuration:

``` nginx /usr/local/etc/nginx/nginx.conf
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
```

To let say `Users/xajler/www`:

``` nginx /usr/local/etc/nginx/nginx.conf
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   /Users/xajler/www;
        index  index.html index.htm;
    }
```

After change stop and start nginix server and nginx is now serving pages from your custom folder!

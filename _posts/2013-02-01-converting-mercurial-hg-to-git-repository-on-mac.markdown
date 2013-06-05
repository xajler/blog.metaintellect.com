---
layout: post
title: "Converting Mercurial (hg) to Git repository on Mac"
date: 2013-02-01 14:16
categories: [git, hg]
comments: true
keywords: Git, Hg, Mercurial, Converting Mercurial to Git, git repository, hg repository.
description: Converting the Mercurial (hg) repository to Git repository with Fast Export on Mac.
---

I had to convert the _Mercurial_ (hg) repository to _Git_ repository, and by lot of advises
on the Internet, I've decided to use [Fast Export](http://repo.or.cz/w/fast-export.git).

## Clone Fast Export

```bash
$ git clone git://repo.or.cz/fast-export.git
```

## Create a new repository

```bash
$ mkdir git_repo
$ cd git_repo
$ git init .
```

## Problems running Fast Export

### Getting to know Python with the Mercurial

By running _Fast Export_ I got error message:

```
'ImportError: No module named mercurial'
```

Essentially this means that _Python_ does not know about _Mercurial_.
The fix for that would be to install _Mercurial_ as a _Python_ module. I've used `easy_install`:

```bash
$ sudo easy_install -U mercurial
```

### Using "force" option

The next error was:

```
'Error: repository has at least one unnamed head...'
```

To fix it there was need to set _Fast Export_ with parameter `--force`.

## Starting Fast Export

Make sure you are in the _Git_ repository that you want to convert to:

```bash
$ cd git_repo
```

Run from it a _Fast Export_ like this:

> *Note*:
>
> Assumes that the _Fast Export_ in directory up and the `hg_repo` is the _Mercurial_
> repository directory from where you want to convert from!

```bash
$ ./../fast-export/hg-fast-export.sh -r ../hg_repo --force
```

After few moments, your _Mercurial (hg)_ repository will be converted to _Git_ one!

And aftrer the conversion is applied run the checkout:

```bash
$ git checkout HEAD
```

Beware if you have a lots of branches, that pushing `master` will only push master. To push evrything to the remote
use:

```bash
$ git push --all origin
```

And now all commits and previous branches are converted to Git!

---
layout: post
title: "Make Powershell, SSH Github and git suck less on Windows"
date: 2012-10-12 03:41
comments: true
categories: [powershell, git, github]
keywords: powershell, git, github, windows, ssh key, posh-git, msysgit
description: Installing the msysgit, configuring git, creating SSH keys for Github, customize the Powershell, installing posh-git. Windows suck less after.
---

There are two Terminals in Windows _Command Prompt_ and _Powershell_, and they both suck by far.
I can understand that _Command Prompt_ is no good, but why the _Powershell_ wasn't done better?!

The things missing in _Powershell_:

* Maximizing is just so mid '90.
* History for a session only (So annoying).
* Painful adding of Aliases.
* Emacs navigation (`Ctrl+a`,`Ctrl+e`,...).
* Full screen and Tranparency (Oh I just want too much).
* The config dir is in `Documents\WindowsPowerShell` (WTF).

Maybe to most Windows users this is strange because this kind of stuff is never used, but if
you're coming from _Linux_ or _Mac_ then the frustration is certain. Because _Linux_ or _Mac_ are
having great Terminal and working in them is just a joy.

The aim of post is to install _git_ on Windowns and then configure it.
Then customize a little bit the _Powershell_ because the defaults are just crime against humanity.
Configure _SSH_ on machine and register _SSH_ key with Github.
Install must-have [posh-git](https://github.com/dahlbyk/posh-git) that will add the branch/status
to _Powershell_ prompt plus auto-completion for _git_.

Note that I'm using Windows 8 and _Powershell_ version 3.0.

## Git Install

For those who might don't know the _git_ is created by Linus Torvalds the creator of _Linux Kernel_.
_Git_ was a  product of his frustration maintaining _Linux Kernel_. He is not really the
huge fan of Windows (nor am I) so _git_ Windows implementation was hard to do because it
really relies on _Unix/Linux_ commands and philosophies that are lacking on Windows.

I know there was a problem I while back with the [official Git version for Windows](http://git-scm.com/downloads)
and I was always using the [msysgit](https://code.google.com/p/msysgit),
don't know if still is the case but I will use _msysgit_ in this post.

Download the latest [msysgit](https://code.google.com/p/msysgit/downloads/list) and install it
with just clicking next few times.  There are few things to configure, but using defaults is safest way.

> Note:
>
> There is also a [Github for Windows](http://windows.github.com/). Probably even easier way to
> install and configure _git_ on Windows, but I like to complicate things.

## Add Git to PATH

By default the _git_ binaries are not set in to PATH, so add it by going to:

    Control Panel/System and Security/System/Advanced system settings

Then in _System Properties_ click on _Environment Variables..._ and in _System Variables_ list box
scroll to _Variable_ `Path`, double-click it and add at the end:

    ;C:\Program Files (x86)\Git\cmd;C:\Program Files (x86)\Git\bin;

Test that the _git_ is available by opening the _Powershell_. Easiest way to open te _Powershell_
(if there is no shortcut) especially in Windows 8 is `Win+r` and type `powershell` to prompt.

In _Powershell_ type:

```
C:\> git
usage: git [--version] [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p|--paginate|--no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           [-c name=value] [--help]
           <command> [<args>]

...
```

If you get something like _usage: git_, then the _git_ is ready!

## Configure Git

Set the user name that will be readable in _git_ log or history:

```text
C:\> git config --global user.name "Kornelije Sajler"
```

Then set your email:

```text
C:\> git config --global user.email "xajler@gmail.com"
```
> Note:
>
> Your email address for Git should be the same one associated with your GitHub account.

## Generate SSH key

With _msysgit_ comes a _Git Bash_ needed to generate _SSH_ keys. If you have one skip this step!

To open _Git Bash_ right-click on any folder in _Windows Explorer_ and choose _Git Bash_.
In _Git Bash_ enter:

```bash
$ ssh-keygen -t rsa -C "xajler@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Kornelije/.ssh/id_rsa):
```
Keygen will ask you for passphrase. In my first attempt I've added one, but on each commit I have to enter
passphrase. That is so annoying.

If you have a _SSH_ key passphrase and it annoys you then enter:

```bash
$ ssh-keygen -p
```
It will ask you for current passphrase, enter the current passphrasse, and with two enters,
you'll now have a blank passphrase!

### Git Bash Copy/Paste

The copy/paste is so awful in _Git Bash_. To paste you need to click the icon in top left corner,
go to `Edit` then `Paste`.

The copy is even more cumbersome, I'll just give you a hint, choose `Select All`!

{% img http://i47.tinypic.com/kei238.png %}

Or read at the end in _Options Tab_ part of _Powershell Customization_ to enable _QuickEdit Mode_.

## Set SSH key to Github

To set the public _SSH_ key in [Github](http://github.com) there is need for getting it from a
`~/.ssh/id_rsa.pub`.

Again open _Git Bash_ right-click on any folder in _Windows Explorer_ and choose _Git Bash_.
In _Git Bash_ enter:

```bash
$ clip < ~/.ssh/id_rsa.pub
```

This command will copy your public _SSH_ key to clipboard. Then go to
[Github / Account Settngs / SSH Keys](https://github.com/settings/ssh) and click the button
`Add SSH Key`.

Enter Title (sorry about my title):

    win-shit

Enter Key:

    Just paste from clipboard

By clicking `Add Key` you have successfully added _SSH_ key to [Github](http://github.com) and
the _git_ pushing to [Github](http://github.com) is now super easy.

## Powershell customization

The visual features of _Powershell_ probably didn't change since Windows 95, and defaults
are probably still dating from '95 and selecting, copy, pasting is awkward, hard and unusable!

### Suck less Powershell

Click the small _Powershell_ icon in top left corner, and in the context menu click on `Properties`.

#### Options Tab

In `Edit Options` check the `QuickEdit Mode`. Quick edit mode enables selecting text from
anywhere in _Powershell_ and with right-click it will copy the selected content.
Also with single right-click pastes the text where the blinking cursor
currently is, similar to _putty_.

This option really boosts the productivity in _Powershell_, it is too bad that this is
not set by default!

#### Font Tab

Even we are in 21st century but the _Powershell_ is still set by default to `Raster Fonts` with
awkward sizes like 16x12, 6x8, that I never really get the meaning of.

In `Font` list choose the _Consolas_ font (or other available mono-space font) and you can check the
`Bold fonts` if you like to have bold text. As for `Size` in list choose whatever you want
I'll stick to `18`.

#### Layout Tab

The _Powershell_ by default is very small, at least to me, maximize is totally unusable, there is
no full screen!

`Screen Buffer Size` and `Window Position` _Width_ height should be same size if you dont want
to have ugly horizontal scroll bar. I set _Width_ to `125` and `Window Position` _Height_ to `35`.

This are all customization, it is not too much but _Powershell_ suck a little less after it,
but there is a room for lots and lots of improvements, while Microsoft spends time on
useless technologies like _Light Switch_.

## Posh-Git: Make your Git shine in Powershell

A set of _Powershell_ scripts which provide _Git/PowerShell_ integration. Includes:

* Prompt for Git repositories - shows the current branch and the state of files (additions, modifications, deletions) within.
* Tab completion for _git_ commands.

### Install

Clone it from _Github_ to any folder, I'll clone it in `source` folder:

```text
C:\source > git clone git://github.com/dahlbyk/posh-git.git
```

Verify execution of _Powershell_ scripts is allowed with:

```text
C:\source > Get-ExecutionPolicy
```

The result should be _RemoteSigned_  or _Unrestricted_.

If scripts are not enabled, run _Powershell_ as Administrator and call:

```text
C:\source > Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Confirm
```

Then `cd` to `posh-git` folder and run:

```text
C:\source\posh-git > .\install.ps1
Creating PowerShell profile...
D:\MyData\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
Adding posh-git to profile...
posh-git successfully installed!
Please reload your profile for the changes to take effect:
    . $PROFILE
```

Then reload your profile, as noted in _posh-git_ after install note:

```text
C:\source\posh-git > . $PROFILE
```

If you're done everything from this post then everything should work just fine!

The outcome of whole post is to have something at the end of the day:

{% img http://i46.tinypic.com/25ian4n.png %}

And just for comparison the _Terminal iTerm2_ on my _Mac OS X Mountain Lion_ with _zsh shell_
and very short aliases, pure awesomeness:

{% img http://i46.tinypic.com/33fggut.png %}

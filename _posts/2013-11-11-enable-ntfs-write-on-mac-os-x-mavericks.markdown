---
layout: post
title: "Enable NTFS write on Mac OS X Mavericks for free and geeky way"
date: 2013-11-11 16:03
comments: true
categories: [Mac, OS X Mavericks, NTFS]
keywords: Mac, Mac OS X Mavericks, NTFS, writable NTFS
description: Enable NTFS write for free on Mac OS X Mavericks, geeky way.
---

By default Mac OS X Mavericks (same goes for older distribution) has _Microsoft_ file system _NTFS_ read-only. There are proprietary software like _Tuxera_ that can enable to write to NTFS. Interesting point is that you don't need to buy it is for free, you just need to geek a bit to make it writable. 

Are you geeky enough?

Surely you guessed it, this will not go without console application in _Mac_ this is _Terminal_, and some of us may us better one _iTerm2_, either way you'll need to know how to open it.

Easiest way is to use _Spotlight_ just hit `Cmd + Space` and write _Terminal_ and here we go...
The other way is in your _Finder_ `Applications > Utilities > Terminal`. (Those who use _iTerm2_, I bet they know how to open it :D)

Before we start on, **MAKE SURE** that your USB stick, external HDD, has single name to it, or better yet without spaces in name! e.g. "`MyPrecious`" is fine, "`My Precious`" is not!  
And what I mean by name is the label name that you get in _Finder_ or _Desktop_ when you plug your device, that is underneath the HDD icon, this is mostly set by manufacture, and if you know how to format on _Windows_, you can also set custom name!

In _Terminal_  create `/etc/fstab` with `nano`, easier for most users, others can use `vim`, `emacs`... (it will ask you for your username password if you have it, write it, if not just hit enter):

```bash
sudo nano /etc/fstab
```

When is created enter this content inside of `/etc/fstab`, be sure that you know the name of your device:

{% img /images/mounted_disk_label_name.png %}

In my case in image is `Elements`, be sure to change below "device-name" to name you got, and there is no spaces in name!!!


```
LABEL=device-name  none    ntfs    rw,auto,nobrowse
```



When you finished entering content, use `Ctrl + X` (it is lowercase x), it will prompt you to save it or not, enter `y`, you'll get another prompt to write to file, just hit enter.  
And that's essentially it, I will not even try to explain what we just did, it will melt brain for most users, just believe me... I know what I'm doing!


The only problem is now, when you plug your device, you're not gonna get a icon on _Desktop_ or _Finder_.  
Mac stores its mounted devices in hidden folder `/Volumes`, so while we have _Terminal_ up and running, make symbolic link of it to the _Desktop_:

```bash
sudo ln -s /Volumes ~/Desktop/Volumes
```

Afterwards unmount your device (right click on device icon and choose `Eject`), sometimes is needed to reboot (restart) your Mac, so it is safe to do so!

Now on your _Desktop_, you'll have a "folder" named `Volumes`, and when you plug your _NTFS_ device, go into this folder, and you'll find there your device armed and ready!  
If you want to save from some other application, when the dialog is open, got to `Desktop > Volumes > [Name of Device]` and save it!

And best thing is, if you are in no need to have support for NTFS anymore, remove `/etc/fstab`:

```bash
sudo rm /etc/fstab
```

... and everything is as before and live happily ever after!

This is how you can have writable _NTFS_ support on your _Mac_, totally for free, and for bonus feeling geeky :)

But nevertheless, if you want to buy the software that does it for you, sure, be my guest... nobody is geek!
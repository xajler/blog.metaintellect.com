---
layout: post
title: "ArchLinux VirtualBox Install Notes"
date: 2013-11-10 14:54
comments: true
categories: [ArchLinux, Linux, installation, VirtualBox]
keywords: ArchLinux, Linux, VirtualBox, ArchLinux installation, ArchLinux configuration, dvorak
description: ArchLinux installation and post installation with installing xorg on VirtualBox Virtual Machine.
---

For a few years now I really wanted to get some time to learn/install and play around with [ArchLinux](https://www.archlinux.org/). _ArchLinux_ is very minimal, highly customizable, but powerful distribution of _Linux_ based on one of oldest distribution _Slackware_ and _BSD_ like Unix.

_ArchLinux_ is not for everybody, if you want everything out-of-box, then probably _Ubuntu_, _Mint_, _Fedora_ or like are for you. But for me, and I have _Mac_, I see no difference between _Ubuntu_ and _Mac_. Because for both, out-of-box you'll get a predefined UI, apps. I'm not going into discussion that most of this can be change in _Ubuntu_, but how much of them really change?

_ArchLinux_ is completely opposite, the install is not really easy (for average PC user), nor GUI like, and after install you are left with bare minimal packages needed for OS to run and configure, and without GUI too.

This installation is not for Linux _n00bs_, some familiarity with console and Linux commands should be expected, because some of notes are really not in depth and there are thing assumed, like basic knowledge of `vim` (or you can replace `vim` with `nano`).  

This is all part of my learning _ArchLinux_, and then to dual boot it on my _Mac_ with _Mac OS X_. And even though I really like _Mac_, I really have huge respect and likeness for _Linux_, and my minimalistic side was in real need of distribution like _ArchLinux_. Why _ArchLinux_?

Just look into their [wiki](https://wiki.archlinux.org/), that much of content and expressiveness (sometimes maybe too much) it is really what it makes _ArchLinux_ great, and second is the [_Arch User Repository_ (_AUR_)](https://aur.archlinux.org/) where you can search for packages, install and comment them (very helpful comments!).

Another strength is that _ArchLinux_ has no release versions, it is rolling released distribution, and if you update weekly or monthly, you'll always have latest versions of your applications and _ArchLinux_ respectively. Another strength (that is for someone is weakness) is that mostly _ArchLinux_ packages are latest stable releases of software, and even this can get you to _dependency hell_, in most cases, you'll end up with latest software with all new features and bug fixes immediately, and not waiting for next version release of OS distribution.

> Note:
>
> I use [Dvorak keyboard layout](http://en.wikipedia.org/wiki/Dvorak_Simplified_Keyboard) and it is throughout the installation, if you use `us` layout ignore all of it, but if you use some other layout e.g. `hr` then replace where `dvorak` to `hr`!

## VirtualBox Pre-installation

* [Download](https://www.virtualbox.org/wiki/Downloads), install and create new _VirtualBox_ machine, for _ArchLinux_ x64.
* Download "dual" ISO from [ArchLinux mirrors](https://www.archlinux.org/download/).
* Add _ISO_ to _VirtualBox_ and set it for CD/DVD.

## Installation

Choose `x86_64` version.  
Change to `Dvorak` layout:

```
loadkeys dvorak
```

Ping _google.com_ to see if there is Internet connection, if not run:

```
dhcpcd
```

Update `pacman` package manager repository:

```
pacman -Sy
```

Install `vim` as a default editor for installation (even though you can use installed plain old `vi` or `nano`):

```
pacman -S vim
```

See what disk structure there is:

```
lsblk
```

For **MBR** installation use `cfdisk`:

```
cfdisk /dev/sda
```

Create new partition with choosing `[ New ]` with 1024 Mb for `boot` directory (not necessary!).

Make it bootable by choosing `[ Bootable ]`.

Create new partition with choosing `[ New ]` with rest of your disk space for `root` folder.

Create filesystems:

```
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2
```

Mount partitions and create `boot` dir, for boot partition:

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```
Change mirror for `pacman`, I use anything in _Germany_, and paste to top of `mirrorlist`:

```
vim /etc/pacman.d/mirrorlist
```

Install base and devel system (this can take for while, so better choose good mirror to be fast as possible!):

```

pacstrap -i /mnt base base-devel
```

Generate `fstab` and check it:

```
genfstab -U -p /mnt >> /mnt/etc/fstab
vim /mnt/etc/fstab
```

### Chroot and configure system

Chroot to `mnt`:

```
arch-chroot /mnt
```

Create custom Keyboard Layout in `vconsole.conf`, and and add keyboard map for _Dvorak_:

```
vim /etc/vconsole.conf

 # add content to it 

"KEYMAP=dvorak"
```

Update with `pacman` and install `vim` (again, omit if you're using `vi` or `nano`):

```
pacman -Sy
pacman -S vim
```

Set up locale, for me uncomment `en_US UTF-8` in `locale.gen`:

```
vim /etc/locale.gen
locale-gen
```

Create `locale.conf` and export locale:

```
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

Set Time Zone by linking to time zone info to `localtime`:

```
 # use to see all zone infos
ls /usr/share/zoneinfo

 # use to see all zone sub infos
ls /usr/share/zoneinfo/Europe

ln -s /usr/share/zoneinfo/Europe/Zagreb /etc/localtime
```

Set Hardware Clock to UTC:

```
hwclock --systohc --utc
```

Set Hostname:

```
echo vbox-arch > /etc/hostname
```

Configure Network:

```
systemctl enable dhcpcd.service
```

### Root passwd and new sudo user

Set `root` password:

```
passwd
```

Create new user that will be sudo:

```
useradd -m -g users -G wheel,video -s /bin/bash <username>
```

Install `sudo` with `pacman`:

```
pacman -S sudo
```

Uncomment `wheel` group from sudoers "`%wheel ALL=(ALL)`", so that user just created can be sudo:

```
visudo
```

Set password to created user:

```
passwd <username>
```

### Bootloader

Since we use **MBR** not **GPT** lets install `GRUB BIOS` bootloader:

```
pacman -S grub-bios
```

Install and configure `GRUB` to `/dev/sda`:

```
grub-install --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

Exit `chroot`, unmount mounted partitions and reboot:

```
exit
umount -R /mnt/boot
umount -R /mnt
reboot
```

## Post Install

Install _Yaourt_ from _archlinuxfr_, for easier interaction/installation from _AUR_.  
Add _archlinuxfr_ repository to `/etc/pacman.conf`:

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

Update `pacman` and install `yaourt`:

```
pacman -Sy yaourt
```

### GUI

Install X server:

```
pacman -S xorg-server xorg-server-utils xorg-xinit
```

Install `mesa` for 3D support

```
pacman -S mesa
```

### VirtualBox

Install Guest additions:

```
pacman -S virtualbox-guest-utils
modprobe -a vboxguest vboxsf vboxvideo
```

Create `virtualbox.conf` in `/etc/modules-load.d`

```
vim /etc/modules-load.d/virtualbox.conf
```

Add modules guest and video to `virtualbox.conf`:

```
vboxguest
vboxsf
vboxvideo
```

Log or `su` as created user.
Copy `.xinitrc` and add following content:

```
cp /etc/X11/xinit/xinitrc ~/.xinitrc
vim ~/.xinitrc

 # add this inside .xinitrc (at the top!)

/usr/bin/VBoxClient-all
```

### Setup keyboard (non US)

We did setup for console now we need to set up for X:

```
localectl set-x11-keymap dvorak
```

### Test X

Install `xterm`, `xclock` and `lwm`:

```
pacman -S xorg-twm xorg-xclock xterm
```

Start X

```
startx
```

{% img /images/arch-alsi.png %}

This is a first part of _ArchLinux_ covering installation, configuration and testing that _X_ or _GUI_ is functioning properly.
Next part is installing and configuring _Window Manager_. And since the _Gnome_ and _KDE_ are most popular, we will not use them at all!
Intention is to use _Tiled Window Manager_...

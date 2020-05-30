# i3	wm - tips and setup
This document describe how to configure Manjaro i3 community edition.

## Power Management
Run `xfce4-power-manager-settings` to display icon, notifications and activate lid closure action. It also allows to parameter brightness reduction, etc. 

## Background lightdm (login screen)
Install `lightdm-settings`, store desired picture with others, select the one.

## Change default browser
Intall `firefox`.

* Replace everything related to 'palemoon' or 'Pale Moon' in .i3/config and .profile
* Then esni procedure (network manager for dns, ipv4-6), options in Firefox and default dns (check cloudflare documentation).

## Terminal transparency
Edit `.Xresources` and search for transparency and replace with:
```
URxvt*inheritPixmap:            true
URxvt*transparent:              true
URxvt*shading:                  20
```

## Better prompt
Edit `.bashrc`:
```
PS1="\[\e[01;36m\]\u\[\e[m\]\[\e[01;32m\]@\[\e[m\]\[\e[01;34m\]\h\[\e[m\]\[\e[01;32m\]:\[\e[m\]\[\e[01;32m\]\w\[\e[m\]\[\e[01;32m\] \[\e[m\]\[\e[01;32m\]\\$\[\e[m\] "
```

## Aliases
Since we are in .bash_rc ..
```
alias ..='cd ..'
alias ll='ls -al'
```

## Install pusle-audio ?
I don't see the immediate need (I mean it works out of the box with `alsa`, but it provides more options), but can be done with `install_pulse`.

## Reinstall non-bugged conky
Uninstall `conky` breaking dependencies:
```
sudo pacman -Rdd conky
```
Then install:
```
conky-git and conky-i3 if not done already
```

## Proper screen resolution and multi-scree
* `xrandr` (console)
* `arandr` (GUI !!)

## Wallpapers
Install `artwork-i3`, `i3-default-artwork`.
To change run `nitrogen`.

## Kernels management
Install `manjaro-settings-manager`.

## Blue light filter
Install `redshift`.
In *.i3/config*:
```
exec --no-startup-id redshift
```

## Change machine name
Because you don't always have inspiration at install time...
```
hostnamectl set-hostname mi3
```
## Install Bluetooth stack
[Bluetooth](https://wiki.archlinux.org/index.php/Bluetooth)

Install `bluez` + `bluez-utils` + `blueberry`.

```
systemctl start bluetooth.service
systemctl enable bluetooth.service
```

Edit *.i3/config*: (system tray icon)
```
exec --no-startup-id blueberry-tray
```

## Resize with mouse
Just <mod>+right-click to resize.

## Status bar (i3status)
Default config path : */etc/i3status.conf*

Copy it to *~/.config/i3status/config* and then update (see file in the repo).

## Replace dmenu by rofi
Test was not conclusive to me without a lot of fine tuning and theming. I prefer dmemu for now.

Install `rofi`.

Edit *.i3/config*:
``` 
bindsym $mod+d exec --no-startup-id rofi -show drun
```
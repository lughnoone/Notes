# FreeBSD Install in X230 + Tips
A few tips to setup FreeBSD at the beginning ... now more like an install procedure on a X230.

## Intall FreeBSD on a X230 laptop

### Full disk encryption install
After selecting auto partitioning ZFS, select encryption options.
More details [here](https://freebsdfoundation.org/wp-content/uploads/2019/11/Configuring-Full-Disk-Encryption.pdf) .

### Hardening FreeBSD
Select all available hardening options, especially ASLR (Adress Space Layout Randomization) or Adress Map Randomization.

### Install `sudo`
```
su
pkg install sudo
visudo
```
In the editor uncomment the ligne authorizing all users part of `wheel` groups as sudoers.

Then give the wheel group to expected user:
```
pw usermod username -G wheel
```

where `username` is the username of the user.

### Install `slim` + `kde5`
```
pkg install xorg graphics/drm-kmod kde5 slim
```

Enable intel graphic drivers + dbus + slim in `/etc/rc.conf`
```
dbus_enable="YES"
slime_enable="YES"
kld_list="i915kms"
```

Next, as normal user, create the file `~/.xinitrc`
Its content must be the path to `startplasma-x11`:
```
exec /usr/local/bin/startplasma-x11
```

Finally lets install `slim` themes to make it a bit more FreeBSD like:
```
pkg install x11-themes/slim-themes
nano /usr/local/etc/slim.conf
```

at the bottom change `current_theme` to:
```
daemon yes
current_theme fbsd
```

Make sure your user has also the `video` group:
```
pw usermode username -G wheel video
```

Type `slim` to test you are able to login and access Kde.

### Wifi configuration

I struggled a bit to find the proper config to finally see the networks at install time.
Here they are afterwards in `/etc/rc.conf`:

```
ifconfig_wlan0="WPA DHCP"
ifconfig_wlan0_ipv6="inet6 accept_rtadv"
create_args_wlan0="country F2 regdomain APAC2"
```

Once the wifi has been activated at install time, you may want to connect to another network.
To do so just run `sudo bsdconfig wireless` - [link](https://www.freebsd.org/cgi/man.cgi?query=bsdconfig&sektion=8)

## FreeBSD in Virtualbox
### Install `slim` + `Xfce4`
```
pkg install xorg xfce slim
```

Next, as normal user, create the file `~/.xinitrc`
Its content must be the path to `startxfce4`:
```
exec /path/to/startxfce4
```

Make sure your user has also the `video` group:
```
pw usermode username -G wheel video
```
Type `slim` to test you are able to login and access Xfce.
If fine, you can run `slim` at boot time:
```
sysrc dbus_enable="YES"
sysrc slim-enable="YES"
```
The system is veru minimalistic. You may want to install `pulseaudio` Xfce related packages to enable the sound, install `Firefox`, and needed applications.

This being said, a simpler alternative is to install `GhostBSD` that is a desktop oriented FreeBSD based OS.

### Install `virtualbox` guest addition
```
pkg search virtualbox
pkg install _virtualbox guest addition package_
```

After install the package install output provides 2 services to be enabled via `sysrc`. Run both commands.

Finally reboot.

## Tips

### Update system

Get FreeBSD latest patchs.

```
freebsd-update fetch
freebsd-update install
```

### Bash + completion

If you like bash better than sh/csh (zsh is still the best :-p)

```
pkg install bash bash-completion
```

### Proper date time formating

In KDE, go to settings, regional settings, Format and change Time settings by selecting `en_SE`. It allows to remove the seconds fronm the short format, which make the lock screen better without those running seconds.

### Avoid changing DNS after DHCP @ attribution

Set your DNS as wanted: (here Cloudflare)

```
nano /etc/resolv.conf

nameserver 1.1.1.1
nameserver 1.0.0.1
nameserver 2606:4700:4700::1111
nameserver 2606:4700:4700::1001
```

then

```
chflags schg /etc/resolv.conf
```

### Network Manager

To simplify network connections, especially for wifi, a network manager is welcome.

```
pkg install net-mgmt/networkmgr
```

If not started after reboot, run as root `networkmgr`. See above for `bsdconfig wireless` that also handle wifi selection, connection.

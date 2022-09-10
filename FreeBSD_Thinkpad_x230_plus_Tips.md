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

### Webcam

Stolen from [David Schlachter](https://www.davidschlachter.com/misc/freebsd-webcam-browser).

```
pkg install webcamd
sysrc webcamd_enable="YES"
sysrc kld_list+="cuse"
```

Then reboot or:

```
kldload cuse
service webcamd start
```

To test the webcam:

```
pkg install pwcview
pwcview -s sqcif -f 25 -d /dev/video0
```

Finally to allow the wecam in browser (FF):

```
pkg install v4l-utils v4l_compat
```

You can test on any webside using camera, like any optician.

### Bluetooth (headphones)

Inspired from [this](https://jrgsystems.com/posts/2022-08-20-how-i-configure-bluetooth-headphones-on-freebsd-13-1/) and [this](https://jrgsystems.com/posts/2022-09-06-blued/)

Probably not needed if your Bluetooth has been detected, at least for me it was already activated in the kernel.
(driver depends on your device)

```
su
kldload ng_ubt
echo 'ng_ubt_load="YES"' >> /boot/loader.conf
kldload cuse
sysrc kld_list+="cuse"
cp /etc/defaults/bluetooth.device.conf /etc/bluetooth/ubt0.conf
sysrc hcsecd_enable="YES"
sysrc bluetooth_enable="YES"
```

Get the device in pairing mode and get the adress:

```
hccontrol -n ubt0hci inquiry

Inquiry result, num_responses=1
Inquiry result #0
        BD_ADDR: f8:9e:94:ee:99:c3
[...]
```

Edit `/etc/bluetooth/hcsecd.conf` to add:

```
device {
    bdaddr  11:11:22:38:7a:85;
    name    "headphones";
    key     nokey;
    pin     "0000";
}
```

Edit `/etc/bluetooth/hosts`:

```
11:11:22:38:7a:85 headphones
```

and restart `hcsecd`:

```
service hcsecd restart
```

Finally to pair the earphones:

```
service hcsecd onestart
service bluetooth start ubt0
hccontrol -n ubt0hci create_connection headphones
hccontrol -n ubt0hci write_authentication_enable 1
virtual_oss -T /dev/sndstat -S -a o,-4 -C 2 -c 2 -r 44100 -b 16 -s 1024 -R /dev/dsp0 -P /dev/bluetooth/headphones -d dsp -t vdsp.ctl
```

Usefull commands:

```
hccontrol -n ubt0hci inquiry # get devices list
hccontrol -n ubt0hci remote_name_request 11:11:22:38:7a:85 # get device name
hccontrol -n ubt0hci create_connection 11:11:22:38:7a:85 # connect to a device using address
hccontrol -n ubt0hci read_connection_list # list connected devices
```

To get an OSS visual sound controller and so tune the volume, just install:

```
sudo pkg install virtual_oss_ctl
```

To switch Firefox to OSS and so get the sound to bluetooth headphones, open `about:config` and add:

```
media.cubeb.backend oss
```

Script to automate connection:

```
#!/bin/sh

set -x
set -e

service hcsecd stop
service bluetooth stop ubt0
service hcsecd onestart
service bluetooth start ubt0
hccontrol -n ubt0hci create_connection headphones
hccontrol -n ubt0hci write_authentication_enable 1
virtual_oss -T /dev/sndstat -S -a o,-4 -C 2 -c 2 -r 44100 -b 16 -s 1024 -R /dev/dsp0 -P /dev/bluetooth/headphones -d dsp -t vdsp.ctl&
```

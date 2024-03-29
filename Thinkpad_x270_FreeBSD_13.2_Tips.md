## Add groups to user

```
pw usermod toto -G wheel,video
```

## Switch pkg from quaterly to lates

```
mkdir -p /usr/local/etc/pkg/repos
cp /etc/pkg/FreeBSD.conf /usr/local/etc/pkg/repos/FreeBSD.conf
```

replace url with:

```
url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest",
```

## Change shell to bash

```
pkg install bash
chsh -s /usr/local/bin/bash user
```

## sudo

Install sudo (seems more effective than doas):

```
pkg install sudo
```

Edit `/usr/local/etc/sudoers` to allow wheel group or users

## Install graphical UI

Install X + Intel graphic drivers: [https://docs.freebsd.org/en/books/handbook/x11/#x11]

Install KDE (minimal is too minimal) + sddm : [https://docs.freebsd.org/en/books/handbook/desktop/]

## How to change sddm background

Edit `/usr/local/share/sddm/themes/breeze/theme.conf`

## How add power buttons to KDE

Create `/usr/local/etc/polkit-1/rules.d/40-wheel-group.rules`

and add:

```
polkit.addRule(function(action, subject) {
    if (subject.isInGroup("wheel")) {
        return polkit.Result.YES;
    }
});
```

## How to change timer of boot loader

Edit `/boot/loader.conf` and add:

```
autoboot_delay="3" for 3 secs
```

## How to freeze DHCP in resolv.conf

```
sudo chflags schg /etc/resolv.conf
```

values: (cloudlfare)

```
nameserver 2606:4700:4700::1111
nameserver 2606:4700:4700::1001
nameserver 1.1.1.1
nameserver 1.0.0.1
```

## How to setup wifi

Add in `/etc/rc.conf`

```
wlans_iwm0="wlan0"
ifconfig_wlan0="mode 11g country FR regdomain ETSI WPA SYNCDHCP"
ifconfig_wlan0_ipv6="inet6 accept_rtadv"
```

The `mode 11g` is used to select 11g compatible device, because currently `iwm` is not compatible with 802.11n/ac (see [iwm man page](https://man.freebsd.org/cgi/man.cgi?query=iwm&apropos=0&sektion=4&manpath=FreeBSD+14.0-CURRENT&arch=default&format=html)).

Edit `/etc/wpa_supplicant.conf`

```
network={
ssid="SSID_NAME"
psk="PASSWORD"
}
```

Install GhostBSD network manager

```sudo pkg install networkmgr```

then allow to run the app with `sudo` without password. Run `sudo visudo` and add:

```
yourusernamepartofwheelgroup ALL = NOPASSWD: /usr/local/bin/networkmgr
```

## Graphic pkg installer

doas pkg install octopkg

## Deactivate file indexing (KDE Baloo)

As baloo is crasing at each startup ... lets disable it.

`doas balooctl disable`

## Brightness keys

Add `acpi_video` to `kld_list` in /etc/rc.conf

(see: [https://wiki.freebsd.org/Laptops/Thinkpad_X270])

It allows brightness handling in KDE power management.

## Webcam

Stolen from [David Schlachter](https://www.davidschlachter.com/misc/freebsd-webcam-browser).

```
pkg install webcamd
sysrc webcamd_enable="YES"
sysrc kld_list+="cuse"
pw groupmod webcamd -m $USER
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

## Remove boot outputs

Set `boot_mute="YES"` in `/boot/loader.conf`.

## Virtual Box install

```
pkg install virtualbox-ose
```

Follow provided instruction to load kernel module, add group to user, network bridge, usb, etc.

Load `vboxdrv` kernel module in _/boot/loader.conf_:

```
vboxdrv_load="YES"
```

add user to  _vboxusers_ group:

```
pw groupmod vboxusers -m mysuer
```

Change permissions in `/etc/devfs.conf`:

```
own     vboxnetctl root:vboxusers
perm    vboxnetctl 0660
```

Setup network in _/etc/rc.conf_:

```
vboxnet_enable="YES"
```

+ (see USB setup in provided procedure)

After reboot, in case you have a mismatch error with the kernel module and the version of the kernel, you might need to rebuild it to match kernel version.

```
cd /usr/ports/emulators/virtualbox-ose-kmod
make
make reinstall
```

## SD card reader

```
pkg install rtsx
```

## Update FreeBSD

```
freebsd-update fetch
freebsd-update install
```

Reboot, then run `cat /etc/os-release` to ensure you are on the new version.

```
pkg update
pkg upgrade
```

## Upgrade FreeBSD

Identifiy the release to upgrade to. Then run:

```
freebsd-update -r 13.2-RELEASE upgrade
freebsd-update install
```

Reboot and install remaning post install updates.

```
freebsd-update install
```

## rc.conf

```
clear_tmp_enable="YES"
syslogd_flags="-ss"
sendmail_enable="NONE"
hostname="machine_name"
ifconfig_em0="DHCP"
ifconfig_em0_ipv6="inet6 accept_rtadv"
sshd_enable="YES"
ntpd_enable="YES"
powerd_enable="YES"
powerd_flags="-a hiadaptive -b adaptive"
performance_cx_lowest="C2"
#powerdxx_enable="YES"
# Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
dumpdev="NO"
zfs_enable="YES"
wlans_iwm0="wlan0"
ifconfig_wlan0="mode 11g country DE regdomain ETSI WPA SYNCDHCP powersave"
ifconfig_wlan0_ipv6="inet6 accept_rtadv"
kld_list="i915kms acpi_video cuse acpi_ibm coretemp"
dbus_enable="YES"
sddm_enable="YES"
webcamd_enable="YES"
#vbox
vboxnet_enable="YES"
devfs_system_ruleset="system"

#     While iwlwifi supports all        802.11 a/b/g/n/ac/ax the compatibility code
#     currently only supports 802.11 a/b/g modes.  Support for 802.11 n/ac is
#     to        come. 802.11ax and 6Ghz support are planned.
#devmatch_blocklist="if_iwm"
#wlans_iwlwifi0="wlan1"
#ifconfig_wlan1="country DE regdomain ETSI WPA SYNCDHCP"
#ifconfig_wlan1_ipv6="inet6 accept_rtadv"
```

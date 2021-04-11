# Install Manjaro Linux 21 on a Thinkpad x270
This document aims at gathering the few info needed to install Manjaro Linux on a Thinkpad x270.
It may also include some customization tips.

## Install

### Bios
Deactivation of secure boot, allow alternate OS to be booted, allow two type of boot (MBR+UEFI), move up USB in the boot sequence.
Else impossible to boot linux on a USB drive.

### Partitioning

#### Option 1

Manual partitioning allows to create one root partition (/) and one encrypted home partition (/home) with LUKS.
(one encrypted partition only causes quite some trouble, especially for suspend/hibernation)

#### Option 2

- FAT32 UEFI (/boot/uefi, flag boot) 300Mio
- btrfs / partition 70Gio encrypted LUKS
- linuxswap 8.9Gio encrypted LUKS
- btrfs /home (rest of available space) encrypted LUKS

btrfs is the replacement for ext4, resizable on demand and easy to backup with Timeshift.

Here everything is encrypted, but still we can setup the disk encryption on top.


### Opal SED

To cope with this issue, it is possible to use SED SSD built-in hardware encryption.

See: https://www.happycoders.eu/manjaro-tutorial-installing-manjaro-linux-dell-xps-15-9570/ and https://github.com/Drive-Trust-Alliance/sedutil/wiki/Encrypting-your-drive

```
gunzip RESCUE64.img.gz
```
and load the image on a USB drive (used for recovery) (win32 imager, dd, rufus, etc.)
Boot on the usb key and login as `root`

```
sedutil-cli --scan
linuxpba
(pwd debug, else it reboot)
(should return `is OPAL NOT LOCKED`)

sedutil-cli --initialsetup debug /dev/sda
sedutil-cli --enablelockingrange 0 debug /dev/sda
sedutil-cli --setlockingrange 0 lk debug /dev/sda
(it's LK not 1K)
sedutil-cli --setmbrdone off debug /dev/sda
gunzip /usr/sedutil/UEFI64-n.nn.img.gz 
(Replace n.nn with the release number)
sedutil-cli --loadpbaimage debug /usr/sedutil/UEFI64-n.nn.img /dev/sda
(Replace n.nn with the release number.)

linuxpba
(pwd debug, else it reboot)
(should return `is OPAL Unlocked`)

sedutil-cli --setsidpassword debug yourrealpassword /dev/sda
sedutil-cli --setadmin1pwd debug yourrealpassword /dev/sda

(test it works)
sedutil-cli --setmbrdone on yourrealpassword /dev/sda
```

in case of error / issue to disable locking

```
sedutil-cli -–disableLockingRange 0 <password> <drive>  
sedutil-cli –-setMBREnable off <password> <drive>
```

to reactivate
```
sedutil-cli -–enableLockingRange 0 <password> <drive>      
sedutil-cli –-setMBREnable on <password> <drive>  
```

to remove OPAL
```
sedutil-cli --revertnoerase <password> <drive>
sedutil-cli --reverttper <password> <drive> 
```

### Battery
(From: https://www.valhalla.fr/2018/12/02/manjaro-thinkpad-checklist/)

Install '''tp_smapi''' and '''acpi_call''' for your kernel.
Also install '''tlpui''.

### CPU scaling and audio powersaving
(From: https://www.valhalla.fr/2018/12/02/manjaro-thinkpad-checklist/ and https://wiki.archlinux.org/index.php/Power_management)

Create and edit ```/etc/tmpfiles.d/energy_performance_preference.conf```
add:
```
w /sys/devices/system/cpu/cpufreq/policy?/energy_performance_preference - - - - balance_power
```

For audio powersavings, create and edit ```/etc/modprobe.d/audio_powersave.conf```
add:
```
options snd_hda_intel power_save=1
```


## Display

### Scaling
Because the screen resolution is 1920x1080 on a 12"5 screen, except if you have hawk eyes, you need to increase the font size to ~125%.

In a terminal:
```
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

Then right click on desktop, display settings and select the 125% scaling.

## Customization tips

### Arc Menu
Instead of the default Gnome application menu you can activate the default Arch Menu.

To do so, run the ```Extension``` appplication and activate Arch Menu

### Transparent top bar and OSD
To have a transparent Gnome top bar & OSD (volume, brightness popup).

First you need to install two extensions from AUR:

```
gnome-shell-extension-transparent-osd-git
gnome-shell-extension-transparent-top-bar-git
```

Then you have to logout / login to restart Gnome.
Then run the ```Extension``` application to activate both extensions

By default the top bar is setup with a transparency of 0.35 (35%).
To change that value (to 0), edit 

```
sudo nano /usr/share/gnome-shell/extensions/transparent-top-bar@zhanghai.me/stylesheet.css
```

and change

```
background-color: rgba(0, 0, 0, 0.35);
```

to

```
background-color: rgba(0, 0, 0, 0);
```

### Transparent Dock
To set the dock transparency, run the ```Extension``` application and adapt the transparency in ```Dash to Dock``` extension settings.

## Not working
For now the finger print reader ... but expected at this stage. To be investigated.

Potential issue (not tested yet) with audio via HDMI.

For both issues, check decicated Arch page below.

## Other
### Links
* Dedicated Arch page: https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X270
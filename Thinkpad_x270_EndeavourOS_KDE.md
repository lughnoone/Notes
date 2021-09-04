# Install EndeavourOS 2021.08.27 on a Thinkpad x270
This document aims at gathering the few info needed to install EndeavourOS on a Thinkpad x270.
It may also include some customization tips.

## Install

### Bios
Deactivation of secure boot, allow alternate OS to be booted, allow two type of boot (MBR+UEFI), move up USB in the boot sequence.
Else impossible to boot linux on a USB drive.

### Partitioning

Default full disk partitioning is fine, with Btrfs and encryption (LUKS).

For hibernation look joined _"Configure Hibernation with LUKS and Btrfs not Breaking Timeshift Snapshots (Arch based system, EndeavourOS)"_ tutorial.

Btrfs is the replacement for ext4, resizable on demand and easy to backup with Timeshift.

Here everything is encrypted, but still we can setup the disk encryption on top.


### Opal SED

With an OPAL drive, it is possible to use SED SSD built-in hardware encryption.

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

**WARNING, it seems some disks, like crucial MX500 do not support disabling OPAL, so you CANNOT decrypt the ssd once activated.

Only way is to do a PSID reset (all data lost) with Crucial Storage Executive (you need the PSID that is writen on the disk itself)**

[https://www.crucial.com/support/storage-executive](https://www.crucial.com/support/storage-executive)
[https://www.crucial.com/support/articles-faq-ssd/overview-crucial-storage-executive](https://www.crucial.com/support/articles-faq-ssd/overview-crucial-storage-executive)

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

Install '''acpi_call''' for your kernel.
Also install '''tlpui''. All setup can be done via the UI.

It seems '''tp_smapi''' is not compatible for x230 and above and lead to kernel error at boot.

### CPU scaling and audio powersaving
***Note: probably useless, since everything can be handled by ```tlpui```***

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

### Fan control
(source: [https://wiki.archlinux.org/title/Fan_speed_control](https://wiki.archlinux.org/title/Fan_speed_control))

Installing ```thinkfan```and ```thinkfan-ui``` seems overrated.

Let's start with detecting the sensors:

```
sudo sensors-detect
```

Just type enter to select the default value and update the configuration file.

### Network
Open KDE settings -> Connections.

For wired and wireless connection, in IPv4 and IPv6 tabs, select method "Automatic (Only Adresses)" and specify the DNSs:

```
1.0.0.0,1.1.1.1
2606:4700:4700::1111,2606:4700:4700::1001
```

### Bluetooth
Bluetooth is not enabled by default because of security risks ...

```
sudo pacman -S --needed bluez bluez-utils pulseaudio-bluetooth
sudo systemctl enable --now bluetooth
```

Then install a UI to manage it: ```bluedevil``` for KDE.

Then go to settings and select _bluetooth_.

See [https://discovery.endeavouros.com/bluetooth/bluetooth/2021/03/](https://discovery.endeavouros.com/bluetooth/bluetooth/2021/03/)

## Display

### Scaling
Because the screen resolution is 1920x1080 on a 12"5 screen, except if you have hawk eyes, you need to increase the font size to ~125%.

This can be done in KDE display settings, and needs to restart the sessions to be taken into account.

### HDMI
Works out of the box, just select the proper audio output in audio settings in case the sound keeps coming from the computer speakers.

## Not working
For now the finger print reader ... but expected at this stage. To be investigated.
* https://github.com/3v1n0/libfprint
* https://snapcraft.io/validity-sensors-tools
```
sudo snap connect validity-sensors-tools:raw-usb 
sudo snap connect validity-sensors-tools:hardware-observe
cd /
sudo validity-sensors-tools.initializer
```
but it fails to upload the firmware

Potential issue (not tested yet) with audio via HDMI.

For both issues, check decicated Arch page below.

## Customization
### Bash

To tweak a bit the default ugly bash prompt, install ```nerd-fonts-hack``` which extends _hack_ font used by Konsole with fancy icons.

Then install _trueline_ bash powerline [https://github.com/petobens/trueline](https://github.com/petobens/trueline):

```
git clone https://github.com/petobens/trueline ~/trueline
echo 'source ~/trueline/trueline.sh' >> ~/.bashrc
```

## Other
### Links
* Dedicated Arch page: https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X270
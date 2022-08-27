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

#### Recover grub for a encrypted Btrfs filesystem


Just in case it can help others, here is what I did to solve [this](https://forum.endeavouros.com/t/grub-2-2-06-r322-gd9b4638c5-1-wont-boot-and-goes-straight-to-the-bios-after-update/30653) grub issue on encrypted btrfs disk.

First boot on live usb from the last EndeavourOS iso downloaded on the website.
Then I followed the expected procedure: https://forum.endeavouros.com/t/the-latest-grub-package-update-needs-some-manual-intervention/30689

But since it might not be obvious to everyone, here his a quick summary.

Get the actual partitions name:

```
sudo fdisk -l
sda1: EFI
sda2: Linux File System
```
(pay attention to adapt the partition name to your case)

Unlock encrypted partition:
`sudo cryptsetup open /dev/sda2 mycryptdevice`

It's now available in `/dev/mapper/mycryptdevice`

Now mount all btrfs subvolumes from the unlocked partition:

```
sudo mount -o subvol=@ /dev/mapper/mycryptdevice /mnt
sudo mount -o subvol=@log /dev/mapper/mycryptdevice /mnt/var/log
sudo mount -o subvol=@cache /dev/mapper/mycryptdevice /mnt/var/cache
sudo mount -o subvol=@home /dev/mapper/mycryptdevice /mnt/home
```

Then mount the ESP from /dev/sda1: (pay attention its indeed /dev/sda**1**)
`sudo mount /dev/sda1 /mnt/boot/efi`

Now I'm able to chroot on my install:
`sudo arch-chroot /mnt`

Finally I was able to repair grub with:

```
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=EndeavourOS-grub
```

Reboot, and it worked !

Edit: you will end-up with a new UEFI entry EndeavourOS-grub (previous one was EndeavourOS). You may need to select the new one after reboot is your BIOS boot menu.

To clean-up everything:
```
sudo downgrade grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=EndeavourOS
```

Then edit pacman.conf to avoid any further update of Grub pending the problem resolution.

```sudo nano /etc/pacman.conf```

and uncomment:

```IgnorePkg   = grub```


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

**WARNING, it seems some disks, like Crucial MX500 do not support disabling OPAL, so you CANNOT decrypt the ssd once activated.

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

## Fingerprint reader
### Setup
Install (AUR) ```python-validity```.

```
systemctl stop python3-validity
sudo validity-sensors-firmware
sudo python3 /usr/share/python-validity/playground/factory-reset.py
systemctl start python3-validity
systemctl status python3-validity
fprintd-enroll
```

After 10 enrolings, the fingerprint reader should detect properly the finger print

### Activate for KDE, login, sudo, etc...

Got to KDE setting, Users, Configure Fingerprint Authentication, clear all finger prints and register new one(s). 

Edit (sudo):

```
/etc/pam.d/system-local-login
/etc/pam.d/system-login
/etc/pam.d/system-auth
/etc/pam.d/su
```

to add at the TOP of the files

```
auth            sufficient      pam_fprintd.so
```

Note: KDE does not activate by default the fingerprint reader, you need to press ENTER first.

## Audio
### Pipewire

To solve audio issue, install Pipewire (Alsa and Pulseaudio client) and make sure `pulseaudio-alsa` has been removed.
`pipewire-alsa`, ` pipewire-pulse`
Also install `lib32-pipewire` + JACK plugin

Reboot or restart audio service

https://wiki.archlinux.org/title/PipeWire#Installation

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

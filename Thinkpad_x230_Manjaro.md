# Thinkpad X230 Manjaro 18.0.4

Quick notes on how to install Manjaro Gnome 18.0.4 on a Lenovo Thinkpad 
X230.
Inspired from ArchLinux wiki: https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X230
Kernel: 5.2.8-1

## Install Manjaro

Just use the graphical installer or architect and follow the steps.
Test every keys of the keyboard to select the proper layout.

Manual partitioning allows to create one root partition (/) and one encrypted home partition (/home) with LUKS.
(one encrypted partition only causes quite some trouble, especially for suspend/hibernation)

To cope with this issue, it is possible to use SED SSD built-in hardware encryption.

See: https://www.happycoders.eu/manjaro-tutorial-installing-manjaro-linux-dell-xps-15-9570/ and https://github.com/Drive-Trust-Alliance/sedutil/wiki/Encrypting-your-drive

```
gunzip RESCUE32.img.gz
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

### What is working

* Kernel: i915 module is installed by default (see `less /proc/modules`)
* Screen is properly setup (update color profile with new led screen)
* Keyboard: brightness, WiFi, lock, play/pause/previous/next shortcuts (other shortcuts not yet tested)
* Volume/Mute buttons
* Touchpad / trackpoint
* Card reader
* WiFi / Bluetooth (Ethernet not tested yet)
* Sound
* Fan control
* Webcam: well detected by the system (`lsusb`), ~~but Cheese Gnome application is unable to use it. Seems a bug report has been opened for Cheese version 3.28 (lost the link). Just use another app in the meantime: Vokoscreen is great replacement, VLC also does the job.~~ and Cheese now works properly.
*...
* Fingerprint reader
Go to Account setting -> Enable finger print login
(battery menu, username, account settings, enable finger print login)


### What needs setup
* Activate hibernation

#### Hibernate on laptop lid closing

By default, nothing happens when the laptop lid is closed.

To suspend the laptop when lid is closed, click on *Menu* ->
*Tweaks* -> *Power* -> and enable *Suspend when laptop lid is
closed*

In `/etc/mkinitcpio.conf`, the HOOKS line needs the “resume” hook added (somewhere after the udev hook, or it won’t work), so it looks like this:

	HOOKS=“base udev autodetect modconf block resume filesystems keyboard keymap fsck”

Then the initramfs files (in /boot) need to be rebuilt with the new mkinitcpio.conf, using this command in the terminal (to rebuild for all installed kernels):

```
sudo mkinitcpio -P
```

This enables resuming from hibernate.

then

```
sudo nano /etc/systemd/logind.conf
```

and add

```
HandleLidSwitch=hibernate
HandleLidSwitchExternalPower=hibernate
```

!!! DEPRECATED !!!
To make it hibernate when lid is closed, you need one more step:

Edit `sudo nano /etc/systemd/logind.conf` and change

```
#HandleLidSwitch=suspend
```
to
```
HandleLidSwitch=hibernate
```

Reboot or run `systemctl restart systemd-logind.service`

#### Fingerprint reader [DEPRECATED]
*Procedure inspired by [this forum post](https://forum.manjaro.org/t/using-the-finger-print-scanner-on-a-lenovo-e530/9216)*

Fingerprint reader is well detected by the system (`lsusb`).

Install `fingerprint-gui` via package manager. It should install `libfprint`. `fprintd` must not to be installed.
Reboot.
Launch `fingerprint-gui`, select the device, next, scan finger. Close.

Now need to edit some PAM services:
* su
* sudo
* system-local-login
* gdm-fingerprint (comment `fprintd` related lines)
* system-auth

to add the following line at the top of each file (after `#%PAM-1.0`)
```
auth    sufficient    pam_fingerprint-gui.so
```

To avoid gtk errors in command line, install `gtk-engine-murrine`.

Re-open `fingerprint-gui`, scan/verify, swipe finger, no, next, test `sudo` and `su`.
If ok, next, close app.

Lock screen, press enter, swipe finger, press 'enter' -> you're logged in !

### What does not work

For now, nothing !

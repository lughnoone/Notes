# Thinkpad X230 Manjaro 17.1.9

Quick notes on how to install Manjaro Gnome 17.1.9 on a Lenovo Thinkpad X230.
Inspired from ArchLinux wiki: https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X230
Kernel: 4.16.4-1

## Install Manjaro

Just use the graphical installer or architect and follow the steps.
Keyboard layout: pay attention depending on your language, may need alternate layout to remove dead keys.

### What is working

* Kernel: i915 module is installed by default (see `less /proc/modules`)
* Screen is properly setup
* Keyboard: brightness, WiFi, lock, play/pause/previous/next shortcuts (other shortcuts not yet tested)
* Volume/Mute buttons
* Touchpad / trackpoint
* Card reader
* WiFi / Bluetooth (Ethernet not tested yet)
* Sound
* Fan control
* ...

### What is almost working
* Webcam: well detected by the system (`lsusb`), but Cheese Gnome application is unable to use it. Seems a bug report has been opened for Cheese version 3.28 (lost the link). Just use another app in the meantime: Vokoscreen is great replacement, VLC also does the job.

### What needs setup
* Fingerprint reader is not working out of the box, but works like a charm after setup.

#### Fingerprint reader
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

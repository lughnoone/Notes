# Thinkpad X230 Manjaro 17.1.9

Quick notes on how to install Manjaro Gnome 17.1.9 on a Lenovo Thinkpad X230.
Inspired from ArchLinux wiki: https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X230

## Install Manjaro

Just use the graphical installer or architect and follow the steps.
Keyboard layout: (alt., no dead keys)

### What is working

* Kernel: i915 module is installed by default (see `less /proc/modules`)
* Screen is properly setup
* Keyboard: brightness shortcuts (other shortcuts not yet tested)
* Volume/Mute buttons
* Touchpad / trackball
* Card reader
* Wifi / Bluetooth (Ethernet not tested yet)
* Sound
* Fan control
* ...

### What is not working
* Webcam: well detected by the system (`lsusb`), but Cheese gnome application is unable to use it. Seems a bug report has been opened for Cheese version 3.28 (lost the link). VLC is able to display it. Just use another app in the meantime.
* Fingerprint reader is not working out of the box

#### Fingerprint reader
*Procedure inspired by [this forum post](https://forum.manjaro.org/t/using-the-finger-print-scanner-on-a-lenovo-e530/9216)*

Fingerprint reader is well detected by the system (`lsusb`).

Install `fingerprint-gui` via package manager. Should install `libfprint`. `fprintd` not to be installed.
Reboot.
Launch `fingerprint-gui`, select the device, next, scan finger. Close.

Now need to edit some PAM services:
* su
* sudo
* system-local-login
* gdm-fingerprint (comment fprintd related lines)
* system-auth

to add the following line at the top of each file (after `#%PAM-1.0`)
```
auth    sufficient    pam_fingerprint-gui.so
```

To avoid gtk error in cmd line, install `gtk-engine-murrine`.

Re-open `fingerprint-gui`, scan/verify, swipe finger, no, next, test `sudo` and `su`.
If ok, next, close app.

Lock sreen, press enter, swipe finger, press 'enter' -> you're logged in !

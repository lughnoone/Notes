# FreeBSD Tips
A few tips to setup FreeBSD.

## Install `sudo`
```
su
pkg install sudo
visudo
```
in the editor uncomment the ligne authorizing all users part of ```wheel``` groups as sudoers.

Then give the wheel group to expected user:
``` pw usermod username -G wheel```
where ```username``` is the username of the user.

## Install ```slim``` + ```Xfce4```
```
pkg install xorg xfce slim
```

Next, as normal user, create the file ```~/.xinitrc```
Its content must be the path to ```startxfce4```:
```
exec /path/to/startxfce4
```

Make sure your user has also the ```video``` group:
```
pw usermode username -G wheel video
```
Type ```slim``` to test you are able to login and access Xfce.
If fine, you can run ```slim``` at boot time:
```
sysrc enable dbus
sysrc enable slim
```
The system is veru minimalistic. You may want to install ```pulseaudio``` Xfce related packages to enable the sound, install ``` Firefox```, and needed applications.

This being said, a simpler alternative is to install ```GhostBSD``` that is a desktop oriented FreeBSD based OS.

## Install ```virtualbox``` guest addition
```
pkg search virtualbox
pkg install _virtualbox guest addition package_
```
After install the package install output provides 2 services to be enabled via ```sysrc```. Run both commands.

Finally reboot.
## BIOS ugrade Lenovo x270 under FreeBSD (or linux)

THIS METHOD IS NOT THE METHOD RECOMMENDED BY LENOVO - FAILED BIOS UPDATE CAN BRICK YOUR DEVICE
!!! USE THIS PROCEDURE AT YOUR OWN RISK !!!

Download BIOS ISO CD image from [Lenovo support website](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x270/downloads/ds120442)

Install `geteltorito` utility:

```
pkg install geteltorito
```

Extract bootabe image from ISO:

```
geteltorito r0iur40w.iso > bios.img
```

Plug your USB stick and mount it. Then run the following command to identify your peripheral:

```
mount
```

You should see in the list your USB key and the associate peripheral path. Here under FreeBSD it's `/dev/da0`.

Unmount device:

```
umount /dev/da0
```

Then use `dd` to write the extracted image to usb stick:

```
dd if=bios.img of=/dev/da0 bs=1M conv=sync
```

Reboot, press enter the F12 to boot from USB device.
=> upgrade

_Note: this also work for linux, but device is likely something like `/dev/sdX`_


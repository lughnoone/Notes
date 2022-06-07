

# Configure Hibernation with LUKS and Btrfs not Breaking Timeshift Snapshots (Arch based system, EndeavourOS)

_(prerequisite: install from AUR ```timeshift```, ```timeshift-autosnap```, ```hibernator``` with ```yay```)_

The aim of this procedure is to explain how to setup Hibernation on a Btrfs file system encrypted with LUKS. Since Btrfs and _swap_ partition do not coexist well on LUKS, the idea is to use a swap file. Even if Btrfs now supports hibernationwith kernel > 5.11 (??), it is not obvious to make it work without breaking the MUCH useful _snapshot_ functionality.

We'll explain here how to make it work, knowledge gathered after much search on Internet.

## Create a Btrfs subvolume and the _swap_ file
Let's create a subvolume _@swap_ with a swap file in it:

```
sudo btrfs subvolume create /@swap
sudo chattr +C /@swap
sudo truncate -s 0 /@swap/swapfile
```

For 8GiB ram, size needed for hibernation is 8,8GiB -> 9011MiB.

```
sudo fallocate -l 9011M /@swap/swapfile
sudo chmod 600 /@swap/swapfile
sudo mkswap /@swap/swapfile
```

_('@' is used to match EndeavourOS @, @home, etc. sub-volumes, any name should work fine)_

## Mount the _swap_ file at startup
Let's create a mount point for the sub-volume:

``` 
sudo mkdir /swap
sudo btrfs subvolume list /
```

From the last command we note the id of @swap sub-volume (first number on the line).

Now let's edit _fstab_
```
sudo nano /etc/fstab
```

Add (replacing the filesystem by yours and subvolid by the one above):

```
/dev/mapper/luks-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX /swap btrfs subvolid=YYY,noatime
/swap/swapfile none swap defaults 0 0
```

_(take the opportunity to add option 'ssd' to all sub-volumes and remove 'autodefrag' which is not recommended for ssd)_

Now tes the configuration:
```
sudo mount -a
```

If all is good, the above command should return nothing. We **now reboot** and then check swap file is well mounted:

```
sudo swapon -s
```

## Enable hibernation
Let's now enable hibernation. For that we need the offset of the swap file:

```
sudo filefrag -v /swap/swapfile


Filesystem type is: 9123683e
File size of /swap/swapfile is 9448718336 (2306816 blocks of 4096 bytes)
 ext:     logical_offset:        physical_offset: length:   expected: flags:
   0:        0..       0:    2014897..   2014897:      1:            
   1:        1..   65535:    2014898..   2080432:  65535:             unwritten
   2:    65536.. 2306815:    2102528..   4343807: 2241280:    2080433: last,unwritten,eof
/swap/swapfile: 2 extents found
```

It's the third number of the _0:_ line, here **2014897**.

Now we enable hibernation with appropriate kernel/system config, but we use ```hibernator``` to do so, since it does it much better that us:

```
sudo hibernator
```

Now we can update the grub config to enable resuming the system after hibernation

```
sudo nano /etc/default/grub
```

Add to the GRUB_CMDLINE_LINUX_DEFAULT the two following options:

```
resume=/dev/mapper/luks-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX  resume_offset=2014897
````

_(Resume path is same as root option, resume offset value is the 3rd one from filefrag command fat line 0, see above.)_

Now let's make the configuration definitive:

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

_(In case of error, do not reboot until fixed)_

Now reboot, hibernation should be enabled and working, we'll test that right after.

## Make sure Btrfs snapshot are working fine

Run ```timeshift``` and try to make a snapshot.
Then try to hibernate and resume.

That's it !

Final procedure: LUKS,Swapfile,hibernation,btrfs snapshot

(prerequisite: install from aur timeshift,timeshift-autosnap,hibernator with yay)

sudo btrfs subvolume create /@swap
sudo chattr +C /@swap
sudo truncate -s 0 /@swap/swapfile

for 8GiB ram, size is 8,8GiB -> 9011MiB

sudo fallocate -l 9011M /@swap/swapfile
sudo chmod 600 /@swap/swapfile
sudo mkswap /@swap/swapfile
sudo mkdir /swap
sudo btrfs subvolume list /

get id of @swap volume

sudo nano /etc/fstab

add (replacing the filesystem by yours and subvolid by the one above):

/dev/mapper/luks-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX /swap btrfs subvolid=YYY,noatime
/swap/swapfile none swap defaults 0 0

(take the opportunity to add option 'ssd' to all subvolume and remove 'autodefrag')

sudo mount -a

If all is good, reboot.
then check swap is mounted
sudo swapon -s

sudo filefrag -v /swap/swapfile  
sudo hibernator
sudo nano /etc/default/grub

resume=/dev/mapper/luks-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX resume_offset=1889268,
(resume is same as root option resume offset value is the 3rd one from filefrag cmd for line 0)

sudo grub-mkconfig -o /boot/grub/grub.cfg

reboot

run timeshift and try to make a snapshot. then try to hibernate and resume.
That's it !

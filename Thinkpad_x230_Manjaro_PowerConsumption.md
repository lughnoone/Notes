# Optimize Laptop Power Consumption

Very widely inspired from 
https://wiki.manjaro.org/index.php/PowerTOP_to_Optimise_Laptop_Power_Consumption

## Install need packages

Install `powertop` and `ethtool`. The first one seems to be installed by default with 
Manjaro 17.

## Diagnostic

Run `sudo powertop --html`. It creates an HTML report `powertop.html` in the home directory 
(created with root permissions).

## Test

Create the optimization script from the tuning powertop report.
(commands are located in the second column of the tuning tab report)

E.g.

echo 'min_power' > '/sys/class/scsi_host/host0/link_power_management_policy'; 
echo 'min_power' > '/sys/class/scsi_host/host5/link_power_management_policy'; 
echo 'min_power' > '/sys/class/scsi_host/host3/link_power_management_policy'; 
echo 'min_power' > '/sys/class/scsi_host/host1/link_power_management_policy'; 
echo 'min_power' > '/sys/class/scsi_host/host4/link_power_management_policy'; 
echo 'min_power' > '/sys/class/scsi_host/host2/link_power_management_policy'; 
echo '1500' > '/proc/sys/vm/dirty_writeback_centisecs'; 
echo 'auto' > '/sys/bus/usb/devices/1-1.2/power/control'; 
echo 'auto' > '/sys/bus/pci/devices/0000:00:1c.0/power/control'; 
echo 'auto' > '/sys/bus/pci/devices/0000:00:1c.1/power/control'; 
echo 'auto' > '/sys/bus/pci/devices/0000:00:1c.2/power/control'; 
ethtool -s enp0s25 wol d; 

Paste the commands in `sudo nano /usr/local/bin/power_optim.sh` and make the script 
executable `sudo chmod +x /usr/local/bin/power_optim.sh`.

Run it manually `sudo /usr/local/bin/power_optim.sh` and let the computer idle for some time 
to be able to compare the laptop power consumption before/after.

If the script has the expected effect, why not running it at startup ?

## Run the script at startup

*I still haven't found a way to run it at startup ... *

Reboot to check the effects. You may want to run the diagnostic again to validate the change 
is well taken into account.

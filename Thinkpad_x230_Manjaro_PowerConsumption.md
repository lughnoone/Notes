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

*Systemd is the way to start script at boot, see below.*
https://wiki.archlinux.org/index.php/Autostarting
https://bbs.archlinux.org/viewtopic.php?id=154338

Create a service `sudo nano /etc/systemd/system/power_optim.service`:

```
[Unit]
Description=Power Optimization
ConditionFileIsExecutable=/usr/local/bin/power_optim.sh

[Service]
Type=oneshot
ExecStart=/usr/local/bin/power_optim.sh
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```
Then test the script is running by launching: `systemctl start power_optim`
If everything is fine, enable the service: `systemctl enable power_optim`

Reboot to check the effects. You may want to run the diagnostic again to validate the change
is well taken into account.

# Well it does not work
**It seems it doesn't work as expected, commands to be put in the power_optim.sh seems to evolve after each boot, so making a static script is not an option.**

Another option would have to use directly `powertop` in the launched script:

```
#!/bin/bash

powertop --auto-tune

exit 0
```

But it seems to not be working either... to be continued...

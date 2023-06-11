# Samba file sharing
Aim of this quick procedure is to setup a local file sharing server.

First install Samba, create one (or more) folder and finally edit the server configuration:
``` 
sudo apt-get install samba
mkdir -p samba_share
nano /etc/samba/smb.conf
```

Add the setup for the created folder (repeat for other folders)
```
[myshare]
path = /path/to/samba_share
writeable=Yes
create mask=0777
directory mask=0777
public=no
```

Then create a user (must be an existing user):
```
sudo smbpasswd -a myusername
```

Finally apply the config:
```
sudo systemctl restart smbd
```

To configure clients, just target the server hostname/ip/user/password. Pay attention to the workgroup to make the sharing visible from Windows machines.
# Netplan
Netplan notes / tips - quite misleading

## Static IP
Let's say the static ip is to be 192.168.0.10, and the gateway 192.168.0.1, with Cloudflare resolvers.

Edit the YAML file in ```/etc/netplan/```:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.0.10/24
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [1.1.1.1, 1.0.0.1, "2606:4700:4700::1111", "2606:4700:4700::1001"]
```

Pay strong attention to the YAML syntax and 2 spaces delimiters.

Then run:
```
sudo netplan try
```

Either accept here, or run:
```
sudo netplan generate
sudo netplan --debug apply
```
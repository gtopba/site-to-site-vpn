1. Local DNS server
2. Sites blacklists
3. DHCP server (optional)
To use pi-hole as DHCP server. Set static IP for the Orange Pi
(Wornning 1 : if connect to different subnet router, you might not be able to access the Opi and VPN will not work. So you brick yourself !!! )
(Wornning 2 : if the router DHCP server is disabled, all your devices will not haveaccess to internet so you can't remote access to fix the problem, you are bricked)
```
sudo nano /etc/netplan/10-dhcp-all-interfaces.yaml
```
Add:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    end0:
      dhcp4: no
      addresses:
        - 192.168.2.206/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.2.1
      nameservers:
        addresses:
          - 192.168.2.206
          - 8.8.8.8
```
```
sudo netplan apply
```
```
ip a show end0
```
In this case, the pi-hole dhcp rang should be `192.168.2.2` to `192. 168.2.205` to prevent ip conflic 

This is the original config:
```
# Added by Armbian
#
# Reference: https://netplan.readthedocs.io/en/stable/netplan-yaml/
#
# Let systemd-networkd manage all Ethernet devices on this system, but be configured by Netplan.

network:
  version: 2
  renderer: networkd
  ethernets:
    all-eth-interfaces:
      match:
        name: "e*"
      dhcp4: yes
      dhcp6: yes
      ipv6-privacy: yes # Enabled by default on most current systems, but networkd currently doesn't enable IPv6 privacy by default, see https://man.archlinux.org/man/systemd.network.5
```

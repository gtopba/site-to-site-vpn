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


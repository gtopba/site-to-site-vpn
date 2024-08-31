1. Local DNS server
2. Sites blacklists
3. DHCP server (optional)
To use pi-hole as DHCP server. Set static IP for the Orange Pi
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
    all-eth-interfaces:
      match:
        name: "e*"
      dhcp4: yes
      dhcp6: yes
      ipv6-privacy: yes
```
```
sudo netplan apply
```
```
ip a show end0
```


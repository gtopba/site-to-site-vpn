1. Local DNS server
2. Sites blacklists
3. DHCP server (optional)
4.
```
auto end0
iface end0 inet static
    address 192.168.2.206
    netmask 255.255.255.0
    gateway 192.168.2.1
    dns-nameservers 192.168.2.206 1.1.1.1
```


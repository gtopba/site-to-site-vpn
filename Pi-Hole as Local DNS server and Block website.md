## 1. Install Pi-hole
Run the Pi-hole installation script: The easiest way to install Pi-hole is by using the automated installation script. Run the following command:
```
curl -sSL https://install.pi-hole.net | bash
```
- add new Adlsit
- 
## 2. Configure Your Router to Use Pi-hole as the DNS Server
1. Log in to your router's admin interface.

2. Navigate to the DNS settings:
    - Look for settings related to DNS, usually under Network, LAN, or DHCP.
    
3. Set the Pi-hole IP address as the primary DNS server:
    - Replace the existing DNS server with the IP address of your Orange Pi (e.g., 192.168.2.x).
    - Save the changes.

4. Optional - Disable IPv6 DNS (if not using IPv6):
  - If your network doesn’t support IPv6 or you didn’t enable IPv6 during the Pi-hole installation, make sure your router is not assigning IPv6 DNS servers.

## 3. Local DNS server
  - Local DNS Resolver: You might have a DNS resolver like Unbound installed on the same machine as Pi-hole. This resolver is configured to provide recursive DNS queries, which Pi-hole uses to resolve domain names. The resolver is set up to listen on 127.0.0.1 (the loopback interface) and on port 5335.
  - Flexibility: This setup allows you to implement advanced DNS features such as DNSSEC validation, filtering, and more complex DNS resolution strategies.
  
## 4. DHCP server (optional)
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

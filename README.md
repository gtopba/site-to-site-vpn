## :heavy_check_mark: Key Characteristics of Site-to-Site VPN:
### Connects Two Networks:

A Site-to-Site VPN connects entire networks (e.g., your local home network and the Northern site network), allowing devices on one network to communicate with devices on the other as if they were on the same local network.

### Uses VPN Gateways:

Each site has a VPN gateway (e.g., your WireGuard server) that manages the VPN connection and routes traffic between the sites.

### Secures Data Transmission:

Data transmitted over the VPN is encrypted, ensuring secure communication over the internet.

### Transparent to End Devices:

Devices on each network do not need to be aware of the VPN; they simply send traffic to the VPN gateway, which handles the encryption and routing.

## :question: My system
### local
1. WireGuard server running on proxmox [install](https://tteck.github.io/Proxmox/#wireguard-lxc)
2. Set static ip for WireGuard server.
3. Forward port 51820
4. Setup DDNS url (for dynamic ip home internet) in the WireGuard server's config file, add your url at pivpnHOST=[ddns url]
   ```
   nano /etc/pivpn/wireguard/setupVars.conf
   ```
5. In this example, my local network subnet is 192.168.0.0/24
   
### remote site
1. Rpi 3 B+ running 64bit PiOS lite [install](https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html)
2. Router 192.168.2.1
   
## :lollipop: WireGuard VPN Setup and Maintenance
This document provides a step-by-step guide to setting up and maintaining a WireGuard VPN connection between a local network and a remote site using a Raspberry Pi with PiOS lite 64bit.

### 1. Install WireGuard on Raspberry Pi
Update your package list:
```
sudo apt-get update
```
Install WireGuard:
```
sudo apt-get install wireguard
```
Install additional tools (optional but recommended):
```
sudo apt-get install wireguard-tools
```
### 2. On WireGuard server, add client
Generate the private key:
```
pivpn add
```
Copy the generated client file to client device

### 3. Set Up the WireGuard Client Configuration File on Raspberry Pi
Create the configuration file:
```
sudo nano /etc/wireguard/wg0.conf
```
Add the following configuration to the file:
```
[Interface]
PrivateKey = <client_private_key>
Address = 10.63.31.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = <server_public_key>
PresharedKey = <preshared_key>
Endpoint = warut.duckdns.org:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Save and exit:
Press Ctrl+X to exit.
Press Y to save the changes.
Press Enter to confirm the file name.
### 4. Enable IP Forwarding and Set Up NAT (Network Address Translation)
Enable IP forwarding:
```
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
Set up iptables rules:
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```
Make iptables rules persistent:
```
sudo apt-get install iptables-persistent
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```
### 5. Start and Enable WireGuard
Start the WireGuard interface:
```
sudo wg-quick up wg0
```
Enable WireGuard to start on boot:
```
sudo systemctl enable wg-quick@wg0
```
### 6. Set Up a Cron Job to Check and Restart VPN Connection Every 5 Minutes
Create the check script:
```
sudo nano /usr/local/bin/check_wireguard.sh
```
Add the following script:
```
#!/bin/bash

# Ping a known reachable address through the VPN
PING_ADDRESS="10.63.31.1" # Change this to an address that should be reachable through the VPN

if ! ping -c 1 -W 1 $PING_ADDRESS > /dev/null; then
    echo "$(date): Ping to $PING_ADDRESS failed. Restarting WireGuard." >> /var/log/wireguard-check.log
    sudo wg-quick down wg0
    sudo wg-quick up wg0
else
    echo "$(date): Ping to $PING_ADDRESS succeeded." >> /var/log/wireguard-check.log
fi
```
Save and exit:
Press Ctrl+X to exit.
Press Y to save the changes.
Press Enter to confirm the file name.

Make the script executable:
```
sudo chmod +x /usr/local/bin/check_wireguard.sh
```
Edit the cron job:
```
sudo crontab -e
```
Add the following line to run every 5 minutes:
```
*/5 * * * * /usr/local/bin/check_wireguard.sh
```
Save and exit:
Press Ctrl+X to exit.
Press Y to save the changes.
Press Enter to confirm the file name.

## Router configurations
### 1. Gateway 

### 2. Static route

## Security Recommendations
Use Strong Authentication: Ensure strong, unique private and public keys for authentication.
Regularly Update Software: Keep WireGuard and all related software up to date.
Proper Firewall Configuration: Use iptables or ufw to manage firewall rules on the VPN server.
Monitor and Review Logs: Regularly monitor VPN server logs for unusual activity.
Restrict Access: Limit VPN server access to trusted IP addresses if possible.
By following these steps and recommendations, you ensure a secure and reliable VPN connection between your local network and the Northern site.


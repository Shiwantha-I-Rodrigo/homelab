# MAP THE NETWORK

## **Step 1: Identify Your Network Interface**

```
ip a
```

## **Step 2: Discover the Subnet**

```
ip route
```

```
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.10
default via 192.168.1.1 dev enp0s3
```

--> Subnet: `192.168.1.0/24`\
--> Gateway: `192.168.1.1`

## **Step 3: Ping Sweep (Basic Host Discovery)**

```
# Using fping
sudo pacman -S fping
sudo fping -a -g 192.168.1.0/24
```

--> `-a` shows alive hosts.\
--> `-g` specifies the range.

```
# Using nmap
sudo pacman -S nmap
sudo nmap -sn 192.168.1.0/24
```

--> `-sn` = ping scan (no port scan). Shows live hosts.

## **Step 4: Map Open Ports / Services**

```
sudo nmap -sS -p 1-65535 192.168.1.10
```

--> `-sS` = SYN scan (fast stealth scan)\
--> `-p 1-65535` = scan all TCP ports

**Scan all hosts in subnet with common ports:**

```
sudo nmap -sS -p 22,80,443,53 192.168.1.0/24
```

## **Step 5: Identify Device Types**

**MAC addresses** (device type/manufacturer)

```
arp -n
```

```
# Using nmap
sudo nmap -O 192.168.1.10
```

--> `-O` attempts OS detection.

## **Step 6: Map Layer 2 Topology**

```
# ARP table
ip neigh show

# Display LLDP neighbors (if LLDP is enabled)
sudo pacman -S lldpd
sudo systemctl enable --now lldpd
sudo lldpctl

# Display CDP neighbors (Cisco)
sudo pacman -S cdpr
sudo cdpr -d enp0s3
```

## **Step 7: Create a Visual Network Map**

`nmap -oX scan.xml` → import into **Zenmap** GUI for network maps

```
sudo nmap -sn -oX network_scan.xml 192.168.1.0/24
```

--> Open `network_scan.xml` in Zenmap and generate a network topology.

---

# AUTOMATION

--> network_map.sh
```
#!/bin/bash

# ------------------------------
# Arch Linux Network Mapping Script
# ------------------------------

# check for root privileges
if [ "$EUID" -ne 0 ]; then
  echo "Please run as root: sudo $0"
  exit
fi

# identify network interface
echo "=== Network Interfaces ==="
ip a | grep -E "^[0-9]+:|inet "

read -p "Enter the interface to scan (ie. enp0s3): " IFACE

# determine subnet
SUBNET=$(ip -o -f inet addr show $IFACE | awk '{print $4}')
echo "Detected subnet: $SUBNET"

# ping sweep to discover hosts
echo "=== Discovering live hosts ==="
sudo pacman -Qs fping > /dev/null || sudo pacman -S --noconfirm fping
fping -a -g $SUBNET 2>/dev/null > live_hosts.txt
echo "Live hosts saved to live_hosts.txt"
cat live_hosts.txt

# gather MAC addresses via ARP
echo "=== MAC Addresses (ARP) ==="
arp -n | grep -F -f live_hosts.txt > mac_table.txt
cat mac_table.txt

# discover open ports and services
echo "=== Scanning open ports on live hosts ==="
sudo pacman -Qs nmap > /dev/null || sudo pacman -S --noconfirm nmap
while read host; do
    echo "--- Scanning $host ---"
    nmap -sS -sV -O $host >> nmap_scan.txt
done < live_hosts.txt
echo "Scan results saved to nmap_scan.txt"

# check LLDP / CDP neighbors
echo "=== LLDP / CDP neighbors ==="
sudo pacman -Qs lldpd > /dev/null || sudo pacman -S --noconfirm lldpd
sudo systemctl enable --now lldpd
lldpctl > lldp_neighbors.txt
cat lldp_neighbors.txt

sudo pacman -Qs cdpr > /dev/null || sudo pacman -S --noconfirm cdpr
cdpr -d $IFACE > cdpr_neighbors.txt
cat cdpr_neighbors.txt

# generate simple network map file
echo "=== Generating basic network map ==="
echo "Network Map for $SUBNET" > network_map.txt
echo "--------------------------" >> network_map.txt
echo "Live Hosts:" >> network_map.txt
cat live_hosts.txt >> network_map.txt
echo "" >> network_map.txt
echo "MAC Table:" >> network_map.txt
cat mac_table.txt >> network_map.txt
echo "" >> network_map.txt
echo "Open Ports / Services:" >> network_map.txt
cat nmap_scan.txt >> network_map.txt
echo "" >> network_map.txt
echo "LLDP Neighbors:" >> network_map.txt
cat lldp_neighbors.txt >> network_map.txt
echo "" >> network_map.txt
echo "CDP Neighbors:" >> network_map.txt
cat cdpr_neighbors.txt >> network_map.txt

echo "Network map created: network_map.txt"
```

--> Run
```
chmod +x network_map.sh
sudo ./network_map.sh
```

--> outputs
- live_hosts.txt → All discovered live hosts
- mac_table.txt → MAC addresses of live hosts
- nmap_scan.txt → Open ports and services
- lldp_neighbors.txt → LLDP neighbor info
- cdpr_neighbors.txt → CDP neighbor info
- network_map.txt → Complete consolidated network map
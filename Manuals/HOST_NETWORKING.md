# IP / SS / NMCLI

| Tool        | Purpose                                               | Best Use Cases                                |
| ----------- | ----------------------------------------------------- | --------------------------------------------- |
| **`ip`**    | Configure and view interfaces, addresses, routes, ARP | Low-level network management, scripting       |
| **`ss`**    | Inspect sockets, network connections, listening ports | Troubleshooting ports, processes, connections |
| **`nmcli`** | Manage connections through NetworkManager             | Desktop/server config, Wi-Fi, static IP, DNS  |

## **1. `ip` — Modern Linux Networking Utility**

`ip` replaces older networking tools such as `ifconfig`, `route`, and `arp`.

**Show Network Interfaces**
```
ip link show
```

**Bring an interface up/down**
```
ip link set eth0 up
ip link set eth0 down
```

**Show or Set IP Addresses**
```
ip addr show
ip addr show eth0
```

**Add/remove an IPv4 address**
```
ip addr add 192.168.1.20/24 dev eth0
ip addr del 192.168.1.20/24 dev eth0
```

**Display routing table**
```
ip route show
```

**Add a default route**
```
ip route add default via 192.168.1.1 dev eth0
```

**Remove a route**
```
ip route del 10.1.1.0/24 dev eth0
```

**Neighbors (ARP Table)**
```
ip neigh show
```

**Manually add ARP entry**
```
ip neigh add 192.168.1.50 lladdr 00:11:22:33:44:55 dev eth0
```

---

## **2. `ss` — Socket Statistics Tool**

`ss` replaces `netstat`.

**Show all listening ports**
```
ss -l
```

Listening TCP + UDP:
```
ss -tuln
```
- **t** : TCP
- **u** : UDP
- **l** : listening
- **n** : numeric output (no DNS resolution)

**Show processes using ports**
```
ss -tulnp
```
- **p** : show process owning the socket.

**Show all connections**
```
ss -ta
```

**Filter by port**
```
ss -tuln sport = :80
ss -tuln dport = :22
```

**Show TCP states**
```
ss -tan state established
ss -tan state syn-recv
```

**Show IPv6 connections**
```
ss -6tan
```

---

## **3. `nmcli` — NetworkManager Command-line Interface**

`nmcli` configures NetworkManager: network connections, Wi-Fi, Ethernet, VPN, hostname, etc.

**Show Network Status**
```
nmcli
nmcli device
nmcli connection show
```

**Enable/Disable Networking**
```
nmcli networking off
nmcli networking on
```

**Enable/disable wireless**
```
nmcli radio wifi off
nmcli radio wifi on
```

**Connect to a Wi-Fi Network**
```
nmcli device wifi list
nmcli device wifi connect "MyWiFi" password "mypassword"
```

**Add a Static IPv4 Address**
**Method 1 — Modify an existing connection**
```
nmcli connection modify eth0 ipv4.addresses "192.168.1.50/24"
nmcli connection modify eth0 ipv4.gateway "192.168.1.1"
nmcli connection modify eth0 ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection modify eth0 ipv4.method manual
nmcli connection up eth0
```
**Method 2 — Create a new connection**
```
nmcli connection add type ethernet ifname eth0 con-name static-eth0 \
    ipv4.addresses 192.168.1.50/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns "8.8.8.8" \
    ipv4.method manual
```

**Enable DHCP**
```
nmcli connection modify eth0 ipv4.method auto
```

**Bring a Connection Up/Down**
```
nmcli connection down eth0
nmcli connection up eth0
```

**Change Hostname**
```
nmcli general hostname server01
```

**Using USB Adapters**

> **IP address resetting constantly**\
> NetworkManager may treat USB interfaces as dynamic or if the hardware briefly disconnects (miliseconds) and reconnects, NetworkManager sees a "new" connection, it falls back to its default behavior (DHCP) unless the profile is strictly locked down.

```
# Set static IP, set manual mode and ensure it auto-connects
sudo nmcli connection modify "Wired connection 2" ipv4.addresses "192.168.11.3/24" ipv4.method manual connection.autoconnect yes
sudo nmcli connection modify "Wired connection 3" ipv4.addresses "192.168.12.4/24" ipv4.method manual connection.autoconnect yes

# Apply the changes immediately
sudo nmcli connection up "Wired connection 2"
sudo nmcli connection up "Wired connection 3"
```

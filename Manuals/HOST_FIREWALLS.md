# IPTABLES / NFTABLES

## **1. `iptables`**

Legacy firewall system for Linux based on the **netfilter** framework.\
Iptables organizes rules into **tables**, which contain **chains**, which contain **rules**.

### 🔹 Main Tables

| Table        | Purpose                               |
| ------------ | ------------------------------------- |
| **filter**   | Basic packet filtering (ACCEPT, DROP) |
| **nat**      | NAT, port forwarding, masquerading    |
| **mangle**   | Packet alteration (TTL, TOS)          |
| **raw**      | Connection tracking exemptions        |
| **security** | SELinux integration                   |

### 🔹 Main Chains

| Chain           | Direction                       |
| --------------- | ------------------------------- |
| **INPUT**       | Traffic **to the local system** |
| **OUTPUT**      | Traffic **from local system**   |
| **FORWARD**     | Routed traffic (not local)      |
| **PREROUTING**  | Changes before routing          |
| **POSTROUTING** | Changes after routing           |

**List Rules**
```
iptables -L -v -n
iptables -t nat -L -v -n
```
- **-L** list
- **-v** verbose
- **-n** no DNS resolution (for speed)

**Allow SSH**
```
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

**Drop all inbound traffic**
```
iptables -P INPUT DROP
```

**Allow loopback**
```
iptables -A INPUT -i lo -j ACCEPT
```

**Delete a rule**
```
iptables -D INPUT -p tcp --dport 22 -j ACCEPT
```

**Delete by line number**
```
iptables -L INPUT --line-numbers
iptables -D INPUT 3
```

**NAT / Port Forwarding**
```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

**Save / Load**
```
iptables-save > /etc/iptables.rules
iptables-restore < /etc/iptables.rules
```

RHEL/CentOS:
```
service iptables save
```

**IPTABLES Config File Example -->**
`/etc/iptables.rules`
```
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]

# Example Configuaration
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p tcp --dport 22 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT

COMMIT
```

---

## **2. `nft`**

Modern firewall framework that replaces **iptables**.

### 🔹 Key Components

| Component     | Description                  |
| ------------- | ---------------------------- |
| **Families**  | inet, ip, ip6, arp, bridge   |
| **Tables**    | Group rules by purpose       |
| **Chains**    | Hook into network flow       |
| **Rules**     | Match traffic + take action  |
| **Sets/Maps** | Efficient lists of IPs/ports |

> Most commonly used family **inet** (works for IPv4 + IPv6).

| **Hook Name**   | **Purpose**                                                                                        |
| --------------- | -------------------------------------------------------------------------------------------------- |
| **input**       | Processes packets **destined for the local system** (after routing decision).                      |
| **output**      | Processes packets **originating from the local system**.                                           |
| **forward**     | Processes packets **passing through the system** (router mode).                                    |
| **prerouting**  | Runs **before routing decisions** are made (used for DNAT, packet marking).                        |
| **postrouting** | Runs **after routing decisions**, just before packets leave the system (used for SNAT/MASQUERADE). |

**Show Rules**
```
nft list ruleset
nft list table inet filter
```

**Add a Table & Chain**
```
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0; }
```

**Accept SSH**
```
nft add rule inet filter input tcp dport 22 accept
```

**Allow loopback**
```
nft add rule inet filter input iif lo accept
```

**Drop everything else**
```
nft add rule inet filter input drop
```

**Delete Rule**

List with handle
```
nft list chain inet filter input
```

Delete using handle
```
nft delete rule inet filter input handle 12
```

**Enable NAT**
```
nft add table ip nat
nft add chain ip nat postrouting { type nat hook postrouting priority 100 ; }
nft add rule ip nat postrouting oif eth0 masquerade
```

**Port forwarding**
```
nft add chain ip nat prerouting { type nat hook prerouting priority -100 ; }
nft add rule ip nat prerouting tcp dport 80 dnat to 192.168.1.10:80
```

**Using Sets (advantage over iptables)**
```
nft add set inet filter blocked_ips { type ipv4_addr ; }
nft add rule inet filter input ip saddr @blocked_ips drop
```

Add entries:
```
nft add element inet filter blocked_ips { 1.2.3.4, 5.6.7.8 }
```

**Save/Load Config**
```
nft list ruleset > /etc/nftables.conf
nft -f /etc/nftables.conf
```

**NFTABLES Config File Example -->**
`/etc/nftables.conf`
```
table inet filter {
    chain input {
        type filter hook input priority 0;

        iif lo accept
        ct state established,related accept
        tcp dport 22 accept
        tcp dport 80 accept
        drop
    }
}
```

---

## FIREWALL STATUS

| **Firewall** | **Enable / Turn On**                                                                                                                                                                                         | **Disable / Turn Off**                                                                                                                                                                                       | **Verify Status**                                            |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| **iptables** | **Debian/Ubuntu:**<br>`sudo systemctl enable netfilter-persistent`<br>`sudo systemctl start netfilter-persistent`<br>**RHEL/CentOS:**<br>`sudo systemctl enable iptables`<br>`sudo systemctl start iptables` | **Debian/Ubuntu:**<br>`sudo systemctl stop netfilter-persistent`<br>`sudo systemctl disable netfilter-persistent`<br>**RHEL/CentOS:**<br>`sudo systemctl stop iptables`<br>`sudo systemctl disable iptables` | `sudo iptables -L -v -n`<br>`sudo systemctl status iptables` |
| **nftables** | `sudo systemctl enable nftables`<br>`sudo systemctl start nftables`                                                                                                                                          | `sudo systemctl stop nftables`<br>`sudo systemctl disable nftables`                                                                                                                                          | `sudo nft list ruleset`<br>`sudo systemctl status nftables`  |

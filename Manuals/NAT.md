**~ The Lies of *NAT* Lamora ~** <sub><sup>by Scott Lynch</sup></sub>

---

# NETWORK ADDRESS TRANSLATION (NAT)

## SUMMARY

- Short term solution to overcome the address requirement to connect with internet.
- Enables an organization to use Private Addressing Scheme (defined in RFC 1918) and still connect to the internet.
- Private Address Spaces :
    1. 10.0.0.0 / 8
    2. 172.16.0.0 / 12
    3. 192.168.0.0 / 16
- NAT Address Types :
    1. Inside Local Address: the IP Address assigned to the host on the inside network.
    2. Inside Global Address: It is the IP address of an inside host (or a group of hosts) as it appears to the outside network.
    3. Outside Local Address: the IP address assigned to an outside host as it appears to the inside network.
    4. Outside Global Address: the actual public IP address of the external device (on the internet).
- NAT Types :
    1. Static NAT : A single local IP address is mapped to single global IP address.
    2. Dynamic NAT : Each inside host is assigned a global address for the duration of the session from a **pool** of global addresses.
    3. Port Address Translation (**PAT**) : Translates multiple local addresses to a single global address using different ports.

**PAT Table**

| Device | Inside Local (IP:Port) | Inside Global (Public IP:Port) | Destination (Website / Server) |
| - | - | - | - |
| PC1   | 192.168.1.10:1050      | 203.0.113.10:40001          | [www.google.com](http://www.google.com) (142.250.72.14:80)|
| PC1   | 192.168.1.10:1051      | 203.0.113.10:40004          | [www.youtube.com](http://www.youtube.com) (142.250.190.14:443)|
| PC2   | 192.168.1.11:1060      | 203.0.113.10:40002          | [www.google.com](http://www.google.com) (142.250.72.14:80)|
| PC3   | 192.168.1.12:1070      | 203.0.113.10:40003          | [www.wikipedia.org](http://www.wikipedia.org) (208.80.154.224:80)|

---

## CONFIGURATIONS

**STATIC NAT**

1 - Define inside and outside router interfaces
```
R1(config)# interface fastethernet0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ip nat inside

R1(config)# interface fastethernet0/2
R1(config-if)# ip address 203.0.113.1 255.255.255.0
R1(config-if)# ip nat outside
```

2 - Configure static NAT
```
R1(config)# ip nat inside source static 192.168.1.10 203.0.113.10
```

**DYNAMIC NAT**

1 - Define inside and outside router interfaces
```
R1(config)# interface fastethernet0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ip nat inside

R1(config)# interface fastethernet0/2
R1(config-if)# ip address 203.0.113.1 255.255.255.0
R1(config-if)# ip nat outside
```

2 - Define NAT pool
```
R1(config)# ip nat pool MYPOOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
```

3 - Define access list for internal devices allowed to NAT
```
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

4 - Configure dynamic NAT
```
R1(config)# ip nat inside source list 1 pool MYPOOL
```

**PAT**

1 - Define inside and outside router interfaces
```
R1(config)# interface fastethernet0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ip nat inside

R1(config)# interface fastethernet0/2
R1(config-if)# ip address 203.0.113.1 255.255.255.0
R1(config-if)# ip nat outside
```

2 - Define access list for internal devices allowed to NAT
```
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

3 - Configure PAT ( NAT overload )
```
R1(config)# ip nat inside source list 1 interface fastethernet0/2 overload
```

---

## MONITORING & MAINTENANCE

```
show ip nat translation
show ip nat translation verbose
debug ip nat
```

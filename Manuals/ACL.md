# ACCESS CONTROL LISTS (ACL)

## SUMMARY

- ACLs can control the traffic entering the network.
    + Firewall Router (IPv4 ACLs, IPv6 ACLs) Layer 3
    + Switch (VLAN ACLs, MAC ACLs) Layer 2
- ACL contains a set of rules that apply to inbound traffic.
- Each rule might apply to one or more of the fields within a packet.
- Traffic is filtered based on the order that the ACL entries occur in the router.
- New statements are added to the end of the list.
- Every ACL has an **implicit deny** at the **end** for any traffic that is not explicitly permitted.
(**An ACL with only one deny entry will deny all traffic !!!**)
- You can define ACLs and still not apply them. ACLs have no effect until they are applied to the interface.
- It is a good practice to apply the ACL on the interface **closest to the source** of the traffic.

**MASKS**

```
Network address to be processed : 10.1.1.5

Subnet mask                     : 255.255.255.128

Inverse mask                    : 255.255.255.255
                                : 255.255.255.128
                                =================
                                :   0.  0.  0.127

Network address 	            : 00001010  00000001  00000001  00000101 <-- 5 (101) is ignored
                                  ↓↓↓↓↓↓↓↓  ↓↓↓↓↓↓↓↓  ↓↓↓↓↓↓↓↓  ↓
Inverse mask                    : 00000000  00000000  00000000  01111111

                                  00001010  00000001  00000001  0   <-- Any address starting with these is affected

                           from : 00001010  00000001  00000001  00000000 ( 10.1.1.0 )
                           to   : 00001010  00000001  00000001  01111111 ( 10.1.1.127 )
```

Based on the **inverse mask**, the first 25 binaries must match the first 25 binaries from the given binary network address.

---

**ACL SUMMARIZATION**

Example 1

```
192.168.32.0/24
192.168.33.0/24
192.168.34.0/24
192.168.35.0/24
192.168.36.0/24
192.168.37.0/24
192.168.38.0/24
192.168.39.0/24

The interesting Octets : 32 : 0010 0000
                       : 33 : 0010 0001
                       : 34 : 0010 0010
                       : 35 : 0010 0011
                       : 36 : 0010 0100
                       : 37 : 0010 0101
                       : 38 : 0010 0110
                       : 39 : 0010 0111
                              ↓↓↓↓ ↓
                              0010 0    ( First 5 is a match )

∴ first 21 ( 8 + 8 + 5 ) bits are a match for all the given subnets.
                           
∴ above five subnets can be summarized as : 192.168.32.0/21 -OR- 192.168.32.0 0.0.7.255
```

Example 2

```
192.168.146.0/24
192.168.147.0/24
192.168.148.0/24
192.168.149.0/24

The interesting Octets : 146 : 1001 0010
                       : 147 : 1001 0011
                       : 148 : 1001 0100
                       : 149 : 1001 0101
                               ↓↓↓↓ ↓
                               1001 0    ( First 5 is a match )

! however these networks cannot be summarized like the previous example.

since the summarized address become : 192.168.144.0/21 ( 192.168.144.0 - 192.168.151.0 ), which include extra addresses.

but the 4 addresses can be summarized into 2 summarized addresses : 
    192.168.146.x and 192.168.147.x, all bits match except for the last one > This can be written as 192.168.146.0/23
    192.168.148.x and 192.168.149.x, all bits match except for the last one > This can be written as 192.168.148.0/23
```

**STANDARD Vs EXTENDED**

| **Feature** | **Standard ACL** | **Extended ACL** |
| - | - | - |
| ACL Range (Number)    | 1–99 (standard), 1300–1999 (expanded)                     | 100–199 (standard), 2000–2699 (expanded)|
| Filtering Criteria    | source IP address                                         | source and destination IP addresses, protocol type, and port numbers|
| Layer of Operation    | Layer 3 (Network Layer)                                   | Layer 3 and Layer 4 (Network and Transport Layers)|
| Placement in Network  | Closer to destination (avoid unnecessary blocking)        | Closer to source (prevent unwanted traffic early)|
| Syntax Example        | `access-list 10 permit 192.168.1.0 0.0.0.255`             | `access-list 110 permit tcp 192.168.1.0 0.0.0.255 10.0.0.0 0.0.0.255 eq 80`|
| Protocol Support      | Works with IP only                                        | Works with IP, TCP, UDP, ICMP, and others|
| Port Filtering        | Not possible                                              | Possible (e.g., filter HTTP, FTP, SSH, etc.)|
| Use Case              | Used for simple network access control                    | Used for complex filtering, like controlling access to specific services or applications|
| Processing Overhead   | Lower                                                     | Higher (because of more detailed inspection)|

- An ACL can be identified as either **named** or **numbered**.
- Only one ACL per interface, per protocol, per direction is allowed.
- **Inbound** packets are processed by an ACL **before** being routed.
- **Outbound** packets are processed by an ACL **after** being routed.
- Unlike subnet mask, wildcard mask **allow non-contiguous bits** in the mask.
    + `access-list 10 permit 192.168.1.0 0.0.3.255` ( matches 192.168.1.0 - 192.168.4.255 ).
    + **0** -> must match exactly.
    + **1** -> simply ignore.
- A "defined but empty" ACL allows all traffic, ! As soon as you add one statement the implicit **“deny all”** rule at the end becomes **active**.

---

## CONFIGURATIONS

An ACL is implemented in two steps:
1. define an ACL with “access-list or ip access-list” command.
2. apply the ACL under specific interface in the required direction with “ip access-group” command.

Standard : `access-list acl-number {permit|deny} {host|source source-wildcard|any}`

```
# Numbered

R1(config)# access-list 10 permit 192.168.1.0 0.0.0.255
```
```
# Named

R1(config)# ip access-list standard MY_LAN
R1(config-std-nacl)# permit 192.168.1.0 0.0.0.255
R1(config-std-nacl)# exit
```

Extended : `access-list acl-number {permit|deny} protocol source wildcard [operator [port]] destination wildcard [operator [port]] [precedence precedence] [tos tos]`

```
# Numbered

access-list 101 permit tcp host 192.168.10.50 host 198.51.100.25 eq 21
access-list 101 permit ip any any precedence critical
access-list 101 permit ip any any tos min-delay
```
```
# Named

R1(config)# ip access-list extended MY_LAN
R1(config-ext-nacl)# remark This is my custom acl configuration
R1(config-std-nacl)# deny ip any host 203.0.113.50                  # Block traffic to host 203.0.113.50
R1(config-std-nacl)# permit udp 192.168.20.0 0.0.0.255 any eq 53    # Allow DNS (UDP 53)
R1(config-std-nacl)# permit tcp any any established                 # Allow only responses to inside-initiated traffic
R1(config-std-nacl)# exit
```

- **precedence** (Indicate priority level of the packet - used for queueing and dropping decision).
- **tos** (describe the desired service type - how the network should handle the packet).

Applying :
```
R1(config)# interface <interface-id>
R1(config-if)# ip access-group {number|name} {in|out}
```

---

## MONITORING & MAINTENANCE

```
show running-configuration | include access-list
show access-list [name | number]
```

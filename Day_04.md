**~ The *Eigrp* of the World~** <sub><sup>by Michael Pye</sup></sub>

---

# Enhanced Interior Gateway Routing Protocol (EIGRP)

| **Protocol** | **Use of AS Number** | **Purpose / Notes**                                                                                                                                         | **Configuration Example**     |
| ------------ | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| **BGP**      | Yes                  | - Identifies routing domains when exchanging routes. <br> - Used for routing policies (e.g., filtering). <br> - Prevents routing loops by tracking AS path. | `router bgp 103`              |
| **EIGRP**    | Yes                  | - AS number identifies EIGRP routing processes. <br> - Routers must be in the **same AS** to exchange routes.                                               | `router eigrp 103`            |
| **OSPF**     | No                   | - Uses **Process number** instead of AS number. <br> - Process number is only locally significant (within the router).                                      | `router ospf 103`             |
| **RIP**      | No direct use        | - Does not use AS numbers or process numbers in the same way as BGP or EIGRP. <br> - Relies on simpler mechanisms for routing updates.                      | `router rip`                  |

**AS NUMBER RANGES**

| **Range**                 | **Value**            | **Use Case**                                              |
| ------------------------- | -------------------- | --------------------------------------------------------- |
| **Well-Known Public**     | `1 – 23455`          | Public ASNs assigned by IANA/RIRs                         |
| **Reserved**              | `23456`              | Reserved for 4-byte to 2-byte ASN transition (AS_TRANS)   |
| **Private ASNs**          | `64512 – 65534`      | For private/internal use (RFC 6996)                       |
| **32-bit (Extended)**     | `65536 – 4294967295` | Public/private use (requires BGP support for 4-byte ASNs) |

- CONFIGURATION
```
R1(config)# router eigrp 1
R1(config-router)# no auto-summary          # disabled by default from ios15+, since auto-summary behaves classfully by default.
R1(config-router)# network 192.168.10.0
R1(config-router)# network 192.168.11.0
R1# show ip eigrp neighbors                 # verify.
R1# show ip eigrp topology                  # show eigrp topology table.
```

- NAMED CONFIGURATION
```
R1(config)# router eigrp network_1
R1(config-router)# address-family ipv4 autonomous-system 1      # set IP version and AS number.
R1(config-router-af)# network 192.168.10.0
```

- PASSIVE INTERFACE
```
R1(config-router)# passive-interface            # Suppresses EIGRP hello packets and routing updates.
R1(config-router)# passive-interface f0/1       # Suppresses EIGRP hello packets and routing updates on f0/1.
```

- OFFSET
```
R1(config)# access-list 100 permit 192.168.10.0
R1(config-router)# offset-list 100 in 10 f0/1       # Applies an offset of 10 to routing metrics on f0/1.
r1(config-router)# metric weights 0 2 0 2 0 0       # modify k1 k2 k3 k4 k5 values of eigrp metric calculation formula.
```

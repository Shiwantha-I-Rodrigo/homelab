**~ Lands of Lost Borders: Out of *BGP* on the Silk Road ~** <sub><sup>by Kate Harris</sup></sub>

---

# BORDER GATEWAY PROTOCOL (BGP)

## SUMMARY

- The routing protocol of the Internet.
- Routes among Autonomous Systems (ASes).
- Autonomous System Number (ASN) is granted by Internet Assigned Numbers Authority (IANA) through five Regional Internet Registries (RIRs).
- BGP uses TCP (port 179) to communicate between routers.
- Key terms
    + **Prefix** : A block of IP addresses (e.g., 8.8.8.0/24).
    + **AS Path** : The list of ASes a route passes through.
    + **Peer** : Another BGP router you exchange routes with.

---

## CONFIGURATIONS

**SETUP BGP**

- To simulate a WAN link between R1 & R2, BGP can be setup.

1. Each router needs an Autonomous System Number (ASN).

    + R1 --> 65001
    + R2 --> 65002

2. Configure BGP on R1.

```
R1(conf)# router bgp 65001
R1(conf-router)# bgp log-neighbor-changes
R1(conf-router)# neighbor 10.0.0.2 remote-as 65002
R1(conf-router)# network 192.168.1.0 mask 255.255.255.0
```

3. Configure BGP on R2.

```
R1(conf)# router bgp 65002
R1(conf-router)# bgp log-neighbor-changes
R1(conf-router)# neighbor 10.0.0.1 remote-as 65001
R1(conf-router)# network 192.168.2.0 mask 255.255.255.0
```

---

## MONITORING & MAINTENANCE

Verify BGP Peering.

```
R1# show ip bgp summary
```
> BGP neighbor should be (10.0.0.2 or 10.0.0.1) in the Established state.

Test Route Exchange.

```
R1# show ip route bgp
```
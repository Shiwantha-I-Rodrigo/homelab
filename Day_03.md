**~The Sorcerer of the Wilde*rip*s~** <sub><sup>by Kai Ashante Wilson</sup></sub>

---

# ROUTING INFORMATION PROTOCOL (RIP)

- INFO
    * A **distance-vector** routing protocol that uses **hop-count** as a metric to determine the best path in a network.
    * An interior gateway protocol.
    * A key limitation of **RIP** is its maximum hop count of 15, which limits maximum network size.
    * Simple to configure, using less processing power and storage compared to complex routing protocols. 

- UPDATES
    * Router running RIP typically broadcast their entire routing table to adjacent routers every 30 seconds.
    * They can also send "triggered updates", immediately if a change in the routing table occurs.

- HOP COUNT
    * The maximum hop count for a valid route is 15.
    * A hop count of 16 or more is considered infinite, which indicates an unreachable network.
    * This limitation is to prevent routing loops. 

- TIMER
    * When a link goes down, a timer starts.
    * A router will wait **180 seconds** for an update from the disconnected neighboring router.
    * If no update is received, the route is marked as **invalid** and will be removed after additional **120 seconds**.

- PROTOCOL TYPE
    * RIP is only aware of the "distance" (hop count) and the "vector" (next-hop router) to each destination.

- DISADVANTEAGE
    * Slow Convergence in large networks.
    * Limited Scalability.
    * Broadcasting the entire routing table is inefficient.
    * It can only use multiple paths if they have the exact same hop count.

- DEFAULTS

| **Feature**                     | **Default Setting**|
| ------------------------------- | - |
| Auto summary                    | Enabled<br>RIPv1 automatically summarizes routes to their classful boundaries (/8, /16, /24), which can cause problems.|
| IP RIP authentication key-chain | No authentication<br>Default mode : clear text|
| IP RIP triggered                | Disabled|
| IP split horizon                | Varies with media : prevents a router from advertising a route back out of the same interface from which it was learned to prevent **LOOPS** |
| Neighbor                        | None defined|
| Network                         | None specified|
| Offset list                     | Disabled|
| Output delay                    | 0 milliseconds : packet is waiting in the output queue of a network device before being transmitted (when communicating with slow devices)|
| Timers basic                    | • Update: 30 seconds : time between sending routing updates<br>• Invalid: 180 seconds : after which a route is declared invalid<br>• Hold-down: 180 seconds : before a route is removed from the routing table<br>• Flush: 240 seconds : time for which routing updates are postponed|
| Validate-update-source          | Enabled|
| Version                         | Receives RIP Version 1 and 2 packets<br>sends Version 1 packets|

- CONFIGURATION

```
R1(config)# ip routing
R1(config)# router rip
R1(config-router)# network 192.168.10.0
R1(config-router)# neighbor 192.168.10.2            # optional
R1(config-router)# timers basic 45 360 400 300      # update invalid holddown flush
R1(config-router)# version 2                        # RIP v2 for individual (interfaces)# ip rip {send | receive} version {1 | 2 | 1 2}
R1# show ip protocols                               # verify
R1(config)# no router rip                           # disable rip
```
**OFFSET**
```
R1(config)# access-list 10 permit 192.168.10.0
R1(config)# router rip
R1(config-router)# offset-list 10 out 2 ethernet0   # add 2 to the hop-count of any route to 192.168.10.0 going out of Ethernet0
```
**AUTHENTICATION**
```
R1(config)# interface f0/1
R1(config-if)# ip rip authentication key-chain trees    # enable authentication
R1(config-if)# ip rip authentication mode md5           # authentication method
R1# show running-config                                 # verify
```
**ROUTE SUMMARIZATION**
```
R1(config)# interface f0/1
R1(config-if)# ip summary-address rip ip address 10.1.1.30 255.255.255.0    # set summary adress manually
R1# show ip interface f0/1                                                  # verify
```
**SPLIT HORIZON**
```
R1(config-if)# no ip split horizon rip                  # disable split horizon ( for hub and spoke topology over a Frame Relay network )
R1# show ip interface f0/1                              # verify
```

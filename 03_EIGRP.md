**~ The *Eigrp* of the World ~** <sub><sup>by Michael Pye</sup></sub>

---

# ENHANCED INTERIOR GATEWAY ROUTING PROTOCOL (EIGRP)

## SUMMARY

| **Protocol** | **Use of AS Number** | **Purpose / Notes**                                                                                                                                         |
| ------------ | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **BGP**      | Yes                  | - Identifies routing domains when exchanging routes. <br> - Used for routing policies (e.g., filtering). <br> - Prevents routing loops by tracking AS path. |
| **EIGRP**    | Yes                  | - AS number identifies EIGRP routing processes. <br> - Routers must be in the **same AS** to exchange routes.                                               |
| **OSPF**     | No                   | - Uses **Process number** instead of AS number. <br> - Process number is only locally significant (within the router).                                      |
| **RIP**      | No direct use        | - Does not use AS numbers or process numbers in the same way as BGP or EIGRP. <br> - Relies on simpler mechanisms for routing updates.                      |

**AS NUMBER RANGES -->**

| **Range**                 | **Value**            | **Use Case**                                              |
| ------------------------- | -------------------- | --------------------------------------------------------- |
| **Well-Known Public**     | `1 – 23455`          | Public ASNs assigned by IANA/RIRs                         |
| **Reserved**              | `23456`              | Reserved for 4-byte to 2-byte ASN transition (AS_TRANS)   |
| **Private ASNs**          | `64512 – 65534`      | For private/internal use (RFC 6996)                       |
| **32-bit (Extended)**     | `65536 – 4294967295` | Public/private use (requires BGP support for 4-byte ASNs) |

**FORWARDING TABLES -->**

| Table              | Purpose                                                                         |
| ------------------ | ------------------------------------------------------------------------------- |
| **Neighbor Table** | Lists directly connected EIGRP routers (neighbors).                             |
| **Topology Table** | Contains ***every learned routes*** from neighbors, including backup routes.    |
| **Routing Table**  | Contains only the *best* routes (successors) used for actual packet forwarding. |

Router A learns about network 10.1.1.0/24 from two neighbors

| Neighbor | Advertised Distance (AD) | Total Metric (FD) | Role                   |
| -------- | ------------------------ | ----------------- | ---------------------- |
| Router B | 100                      | 200               | **Successor**          |
| Router C | 150                      | 250               | **Feasible Successor** |

EIGRP topology table on Router A will list both routes, but only the successor (via Router B) goes into the routing table for active forwarding.

**LOAD BALANCING -->**

balanced : better (lower-metric) routes get more traffic, and less-efficient routes get less.
min : only use the best path for forwarding traffic.

best metric = 1000
variance multiplier = 2
acceptable limit ≤ ( 1000 x 2 )

| Route    | Metric | comparison    | Used  |
| -------- | ------ | ------------- | ----- |
| A (best) | 1000   | 1000 ≤ 2000   | used  |
| B        | 1500   | 1500 ≤ 2000   | used  |
| C        | 2500   | 2500 ≤ 2000   | unused|

**NAMED VS NUMBERED -->**

**Numbered** : You specify the autonomous system (AS) number directly when enabling the process.
- Configuration is done globally under the EIGRP process.
- IPv4 and IPv6 require separate configurations.
- Used on older IOS versions.

**Named** : Instead of an AS number alone, you give the process a name.
- Supports multiple address families (IPv4 and IPv6) under a single named process.
- Allows modular configuration (per-interface, per-address-family).
- Easier to manage dual-stack (IPv4 + IPv6) networks.

---

## CONFIGURATIONS

**ENABLE EIGRP**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 1`                                     | Enables **EIGRP (Enhanced Interior Gateway Routing Protocol)** on the router with **Autonomous System (AS) number 1** — this is the **classic EIGRP configuration mode**.|
| `R1(config-router)# network 172.16.0.0`                          | Specifies the **network** to be advertised by EIGRP. Interfaces with IP addresses within this network will participate in EIGRP routing.|
|||
| `R1(config)# router eigrp virtual-name1`                         | Begins configuration of **Named EIGRP** (newer configuration style) using the **virtual name** `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000` | Enters **address-family configuration mode** for **IPv4** under the **Named EIGRP** instance, using **AS number 45000**.|
| `R1(config-router-af)# network 172.16.0.0`                       | Specifies the **network** to be included in the EIGRP process under the **IPv4 address family** configuration.|

**EIGRP PARAMETERS**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 1`                         | Enables **classic EIGRP** process with **Autonomous System (AS) 1** and enters EIGRP router configuration mode.|
| `R1(config-router)# network 172.16.0.0`              | Advertises the **172.16.0.0** network in EIGRP; interfaces within this range participate in routing.|
| `R1(config-router)# passive-interface`               | Makes an interface **passive**, meaning it will **advertise** the network but **not form EIGRP neighbor adjacencies** on that interface.|
| `R1(config-router)# offset-list 21 in 10 ethernet 0` | Applies an **offset-list** (ACL 21) to **incoming routes** on **Ethernet 0**, adding **10** to the metric of matched routes — useful for route manipulation.|
| `R1(config-router)# metric weights 0 2 0 2 0 0`      | Manually sets **K values** for the EIGRP metric calculation (here: K1=2, K2=0, K3=2, K4=0, K5=0). Determines how bandwidth, delay, etc. affect routing metrics.|
| `R1(config-router)# no auto-summary`                 | Disables **automatic route summarization**, ensuring EIGRP advertises **specific subnet routes** instead of classful networks.|
|||
| `R1(config)# router eigrp virtual-name1`                                    | Starts **Named EIGRP** configuration mode using the instance name `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000`            | Defines the **IPv4 address family** for the EIGRP process, using **AS 45000**.|
| `R1(config-router-af)# network 172.16.0.0`                                  | Specifies the **network** to include in the EIGRP routing process for IPv4.|
| `R1(config-router-af)# metric weights 0 2 0 2 0 0 0`                        | Configures **K values** for the EIGRP metric calculation (includes an extra zero parameter used in Named EIGRP syntax).|
| `R1(config-router-af)# metric rib-scale 100`                                | Adjusts the **scaling factor** applied to EIGRP metrics when installing routes in the **Routing Information Base (RIB)**.|
| `R1(config-router-af)# af-interface gigabitethernet 0/0/1`                  | Enters **address-family interface configuration** mode for **GigabitEthernet 0/0/1**.|
| `R1(config-router-af-interface)# passive-interface`                         | Marks **GigabitEthernet 0/0/1** as **passive**, preventing neighbor relationships on that interface.|
| `R1(config-router-af-interface)# bandwidth-percent 75`                      | Allocates **75%** of the interface bandwidth for EIGRP traffic (hello packets, updates, etc.).|
| `R1(config-router-af-interface)# exit-af-interface`                         | Exits the **address-family interface** configuration mode and returns to address-family configuration.|
| `R1(config-router-af-interface)# topology base`                             | Enters the **base topology** configuration mode, where route-metric and topology-related settings are applied.|
| `R1(config-router-af-topology)# offset-list 21 in 10 gigabitethernet 0/0/1` | Applies an **offset-list (ACL 21)** to **incoming routes** on **GigabitEthernet 0/0/1**, adding **10** to their metrics.|
| `R1(config-router-af-topology)# no auto-summary`                            | Disables **automatic summarization** under the **base topology**, ensuring only specific routes are advertised.|

**ROUTE SUMMARIZATION**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 101`                                | Enables the **classic EIGRP routing process** using **Autonomous System (AS) 101** and enters EIGRP router configuration mode.|
| `R1(config-router)# no auto-summary`                          | Disables **automatic route summarization**, ensuring **subnet-level routes** are advertised instead of classful ones.|
| `R1(config-router)# exit`                                     | Exits EIGRP router configuration mode and returns to global configuration mode.|
| `R1(config)# interface Gigabitethernet 1/0/1`                 | Enters **interface configuration mode** for **GigabitEthernet 1/0/1**.|
| `R1(config-if)# no switchport`                                | Converts the interface to a **Layer 3 interface** (used for routing instead of switching).|
| `R1(config-if)# ip summary-address eigrp 100 0.0.0.0 0.0.0.0` | Configures a **summary route** on the interface for **EIGRP AS 100**, summarizing all routes to a **default route (0.0.0.0/0)**.|
| `R1(config-if)# ip bandwidth-percent eigrp 209 75`            | Limits **EIGRP process 209** to use a maximum of **75% of the interface bandwidth** for EIGRP traffic (updates, hellos, etc.).|
|||
| `R1(config)# router eigrp virtual-name1`                                           | Starts **Named EIGRP** configuration mode using the instance name `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000`                   | Enters the **IPv4 address family** configuration mode under **AS 45000**.|
| `R1(config-router-af)# af-interface ethernet 0/0`                                  | Enters **address-family interface configuration** for **Ethernet 0/0**, allowing interface-specific EIGRP settings.|
| `R1(config-router-af-interface)# summary-address 192.168.0.0 255.255.0.0`          | Configures a **manual summary address** for EIGRP on this interface, summarizing routes in the **192.168.0.0/16** network.|
| `R1(config-router-af-interface)# exit-af-interface`                                | Exits **address-family interface configuration** mode and returns to address-family mode.|
| `R1(config-router-af)# topology base`                                              | Enters the **base topology configuration** mode, where topology-specific and metric parameters are defined.|
| `R1(config-router-af-topology)# summary-metric 192.168.0.0/16 10000 10 255 1 1500` | Defines a **summary route metric** for the **192.168.0.0/16** network — setting custom **bandwidth (10000), delay (10), reliability (255), load (1), and MTU (1500)** values used for route advertisement.|

**EVENT LOGGING**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 1`                         | Enables **classic EIGRP** process with **Autonomous System (AS) 1** and enters EIGRP router configuration mode.|
| `R1(config-router)# eigrp event-log-size 5000`       | Sets the **EIGRP event log buffer size** to **5000 entries**, allowing the router to store more EIGRP-related log events (such as topology changes, updates, and neighbor activity).|
| `R1(config-router)# eigrp log-neighbor-changes`      | Enables **logging of EIGRP neighbor adjacency changes** (up or down events) to the system log — useful for monitoring EIGRP stability.|
| `R1(config-router)# eigrp log-neighbor-warnings 300` | Configures the router to **log a warning** if an EIGRP neighbor has been **silent (no hello packets)** for **300 seconds** — helps detect neighbor communication issues before adjacency loss.|
|||
| `R1(config)# router eigrp virtual-name1`                         | Starts **Named EIGRP** configuration mode using the instance name `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000` | Enters **IPv4 address-family configuration** for the **EIGRP process with AS 45000**.|
| `R1(config-router-af)# eigrp log-neighbor-warnings 300`          | Sets a **300-second neighbor inactivity warning timer** for IPv4 in the Named EIGRP instance.|
| `R1(config-router-af)# eigrp log-neighbor-changes`               | Enables **neighbor change logging** under the IPv4 address family — logs when EIGRP adjacencies go up or down.|
| `R1(config-router-af)# topology base`                            | Enters **base topology configuration mode**, where detailed EIGRP parameters can be tuned.|
| `R1(config-router-af-topology)# eigrp event-log-size 10000`      | Sets the **EIGRP event log buffer size** to **10,000 entries** for the **base topology**, allowing extensive tracking of EIGRP operational events.|


**LOAD BALANCING**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 1`                | Enables **classic EIGRP** with **Autonomous System (AS) 1** and enters EIGRP router configuration mode.|
| `R1(config-router)# traffic-share balanced` | Instructs EIGRP to **distribute traffic proportionally** across multiple feasible successor routes based on their metrics (the default mode). This ensures load sharing according to route cost.|
| `R1(config-router)# maximum-paths 5`        | Allows up to **5 equal-cost or feasible successor routes** to be installed in the routing table for load balancing.|
| `R1(config-router)# variance 1`             | Sets the **variance multiplier** to **1**, meaning only **equal-cost paths** are used for load balancing. (Higher values allow unequal-cost load balancing.)|
|||
| `R1(config)# router eigrp virtual-name1`                         | Starts **Named EIGRP** configuration using the instance name `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000` | Enters the **IPv4 address family** configuration mode for **EIGRP AS 45000**.|
| `R1(config-router-af)# topology base`                            | Enters the **base topology configuration** mode, where routing calculations and load balancing parameters are tuned.|
| `R1(config-router-af-topology)# traffic-share balanced`          | Enables **balanced traffic sharing** across multiple EIGRP paths in the base topology (default behavior).|
| `R1(config-router-af-topology)# maximum-paths 5`                 | Configures EIGRP to use up to **5 parallel routes** for traffic forwarding.|
| `R1(config-router-af-topology)# variance 1`                      | Sets the **variance value** to **1**, limiting EIGRP to only use **equal-cost paths** for load balancing.|

**HELLO PACKETS & HOLD TIME**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 1`                    | Enables **classic EIGRP** process with **Autonomous System (AS) 1** and enters EIGRP router configuration mode.|
| `R1(config-router)# exit`                       | Exits from EIGRP router configuration mode back to global configuration mode.|
| `R1(config)# interface Gigabitethernet 1/0/9`   | Enters **interface configuration mode** for **GigabitEthernet 1/0/9**.|
| `R1(config-if)# no switchport`                  | Converts the interface to **Layer 3 mode** (used for routing instead of switching).|
| `R1(config-if)# ip hello-interval eigrp 109 10` | Sets the **EIGRP hello interval** to **10 seconds** for **EIGRP AS 109** on this interface. Hello packets are used to establish and maintain neighbor relationships.|
| `R1(config-if)# ip hold-time eigrp 109 40`      | Sets the **EIGRP hold time** to **40 seconds** for **AS 109**. If no hello packets are received from a neighbor within this time, the neighbor is declared down.|
|||
| `R1(config)# router eigrp virtual-name1`                         | Starts **Named EIGRP** configuration using the instance name `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000` | Enters **IPv4 address-family configuration mode** for **EIGRP AS 45000**.|
| `R1(config-router-af)# af-interface ethernet 0/0`                | Enters **address-family interface configuration** for **Ethernet 0/0**, where interface-specific EIGRP parameters are set.|
| `R1(config-router-af-interface)# hello-interval 10`              | Sets the **EIGRP hello interval** to **10 seconds** on the Ethernet 0/0 interface for the Named EIGRP instance.|
| `R1(config-router-af-interface)# hold-time 50`                   | Sets the **EIGRP hold time** to **50 seconds** — the duration a router will wait without hearing a hello before declaring the neighbor down.|


**SPLIT HORIZON**

| **Command** | **Description** |
| - | - |
| `R1(config)# router eigrp 1`                   | Enables **classic EIGRP** with **Autonomous System (AS) 1** and enters EIGRP router configuration mode.|
| `R1(config-router)# exit`                      | Exits from EIGRP router configuration mode back to global configuration mode.|
| `R1(config)# interface Ethernet 0/1`           | Enters **interface configuration mode** for **Ethernet 0/1**.|
| `R1(config-if)# no ip split-horizon eigrp 101` | **Disables split horizon** for **EIGRP AS 101** on this interface. Normally, split horizon prevents a router from advertising routes back out of the interface on which they were learned — disabling it is common on **hub interfaces** in **hub-and-spoke topologies** (like Frame Relay or DMVPN).|
|||
| `R1(config)# router eigrp virtual-name1`                         | Starts **Named EIGRP** configuration using the instance name `virtual-name1`.|
| `R1(config-router)# address-family ipv4 autonomous-system 45000` | Enters **IPv4 address-family configuration mode** for **EIGRP AS 45000**.|
| `R1(config-router-af)# af-interface ethernet 0/0`                | Enters **address-family interface configuration mode** for **Ethernet 0/0**, where per-interface EIGRP parameters are defined.|
| `R1(config-router-af-interface)# no split-horizon`               | **Disables split horizon** under the Named EIGRP configuration for this interface, allowing learned routes to be advertised back out the same interface (used in multipoint or hub-and-spoke designs).|
| `R1(config-router-af-interface)# no next-hop-self no-ecmp-mode`  | Disables both **next-hop-self** (prevents the router from changing the next-hop address in EIGRP updates) and **ECMP mode** (Equal-Cost Multi-Path) for this interface — often used for advanced traffic engineering or when preserving original next-hop information is required.|

**REDISTRIBUTION**

| Command | Description |
| - | - |
| `R1(config)# router eigrp 1`                              | Enters EIGRP configuration mode for autonomous system (AS) 1.|
| `R1(config-router)# network 172.16.0.0`                   | Specifies the networks that EIGRP will advertise and participate in.|
| `R1(config-router)# redistribute rip`                     | Redistributes routes learned from RIP into EIGRP.|
| `R1(config-router)# distance eigrp 80 130`                | Sets the administrative distance of EIGRP to 80 for internal and 130 for external routes.|
| `R1(config-router)# default-metric 1000 100 250 100 1500` | Defines a default metric for routes redistributed into EIGRP: bandwidth=1000, delay=100, reliability=250, load=100, MTU=1500.|

---

## MONITORING & MAINTENANCE

| **Command** | **Description** |
| - | - |
| `R1# show ip eigrp neighbors`             | Displays EIGRP neighbors, their hold times, SRTT, and uptime.|
| `R1# show ip eigrp neighbors detail`      | Shows detailed neighbor info, including authentication and stub settings.|
| `R1# show ip eigrp interfaces`            | Lists interfaces participating in EIGRP and their parameters.|
| `R1# show ip eigrp interfaces detail`     | Displays detailed EIGRP interface information.|
| `R1# show ip eigrp topology`              | Displays EIGRP topology table (successor and feasible successor routes).|
| `R1# show ip eigrp topology all-links`    | Shows all known routes, even those not meeting the feasibility condition.|
| `R1# show ip route eigrp`                 | Displays routes learned through EIGRP in the routing table.|
| `R1# show ip eigrp traffic`               | Shows EIGRP packet counts (Hellos, Updates, Queries, Replies, ACKs).|
| `R1# show ip protocols`                   | Displays active routing protocols, EIGRP AS number, networks, and timers.|
| `R1# show ip interface <interface>`       | Verifies if EIGRP authentication (MD5/HMAC) is configured.|
| `R1# show ip eigrp neighbors detail`      | Confirms if a router or neighbor is configured as a stub.|
| `R1# show run\| section eigrp`            | Checks which interfaces are passive (not forming adjacencies).|
| `R1# show ip eigrp topology <network>`    | Displays feasible distance (FD), advertised distance (AD), and next hops.|
| `R1# show ip eigrp events`                | chronological log of key EIGRP process activities.|
| `R1# show ip eigrp vrf VRF1 accounting`   | displays prefix accounting information for the EIGRP processe|
| `R1# show ipv6 eigrp neighbors`           | Displays IPv6 EIGRP neighbors.|
| `R1# show ipv6 eigrp topology`            | Shows IPv6 EIGRP topology table.|
| `R1# show ipv6 route eigrp`               | Lists routes learned via EIGRP for IPv6.|
| `R1# show ipv6 protocols`                 | Displays active IPv6 routing protocols and parameters.|
| `R1# debug eigrp packets`                 | Displays real-time EIGRP packet exchanges.|
| `R1# debug eigrp neighbors`               | Monitors neighbor formation and loss.|
| `R1# debug eigrp fsm`                     | Tracks EIGRP finite state machine transitions (route state changes).|
| `R1# debug eigrp summary`                 | Summarizes EIGRP activity.|
| `R1# undebug all` or `R1# u all`          | Turns off all debugging.|
| `R1# clear ip eigrp neighbors`            | Resets EIGRP neighbor adjacencies manually.|
| `R1# clear ip eigrp <AS-number> process`  | Restarts the entire EIGRP process (rebuilds adjacencies and topology).|

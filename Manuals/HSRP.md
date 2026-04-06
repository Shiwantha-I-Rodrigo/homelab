# HSRP (Hot Standby Router Protocol)

## SUMMARY

- HSRP is a Cisco proprietary redundancy protocol used in routing and switching to ensure network gateway high availability.
- HSRP allows multiple routers (or Layer 3 switches) to work together to present one virtual default gateway to end devices.

**HOW IT WORKS**

1. Routers are grouped into an HSRP group (usually 2 routers).
2. They share a virtual IP address and a virtual MAC address that hosts use as their default gateway.
3. Within the group:
    - One router is the Active Router (forwards traffic).
    - Another is the Standby Router (takes over if the active fails).
    - Others (if any) are in Listen state (monitoring).
4. Routers exchange hello messages (UDP port 1985) to monitor each other’s status.

**HSRP STATES**

| **State**   | **Description**                                   |
| ----------- | ------------------------------------------------- |
| **Initial** | Router not yet participating in HSRP.             |
| **Learn**   | Waiting to learn the virtual IP address.          |
| **Listen**  | Aware of HSRP group but not active/standby.       |
| **Speak**   | Sending and receiving hello messages.             |
| **Standby** | Backup router ready to take over if active fails. |
| **Active**  | Currently forwarding traffic for the virtual IP.  |

**KEY CONCEPTS**

| **Term**        | **Description**                                                                         |
| --------------- | --------------------------------------------------------------------------------------- |
| **Virtual IP**  | Shared IP address used as the default gateway for hosts.                                |
| **Virtual MAC** | MAC address associated with the HSRP group.                                             |
| **Hello Timer** | Default 3 seconds — interval between hello messages.                                    |
| **Hold Timer**  | Default 10 seconds — how long a router waits before assuming the active router is down. |
| **Priority**    | Determines which router becomes active (higher = more preferred). Default is 100.       |
| **Preemption**  | Allows a higher-priority router to take back the active role once it’s back online.     |

| **HSRP Version** | **Description**                                                                      |
| ---------------- | ------------------------------------------------------------------------------------ |
| **HSRPv1**       | Default version; uses multicast address 224.0.0.2 and UDP port 1985.                 |
| **HSRPv2**       | Supports IPv6, more groups (0–4095), and uses multicast 224.0.0.102 / UDP port 1985. |

---

## CONFIGUARATION

R1
```
R1(config-if)# ip address 192.168.1.2 255.255.255.0
R1(config-if)# standby 1 ip 192.168.1.1
R1(config-if)# standby 1 priority 110
R1(config-if)# standby 1 preempt
```

R2
```
R2(config-if)# ip address 192.168.1.3 255.255.255.0
R2(config-if)# standby 1 ip 192.168.1.1
```

---

## MONITORING & MAINTENANCE

| **Category**                         | **Command**                                              | **Purpose / Description**                                                                                          |
| ------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **View Specific Interface or Group** | `show standby interface <interface>`                     | Shows HSRP status, role, and timers for one interface (e.g., `show standby interface gig0/1`).                     |
|                                      | `show standby <interface> <group-number>`                | Displays detailed info about a specific HSRP group — priorities, virtual MAC, timers, and preemption.              |
| **Verify IP and MAC Associations**   | `show ip interface brief`                                | Confirms physical and virtual IP addresses.                                                                        |
|                                      | `show arp`                                               | Verifies that the virtual IP and virtual MAC are present in the ARP table.                                         |
|                                      | `show mac address-table`                                 | Confirms which port is forwarding the HSRP virtual MAC.                                                            |
| **View Interface and System Logs**   | `show logging`                                           | Displays system log messages (useful for detecting HSRP state changes).                                            |
|                                      | `debug standby events`                                   | Shows detailed real-time HSRP events (use with caution in production).                                             |
|                                      | `debug standby packets`                                  | Displays HSRP hello and state change packets (troubleshooting only).                                               |
| **Maintenance Commands**             | `clear standby counters`                                 | Resets HSRP statistics and counters for a fresh baseline.                                                          |
|                                      | `clear arp-cache`                                        | Clears ARP table — useful if virtual MAC/IP inconsistencies occur after failover.                                  |
|                                      | `no standby <group> preempt` / `standby <group> preempt` | Enable or disable preemption behavior for maintenance or testing.                                                  |

**COMMON CHECKS -->**

| **What to Verify**   | **Command**          | **Expected Result**                                                   |
| -------------------- | -------------------- | --------------------------------------------------------------------- |
| Active/Standby roles | `show standby brief` | One router shows **Active**, the other **Standby**.                   |
| Virtual IP address   | `show standby`       | Same on both routers.                                                 |
| Priority values      | `show standby`       | Active router has the **higher** priority (if preemption is enabled). |
| Hello/Hold timers    | `show standby`       | Should match on both routers.                                         |
| Virtual MAC address  | `show standby`       | Same on both routers; changes if group number changes.                |
| Failover test        | `ping <gateway IP>`  | Should remain reachable even when active router goes down.            |

**~ The King’s Guard ~** <sub><sup>by Rae Carson</sup></sub>

---

# PORT SECURITY

## SUMMARY

- Restricts input to an interface by limiting and identifying MAC addresses of the devices allowed to access the port.
- Key Purposes :
    + Prevent unauthorized devices from connecting.
    + Limit the number of devices per port.
    + Protect against MAC flooding attacks.

| Feature                 | Description                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **Static MAC Address**  | Manually configure which MAC addresses can connect.                                  |
| **Dynamic MAC Address** | Switch learns MAC addresses dynamically (up to a maximum limit).                     |
| **Sticky MAC Address**  | Switch dynamically learns MAC addresses and saves them in the running configuration. |
| **Violation Modes**     | What happens when a violation occurs: `protect`, `restrict`, or `shutdown`.          |

Violation Modes :
1. **Protect** – Drops packets from unauthorized MACs; no log generated.
2. **Restrict** – Drops packets and generates a log/alert.
3. **Shutdown** – Puts the port in an error-disabled state (most secure).

---

## CONFIGURATIONS

| **Command**                                                            | **Purpose / Description**                                                                                                                                                                                   |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `switchport mode access`                                               | Sets the port as an **access port** (used for end devices, not trunking).                                                                                                                                   |
| `switchport port-security`                                             | **Enables port security** on the interface.                                                                                                                                                                 |
| `switchport port-security maximum 2`                                   | Limits the port to **a maximum of 2 MAC addresses**.                                                                                                                                                        |
| `switchport port-security aging type {inactive \| absolute}`           | Defines how learned MAC addresses age out:<br>• **inactive** – removes MACs that are not actively sending traffic.<br>• **absolute** – removes MACs after the configured aging time regardless of activity. |
| `switchport port-security aging time 5`                                | Sets the **aging time** for secure MAC addresses to **5 minutes**.                                                                                                                                          |
| `switchport port-security violation {protect \| restrict \| shutdown}` | Defines the **action taken on a security violation**:<br>• **protect** – drops violating traffic silently.<br>• **restrict** – drops and logs the violation.<br>• **shutdown** – disables the port.         |
| `switchport port-security mac-address sticky`                          | Enables **sticky MAC learning**, allowing dynamically learned MACs to be stored in the running configuration.                                                                                               |

---

## MONITORING & MAINTENANCE

| **Command**                          | **Purpose / Description**                                                                                                                                                                                                                                                                                       |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `show port-security`                 | Displays **overall port security status** on the switch, including:<br>• Number of secure ports<br>• Number of security violations<br>• Maximum allowed MAC addresses per port<br>• Total number of secure MAC addresses learned.                                                                               |
| `show port-security interface fa0/1` | Displays **detailed port security information** for a specific interface (`fa0/1`), including:<br>• Port security status (enabled/disabled)<br>• Violation mode (protect/restrict/shutdown)<br>• Maximum and current secure MAC addresses<br>• Number of security violations<br>• Aging type and time settings. |

---
---
---

# DHCP SNOOPING

- Acts as a **LAYER 2** firewall between **untrusted hosts** and **trusted DHCP servers**.
- DHCP Snooping categorizes switch ports into two types:
    + **Trusted Port** : Allowed to send DHCP server responses (DHCPOFFER, DHCPACK). Typically uplinks to legitimate DHCP servers.
    + **Untrusted Port** : Allowed to send DHCP requests only (from clients). DHCP server messages from these ports are **blocked**.
- This can help prevent :
    + Rogue (unauthorized) DHCP servers handing out fake IP addresses.
    + DHCP starvation attacks (where an attacker exhausts all IP addresses in a pool).
- Process :
    1. A DHCP client sends a DHCPDISCOVER message (broadcast).
    2. The switch receives it and forwards it to trusted ports only.
    3. The DHCP server (on a trusted port) replies with an offer (DHCPOFFER).
    4. The switch inspects and records the binding (MAC, IP, VLAN, Interface) in the DHCP Snooping Binding Table.
    5. If an untrusted port sends a server message (DHCPOFFER or DHCPACK), the switch drops it.
- Snooping Binding Table : `show ip dhcp snooping binding`
    + MAC address
    + IP address
    + VLAN
    + Interface
    + Lease time
- binding table is used by other security features like Dynamic ARP Inspection (DAI) and IP Source Guard (IPSG).

## CONFIGURATION

| **Command**                                                                       | **Description**                                                                              |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `S1(config)# ip dhcp snooping`                                                    | Turns on DHCP snooping for the entire switch.                                                |
| `S1(config)# ip dhcp snooping vlan 10`                                            | DHCP snooping on VLAN 10.                                                                    |
| `S1(config-if)# ip dhcp snooping trust`                                           | Allows DHCP messages from servers on this port; untrusted ports only accept client messages. |
| `S1(config-if)# switchport mode trunk`<br>`S1(config-if)# ip dhcp snooping trust` | Ensures DHCP server messages can pass through the trunk.                                     |
| `S1(config-if)# ip dhcp snooping limit rate 10`                                   | Prevents DHCP starvation attacks by limiting the number of DHCP packets per second.          |

## MONITORING & MAINTENANCE

| **Command**                                                                                          | **Description**                                                                                          |
| ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `S1# show ip dhcp snooping`                                                                          | Shows global DHCP snooping status, VLANs under protection, trusted interfaces, and rate limits per port. |
| `S1# show ip dhcp snooping binding`                                                                  | Displays the learned IP–MAC–VLAN–Interface mappings from DHCP leases.                                    |
| `S1# show ip dhcp snooping trust`                                                                    | Shows which interfaces are configured as trusted.                                                        |
| `S1# show ip dhcp snooping database`                                                                 | Displays the DHCP snooping database file location, write delay interval, and number of bindings stored.  |
| `S1# show ip dhcp snooping statistics`                                                               | Displays counts of DHCP packets dropped or forwarded per interface.                                      |
| `S1# clear ip dhcp snooping binding`                                                                 | Clears the DHCP snooping binding table.                                                                  |
| `S1(config)# ip dhcp snooping binding 00:1A:2B:3C:4D:5E vlan 10 ip 192.168.10.5 interface Gi0/10`    | Manually adds a static DHCP binding.                                                                     |
| `S1(config)# no ip dhcp snooping binding 00:1A:2B:3C:4D:5E vlan 10 ip 192.168.10.5 interface Gi0/10` | Removes a manually added static DHCP binding.                                                            |
| `S1(config)# ip dhcp snooping database flash:dhcp_snoop.db`                                          | Specifies the file to store DHCP binding info for recovery after reload.                                 |
| `S1(config)# ip dhcp snooping database write-delay 60`                                               | Sets the write delay interval (in seconds) for updating the DHCP snooping database.                      |

---
---
---

# DYNAMIC ARP INSPECTION

- Often used alongside DHCP Snooping.
- DAI protects your network from ARP spoofing or ARP poisoning attacks.
- ARP (Address Resolution Protocol) is trust-based. any device can send an ARP reply saying, **“Hey, IP 10.0.0.1 is at MAC aa:bb:cc:dd:ee:ff”**.
- DAI intercepts and validates all ARP packets on untrusted ports.
- It compares the MAC and IP addresses in the ARP packet against the DHCP Snooping binding table ( trusted IP–MAC pairs learned from DHCP ).
    + If the ARP info matches the DHCP Snooping database → packet is forwarded.
    + If it doesn’t match → packet is dropped, and a log entry is generated.
- DAI depends on DHCP Snooping being enabled first.

## CONFIGUARATION

| **Command**                                      | **Description**                                                                        |
| ------------------------------------------------ | -------------------------------------------------------------------------------------- |
| `S1(config)# ip arp inspection vlan 10`          | Enables Dynamic ARP Inspection (DAI) on VLAN 10.                                       |
| `S1(config-if)# ip arp inspection trust`         | Marks the interface as trusted so ARP packets from this port are allowed.              |
| `S1(config-if)# ip arp inspection limit rate 15` | Limits the number of ARP packets per second on untrusted ports to prevent ARP attacks. |

## MONITORING & MAINTENANCE

| **Command**                                                                                                                                                                                | **Description**                                                                |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| `S1# show ip arp inspection`                                                                                                                                                               | Displays the global DAI status.                                                |
| `S1# show ip arp inspection statistics`                                                                                                                                                    | Shows DAI statistics, including dropped and forwarded ARP packets.             |
| `S1# show ip arp inspection interface GigabitEthernet0/10`<br>`S1# show ip arp inspection vlan 10`                                                                                         | Displays DAI configuration for a specific interface or VLAN.                   |
| `S1# clear ip arp inspection statistics`                                                                                                                                                   | Clears DAI counters or statistics.                                             |
| `S1# show interfaces status err-disabled`<br>`S1(config)# errdisable recovery cause arp-inspection`<br>`S1(config)# errdisable recovery interval 30`                                       | Resets error-disabled ports triggered by DAI and sets the recovery interval.   |
| `S1(config)# arp access-list STATIC-HOSTS`<br>`S1(config-arp-nacl)# permit ip host 192.168.10.5 mac host 00:1A:2B:3C:4D:5E`<br>`S1(config)# ip arp inspection filter STATIC-HOSTS vlan 10` | Applies ARP ACLs to allow static hosts not in the DHCP snooping binding table. |

---
---
---

# IP SOURCE GUARD

- Allow traffic only from IP addresses that are known and valid for a given port.
- A host cannot use a fake IP address to impersonate another device on the network.
- Relies on DHCP Snooping Table or Static Bindings.
- IPSG is applied per interface, usually on access ports.
- Works with Dynamic ARP Inspection and DHCP Snooping for layered security.

## CONFIGURATION

| **Command**                                                                             | **Description**                                                                                                      |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `S1(config-if)# ip verify source`                                                       | Enables IP Source Guard on the interface to validate IP addresses against the DHCP snooping or static binding table. |
| `S1(config)# ip source binding 192.168.10.5 00:1A:2B:3C:4D:5E vlan 10 interface Gi0/10` | Manually creates a static IP–MAC–VLAN–interface binding for IP Source Guard on a specific host.                      |

## MONITORING & MAINTENANCE

| **Command**                                             | **Description**                                                                                |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `S1# show ip verify source`                             | Displays global IP Source Guard status and per-interface configuration.                        |
| `S1# show ip source binding`                            | Shows the IP–MAC–VLAN–interface bindings used by IPSG (from DHCP snooping or static bindings). |
| `S1# clear ip source binding`                           | Clears all static IPSG bindings or refreshes dynamic entries.                                  |
| `S1# show interfaces status err-disabled`               | Checks if any ports were error-disabled due to IPSG violations.                                |
| `S1(config)# errdisable recovery cause ip-source-guard` | Enables automatic recovery of error-disabled ports caused by IPSG violations.                  |
| `S1(config)# errdisable recovery interval <seconds>`    | Sets the interval for automatically re-enabling error-disabled ports.                          |

---
---
---

# PROTECTED PORTS

- Feature of Cisco switches that restrict communication between certain ports.
- Ports that are configured as protected cannot communicate with other protected ports on the same switch.
- Protected ports can still communicate with unprotected ports.
- Protected ports work at the switching layer (2).
- Often used to isolate hosts on the same VLAN without requiring private VLANs.
- Traffic destined for a router or Layer 3 interface is unaffected.

## CONFIGURATION

| **Command**                                             | **Description**                            |
| ------------------------------------------------------- | ------------------------------------------ |
| `S1(config-if-range)# switchport protected`             | Set protected port.                        |

## MONITORING

| **Command**                           | **Description**                                          |
| --------------------------------------| -------------------------------------------------------- |
| `S1# show running-config`             | If switchport protected is listed, the port is protected.|

---
---
---

# SPANNING TREE ROOT GUARD

- Layer 2 security feature on Cisco switches that prevents a designated port from becoming the root port.
- Prevent allowing a superior Bridge Protocol Data Unit (BPDU) to take over as the root bridge in STP.
- Normally, any switch can become root if it sends BPDUs claiming a lower bridge ID.
- Root Guard blocks ports that should never become root, protecting the network topology.
    + Prevent accidental misconfigurations.
    + Ensure that only designated switches (like core switches) become root in a network.
- If the port receives a superior BPDU claiming a better root bridge.
    + The port goes into a “root-inconsistent” state.
    + It stops forwarding traffic but still receives BPDUs.
    + Once the superior BPDU stops, the port returns to normal.

## CONFIGURATION

| **Command**                                             | **Description**                            |
| ------------------------------------------------------- | ------------------------------------------ |
| `S1(config-if)# spanning-tree guard root`               | Now, this port cannot become a root port.  |

## MONITORING & MAINTENANCE

| **Command**                                             | **Description**                            |
| ------------------------------------------------------- | ------------------------------------------ |
| `S1# show spanning-tree inconsistentports`              | Check status of Root Guard on a port.      |
| `S1# show spanning-tree interface fa0/10 detail`        | Check interface STP status.                | 

---
---
---

# STP BPDU GUARD

| Feature        | Purpose                                   | Behavior                                   |
| -------------- | ----------------------------------------- | ------------------------------------------ |
| **Root Guard** | Prevent a port from becoming root         | Puts port in root-inconsistent state       |
| **BPDU Guard** | Protect access ports from receiving BPDUs | Shuts down port immediately (err-disabled) |

- BPDU Guard prevents.
    + Accidental loops: If someone connects a switch to an access port, it could cause loops.
    + Malicious attacks: A rogue switch could claim root bridge or send BPDUs to destabilize STP.
    + Topology instability: Only designated switches should participate in STP.
- To restore the port, it must be manually or automatically re-enabled.

| **Command**                                            | **Description**                            |
| ------------------------------------------------------ | ------------------------------------------ |
| `S1(config)# spanning-tree portfast bpduguard default` | Enable BPDU Guard globally on access port. |
| `S1(config-if)# spanning-tree bpduguard enable`        | Enable BPDU Guard on a single interface.   | 

## MONITORING & MAINTENANCE

| **Command**                                                                                | **Description**                                        |
| -------------------------------------------------------------------------------------------| ------------------------------------------------------ |
| `S1# show err-disabled`                                                                    | Check which ports are err-disabled due to BPDU Guard.  |
| `S1(config)# interface fa0/10`<br>`S1(config-if)# shutdown`<br>`S1(config-if)# no shutdown`| Restore err-disabled port.|

---
---
---

# STORM CONTROL

- Layer 2 switch feature that prevents network traffic storms caused by broadcast, multicast, or unknown unicast traffic.
- You set a threshold (as a percentage of the interface bandwidth or in packets per second).
- If traffic exceeds the threshold.
    + The port drops excessive frames of that type.
    + It can also generate logs, SNMP traps, or notifications.
- Storm Control **does not** completely shut down the port, unless configured to do so.

| **Command**                                          | **Purpose**                                                                                                       |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `S1(config-if)# storm-control broadcast level 10.00` | Sets the broadcast traffic threshold to 10% of the interface bandwidth. Excess traffic beyond this is suppressed. |
| `S1(config-if)# storm-control action shutdown`       | Configures the port to go into **err-disabled** (shutdown) state if the storm threshold is exceeded.              |

| **Command**                              | **Purpose**                                                                                                                                                  |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `S1# show storm-control`                 | Displays **storm control configuration and status** for all interfaces on the switch. Shows thresholds, traffic type monitored, and statistics.              |
| `S1# show storm-control interface fa0/1` | Displays **storm control configuration and status for a specific interface** (Fa0/1), including thresholds, packets suppressed, and monitored traffic types. |

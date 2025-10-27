# SETUP

**DATE : 2024/01/03**

## EQUIPMENT

1. Router Cisco 1841 (64MB)
2. Router Cisco 1841 adventerprise k9 (64MB)
3. Switch Catalyst WS-C3750V2-24TS (24x)
4. Switch Catalyst WS-C2960-24TC-S (24x)

## SETUP

```
R1 : 	fastethernet0/0 : ip - 192.168.10.1 / 255.255.255.0 (R1-R2)
	    fastethernet0/1 : ip - 192.168.11.1 / 255.255.255.0 (R1-S1)
R2 : 	fastethernet0/0 : ip - 192.168.10.2 / 255.255.255.0 (R2-R1)
	    fastethernet0/1 : ip - 192.168.12.1 / 255.255.255.0 (R2-S2)

S1 :    vlan1 : ip - 192.168.11.2 / 255.255.255.0
S2 :    vlan1 : ip - 192.168.12.2 / 255.255.255.0

PC1 : dev : enp3s0 : ip - 192.168.11.3
PC2 : dev : enp3s0 : ip - 192.168.12.3
```

![setup](/images/initial.png)

## CREDENTIALS ( • ᴗ - ) ✧

**ENABLE** : enable\
**USER** : admin\
**PASSWORD** : admin\
**DOMAIN** : admin.com

## FACTORY RESET

0. Establish a console connection and open the cli.
1. Power off the switch while keeping the console cli open.
2. Power on the switch & hold the mode button until recovery mode is shown on the cli.
3. `flash_init`
4. `dir flash:`
5. `del flash:vlan.dat`
6. `del flash:config.text`
7. `del flash:private-config.text` (if present)
8. `boot`

## INITIAL CONFIGURATION

```
Common >

configure terminal                        |enter config mode
(config)hostname NAME                     |change host name
(config)enable secret PASSWORD            |set enable password
(config)no ip domain-lookup               |stop looking for domains when a command is unrecognized

Switch S1 >

S1(config)# interface vlan 1                            |create a virtual host on the switch
S1(config-if)# ip address 192.168.11.2  255.255.255.0   |assign an IP address
S1(config-if)# no shut                                  |must turn it on
S1(config-if)# exit                                     |leave interface config and return to global config
S1(config)# ip default-gateway 192.168.11.1             |must be on same subnet as Mgt interface

Router R1 >

R1(config)# interface f0/1                              |enter configuration mode for an interface
R1(config-if)# ip address 192.168.11.1 255.255.255.0    |assign the IP Address and subnet mask
R1(config-if)# no shut                                  |turn on the interface
R1(config)# interface f0/0                              |enter configuration mode for an interface
R1(config-if)# ip address 192.168.10.1 255.255.255.0    |assign the IP Address and subnet mask
R1(config-if)# no shut                                  |turn on the interface

Router R2 >

R1(config)# interface f0/1                              |enter configuration mode for an interface
R1(config-if)# ip address 192.168.12.1 255.255.255.0    |assign the IP Address and subnet mask
R1(config-if)# no shut                                  |turn on the interface
R1(config)# interface f0/0                              |enter configuration mode for an interface
R1(config-if)# ip address 192.168.10.2 255.255.255.0    |assign the IP Address and subnet mask
R1(config-if)# no shut                                  |turn on the interface

Switch S2 >

S1(config)# interface vlan 1                            |create a virtual host on the switch
S1(config-if)# ip address 192.168.12.2  255.255.255.0   |assign an IP address
S1(config-if)# no shut                                  |must turn it on
S1(config-if)# exit                                     |leave interface config and return to global config
S1(config)# ip default-gateway 192.168.12.1             |must be on same subnet as Mgt interface
```

# FUTURE PLANS

| **Category**                              | **Task**                             | **Type**  | **Description / Goal**                                                                         |
| ----------------------------------------- | ------------------------------------ | --------- | ---------------------------------------------------------------------------------------------- |
| **Networking Fundamentals**               | Explain OSI Model                    | Explain   | Understand the 7-layer model (Physical to Application) and map common protocols to each layer. |
|                                           | Implement Subnet                     | Implement | Practice subnetting IPv4 networks, calculate subnets, host ranges, and masks.                  |
|                                           | Explain CIDR Notation                | Explain   | Learn Classless Inter-Domain Routing and its use in efficient IP allocation.                   |
|                                           | Explain IPv4 vs IPv6                 | Explain   | Compare addressing structure, header format, and routing behavior.                             |
| **Routing Protocols**                     | Explain / Implement RIP              | Both      | Study distance-vector operation, hop count metric, and configure RIP in lab.                   |
|                                           | Explain / Implement OSPF             | Both      | Explore link-state protocol, LSAs, areas, DR/BDR roles. Configure OSPF in topology.            |
|                                           | Explain / Implement EIGRP            | Both      | Examine Cisco proprietary hybrid protocol. Configure metrics, neighbors, and authentication.   |
|                                           | Explain / Implement BGP              | Both      | Study exterior gateway protocol, ASNs, path selection, and peer configuration.                 |
| **Layer 2 Technologies**                  | Implement VLAN                       | Implement | Create VLANs, assign ports, and test inter-VLAN routing.                                       |
|                                           | Implement VxLAN                      | Implement | Learn overlay networking, MAC-in-UDP encapsulation, and configure in lab.                      |
|                                           | Implement Multicast (PIM/IGMP)       | Implement | Practice multicast group membership (IGMP) and routing (PIM Sparse/Dense).                     |
|                                           | Explain STP                          | Explain   | Understand Spanning Tree Protocol operation, root bridge, port roles, and convergence.         |
|                                           | Explain Spine-Leaf Architecture      | Explain   | Study modern data center design with ECMP and horizontal scalability.                          |
| **Network Security**                      | Implement / Explain Port Security    | Both      | Restrict MAC addresses per port, understand violation modes.                                   |
|                                           | Implement Cisco Firewall             | Implement | Configure access rules, NAT, inspection, and policies.                                         |
|                                           | Implement NAT                        | Implement | Practice static, dynamic, and PAT configurations.                                              |
|                                           | Implement ACL                        | Implement | Create standard and extended ACLs for traffic filtering.                                       |
|                                           | Implement VPN (IPSec / SSL / VPLS)   | Implement | Study encryption, tunneling, and deploy site-to-site and remote-access VPNs.                   |
|                                           | Explain SSID / WPA2 / Channels       | Explain   | Wireless fundamentals: network names, security types, and frequency management.                |
| **Network Tools**                         | ip, ss, ifconfig, route              | Command   | Learn Linux network configuration and interface management tools.                              |
|                                           | iptables                             | Command   | Configure firewall rules and packet filtering in Linux.                                        |
|                                           | nmcli                                | Command   | Use NetworkManager CLI for interface and connection control.                                   |
|                                           | ping, traceroute                     | Command   | Basic connectivity and path verification.                                                      |
|                                           | netstat                              | Command   | View active connections, ports, and socket stats.                                              |
|                                           | nslookup                             | Command   | DNS lookup and troubleshooting.                                                                |
|                                           | tcpdump, Wireshark                   | Command   | Packet capture and analysis.                                                                   |
|                                           | Read forwarding/routing tables       | Command   | Understand routing logic and kernel routing tables.                                            |
|                                           | Read headers with Wireshark/tcpdump  | Analyze   | Decode Ethernet, IP, TCP/UDP headers.                                                          |
|                                           | Check ports with netcat/nmap         | Analyze   | Scan open ports and test connectivity.                                                         |
| **Monitoring & Logging**                  | Explain SNMP                         | Explain   | Learn Simple Network Management Protocol architecture and versions.                            |
|                                           | Implement SYSLOG                     | Implement | Centralize logs from network devices.                                                          |
|                                           | Implement NETFLOW                    | Implement | Capture flow data for traffic analysis.                                                        |
| **Redundancy & Load Balancing**           | Implement HSRP                       | Implement | Configure Hot Standby Router Protocol for gateway redundancy.                                  |
|                                           | Implement VRRP                       | Implement | Practice open-standard redundancy equivalent of HSRP.                                          |
|                                           | Implement Load Balancing with Trunk  | Implement | Aggregate links and balance traffic using EtherChannel or LACP.                                |
| **Automation & Configuration Management** | Implement Ansible                    | Implement | Automate network configuration and validation.                                                 |
|                                           | Implement Python / Jinja2 Management | Implement | Build scripts and templates for network automation.                                            |
| **Core Network Services**                 | Implement DHCP                       | Implement | Set up DHCP server for dynamic IP allocation.                                                  |
|                                           | Implement DNS                        | Implement | Configure DNS zones, records, and resolution testing.                                          |
| **Advanced Networking**                   | Explain MPLS (RSVP / LDP)            | Explain   | Study label-switched paths, label distribution, and QoS integration.                           |

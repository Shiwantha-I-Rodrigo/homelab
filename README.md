# NOC / SOC HomeLAB

### **Hardware**

+ **Cisco Catalyst 1841 Routers** x 2
+ **Cisco Catalyst 2960 Switch** x 1
+ **Cisco Catalyst 3750 Switch** x 1
+ **Network Interface Cards** x 3
+ **Acer Aspire 7 Laptop** x 1

![lab](images/lab1.png)

### **Software**

* Operating Systems
    + Arch Linux (Host/Attacker)
    + IPFire (Firewall)
    + Metasploitable2 (Endpoint)
* KVM/QEMU (Virtualization Platform)
* Wireshark (Packet Analyzer)
* Splunk (SIEM)
* Suricata (IPS/IDS)
* Cisco Packet Tracer (for Large Network Simulations)

### **Base Network Topography**

![lab](images/seclab.png)

---

# Networking_Fundamentals

* OSI_&_TCP/IP
* IPv4+IPv6
* VLANs,STP,EtherChannel
* Routing:OSPF,EIGRP,BGP
* NAT,ACLs,DHCP

## Labs

### Core_Networking

* [1-01_Multi-router_topology_(static+dynamic_routing)](/Networking_Fundamentals_Labs/1-01%20Multi-router%20topology.md)
* 1-02_VLANs+inter-VLAN_routing
* 1-03_ACLs+firewall_simulation

### Switching_&_Layer_2

* 1-04_Break_STP_and_recover_(root_priority)
* 1-05_Port_Security_(sticky_MAC,violations),BPDU_Guard,PortFast
* 1-06_Capture_STP,ARP,DHCP_in_Wireshark
* 1-07_Simulate_rogue_switch_(STP_behavior)

### Routing_&_Traffic_Flow

* 1-08_Routing_black_holes+troubleshooting_(`traceroute`,CEF)
* 1-09_Route_redistribution_(OSPF<->EIGRP<->BGP)
* 1-10_DHCP_relay_(`ip_helper-address`)
* 1-11_DHCP_exhaustion+recovery
* 1-12_NAT_overload_vs_static_NAT_(packet_capture)
* 1-13_Link_failure+convergence_time
* 1-14_Metric_manipulation+path_selection

---

# Advanced_Networking_(IPv6+High_Availability+QoS)

* IPv6
* Redundancy_protocols
* Performance_engineering

## Labs

### IPv6

* 2-01_IPv6_addressing+subnetting
* 2-02_Dual-stack_networks
* 2-03_OSPFv3/EIGRP_for_IPv6
* 2-04_SLAAC_vs_DHCPv6
* 2-05_IPv6_ACLs

### High_Availability

* 2-06_First_Hop_Redundancy:
    * HSRP/VRRP/GLBP
* 2-07_Dual_ISP_failover_(floating_routes+IP_SLA)
* 2-08_Simulate_gateway_failover
* 2-09_Load_balancing_vs_failover_testing

### QoS_&_Performance

* 2-10_DSCP_marking_&_classification
* 2-11_Traffic_shaping_vs_policing
* 2-12_Congestion_simulation
* 2-13_Prioritize_voice/video_traffic

---

# Network_Operations

* Monitoring
* Telemetry
* Observability

## Labs

### Monitoring_Setup

* 3-01_SNMP+NetFlow/sFlow
* 3-02_SNMP_traps_(link_up/down)
* 3-03_Simulate_outages+alerting

### Observability

* 3-04_Dashboards_(utilization,performance)
* 3-05_Baseline_metrics:
    * CPU
    * Interface_usage
    * Error_rates
* 3-06_Tune_alert_thresholds

### Log_Analysis

* 3-07_Correlate_alerts+syslog
* 3-08_Send_logs_to_Splunk:
    * Interface_errors
    * Config_changes
    * Reboots

### Detection_Engineering

* 3-09_Saved_searches:
    * Interface_down
    * Auth_failures
    * STP_changes

### Operations_Runbooks

* 3-10_Build_NOC_runbooks:
    * Interface_down
    * High_CPU
    * Packet_loss

---

# Network_Security_&_Threat_Detection

* IDS/IPS
* SIEM
* Threat_hunting
* Hardening

## Labs

### Foundations

* [4-01_Map_network](/Network_Security_Labs/4-01_Map_network.md)
* 4-02_Send_malicious_traffic+observe_logs
* 4-03_IDS/IPS_tuning_(Suricata)

### Traffic_Analysis

* 4-04_Packet_capture_(Wireshark)
* 4-05_Validate_IDS_alerts_vs_PCAP

### Attack_Simulation

* 4-06_Generate_attacks:
    * Nmap_scans
    * SSH_brute_force
    * Suspicious_DNS

### Detection_Engineering

* 4-07_Custom_Suricata_rules:
    * Port_scans
    * ICMP_tunneling
    * Outbound_anomalies

### SIEM

* 4-08_Splunk_dashboards:
    * Top_source_IPs
    * Failed_logins
    * Lateral_movement

### Threat_Hunting

* 4-09_Hunt_for:
    * Beaconing
    * East-west_anomalies
    * New_services

### Framework_Mapping

* 4-10_Map_to:
    * MITRE_ATT&CK
    * NIST_Cybersecurity_Framework

### Incident_Response

* 4-11_Full_incident_workflow:
    1._Detection
    2._Triage
    3._Containment
    4._Eradication
    5._Lessons_learned

### Deception_&_Hardening

* 4-12_Honeypot_deployment
* 4-13_Detect_interaction
* [4-14_Device_hardening](/Network_Security_Labs/4-14_Device_hardening.md)
    * Port_security
    * DHCP_snooping
    * DAI
    * IP_source_guard
    * STP_protections

---

# Automation_&_Network_Programming

* Scripting
* APIs
* Infrastructure_as_Code

## Labs

### Python_Automation

* 5-01_Use_Python+Netmiko_to_push_configs
* 5-02_Parse_CLI_output+structured_data
* 5-03_Generate_network_reports

### Multi-Vendor_Automation

* 5-04_Use_NAPALM
* 5-05_Config_compliance_checks

### Infrastructure_as_Code

* 5-06_Automate_VLAN_deployments
* 5-07_Config_backups_(scheduled)
* 5-08_Config_drift_detection

### Orchestration

* 5-09_Use_Ansible:
    * Playbooks
    * Idempotent_configs

### Advanced
* 5-10_REST_APIs_(device_APIs/controllers)
* 5-11_Zero-touch_provisioning_simulation

---

# Cloud_Networking

* Cloud_networking
* Hybrid_environments

## Labs

### Core_Cloud_Networking

* 6-01_Build_VPC_in_Amazon_Web_Services
    * Public/private_subnets
    * Route_tables
* 6-02_Security_Groups_vs_NACLs
* 6-03_VPC_Flow_Logs_analysis

### Hybrid_Networking

* 6-04_Site-to-site_VPN_(on-prem_<->_cloud)
* 6-05_Simulate_hybrid_routing

### Traffic_&_Scaling

* 6-06_Load_balancers_(L4_vs_L7)
* 6-07_Failover_testing_in_cloud

---

# Identity_&_Access_Control

* AAA
* Centralized_authentication

## Labs

* 7-01_Configure_RADIUS
* 7-02_Configure_TACACS+
* 7-03_Role-based_CLI_access
* 7-04_Log_authentication_failures

---

# Wireless_Networking

* RF_basics
* WLAN_security

## Labs

* 8-01_Deploy_WLAN_(WPA2/WPA3)
* 8-02_Rogue_AP_simulation
* 8-03_Wireless_packet_capture
* 8-04_2.4_GHz_vs_5_GHz_behavior

---

# Advanced_Troubleshooting

* Systematic_debugging
* Real-world_problem_solving

## Labs

* 9-01_OSI-based_troubleshooting_framework
* 9-02_Control_plane_vs_data_plane_isolation
* 9-03_Blind_troubleshooting_labs
* 9-04_Timed_failure_scenarios

---

# Lab_Infrastructure

## Labs

* 10-01_Integrate_Linux_servers:
    * DNS
    * DHCP
    * Web
    * NTP

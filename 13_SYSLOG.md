**~ *Syslog* 451 ~** <sub><sup>by Ray Bradbury</sup></sub>

---

# SYSLOG

## SUMMARY

- Syslog is the standard mechanism Cisco devices use to report events (security, interface up/down, routing changes, system errors, debugging, etc.)
- Messages are categorized by facility and severity (0–7):
    + 0 Emergency
    + 1 Alert
    + 2 Critical
    + 3 Error
    + 4 Warning
    + 5 Notice
    + 6 Informational
    + 7 Debugging

---

## CONFIGUARATION

```
# Set hostname
R1(config)# hostname R1

# Enable timestamps for logs and ensures human-readable timestamps.
R1(config)# service timestamps log datetime msec localtime
R1(config)# service timestamps debug datetime msec localtime

# Set NTP for accurate time.
R1(config)# ntp server 203.0.113.5

# Create a loopback interface for stability.
R1(config)# interface Loopback0
R1(config-if)# ip address 10.0.0.10 255.255.255.255
R1(config-if)# exit

# Specify syslog source interface.
R1(config)# logging source-interface Loopback0

# Set syslog server (collector).
R1(config)# logging host 192.0.2.10
R1(config)# logging host 192.0.2.10 transport tcp port 6514 (optional for tcp/tls mode)

# Set logging severity level
R1(config)# logging trap informational

# Enable local log buffer
R1(config)# logging buffered 8192 debugging

# Set console and monitor logging levels
R1(config)# logging console warnings
R1(config)# logging monitor informational

# Lets syslog collector separate Cisco logs by facility.
R1(config)# logging facility local7

# Enable syslog service globally
R1(config)# logging on
```

---

## MONITORING & MAINTENANCE

R1# show logging
R1# terminal monitor
R1# show ntp status
R1# show interfaces
R1# debug logging
R1# send log 6 "Syslog test from R1"
R1# clear logging

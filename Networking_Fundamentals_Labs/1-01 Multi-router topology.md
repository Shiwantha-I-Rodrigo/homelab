# PRE CONFIGURATION

## FACTORY RESET

### Establish a console connection and open the cli.

> Find the connection

```
ls /dev/ttyUSB* /dev/ttyS*
```

- /dev/ttyUSB0: Common if using a USB-to-RJ45 (blue) console cable.
- /dev/ttyS0: Common if using a native DB9 serial port.

> Connect using Screen

```
screen /dev/ttyUSB0 9600
```

`screen [device] [baud_rate]`

### Power off the switch / router while keeping the console cli open.

> Enter Recovery Mode

- Router
    + Within the first 60 seconds of boot-up, send a Break sequence. In screen, this is: Press `Ctrl+a`, then press `Ctrl+b`.
- Switch
    + Power on the switch & hold the mode button until recovery mode is shown on the cli.

### Reset

```
flash_init
dir flash:
del flash:vlan.dat
del flash:config.text
del flash:private-config.text (if present)
boot
```

---

## INITIAL SETUP

1. Enter Configuration Mode
    ```
    configure terminal
    ```
2. Set a Hostname (Required for SSH)
    ```
    hostname S1
    ```
3. Set a Domain Name (Required for SSH)
    ```
    ip domain-name lab.local
    ```
4. Generate Encryption Keys
    ```
    crypto key generate rsa
    # Use at least 2048 bits for SSH version 2
    ```
5. Create a Local User
    ```
    username admin privilege 15 secret admin
    ```
6. Configure the VTY lines (virtual 'ports' for SSH)
    ```
    line vty 0 15
    login local
    transport input ssh
    exit
    ```
7. Enable SSH Version 2
    ```
    ip ssh version 2
    ```
8. Set Management IP
    ```
    # S2
    interface vlan 1
        ip address 192.168.10.1 255.255.255.0
        no shutdown

    # S1
    interface vlan 1
        ip address 192.168.50.3 255.255.255.0
        no shutdown
    ```
9. Set enable password
    ```
    configure terminal
        enable secret enable
        exit
        ```
10. Save Configs
    ```
    copy running-config startup-config
    ```

> CREDENTIALS ( • ᴗ - ) ✧\
> 
> **ENABLE** : enable\
> **USER** : admin\
> **PASSWORD** : admin\
> **DOMAIN** : admin.com

---

## HOST SETUP

1. Arch Linux Compatibility with older Cisco SSH Algorithms (temporary)
    ```
    ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa admin@192.168.12.1
    ```
2. add compatibility profiles (permanant)
    ```
    # vim ~/.ssh/config
    # add

    Host 192.168.10.*
    HostKeyAlgorithms +ssh-rsa
    KexAlgorithms +diffie-hellman-group14-sha1
    User admin

    Host 192.168.50.*
    HostKeyAlgorithms +ssh-rsa
    KexAlgorithms +diffie-hellman-group14-sha1
    User admin
    ```

---

# NETWORK CONFIGURATION

## TOPOLOGY

![topology](../images/seclab.png)

### NIC-1 (enp0s3)

```
sudo nmcli connection modify "Wired connection 1" ipv4.addresses "192.168.10.2/24" ipv4.method manual connection.autoconnect yes
sudo nmcli connection up "Wired connection 1"
```

### NIC-2 (enp5s0f3u1c2)

```
sudo nmcli connection modify "Wired connection 2" ipv4.addresses "192.168.50.2/24" ipv4.method manual connection.autoconnect yes
sudo nmcli connection up "Wired connection 2"
```

### NIC-3 (enp5s0f4u1c2)

```
sudo ip addr flush dev enp5s0f4u1c2
sudo nmcli connection modify "Wired connection 3" ipv4.method disabled ipv6.method ignore
sudo nmcli connection modify "Wired connection 3" connection.autoconnect yes
sudo nmcli connection up "Wired connection 3"
```

### Catalyst 3750 / S2

enable routing globally
```
conf t
 ip routing
```

management ip address
```
interface Vlan1
 ip address 192.168.10.1 255.255.255.0

interface FastEthernet2/0/2
 switchport mode access
 switchport access vlan 1
```

routing physical ports
```
interface FastEthernet2/0/23
 no switchport
 ip address 192.168.20.2 255.255.255.252
```

set deafault route
```
ip route 0.0.0.0 0.0.0.0 192.168.20.1
```

duplicate traffic on *FastEthernet2/0/23* to *FastEthernet2/0/14*.
```
conf t
 no monitor session 1
 monitor session 1 source interface FastEthernet2/0/23 both
 monitor session 1 destination interface FastEthernet2/0/14
```

### Catalyst 1841 / R2

assign ip addresses
```
conf t
interface FastEthernet0/1
 description Internal LAN
 ip address 192.168.20.1 255.255.255.252
 no shutdown
 exit
```

setup vlan routing
```
conf t

# Remove the IP from the physical interface
interface FastEthernet0/0
 no ip address
 no shutdown
 exit

# Configure Sub-interface for VLAN 30
interface FastEthernet0/0.30
 description Gateway for VLAN 30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit

# Configure Sub-interface for VLAN 40
interface FastEthernet0/0.40
 description Gateway for VLAN 40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 exit

# Configure Sub-interface for VLAN 50
interface FastEthernet0/0.50
 description Gateway for VLAN 50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
 exit
```

### Catalyst 2960 / S1

create vlans
```
conf t
vlan 30
 name Marketing_VLAN
vlan 40
 name Engineering_VLAN
vlan 50
 name NIC2_VLAN
exit
```

configure trunk ports
```
conf t

interface GigabitEthernet0/1
 description Trunk to R2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 # Optional: Limit allowed VLANs for security
 switchport trunk allowed vlan 30,40,50
 no shutdown

interface FastEthernet0/2
 description Trunk to NIC-2
 switchport mode trunk
 switchport trunk allowed vlan 30,40,50
 # Set the Native VLAN (If the Host OS of NIC-2 (192.168.50.2) sends untagged traffic, it will fall into VLAN 50).
 switchport trunk native vlan 50
 no shutdown
 exit
```

set management ip
```
interface Vlan50
 description Management Interface
 ip address 192.168.50.3 255.255.255.0
 no shutdown
exit
```

set deafault route
```
ip default-gateway 192.168.50.1
```

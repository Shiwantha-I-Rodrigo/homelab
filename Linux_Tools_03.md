# PING / TRACEROUTE / NETSTAT / NSLOOKUP / NETCAT / NMAP

## **1. PING**

**Purpose:** Test connectivity, latency, and packet loss.

| Use Case                        | Example Command                                                                       |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| Check if host is reachable      | `ping google.com`                                                                     |
| Specify number of packets       | `ping -c 5 google.com` (Linux/macOS)<br>`ping -n 5 google.com` (Windows)              |
| Set packet size                 | `ping -s 1000 google.com` (Linux/macOS)                                               |
| Ping via specific interface     | `ping -I eth0 8.8.8.8` (Linux)                                                        |
| Continuous ping (until stopped) | `ping -t google.com` (Windows)<br>`ping google.com` (Linux/macOS, default continuous) |

| Option           | Explanation                                       |
| ---------------- | ------------------------------------------------- |
| `-c <count>`     | Number of ping packets to send (Linux/macOS).     |
| `-n <count>`     | Number of ping packets (Windows).                 |
| `-s <size>`      | Packet size in bytes.                             |
| `-I <interface>` | Use a specific network interface.                 |
| `-t`             | Continuous ping until manually stopped (Windows). |

---

## **2. TRACEROUTE / TRACERT**

**Purpose:** Map network path and identify bottlenecks.

| Use Case                   | Example Command                                                         |
| -------------------------- | ----------------------------------------------------------------------- |
| Trace route to host        | `traceroute google.com` (Linux/macOS)<br>`tracert google.com` (Windows) |
| Limit maximum hops         | `traceroute -m 10 google.com`                                           |
| Use specific port/protocol | `traceroute -T -p 80 google.com` (TCP, Linux)                           |
| Change packet size         | `traceroute -q 3 -w 2 google.com` (3 probes per hop, 2 sec timeout)     |
| Continuous path checking   | Use a loop with `ping` or repeat traceroute in scripts                  |

| Option          | Explanation                                      |
| --------------- | ------------------------------------------------ |
| `-m <max_hops>` | Maximum number of hops to trace (Linux/macOS).   |
| `-T`            | Use TCP instead of default UDP (Linux/macOS).    |
| `-p <port>`     | Specify port for TCP trace.                      |
| `-q <probes>`   | Number of probe packets per hop (Linux/macOS).   |
| `-w <timeout>`  | Timeout in seconds for each probe (Linux/macOS). |

---

## **3. NETSTAT**

**Purpose:** Monitor active connections, listening ports, and routing.

| Use Case                     | Example Command                                    |                          |
| ---------------------------- | -------------------------------------------------- | ------------------------ |
| Show all connections         | `netstat -a`                                       |                          |
| Show listening ports only    | `netstat -l` (Linux), `netstat -an                 | find "LISTEN"` (Windows) |
| Show numeric addresses/ports | `netstat -n`                                       |                          |
| Show connections by protocol | `netstat -t` (TCP), `netstat -u` (UDP)             |                          |
| Show processes using ports   | `netstat -tulpn` (Linux)<br>`netstat -b` (Windows) |                          |
| Show routing table           | `netstat -r`                                       |                          |

| Option | Explanation                                                    |
| ------ | -------------------------------------------------------------- |
| `-a`   | Show all connections and listening ports.                      |
| `-l`   | Show only listening ports (Linux/macOS).                       |
| `-n`   | Show numeric addresses and ports.                              |
| `-t`   | Show only TCP connections (Linux).                             |
| `-u`   | Show only UDP connections (Linux).                             |
| `-p`   | Show process ID (PID) and program name using the port (Linux). |
| `-r`   | Show routing table.                                            |
| `-b`   | Show executable involved in connection (Windows).              |

---

## **4. NSLOOKUP**

**Purpose:** Query DNS for hostname or IP information.

| Use Case                              | Example Command                                                                    |
| ------------------------------------- | ---------------------------------------------------------------------------------- |
| Resolve domain to IP                  | `nslookup google.com`                                                              |
| Reverse lookup (IP → domain)          | `nslookup 8.8.8.8`                                                                 |
| Use specific DNS server               | `nslookup google.com 8.8.8.8`                                                      |
| Query specific record type            | `nslookup -type=MX example.com` (mail servers)<br>`nslookup -type=TXT example.com` |
| Interactive mode for multiple queries | `nslookup` → then type domains                                                     |

| Option           | Explanation                                                  |
| ---------------- | ------------------------------------------------------------ |
| `-type=<record>` | Specify DNS record type (A, MX, CNAME, TXT, etc.).           |
| `server <DNS>`   | Use a specific DNS server for queries.                       |

---

## **5. NETCAT (nc)**

**Purpose:** TCP/UDP testing, file transfer, and network debugging.

| Use Case                   | Example Command                                                               |
| -------------------------- | ----------------------------------------------------------------------------- |
| Connect to a host and port | `nc 192.168.1.10 80`                                                          |
| Listen on a port           | `nc -l 1234` (Linux/macOS)<br>`nc -l -p 1234` (Windows)                       |
| Scan for open ports        | `nc -zv 192.168.1.1 20-80`                                                    |
| File transfer              | Sender: `nc -l 1234 > file.txt`<br>Receiver: `nc 192.168.1.2 1234 < file.txt` |
| Create simple chat         | `nc -l 1234` on one host, `nc host 1234` on the other                         |

| Option      | Explanation                                                  |
| ----------- | ------------------------------------------------------------ |
| `-l`        | Listen mode (act as a server).                               |
| `-p <port>` | Specify port to listen on (Windows syntax differs slightly). |
| `-z`        | Zero-I/O mode: used for scanning if ports are open.          |
| `-v`        | Verbose mode: shows connection details.                      |
| `<` / `>`   | Redirect file input/output for transfer.                     |

---

## **6. NMAP**

**Purpose:** Network discovery, port scanning, and security auditing.

| Use Case                     | Example Command                   |
| ---------------------------- | --------------------------------- |
| Scan a single host           | `nmap 192.168.1.1`                |
| Scan a subnet                | `nmap 192.168.1.0/24`             |
| Detect service versions      | `nmap -sV 192.168.1.1`            |
| OS detection                 | `nmap -O 192.168.1.1`             |
| Scan specific ports          | `nmap -p 22,80,443 192.168.1.1`   |
| Aggressive scan with scripts | `nmap -A 192.168.1.1`             |
| Scan stealthily              | `nmap -sS 192.168.1.1` (SYN scan) |

| Option       | Explanation                                                               |
| ------------ | ------------------------------------------------------------------------- |
| `-sV`        | Detect version of services running on open ports.                         |
| `-O`         | Enable OS detection.                                                      |
| `-p <ports>` | Scan specific ports.                                                      |
| `-A`         | Aggressive scan: enables OS detection, version scan, scripts, traceroute. |
| `-sS`        | SYN scan: stealthy TCP port scan (does not complete handshake).           |

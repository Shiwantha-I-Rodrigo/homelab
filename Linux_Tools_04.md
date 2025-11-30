# TCPDUMP / WIRESHARK

## **1. TCPDUMP**

**Purpose:**
Command-line packet sniffer used to capture and inspect network traffic in real-time.

**Common usages:**

| Use Case                             | Example Command                    | Explanation                                   |
| ------------------------------------ | ---------------------------------- | --------------------------------------------- |
| Capture all packets on an interface  | `tcpdump -i eth0`                  | Monitors all traffic on `eth0`.               |
| Capture a specific number of packets | `tcpdump -i eth0 -c 10`            | Captures only 10 packets.                     |
| Filter by protocol (e.g., TCP)       | `tcpdump -i eth0 tcp`              | Captures only TCP traffic.                    |
| Filter by host IP                    | `tcpdump -i eth0 host 192.168.1.5` | Captures packets to/from a specific host.     |
| Filter by port                       | `tcpdump -i eth0 port 80`          | Captures traffic on HTTP port.                |
| Save capture to a file               | `tcpdump -i eth0 -w capture.pcap`  | Saves captured packets for later analysis.    |
| Read packets from a file             | `tcpdump -r capture.pcap`          | Reads and displays packets from a saved file. |

| Option               | Explanation                                                           |
| -------------------- | --------------------------------------------------------------------- |
| `-i <interface>`     | Specify the network interface to capture packets from (e.g., `eth0`). |
| `-c <count>`         | Capture only a specified number of packets (e.g., `-c 10`).           |
| `tcp`, `udp`, `icmp` | Protocol filter: captures only packets of the specified protocol.     |
| `host <IP>`          | Capture packets to/from a specific host IP.                           |
| `port <port>`        | Capture packets on a specific port (e.g., HTTP `80`).                 |
| `-w <file>`          | Write captured packets to a file (e.g., `capture.pcap`).              |
| `-r <file>`          | Read packets from a previously saved file.                            |


**Common usages in practice:**

* Troubleshooting connectivity issues.
* Debugging application network behavior.
* Detecting suspicious network activity.
* Capturing packets for detailed analysis in TShark.

---

## **2. TSHARK**

**Purpose:**
Command-line network protocol analyzer (CLI version of Wireshark). Allows capturing, filtering, and analyzing packets without a GUI.

**Common usages:**

| Use Case                           | Example Command                                        | Explanation                                                           |
| ---------------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------- |
| Capture live traffic               | `tshark -i eth0`                                       | Monitors all packets on the interface in real-time.                   |
| Capture specific number of packets | `tshark -i eth0 -c 10`                                 | Stops after capturing 10 packets.                                     |
| Filter by protocol                 | `tshark -i eth0 -f "tcp"`                              | Captures only TCP traffic (BPF filter).                               |
| Filter by display filter           | `tshark -i eth0 -Y "http.request"`                     | Captures only HTTP request packets (Wireshark display filter syntax). |
| Save capture to file               | `tshark -i eth0 -w capture.pcap`                       | Saves captured packets for later analysis.                            |
| Read and analyze file              | `tshark -r capture.pcap`                               | Reads and prints packets from a saved file.                           |
| Extract specific fields            | `tshark -r capture.pcap -T fields -e ip.src -e ip.dst` | Shows only source and destination IPs.                                |

| Option                  | Explanation                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| `-i <interface>`        | Specify the network interface to capture packets from.                                   |
| `-c <count>`            | Stop capture after a specified number of packets.                                        |
| `-f "<BPF filter>"`     | Capture filter using Berkeley Packet Filter syntax (e.g., `tcp`).                        |
| `-Y "<display filter>"` | Display filter using Wireshark syntax to filter captured packets (e.g., `http.request`). |
| `-w <file>`             | Save captured packets to a file (e.g., `capture.pcap`).                                  |
| `-r <file>`             | Read and analyze packets from a saved file.                                              |
| `-T fields -e <field>`  | Extract specific fields from packets (e.g., source and destination IPs).                 |

**Common usages in practice:**

* Real-time packet capture on servers without GUI.
* Scripted network monitoring or automation.
* Security analysis or intrusion detection.
* Parsing and extracting specific packet data for reports.

---

**Key difference between TCPDUMP and TSHARK:**

* **TCPDUMP:** Lightweight, mostly used for quick captures and raw packet inspection.
* **TShark:** Provides protocol-level decoding like Wireshark but in command-line form; better for detailed protocol analysis.

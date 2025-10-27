**~ The Book of Lost *Packets* ~** <sub><sup>by J.R.R.Tolkien</sup></sub>

---

# CONNECTIVITY TROUBLESHOOTING

**DATE : 2024/01/16**

I started the day testing connectivity in my home lab. 2 PCs sat on opposite ends of my small network, each on a different subnet. The plan was to make sure they could reach each other.

| **Source** | **Destination** | **Result** | **Remarks**                       |
| ---------- | --------------- | ---------- | --------------------------------- |
| PC1        | PC2             | Success    | PC1 can reach PC2 without issues  |
| PC2        | PC1             | Fail       | No replies                        |

**ISOLATING THE PROBLEM**

My first thought was to trace the path. I **SSH**-ed into **S2**, the switch closest to **PC2** and tried pinging **PC1** from there without success. 
Then, I **SSH**-ed into **R2**, the router just upstream of **S2**. I checked its routing table and **PC1**’s subnet was listed correctly. 
I moved to the other side. From **S1** ( the switch nearest to **PC1** ) pinged **PC1** and it succeeded. 
The same was true from **R1**, the router directly connected to **S1**.
But from **R2** ( the next hop from **R1**), still nothing.
something may be wrong with traffic moving from R2 to R1 or a mistaken routing decision is made along the way ( PC1 > S1 > R1 ) of the response.

**PACKET SNIFFING**

Using **Wireshark** I started a capture on **PC1’s eth1 interface** (the one connected to **S1**) and ran two tests:

| **Source** | **Destination** | **Result** | **Remarks**                                       |
| ---------- | --------------- | ---------- | ------------------------------------------------- |
| S1         | PC1             | Success    | ICMP request and reply pairs observed             |
| PC2        | PC1             | Failed     | Only ICMP requests captured; no replies from PC1  |

S1 > PC1
```
120 22.913239952 192.168.11.2 → 192.168.11.3 ICMP 114 Echo (ping) request  id=0x0004, seq=0/0, ttl=255
121 22.913254720 192.168.11.3 → 192.168.11.2 ICMP 114 Echo (ping) reply    id=0x0004, seq=0/0, ttl=64 (request in 120)
124 22.915508684 192.168.11.2 → 192.168.11.3 ICMP 114 Echo (ping) request  id=0x0004, seq=1/256, ttl=255
125 22.915527309 192.168.11.3 → 192.168.11.2 ICMP 114 Echo (ping) reply    id=0x0004, seq=1/256, ttl=64 (request in 124)
128 22.917771996 192.168.11.2 → 192.168.11.3 ICMP 114 Echo (ping) request  id=0x0004, seq=2/512, ttl=255
129 22.917778448 192.168.11.3 → 192.168.11.2 ICMP 114 Echo (ping) reply    id=0x0004, seq=2/512, ttl=64 (request in 128)
```

PC2 > PC1
```
5 1.801374790 192.168.10.2 → 192.168.11.3 ICMP 114 Echo (ping) request  id=0x000a, seq=0/0, ttl=254
12 3.798425453 192.168.10.2 → 192.168.11.3 ICMP 114 Echo (ping) request  id=0x000a, seq=1/256, ttl=254
17 5.798473447 192.168.10.2 → 192.168.11.3 ICMP 114 Echo (ping) request  id=0x000a, seq=2/512, ttl=254
```

The packet capture confirmed that **PC1** was receiving pings from **PC2** but never sends replies, *at least not on **eth1***. 
Hence, Either the replies are being blocked or they are leaving via a different interface.

**FOLLOWING THE PACKETS**

Firewall ? i confirmed that no rules are interfering with ICMP traffic.
Then i checked the **the default gateway** of **PC1**.
Sure enough, **PC1**’s default route has been changed to my **home Wi-Fi network on wlan0**. 
That meant that **PC1** maybe happily sending ICMP replies out through its **wlan0** interface instead of back through to the lab network. 
To confirm, I fired up Wireshark on **wlan0** and pinged **PC1** from **PC2** again.

PC1 > wlan0
```
18 6.853390970 192.168.11.3 → 192.168.10.2 ICMP Echo reply
19 8.850554715 192.168.11.3 → 192.168.10.2 ICMP Echo reply
20 10.850340025 192.168.11.3 → 192.168.10.2 ICMP Echo reply
```

Mystery solved : the replies were leaving through the wrong door.

**THE FIX**

I changed PC1’s **default gateway** to **R1**’s IP address, the proper next hop in my lab topology.
A quick ping test confirmed that both PCs were exchanging packets both ways.

> ***Lessons Learned :***

- What started as a simple connectivity test turned into a reminder of how easily routing **asymmetry** can break communication.
- A single misconfigured default gateway made it look like a firewall issue.
- In the end, I learned that before blaming complex configurations, it’s always worth checking the basics.


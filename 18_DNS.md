**~ Guild of *DNS* ~** <sub><sup>by Jon Cronshaw</sup></sub>

---

# DNS

## SUMMARY

**DNS PROCESS**

| **Step** | **Description** |
| - | - |
| **1. Browser Cache Check**                     | Browser first checks if it already knows the IP address of `www.example.com` (from a previous visit).|
| **2. Operating System Cache Check**            | If not in the browser, the browser asks the operating system (OS). The OS might have it cached too.|
| **3. DNS Resolver Query (Recursive Resolver)** | If the OS doesn’t know, it asks a DNS resolver, usually provided by your ISP (Internet Service Provider) or a public resolver like Google (`8.8.8.8`) or Cloudflare (`1.1.1.1`).|
| **4. Root Server Query**                       | If the resolver doesn’t know, it asks a root DNS server. The root server doesn’t know the exact address, but it points to the Top-Level Domain (TLD) server responsible for `.com`.|
| **5. TLD Server Query**                        | The resolver then asks the `.com` TLD server, which points to the authoritative name server for `example.com`.|
| **6. Authoritative Name Server Query**         | Finally, the resolver asks the authoritative server, which holds the actual DNS record for `www.example.com` and returns the correct IP address.|
| **7. Response to Browser**                     | The resolver sends the IP address back to your computer.|

**DNS RECORD TYPES**

| Record Type | Meaning | Example |
| - | - | - |
| **A**       | Maps domain → IPv4 address              | `example.com → 93.184.216.34`|
| **AAAA**    | Maps domain → IPv6 address              | `example.com → 2606:2800:220:1:248:1893:25c8:1946`|
| **CNAME**   | Alias (canonical name) → another domain | `www.example.com → example.com`|
| **MX**      | Mail exchange (for email servers)       | `example.com → mail.example.com`|
| **TXT**     | Text data (for verification, SPF, etc.) | `v=spf1 include:_spf.google.com ~all`|
| **NS**      | Name server for the domain              | `ns1.example.com`|
| **PTR**     | Reverse lookup (IP → domain)            | `93.184.216.34 → example.com`|

**DNS SERVERS**

Handle caching, recursion, forwarding, zone transfers, and high-availability operations.

| Component | Description |
| - | - |
| **Query Processor**        | Parses incoming DNS queries and decides how to handle them (cache lookup, forwarder, authoritative answer, recursion).|
| **Cache / Resolver Cache** | Stores recently resolved records (A, AAAA, MX, etc.) with their TTLs to speed up subsequent lookups.|
| **Authoritative Engine**   | Responds with authoritative answers for domains the server owns (zones it’s configured to serve).|
| **Recursive Resolver**     | If not authoritative, recursively queries other DNS servers (root → TLD → authoritative).|
| **Zone Management**        | Handles zone files, AXFR/IXFR transfers, DNSSEC signing, and updates.|
| **Network I/O Engine**     | Manages UDP/TCP sockets, DNS-over-TLS (DoT), DNS-over-HTTPS (DoH), rate limiting, and query queueing.|

Query Process -->

1. Receive query → UDP or TCP port 53 (or HTTPS/TLS for DoH/DoT).
2. Check cache → If the record is cached and TTL valid, respond immediately.
3. Authoritative check → If the server is authoritative for that zone, answer directly.
4. Recursion / Forwarding → If not, recursively query upstream servers (root → TLD → authoritative) or use a configured forwarder.
5. Response assembly → Add EDNS options, DNSSEC signatures if needed.
6. Send response and update cache.

DNS Load Balancing Modes -->

| **Mode** | **Type** | **Mechanism** | **Description** | **Pros** | **Limitations** |
| - | - | - | - | - | - |
| **Round-Robin DNS**                  | DNS record                        | Multiple A/AAAA records returned in rotation                   | Distributes requests evenly by rotating record order | Simple, widely supported              | No health checks, no true load awareness|
| **Weighted Round Robin**             | DNS record                        | Assigns weights to each IP                                     | Servers with higher weight appear more often         | Easy traffic ratio control            | Still no health or performance feedback|
| **GeoDNS / GeoIP Routing**           | DNS record                        | Uses client’s geographic location to choose response           | Directs clients to nearest regional server           | Improves latency and localization     | Requires accurate GeoIP database|
| **Latency-Based Routing**            | Managed DNS (ie. Route 53, NS1)   | Uses network latency measurements                              | Returns IP with lowest latency to client             | Optimized user experience             | Needs active monitoring infrastructure|
| **Health-Check / Failover DNS**      | Managed DNS / local DNS appliance | Monitors endpoint health                                       | Removes unhealthy servers from responses             | High availability, automatic failover | Propagation delay due to TTL caching|
| **Anycast DNS**                      | Network layer                     | Same IP announced from multiple locations via BGP              | Routes client queries to the nearest DNS node        | Extremely scalable, global resilience | Harder debugging, relies on BGP routing|
| **Load-Balanced DNS Cluster**        | Server infrastructure             | Front-end load balancer (LVS, HAProxy) distributes DNS traffic | Balances incoming UDP/TCP queries across DNS daemons | High query throughput, redundancy     | Added complexity, single VIP dependency|
| **EDNS Client Subnet (ECS) Routing** | DNS protocol extension            | Passes client subnet info to authoritative DNS                 | Enables more accurate geo/load-aware answers         | Improves accuracy for CDNs            | Privacy issues, not all resolvers support ECS|

---

## CONFIGUARATION

1 - Install Required Packages.

```
sudo pacman -Syu bind
```
- This installs `named` / `dig` / `nslookup`

2 - Configure the Main BIND File.

/etc/named.conf
```
options {
    directory "/var/named";
    listen-on port 53 { any; };
    allow-query { any; };
    recursion yes;
    forwarders { 1.1.1.1; 8.8.8.8; };  // optional, for upstream DNS
    dnssec-validation auto;
};

// Example local zone
zone "example.local" IN {
    type master;
    file "example.local.zone";
    allow-update { none; };
};
```

3 - Create the Zone File.

/var/named/example.local.zone
```
$TTL 1h
@   IN  SOA ns1.example.local. admin.example.local. (
        2025111001 ; serial
        1h         ; refresh
        15m        ; retry
        1w         ; expire
        1h )       ; minimum

    IN  NS  ns1.example.local.
ns1 IN  A   192.168.56.10
app IN  A   192.168.56.11
web IN  A   192.168.56.12
```

4 - Set Permissions.
```
sudo chown -R named:named /var/named
sudo chmod -R 640 /var/named/*
sudo chmod 755 /var/named
```

5 - Enable and Start the DNS Service.
```
sudo systemctl enable named.service
sudo systemctl start named.service
```

6 - Add a Secondary (Slave) DNS Server.

In the master’s zone VM1 definition, add:
```
zone "example.local" IN {
    type master;
    file "example.local.zone";
    allow-transfer { 192.168.56.20; };  // IP of the slave
};
```

On the slave VM2, configure:
```
zone "example.local" IN {
    type slave;
    masters { 192.168.56.10; };  // master IP
    file "slaves/example.local.zone";
};
```

7 - Simulate Load Balancing.

In the example.local.zone, add multiple A records:
```
www IN A 192.168.56.11
www IN A 192.168.56.12
www IN A 192.168.56.13
```

When you run:
```
dig @127.0.0.1 www.example.local
```
You’ll see the IPs rotate. that’s round-robin DNS load balancing.

---

## MONITORING & MAINTENANCE

Check service status.
```
sudo systemctl status named.service
```

Test the server.

from  a seperate pc
```
dig @127.0.0.1 app.example.local
```

answer should include.
```
;; ANSWER SECTION:
app.example.local.  3600 IN A 192.168.56.11
```

| Command                                                            | Purpose                            |
| ------------------------------------------------------------------ | ---------------------------------- |
| `sudo named-checkconf`                                             | Validates `/etc/named.conf` syntax |
| `sudo named-checkzone example.local /var/named/example.local.zone` | Validates zone file                |
| `sudo journalctl -u named`                                         | View logs                          |
| `dig @127.0.0.1 example.local`                                     | Query DNS server directly          |

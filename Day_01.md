# REBOOT SUPRISE !

After power cycling the lab, I noticed that only the port connected to my PC lit on switch S1 and all the other connections were inactive which suggested that the other links weren’t active or that the switches lost their configurations. To start from the top checked the IP address on my PC and pinged the neighbor switch. I was able to SSH into S1 without any trouble, which was a good sign. Then i tried to ping router R1, but it didn’t respond. Checking the ARP table confirmed my suspicion as it only showed entries for my PC and S1. Clearly, the rest of the network wasn’t reachable yet.

![setup](/images/initial.png)

Next, I connected the cable to S2 and configured my PC’s IP address accordingly. However, I couldn’t ping S2 either and running `arp -a` showed an incomplete table. That points toward a configuration issue on S2. I began to suspect that its configuration hadn’t been saved before the lab was powered down. I connected a console cable to S2 and established a direct session. My suspicion was confirmed as i was greeted by the initial configuration dialog, meaning S2 had lost its previous settings. I reconfigured S2 manually, set up the basics, and enabled SSH access. After doing that, I could ping S2 successfully.

However, when I tried to SSH into S2, I got a warning about the host key mismatch so i cleared the relevant host entry from my `.ssh/known_hosts` file and successfully connected.

Moving on, I couldn’t ping R2. I connected a console cable to R1. The router still had its hostname and basic configuration, so it hadn’t lost everything. The ARP table on R1  was incomplete as only the f0/0 was active. I realized that i have forgotten to assign an IP to f0/1 earlier. i set the ip and initialized f0/1.

Running `show cdp neighbors` on R1 displayed S1 but not R2, so I moved to R2 with the console cable. On R2, the ARP table was empty, which meant no interfaces had IP addresses assigned. I assigned IP addresses to both interfaces f0/0 and f0/1 and now ARP showed both connections. `show cdp neighbors` on R2 listed both R1 and S2, which meant the physical and Layer 2 connectivity was restored, therefore i disconnected the console cable.

Back on my PC, I tried to SSH into S1 again and got “network unreachable”. I checked the interface with `ip addr show`. It revealed that the IP address of my Ethernet interface was mysteriously reset, even after i tried to set the ip, it kept resetting. After some googling, I realized that NetworkManager was interfering the adapter configuration set by `ip` command by applying its own configuration periodically.

I checked the available connections using `nmcli connection show` and then manually set the IP configuration using, `sudo nmcli connection modify "conn name" ipv4.method manual ipv4.addresses x.x.x.x/24`.

After this, I was able to ping S1 successfully again and from there, I SSHed into S1, then into R1 through S1, into R2 through R1, and finally into S2 through R2. At this point, the entire network was reachable via SSH (through proxy jumping).

However, ping tests told a different story as i could ping S1 and R1, but pings to R2 and S2 failed. That was a curious mismatch, since SSH worked through the chain, but ICMP packets weren't making it through.

| Device    | SSH (Proxy Jumps) | Ping|
|-          |-                  |-|
|S1         |success            |success|
|R1         |success            |success|
|R2         |success            |fail|
|S2         |success            |fail|

I deduce that the likely reason for that is unavailability of any routing. SSH sessions works because each hop is an independent connection over a directly connected interface. But for pinging across devices, packets need proper IP routing between subnets and since `show ip protocol` returned none, it meant that no dynamic routing protocol was running. The devices could communicate with their direct neighbors but didn’t know about networks beyond them.

So, in short, some connectivity was reestablished, but end-to-end IP reachability was incomplete because the routers weren’t exchanging routes. I saved the current configurations with `copy running-config startup-config` to prevent another round of post-reboot surprises.

The plan for the next day :
1. Enable various routing protocols and observe their behaviors and additionally configure static routes so the entire network is reachable.

Lessons Learned :
1. **Always** save configs before leaving the device.
2. SSH fingerprint must be cleared if the target device key is changed.
3. Multiple programs might be modifying the system configurations at the same time.
4. Sections of the network unreachable directly, might be reachable though **Proxy Jumping**.
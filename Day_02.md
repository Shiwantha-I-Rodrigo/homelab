# SSH

## CONFIGURATION

```
S1(config)# hostname S1
S1(config)# username admin password admin
S1(config)# ip domain-name admin.com
S1(config)# crypto key generate rsa
S1(config)# line vty 0 15
S1(config-line)# login local
S1(config-line)# transport input ssh
S1(config-line)# end
S1(config)#ip ssh version 2
```

> Use similar config for all 4 devices S1,R1,R2,S2 !

**LIMIT ACCESS TO A SPECIFIC SUBNET**

```
R1(config)#access-list 23 permit 10.10.10.0 0.0.0.255
R1(config)#line vty 0 15
R1(config-line)#access-class 23 in
R1(config-line)#exit
```

## TORUBLESHOOTING

It started with a simple goal : SSH into four network devices directly **S1**, **R1**, **R2** and **S2** all part of my lab setup. The first round of SSH attempts failed across the board.

* **S1** and **R1** responded with the error.
```
Unable to negotiate with 192.168.11.2 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1
```
* **R2** and **S2** didn’t respond at all.

At first glance, it looks like a compatibility issue. Some Googling confirmed my suspicion. Legacy devices still rely on outdated algorithms like `diffie-hellman-group1-sha1` and older host key algorithms like `ssh-rsa`.

I activated legacy algorithms by adding some SSH client configurations.

```
sudo vim /etc/ssh/ssh_config
```

```
Host S1
        Hostname 192.168.11.2
        User admin
        HostKeyAlgorithms +ssh-rsa
        KexAlgorithms +diffie-hellman-group1-sha1
        Ciphers +aes256-cbc
```

> Added similar configurations for all 4 devices.

Cleaned up any stale host fingerprints in,

```
sudo vim ~/.ssh/known_hosts
```

With that, I am finally able to **SSH directly into S1**, however R1 (192.168.10.1) still wasn’t accepting direct connections.
So, I attempted to SSH into it **via S1 using a proxy jump**:

R1 has two interfaces (192.168.10.1) and (192.168.11.1 < directly connected to S1 >).

```
ssh -J admin@192.168.11.2 admin@192.168.10.1 <fail>
ssh -J admin@192.168.11.2 admin@192.168.11.1 <success>
```

Running `show ip protocols` on R1 returnes nothing.
Seems no routing protocols are enabled.

On R1, I enabled **RIP** and added the two directly connected networks to its routing table.

Still i cannot directly ssh in to R1 through its second interface (192.168.10.1).

I suspected the problem is on **S1**, maybe a missing default route.
So, apone checking **S1 didn’t have a default gateway configured**. Therefore, i set it manually to point to R1.

`show running-config`
`ip default-gateway 192.168.11.1`

```
ssh -J admin@192.168.11.2 admin@192.168.10.1 <fail>
ssh -J admin@192.168.11.2 admin@192.168.11.1 <success>
```

At this point, **only S1 is directly accessible** and **others were not**.

That's when it hit me: **my PC has multiple network connections** one for the lab network, another for internet. A quick look at the routing table with `ip route` showed that **the default gateway pointed to my internet connection**, not the lab. I didn’t want to mess with that changing it could interrupt my internet access. So, I accepted the reality, that **To access R2 and S2, I’d have to proxy jump through S1**.

Lessons Learned :

* **Legacy devices require legacy crypto algorithms**: Modern SSH clients won’t talk to them unless you manually enable old algorithms.
* **routing protocols enable reachability for remote networks**: RIP was the key to getting the devices to talk to each other.
* **Default gateways matter**: Both on the network devices and my own machine.
* **ProxyJump is your friend**: Especially when direct SSH access isn’t feasible.

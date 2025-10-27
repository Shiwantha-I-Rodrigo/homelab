**~ King Solomon’s Carpet ~** <sub><sup>by Barbara Vine</sup></sub>

---

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
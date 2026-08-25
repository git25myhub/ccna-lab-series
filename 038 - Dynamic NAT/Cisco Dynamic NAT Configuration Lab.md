# Cisco Dynamic NAT Configuration Lab

## Lab Objective

In this lab, you will troubleshoot why internal PCs cannot initially reach an external server and configure **Dynamic NAT (Network Address Translation)** on R1 to allow the inside network to communicate through the simulated Internet.

You will:

- Understand why RIP alone does not provide end-to-end connectivity through the simulated Internet.
- Configure R1 as a NAT router.
- Translate the `192.168.1.0/24` inside network into a public address pool.
- Configure a dynamic NAT pool using addresses `1.2.3.10–1.2.3.20`.
- Test connectivity from the PCs to SRV1.
- Verify NAT translations on R1.

---

## Network Scenario

RIP has already been configured so that R1 and R2 can reach their respective inside networks.

However, PC1, PC2, and PC3 are unable to successfully ping **SRV1**.

The serial connection between R1 and R2 represents the **Internet** and has ACLs configured to simulate Internet filtering.

The private addresses used by the PCs are not permitted to cross this simulated Internet connection directly. Therefore, R1 must translate the private source addresses into addresses from the public NAT pool.

### Important Networks

| Purpose | Network / Address |
|---|---|
| Inside network | `192.168.1.0/24` |
| NAT pool | `1.2.3.10–1.2.3.20` |
| SRV1 | `1.1.1.100` |
| NAT inside interface | R1 `G0/0` |
| NAT outside interface | R1 `S0/3/0` |

---

# Part 1 — Troubleshoot the Connectivity Problem

Before configuring NAT, test connectivity from the PCs to SRV1.

From PC1, PC2, and PC3, run:

```text
ping 1.1.1.100
```

### Expected Result

The pings should fail.

### Why Do the Pings Fail?

RIP allows R1 and R2 to learn routes to their respective internal networks, but **routing alone does not solve the problem**.

The PCs use private IP addresses from:

```text
192.168.1.0/24
```

When traffic travels toward the simulated Internet, the source address is still a private address.

The ACLs on the serial connection are designed to simulate an Internet environment where these private addresses cannot be used directly.

Therefore, R1 needs to perform **Network Address Translation (NAT)**.

NAT will replace the private source address with a public address from the configured NAT pool.

---

# Part 2 — Configure Dynamic NAT on R1

## Step 1 — Identify the NAT Inside Interface

The interface connected to the internal LAN is R1's:

```text
GigabitEthernet0/0
```

Configure it as the NAT inside interface:

```cisco
R1# configure terminal
R1(config)# interface gigabitEthernet0/0
R1(config-if)# ip nat inside
```

---

## Step 2 — Identify the NAT Outside Interface

The serial interface connects R1 toward the simulated Internet.

Configure:

```cisco
R1(config-if)# exit
R1(config)# interface serial0/3/0
R1(config-if)# ip nat outside
R1(config-if)# exit
```

The resulting configuration should conceptually be:

```text
G0/0       → NAT inside
S0/3/0     → NAT outside
```

---

# Step 3 — Create an Access List for the Inside Network

The NAT configuration needs to identify which source addresses should be translated.

The inside network is:

```text
192.168.1.0/24
```

Configure standard ACL 1:

```cisco
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

This tells R1:

> Permit addresses belonging to `192.168.1.0/24` to be considered for NAT.

---

# Step 4 — Create the Dynamic NAT Pool

Create a NAT pool named `pool1` using the addresses:

```text
1.2.3.10 – 1.2.3.20
```

The subnet mask is:

```text
255.255.255.0
```

Configure:

```cisco
R1(config)# ip nat pool pool1 1.2.3.10 1.2.3.20 netmask 255.255.255.0
```

### Important

The NAT pool command requires both the **starting and ending address**:

```text
ip nat pool <name> <start-address> <end-address> netmask <mask>
```

For this lab:

```text
pool name       = pool1
start address   = 1.2.3.10
end address     = 1.2.3.20
netmask         = 255.255.255.0
```

---

# Step 5 — Associate the ACL with the NAT Pool

Now connect ACL 1 to the NAT pool:

```cisco
R1(config)# ip nat inside source list 1 pool pool1
```

This configuration tells R1:

> Translate traffic from addresses permitted by ACL 1 using addresses from NAT pool `pool1`.

---

# Step 6 — Save the Configuration

Save the configuration:

```cisco
R1(config)# end
R1# write memory
```

or:

```cisco
R1# copy running-config startup-config
```

---

# Part 3 — Test Connectivity

Return to PC1, PC2, and PC3 and ping SRV1.

```text
ping 1.1.1.100
```

The first ping may fail while the NAT translation is being created.

Run the ping again if necessary.

### Expected Result

You should eventually receive replies similar to:

```text
Reply from 1.1.1.100: bytes=32 time=10ms TTL=126
Reply from 1.1.1.100: bytes=32 time=2ms TTL=126
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
```

The PCs can now communicate with SRV1 because their private addresses are being translated into addresses from the NAT pool.

---

# Part 4 — Verify NAT Translations

On R1, use:

```cisco
R1# show ip nat translations
```

You should see entries similar to:

```text
Pro  Inside global     Inside local       Outside local      Outside global
icmp 1.2.3.11:1        192.168.1.12:1     1.1.1.100:1        1.1.1.100:1
icmp 1.2.3.12:1        192.168.1.13:1     1.1.1.100:1        1.1.1.100:1
```

The exact entries and ICMP identifiers may differ depending on which PC generated traffic.

---

## Understanding the NAT Translation Table

The output contains four important columns:

| Column | Meaning |
|---|---|
| Inside global | Translated/public address |
| Inside local | Original private address |
| Outside local | Destination address as seen locally |
| Outside global | Actual outside destination address |

For example:

```text
Inside global     Inside local
1.2.3.11          192.168.1.12
```

means:

```text
192.168.1.12 → 1.2.3.11
```

R1 is translating the private PC address into a public address from the NAT pool.

---

# Useful Verification Commands

### Display NAT translations

```cisco
R1# show ip nat translations
```

### Display NAT statistics

```cisco
R1# show ip nat statistics
```

### Verify the NAT configuration

```cisco
R1# show running-config | include ip nat
```

### Verify the ACL

```cisco
R1# show access-lists
```

### Verify the routing table

```cisco
R1# show ip route
```

---

# Troubleshooting

If the PCs still cannot ping SRV1, check the following.

### 1. Verify NAT interfaces

```cisco
R1# show running-config interface gigabitEthernet0/0
R1# show running-config interface serial0/3/0
```

Confirm:

```text
G0/0 → ip nat inside
S0/3/0 → ip nat outside
```

### 2. Verify the ACL

```cisco
R1# show access-lists
```

Confirm ACL 1 permits:

```text
192.168.1.0 0.0.0.255
```

### 3. Verify the NAT pool

```cisco
R1# show running-config | include ip nat pool
```

Expected:

```cisco
ip nat pool pool1 1.2.3.10 1.2.3.20 netmask 255.255.255.0
```

### 4. Verify the NAT rule

```cisco
R1# show running-config | include ip nat inside source
```

Expected:

```cisco
ip nat inside source list 1 pool pool1
```

### 5. Generate traffic

NAT translations are created dynamically when traffic passes through the router.

From a PC:

```text
ping 1.1.1.100
```

Then check:

```cisco
R1# show ip nat translations
```

---

# Key Concepts Learned

## RIP vs NAT

RIP is responsible for **routing**.

NAT is responsible for **translating addresses**.

In this lab:

```text
RIP
 ↓
Provides routes

NAT
 ↓
Translates private addresses

ACLs
 ↓
Simulate Internet restrictions

SRV1
 ↓
1.1.1.100
```

Having a valid route does not automatically mean that private IP addresses can successfully communicate through an Internet-like environment.

---

# Final Configuration

The important R1 configuration should contain:

```cisco
interface GigabitEthernet0/0
 ip nat inside

interface Serial0/3/0
 ip nat outside

access-list 1 permit 192.168.1.0 0.0.0.255

ip nat pool pool1 1.2.3.10 1.2.3.20 netmask 255.255.255.0

ip nat inside source list 1 pool pool1
```

Save the configuration:

```cisco
R1# write memory
```

---

# Completion Criteria

The lab is successfully completed when:

- [ ] R1's `G0/0` is configured as `ip nat inside`.
- [ ] R1's `S0/3/0` is configured as `ip nat outside`.
- [ ] ACL 1 permits `192.168.1.0/24`.
- [ ] NAT pool `pool1` contains addresses `1.2.3.10–1.2.3.20`.
- [ ] Dynamic NAT is associated with ACL 1 and `pool1`.
- [ ] PC1 can ping SRV1.
- [ ] PC2 can ping SRV1.
- [ ] PC3 can ping SRV1.
- [ ] `show ip nat translations` displays active translations.
- [ ] The configuration has been saved.

---

## Key Takeaway

**RIP provides reachability, but NAT provides the address translation required for the private `192.168.1.0/24` network to communicate through the simulated Internet.**

The essential NAT relationship is:

```text
192.168.1.x
    ↓
   R1
    ↓ NAT
1.2.3.10–1.2.3.20
    ↓
Simulated Internet
    ↓
1.1.1.100 (SRV1)
```
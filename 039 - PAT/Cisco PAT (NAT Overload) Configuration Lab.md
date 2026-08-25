# Cisco PAT (NAT Overload) Configuration Lab

## Lab Objective

In this lab, you will troubleshoot connectivity between an internal LAN and an external server, then configure **Port Address Translation (PAT)** on R1.

PAT allows multiple devices using private IP addresses to access an external network by sharing a **single public IP address**.

You will:

- Understand why PC1, PC2, and PC3 initially cannot reach SRV1.
- Configure R1's inside and outside NAT interfaces.
- Identify the internal network using an ACL.
- Configure PAT using R1's `S0/3/0` interface.
- Enable NAT overload.
- Test connectivity from all PCs to SRV1.
- Verify the resulting PAT translations.

---

# Network Scenario

RIP has already been configured so that R1 and R2 can reach their respective inside networks.

However, PC1, PC2, and PC3 initially cannot successfully ping SRV1.

The serial connection between R1 and R2 represents the **Internet** and has ACLs configured to simulate Internet filtering.

The PCs use private addresses from:

```text
192.168.1.0/24
```

These private addresses cannot be used directly across the simulated Internet.

R1 must therefore translate the private source addresses into its own outside interface address.

This is accomplished using **PAT with overload**.

---

# Addressing Information

| Device / Network | Address |
|---|---|
| Inside LAN | `192.168.1.0/24` |
| PC1 | `192.168.1.11` |
| PC2 | `192.168.1.12` |
| PC3 | `192.168.1.13` |
| SRV1 | `1.1.1.100` |
| R1 inside interface | `G0/0` |
| R1 outside interface | `S0/3/0` |
| PAT public address | R1 `S0/3/0` address |

> **Note:** The exact IP address configured on `S0/3/0` is not hard-coded into the PAT configuration. Instead, R1 dynamically uses the IP address currently assigned to that interface.

---

# Part 1 — Troubleshoot the Connectivity Problem

Before configuring PAT, test connectivity from the PCs to SRV1.

From PC1, PC2, and PC3:

```text
ping 1.1.1.100
```

### Expected Result

The initial ping should fail:

```text
Request timed out.
Request timed out.
Request timed out.
Request timed out.
```

### Why Does It Fail?

RIP provides the necessary routing information, but routing alone is not enough.

The PCs use private addresses:

```text
192.168.1.11
192.168.1.12
192.168.1.13
```

These addresses are not translated before entering the simulated Internet.

The ACLs on the serial connection are simulating an Internet environment that does not allow these private addresses to pass directly.

Therefore, R1 needs to translate the private addresses into a usable outside address.

---

# Part 2 — Configure PAT on R1

PAT is sometimes called:

- NAT Overload
- Port Address Translation
- Many-to-One NAT

Unlike Dynamic NAT, which uses a pool of multiple public addresses, PAT allows **many internal devices to share one public IP address**.

In this lab, the single public IP address is the address assigned to:

```text
R1 S0/3/0
```

---

## Step 1 — Configure the Inside NAT Interface

R1's `G0/0` interface connects to the internal LAN.

Enter:

```cisco
R1# configure terminal
R1(config)# interface gigabitEthernet0/0
R1(config-if)# ip nat inside
```

This tells R1 that traffic entering through `G0/0` originates from the NAT inside network.

---

## Step 2 — Configure the Outside NAT Interface

R1's `S0/3/0` interface connects toward the simulated Internet.

Configure:

```cisco
R1(config-if)# exit
R1(config)# interface serial0/3/0
R1(config-if)# ip nat outside
R1(config-if)# exit
```

The NAT interfaces should now be:

```text
G0/0       → NAT inside
S0/3/0     → NAT outside
```

---

# Step 3 — Create an ACL for the Inside Network

The ACL identifies the addresses that should be translated.

The inside network is:

```text
192.168.1.0/24
```

Configure:

```cisco
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

This ACL matches all hosts in:

```text
192.168.1.0 – 192.168.1.255
```

---

# Step 4 — Configure PAT Using the Outside Interface

Now configure PAT to use the IP address assigned to R1's `S0/3/0` interface.

Enter:

```cisco
R1(config)# ip nat inside source list 1 interface serial0/3/0 overload
```

This command is the key configuration in this lab.

It means:

```text
Inside source
      ↓
ACL 1
      ↓
Use R1's Serial0/3/0 address
      ↓
OVERLOAD
```

The `overload` keyword is extremely important.

Without `overload`, the router would not be configured to allow multiple internal hosts to share the same outside address using different port numbers.

---

# Step 5 — Save the Configuration

Exit configuration mode:

```cisco
R1(config)# end
```

Save the configuration:

```cisco
R1# write memory
```

or:

```cisco
R1# copy running-config startup-config
```

---

# Part 3 — Test Connectivity

From each PC, ping SRV1:

```text
ping 1.1.1.100
```

For example:

```text
C:\>ping 1.1.1.100
```

You should receive replies similar to:

```text
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
Reply from 1.1.1.100: bytes=32 time=4ms TTL=126
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
```

You may need to run the ping again if the first attempt times out.

This is normal because the first packet can trigger the creation of the NAT translation.

---

# Part 4 — Verify PAT Translations

After generating traffic from the PCs, check the NAT table on R1:

```cisco
R1# show ip nat translations
```

Your output should look similar to:

```text
Pro  Inside global     Inside local       Outside local      Outside global
icmp 1.2.3.1:1024      192.168.1.13:1     1.1.1.100:1        1.1.1.100:1024
icmp 1.2.3.1:1025      192.168.1.13:2     1.1.1.100:2        1.1.1.100:1025
icmp 1.2.3.1:1         192.168.1.12:1     1.1.1.100:1        1.1.1.100:1
icmp 1.2.3.1:5         192.168.1.11:5     1.1.1.100:5        1.1.1.100:5
```

The exact entries will vary depending on the order and number of pings generated.

---

# Understanding the PAT Table

The most important part of the output is the relationship between:

```text
Inside global
```

and:

```text
Inside local
```

For example:

```text
Inside global     Inside local
1.2.3.1:5         192.168.1.11:5
1.2.3.1:1         192.168.1.12:1
1.2.3.1:1024      192.168.1.13:1
```

This demonstrates that multiple internal devices are using the **same outside IP address**:

```text
1.2.3.1
```

The port/identifier information allows R1 to distinguish the different connections.

Conceptually:

```text
PC1 192.168.1.11 ─┐
                   │
PC2 192.168.1.12 ─┼──> R1 PAT ──> 1.2.3.1 ──> Internet
                   │
PC3 192.168.1.13 ─┘
```

All three PCs can therefore share R1's single outside interface address.

---

# Dynamic NAT vs PAT

This lab is closely related to the previous Dynamic NAT lab.

## Dynamic NAT

Dynamic NAT uses a pool of public addresses:

```text
192.168.1.11 → 1.2.3.10
192.168.1.12 → 1.2.3.11
192.168.1.13 → 1.2.3.12
```

Each internal host can receive a different public address from the pool.

## PAT

PAT allows multiple internal hosts to share one address:

```text
192.168.1.11 ─┐
192.168.1.12 ─┼──> 1.2.3.1
192.168.1.13 ─┘
```

R1 uses port/identifier information to keep the translations separate.

---

# Useful Verification Commands

## Display NAT translations

```cisco
R1# show ip nat translations
```

This is the primary command required by the lab.

---

## Display NAT statistics

```cisco
R1# show ip nat statistics
```

This can show information such as:

- Active translations
- Inside/outside interfaces
- NAT hits
- NAT misses
- Configured NAT rules

---

## Verify NAT interfaces

```cisco
R1# show running-config
```

Look for:

```cisco
interface GigabitEthernet0/0
 ip nat inside

interface Serial0/3/0
 ip nat outside
```

---

## Verify the ACL

```cisco
R1# show access-lists
```

You should see:

```text
access-list 1 permit 192.168.1.0 0.0.0.255
```

---

## Verify the PAT rule

```cisco
R1# show running-config | include ip nat
```

You should see something similar to:

```text
ip nat inside
ip nat outside
ip nat inside source list 1 interface Serial0/3/0 overload
```

---

# Troubleshooting

If the PCs cannot reach SRV1 after configuring PAT, check the following.

### Check 1 — NAT inside interface

```cisco
R1# show running-config interface gigabitEthernet0/0
```

Confirm:

```cisco
ip nat inside
```

---

### Check 2 — NAT outside interface

```cisco
R1# show running-config interface serial0/3/0
```

Confirm:

```cisco
ip nat outside
```

---

### Check 3 — ACL

```cisco
R1# show access-lists
```

Confirm ACL 1 contains:

```text
192.168.1.0 0.0.0.255
```

---

### Check 4 — PAT configuration

Use:

```cisco
R1# show running-config | include ip nat
```

The important command should be:

```cisco
ip nat inside source list 1 interface Serial0/3/0 overload
```

Make sure **`overload`** is present.

---

### Check 5 — Generate traffic

NAT translations are created dynamically.

From a PC:

```text
ping 1.1.1.100
```

Then immediately check:

```cisco
R1# show ip nat translations
```

---

# Final Configuration

The essential R1 configuration for this lab is:

```cisco
interface GigabitEthernet0/0
 ip nat inside

interface Serial0/3/0
 ip nat outside

access-list 1 permit 192.168.1.0 0.0.0.255

ip nat inside source list 1 interface Serial0/3/0 overload
```

Save the configuration:

```cisco
R1# write memory
```

---

# Lab Completion Checklist

- [ ] R1 `G0/0` is configured as `ip nat inside`.
- [ ] R1 `S0/3/0` is configured as `ip nat outside`.
- [ ] ACL 1 permits `192.168.1.0/24`.
- [ ] PAT uses R1's `S0/3/0` interface.
- [ ] The PAT configuration includes the `overload` keyword.
- [ ] PC1 can ping `1.1.1.100`.
- [ ] PC2 can ping `1.1.1.100`.
- [ ] PC3 can ping `1.1.1.100`.
- [ ] `show ip nat translations` displays translations.
- [ ] Multiple internal addresses are translated to the same outside interface address.
- [ ] The configuration is saved.

---

# Key Takeaway

**PAT allows multiple private devices to share one public IP address.**

In this lab, R1 translates:

```text
192.168.1.11 ─┐
192.168.1.12 ─┼──> R1 S0/3/0 address
192.168.1.13 ─┘
```

The critical configuration is:

```cisco
ip nat inside source list 1 interface serial0/3/0 overload
```

The word **`overload`** is what enables multiple internal hosts to share the same outside interface address.

The `show ip nat translations` command provides visible evidence of this behavior by showing multiple `192.168.1.x` addresses being translated to the same `1.2.3.1` address with different identifiers/ports.
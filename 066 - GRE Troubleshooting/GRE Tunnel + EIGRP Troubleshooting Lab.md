# GRE Tunnel + EIGRP Troubleshooting Lab

## 📌 Lab Overview

This lab focuses on troubleshooting a **GRE tunnel running EIGRP** between two Cisco routers.

The network is designed to provide connectivity between:

- **10.0.1.0/24** — R1 LAN
- **10.0.2.0/24** — R2 LAN

However, there are **TWO misconfigurations on each router**.

The objective is to identify and correct all four problems and restore end-to-end connectivity between the two LANs.

---

# 🎯 Lab Requirements

The network must satisfy the following requirements:

1. The **GRE tunnel between R1 and R2** must operate correctly.
2. The GRE tunnel must use the correct WAN interfaces as its source.
3. The GRE tunnel endpoints must be reachable through the underlying WAN network.
4. **EIGRP AS 100** must operate over the GRE tunnel.
5. R1 must advertise `10.0.1.0/24`.
6. R2 must advertise `10.0.2.0/24`.
7. The GRE tunnel network `192.168.1.0/30` must participate in EIGRP.
8. R1 and R2 must establish an EIGRP neighbor relationship.
9. R1 must learn `10.0.2.0/24`.
10. R2 must learn `10.0.1.0/24`.
11. Hosts on the two LANs must be able to communicate.

---

# 🗺️ Network Addressing

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | G0/0 | 10.0.1.1/24 | R1 LAN |
| R1 | S0/0/0 | 100.0.0.2/30 | WAN |
| R1 | Tunnel0 | 192.168.1.1/30 | GRE Tunnel |
| R2 | G0/0 | 10.0.2.1/24 | R2 LAN |
| R2 | S0/0/0 | 200.0.0.2/30 | WAN |
| R2 | Tunnel0 | 192.168.1.2/30 | GRE Tunnel |

### Network Diagram

```text
             GRE Tunnel
       192.168.1.0/30
   ┌────────────────────────┐
   │                        │
   │                        │
R1 Tunnel0                R2 Tunnel0
192.168.1.1              192.168.1.2
   │                        │
   │                        │
  R1                      R2
   │                        │
G0/0                     G0/0
10.0.1.1                 10.0.2.1
   │                        │
10.0.1.0/24             10.0.2.0/24
   │                        │
 PC1                      PC2

WAN:
R1 S0/0/0 = 100.0.0.2
R2 S0/0/0 = 200.0.0.2
```

---

# 🔎 Initial Troubleshooting

The first step was to inspect the interface status on both routers.

## R1 Initial Status

```cisco
R1# show ip interface brief
```

Initial output:

```text
Interface              IP-Address      Status                Protocol
GigabitEthernet0/0     10.0.1.1        up                    up
Serial0/0/0            100.0.0.2       up                    down
Tunnel0                192.168.1.1     up                    down
```

The serial interface initially showed:

```text
up/down
```

and the GRE tunnel was also:

```text
up/down
```

This indicated that the tunnel was not yet operational.

---

# ❌ R1 Misconfiguration #1 — Incorrect GRE Destination

The GRE tunnel configuration was inspected using:

```cisco
R1# show interface tunnel 0
```

The problematic configuration showed:

```text
Tunnel source 100.0.0.2 (Serial0/0/0), destination 192.168.1.2
```

The tunnel destination was incorrectly configured as:

```text
192.168.1.2
```

This is the **remote tunnel IP**, not the remote GRE transport endpoint.

The GRE destination must point to the remote router's reachable WAN address:

```text
200.0.0.2
```

### Fix

```cisco
R1(config)# interface tunnel 0
R1(config-if)# tunnel destination 200.0.0.2
```

After correcting the destination, Tunnel0 changed to:

```text
Tunnel0 is up, line protocol is up
```

---

# ❌ R1 Misconfiguration #2 — Incorrect EIGRP Network Statement

The EIGRP configuration was checked:

```cisco
R1# show ip protocols
```

The initial routing networks included:

```text
10.0.1.0/24
100.0.0.0/30
```

The problem was that EIGRP was enabled on the **WAN network**:

```text
100.0.0.0/30
```

instead of the GRE tunnel network:

```text
192.168.1.0/30
```

EIGRP should form its adjacency across **Tunnel0**, not across the physical WAN transport network.

### Fix

Remove the incorrect WAN network:

```cisco
R1(config)# router eigrp 100
R1(config-router)# no network 100.0.0.0 0.0.0.3
```

Add the GRE tunnel network:

```cisco
R1(config-router)# network 192.168.1.0 0.0.0.3
```

### Correct R1 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.1.0 0.0.0.255
 network 192.168.1.0 0.0.0.3
```

---

# ❌ R2 Misconfiguration #1 — Incorrect GRE Tunnel Source

The GRE tunnel configuration on R2 was inspected:

```cisco
R2# show interface tunnel 0
```

The output showed:

```text
Tunnel source 10.0.2.1 (GigabitEthernet0/0), destination 100.0.0.1
```

The tunnel source was incorrectly configured to use:

```text
GigabitEthernet0/0
10.0.2.1
```

The GRE tunnel should use the WAN serial interface as its transport source.

The correct source is:

```text
Serial0/0/0
200.0.0.2
```

### Fix

```cisco
R2(config)# interface tunnel 0
R2(config-if)# tunnel source Serial0/0/0
```

This changes the GRE transport source from the LAN interface to the WAN interface.

---

# ❌ R2 Misconfiguration #2 — Incorrect GRE Tunnel Destination

The original R2 tunnel configuration showed:

```text
Tunnel destination 100.0.0.1
```

The required remote GRE endpoint is R1's WAN address:

```text
100.0.0.2
```

Therefore, the tunnel destination was incorrect.

### Fix

```cisco
R2(config)# interface tunnel 0
R2(config-if)# tunnel destination 100.0.0.2
```

### Correct R2 GRE Configuration

```cisco
interface Tunnel0
 ip address 192.168.1.2 255.255.255.252
 tunnel source Serial0/0/0
 tunnel destination 100.0.0.2
```

---

# 🛠️ Complete Corrected Configuration

## R1

```cisco
interface GigabitEthernet0/0
 ip address 10.0.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 ip address 100.0.0.2 255.255.255.252
 no shutdown

interface Tunnel0
 ip address 192.168.1.1 255.255.255.252
 tunnel source Serial0/0/0
 tunnel destination 200.0.0.2

ip route 0.0.0.0 0.0.0.0 100.0.0.1

router eigrp 100
 network 10.0.1.0 0.0.0.255
 network 192.168.1.0 0.0.0.3
```

---

## R2

```cisco
interface GigabitEthernet0/0
 ip address 10.0.2.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 ip address 200.0.0.2 255.255.255.252
 no shutdown

interface Tunnel0
 ip address 192.168.1.2 255.255.255.252
 tunnel source Serial0/0/0
 tunnel destination 100.0.0.2

ip route 0.0.0.0 0.0.0.0 200.0.0.1

router eigrp 100
 network 10.0.2.0 0.0.0.255
 network 192.168.1.0 0.0.0.3
```

---

# 🌐 Why the Default Routes Are Important

GRE requires the tunnel destination to be reachable through the underlying network.

R1 therefore needs a route toward R2's WAN endpoint:

```text
200.0.0.2
```

R2 needs a route toward R1's WAN endpoint:

```text
100.0.0.2
```

The lab uses default routes:

### R1

```cisco
ip route 0.0.0.0 0.0.0.0 100.0.0.1
```

### R2

```cisco
ip route 0.0.0.0 0.0.0.0 200.0.0.1
```

These routes allow the routers to reach the remote GRE endpoints.

---

# 🔄 Verify the GRE Tunnel

On R1:

```cisco
R1# show interface tunnel 0
```

The expected result is:

```text
Tunnel0 is up, line protocol is up
```

The tunnel should show:

```text
Internet address is 192.168.1.1/30
Tunnel source 100.0.0.2 (Serial0/0/0)
Tunnel destination 200.0.0.2
Tunnel protocol/transport GRE/IP
```

On R2:

```cisco
R2# show interface tunnel 0
```

Expected:

```text
Tunnel0 is up, line protocol is up
```

With:

```text
Internet address is 192.168.1.2/30
Tunnel source 200.0.0.2 (Serial0/0/0)
Tunnel destination 100.0.0.2
Tunnel protocol/transport GRE/IP
```

---

# 🧪 Test GRE Connectivity

From R1:

```cisco
R1# ping 192.168.1.2
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

From R2:

```cisco
R2# ping 192.168.1.1
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

If these pings succeed, the GRE tunnel is working correctly.

---

# 🤝 Verify EIGRP

Check the EIGRP neighbors:

```cisco
R1# show ip eigrp neighbors
```

R1 should see:

```text
192.168.1.2
```

over:

```text
Tunnel0
```

R2 should see:

```text
192.168.1.1
```

over:

```text
Tunnel0
```

The important point is that the EIGRP adjacency should be formed **over the GRE tunnel**.

---

# 🗺️ Verify Routing Tables

## R1

Run:

```cisco
R1# show ip route
```

R1 should learn R2's LAN:

```text
D       10.0.2.0/24 via 192.168.1.2, Tunnel0
```

The `D` indicates an **EIGRP-learned route**.

---

## R2

Run:

```cisco
R2# show ip route
```

R2 should learn R1's LAN:

```text
D       10.0.1.0/24 via 192.168.1.1, Tunnel0
```

Again, the `D` indicates that the route was learned through EIGRP.

---

# 💻 End-to-End Connectivity Test

After correcting all four misconfigurations, test connectivity between the LANs.

From a PC on the `10.0.2.0/24` network:

```text
C:\> ping 10.0.1.100
```

Or from a PC on the `10.0.1.0/24` network:

```text
C:\> ping 10.0.2.100
```

The expected result is successful replies.

```text
Reply from 10.0.1.100: bytes=32 time<1ms TTL=126
Reply from 10.0.1.100: bytes=32 time<1ms TTL=126
Reply from 10.0.1.100: bytes=32 time<1ms TTL=126
```

---

# 🔍 Troubleshooting Methodology

The following troubleshooting sequence is useful for GRE + EIGRP problems.

### Step 1 — Check physical interfaces

```cisco
show ip interface brief
```

Confirm that the WAN and LAN interfaces are operational.

### Step 2 — Check WAN reachability

From R1:

```cisco
ping 200.0.0.2
```

From R2:

```cisco
ping 100.0.0.2
```

### Step 3 — Check GRE configuration

```cisco
show interface tunnel 0
```

Verify:

- Tunnel source
- Tunnel destination
- Tunnel IP address
- Tunnel status

### Step 4 — Test the tunnel

```cisco
ping 192.168.1.2
```

or:

```cisco
ping 192.168.1.1
```

### Step 5 — Check EIGRP

```cisco
show ip protocols
```

Verify that EIGRP includes:

```text
10.0.x.0/24
192.168.1.0/30
```

and does not unnecessarily form the EIGRP adjacency over the WAN transport network.

### Step 6 — Check EIGRP neighbors

```cisco
show ip eigrp neighbors
```

### Step 7 — Check EIGRP routes

```cisco
show ip route eigrp
```

### Step 8 — Test end-to-end connectivity

```text
ping <remote-PC-IP>
```

---

# 🧠 Key Concepts Learned

## GRE Tunnel

GRE creates a virtual tunnel between two routers.

The tunnel uses:

```text
192.168.1.1/30
        |
        |
     GRE Tunnel
        |
        |
192.168.1.2/30
```

The GRE packets are transported across the underlying WAN network.

---

## GRE Source vs Destination

A common troubleshooting mistake is confusing the **tunnel IP addresses** with the **GRE transport endpoint addresses**.

In this lab:

| Router | Tunnel IP | GRE Source | GRE Destination |
|---|---|---|---|
| R1 | 192.168.1.1 | 100.0.0.2 | 200.0.0.2 |
| R2 | 192.168.1.2 | 200.0.0.2 | 100.0.0.2 |

The tunnel destination must be the **remote router's reachable WAN address**, not its tunnel address.

---

## EIGRP Over GRE

EIGRP is configured on the GRE tunnel network:

```cisco
network 192.168.1.0 0.0.0.3
```

This allows R1 and R2 to establish their EIGRP neighbor relationship over Tunnel0.

The physical WAN network is used to **transport the GRE packets**, while the GRE tunnel provides the logical path over which EIGRP operates.

---

# 📝 Misconfiguration Summary

| Router | Misconfiguration | Problem | Correction |
|---|---|---|---|
| R1 | GRE destination | Destination pointed to `192.168.1.2` | Change to `200.0.0.2` |
| R1 | EIGRP network | EIGRP enabled on `100.0.0.0/30` instead of GRE | Remove WAN network and add `192.168.1.0/30` |
| R2 | GRE source | Tunnel sourced from G0/0 | Change source to S0/0/0 |
| R2 | GRE destination | Destination pointed to `100.0.0.1` | Change to `100.0.0.2` |

---

# ✅ Completion Checklist

The lab is successfully completed when:

- [x] R1 WAN interface is operational.
- [x] R2 WAN interface is operational.
- [x] R1 can reach R2's WAN endpoint.
- [x] R2 can reach R1's WAN endpoint.
- [x] R1 Tunnel0 is up/up.
- [x] R2 Tunnel0 is up/up.
- [x] R1 Tunnel0 uses Serial0/0/0 as its source.
- [x] R2 Tunnel0 uses Serial0/0/0 as its source.
- [x] R1 Tunnel destination is `200.0.0.2`.
- [x] R2 Tunnel destination is `100.0.0.2`.
- [x] EIGRP AS 100 is configured on both routers.
- [x] EIGRP operates over Tunnel0.
- [x] R1 and R2 form an EIGRP adjacency.
- [x] R1 learns `10.0.2.0/24`.
- [x] R2 learns `10.0.1.0/24`.
- [x] PC1 can communicate with PC2.
- [x] PC2 can communicate with PC1.

---

# 📚 Final Summary

This troubleshooting lab demonstrates how multiple configuration errors can prevent a GRE tunnel and EIGRP from functioning correctly.

The four issues were:

1. **R1 had an incorrect GRE destination.**
2. **R1 had the wrong EIGRP network statement.**
3. **R2 had an incorrect GRE tunnel source.**
4. **R2 had an incorrect GRE tunnel destination.**

After correcting these issues, the expected operation is:

```text
        EIGRP AS 100
             |
             v
PC1 ── R1 ═══════════ R2 ── PC2
      │   GRE Tunnel    │
      │ 192.168.1.0/30  │
      │                 │
 10.0.1.0/24       10.0.2.0/24
```

The GRE tunnel provides the logical connection between the routers, while EIGRP dynamically advertises the LAN networks across that tunnel, allowing end-to-end communication between the two sites.
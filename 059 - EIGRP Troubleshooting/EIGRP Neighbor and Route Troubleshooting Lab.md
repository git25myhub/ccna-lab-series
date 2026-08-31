# EIGRP Neighbor and Route Troubleshooting Lab

## 📌 Lab Overview

In this lab, the network contains several EIGRP configuration errors that prevent routers from becoming EIGRP neighbors and learning the correct routes.

The objective is to troubleshoot the EIGRP configuration on each router, identify **one misconfiguration per router**, correct the issues, and verify full EIGRP convergence.

### Lab Requirements

- Routers must successfully establish EIGRP neighbor adjacencies.
- All routers must use **EIGRP Autonomous System 100**.
- The appropriate connected networks must be advertised through EIGRP.
- Loopback interfaces should remain passive.
- Automatic summarization must be disabled.
- **R3 must advertise a `10.0.0.0/8` summary toward R5.**
- R5 must receive the `10.0.0.0/8` summary route from R3.

---

## 🗺️ Network Topology

The topology consists of five routers:

```text
             10.14.0.0/24
        R1 ---------------- R4
        |                   |
        |                   |
 10.12.0.0/24          10.34.0.0/24
        |                   |
        R2 ---------------- R3
             10.23.0.0/24
                              \
                               \
                            10.35.0.0/24
                                 |
                                 R5
```

### Loopback Addresses

| Router | Loopback |
|--------|----------|
| R1 | `1.1.1.1/32` |
| R2 | `2.2.2.2/32` |
| R3 | `3.3.3.3/32` |
| R4 | `4.4.4.4/32` |
| R5 | `5.5.5.5/32` |

### Inter-Router Networks

| Link | Network |
|------|---------|
| R1 ↔ R2 | `10.12.0.0/24` |
| R1 ↔ R4 | `10.14.0.0/24` |
| R2 ↔ R3 | `10.23.0.0/24` |
| R3 ↔ R4 | `10.34.0.0/24` |
| R3 ↔ R5 | `10.35.0.0/24` |

---

# 🔍 Troubleshooting Process

## 1. Verify EIGRP Neighbors

The first step was to check whether EIGRP adjacencies were being formed.

```cisco
show ip eigrp neighbors
```

Initially, R1 displayed:

```text
IP-EIGRP neighbors for process 10
```

but no neighbors were listed.

This indicated an EIGRP process mismatch.

---

# 🛠️ Router Fixes

## R1 — EIGRP Autonomous System Mismatch

### Problem

R1 was configured with:

```cisco
router eigrp 10
```

while the rest of the network was using EIGRP AS **100**.

Because EIGRP neighbors must use the same autonomous-system number, R1 could not establish an adjacency with the other routers.

### Fix

The incorrect EIGRP process was removed and recreated using AS 100:

```cisco
no router eigrp 10

router eigrp 100
 network 10.0.0.0
 network 1.1.1.1 0.0.0.0
 passive-interface Loopback0
 no auto-summary
```

### Verification

After the correction, R1 established an adjacency with R4:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 10.14.0.4 (GigabitEthernet0/0) is up
```

R1 subsequently established an adjacency with R2 as well:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 10.12.0.2 (FastEthernet1/0) is up
```

---

## R2 — EIGRP Configuration

R2 was verified as part of the EIGRP AS 100 domain and successfully formed an adjacency with R3.

The important requirement was ensuring that R2 participates in the same EIGRP process and advertises its connected networks.

### Verification

An EIGRP neighbor relationship was observed between R2 and R3:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 10.23.0.3 (FastEthernet1/0) is up
```

---

## R3 — EIGRP Summary Advertisement Misconfigured

### Problem

R3 was required to send a summarized route for the `10.0.0.0/8` network toward R5.

However, the summary was configured on the wrong interface:

```cisco
interface FastEthernet1/0
 ip summary-address eigrp 100 10.0.0.0 255.0.0.0 5
```

`FastEthernet1/0` connects R3 to R2:

```text
R3 Fa1/0 → R2
```

The summary therefore needed to be configured on the interface connecting R3 to R5.

### Fix

The incorrect summary was removed:

```cisco
interface FastEthernet1/0
 no ip summary-address eigrp 100 10.0.0.0 255.0.0.0 5
```

The summary was then configured on R3's GigabitEthernet interface toward R5:

```cisco
interface GigabitEthernet0/0
 ip summary-address eigrp 100 10.0.0.0 255.0.0.0 5
```

The `5` represents the administrative distance for the summary route.

### Verification

R3 successfully maintained an EIGRP adjacency with R5:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 10.35.0.5 (GigabitEthernet0/0) is up
```

R3 now advertises the `10.0.0.0/8` summary toward R5.

---

## R4 — Incorrect Loopback Network Advertisement

### Problem

R4's EIGRP configuration contained:

```cisco
network 44.4.4.4/32
```

However, R4's actual Loopback0 address was:

```text
4.4.4.4/32
```

The incorrect network statement prevented the loopback from being properly advertised through EIGRP.

### Fix

The incorrect network statement was removed:

```cisco
no network 44.4.4.4 0.0.0.0
```

The correct Loopback0 network was added:

```cisco
network 4.4.4.4 0.0.0.0
```

The Loopback0 interface remained passive.

### Verification

R4 maintained EIGRP adjacencies with R1 and R3:

```text
Neighbor 10.14.0.1 (GigabitEthernet0/0) is up
Neighbor 10.34.0.3 (FastEthernet1/0) is up
```

R4's loopback can now be advertised correctly through EIGRP.

---

## R5 — Incorrect EIGRP Network Wildcard

### Problem

R5 originally contained:

```cisco
router eigrp 100
 passive-interface Loopback0
 network 5.5.5.5 0.0.0.0
 network 10.0.0.0 0.0.0.0
```

The statement:

```cisco
network 10.0.0.0 0.0.0.0
```

matches only the exact address `10.0.0.0`.

It does **not** correctly match the `10.35.0.0/24` interface network.

### Fix

The incorrect statement was removed:

```cisco
no network 10.0.0.0 0.0.0.0
```

and replaced with:

```cisco
network 10.0.0.0
```

This enables EIGRP to identify the appropriate `10.x.x.x` interfaces.

### Verification

R5 successfully established an EIGRP adjacency with R3:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 10.35.0.3 (GigabitEthernet0/0) is up
```

R5 then learned the required summary:

```text
D       10.0.0.0/8 [90/3072] via 10.35.0.3
```

This confirms that R5 is receiving the `10.0.0.0/8` EIGRP summary from R3.

---

# ✅ Final Verification

## R5 Routing Table

After all corrections, R5's routing table contained:

```text
D       1.1.1.1/32 [90/156672] via 10.35.0.3
D       2.2.2.2/32 [90/156416] via 10.35.0.3
D       3.3.3.3/32 [90/130816] via 10.35.0.3
D       4.4.4.4/32 [90/156416] via 10.35.0.3
C       5.5.5.5/32 is directly connected
D       10.0.0.0/8 [90/3072] via 10.35.0.3
C       10.35.0.0/24 is directly connected
```

The most important entry is:

```text
D 10.0.0.0/8 [90/3072] via 10.35.0.3
```

This confirms that R5 is receiving the required EIGRP summary from R3.

---

# 🔎 Useful Verification Commands

The following commands are useful for troubleshooting and verifying EIGRP:

### Check EIGRP neighbors

```cisco
show ip eigrp neighbors
```

### Check EIGRP configuration

```cisco
show ip protocols
```

### Check the routing table

```cisco
show ip route
```

### Display EIGRP routes

```cisco
show ip route eigrp
```

### Check the running configuration

```cisco
show running-config
```

### Verify EIGRP topology

```cisco
show ip eigrp topology
```

---

# 🧠 Key Troubleshooting Lessons

This lab demonstrates several important EIGRP troubleshooting concepts:

1. **EIGRP neighbors must use the same AS number.**
   - R1 using AS 10 could not neighbor with routers using AS 100.

2. **EIGRP network statements must match the intended interfaces.**
   - An incorrect wildcard mask can prevent an interface from participating in EIGRP.

3. **Loopback addresses must be correctly advertised.**
   - R4 had `44.4.4.4` configured instead of its actual `4.4.4.4` address.

4. **EIGRP summarization is interface-specific.**
   - The `10.0.0.0/8` summary needed to be configured on R3's interface toward R5.

5. **Passive interfaces can advertise networks without forming EIGRP adjacencies.**
   - Loopback interfaces were configured as passive.

6. **`show ip eigrp neighbors` is one of the first commands to use when troubleshooting EIGRP.**

7. **The routing table confirms whether EIGRP routes are actually being installed.**

---

# 🎯 Lab Completion Criteria

The lab is successfully completed when:

- [x] R1 participates in EIGRP AS 100.
- [x] R2 participates in EIGRP AS 100.
- [x] R3 participates in EIGRP AS 100.
- [x] R4 participates in EIGRP AS 100.
- [x] R5 participates in EIGRP AS 100.
- [x] EIGRP neighbor adjacencies are established.
- [x] Loopback interfaces are passive.
- [x] Automatic summarization is disabled.
- [x] R4 advertises `4.4.4.4/32`.
- [x] R3 summarizes the `10.0.0.0/8` network toward R5.
- [x] R5 receives `10.0.0.0/8` through EIGRP.
- [x] EIGRP routes appear correctly in the routing tables.

## 🏁 Conclusion

The lab was completed by identifying and correcting EIGRP configuration errors across the network.

The major issues involved an **EIGRP AS mismatch on R1**, an **incorrect network advertisement on R4**, an **incorrect EIGRP network statement on R5**, and an **EIGRP summary configured on the wrong interface on R3**.

After correcting the configurations, the routers successfully formed EIGRP neighbor relationships and exchanged routing information. R5 ultimately received the required:

```text
10.0.0.0/8
```

summary route from R3 through EIGRP.
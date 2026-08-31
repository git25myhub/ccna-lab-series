# OSPF Troubleshooting – Multi-Area OSPF Route Summarization

## 📌 Lab Overview

This lab focuses on troubleshooting a **Multi-Area OSPF** network where routers are not receiving the routes they should.

The primary problem reported is:

> **R5 is not receiving the `10.0.0.0/8` OSPF summary route it should receive.**

The network contains multiple OSPF areas, with **R2 and R3 acting as Area Border Routers (ABRs)**.

The objective is to identify the misconfigured devices, correct the OSPF configuration, restore proper OSPF adjacencies, and ensure that R5 receives the expected summarized route.

---

# 🎯 Objectives

The objectives of this lab are:

1. Troubleshoot Multi-Area OSPF connectivity.
2. Identify which routers are misconfigured.
3. Verify OSPF neighbor relationships.
4. Verify OSPF network statements and area assignments.
5. Ensure the Area 0 backbone is correctly configured.
6. Correct the OSPF area assignment on R3.
7. Ensure R2 and R3 function correctly as ABRs.
8. Correct OSPF route summarization.
9. Ensure R5 receives the `10.0.0.0/8` summary route.
10. Verify the final routing tables and OSPF adjacencies.

---

# 🗺️ Network Topology

The network uses three OSPF areas:

```text
                         AREA 1
                  ┌─────────────────┐
                  │                 │
                 R1                R4
                  │                 │
                  └───────R2────────┘
                          │
                          │
                       AREA 0
                          │
                          R3
                          │
                          │
                       AREA 2
                          │
                          R5
```

### Area Assignment

| Router | Network | Area |
|---|---|---:|
| R1 | 10.12.0.0/24 | Area 1 |
| R1 | 10.14.0.0/24 | Area 1 |
| R2 | 10.12.0.0/24 | Area 1 |
| R2 | 10.23.0.0/24 | Area 0 |
| R2 | Loopback0 | Area 0 |
| R3 | 10.23.0.0/24 | Area 0 |
| R3 | 10.35.0.0/24 | Area 2 |
| R3 | Loopback0 | Area 0 |
| R4 | 10.14.0.0/24 | Area 1 |
| R5 | 10.35.0.0/24 | Area 2 |

---

# 🔎 Initial Problem

The key symptom was observed on R5.

Initially, R5 had individual inter-area routes:

```text
O IA  10.12.0.0
O IA  10.14.0.0
O IA  10.23.0.0
```

However, R5 was expected to receive the summarized:

```text
10.0.0.0/8
```

as an OSPF inter-area route.

The required final route was:

```text
O IA 10.0.0.0/8
```

---

# 1. Initial Troubleshooting – R1

The first step was to inspect R1's OSPF neighbors:

```cisco
R1# show ip ospf neighbor
```

R1 initially showed:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   2WAY/DROTHER    00:00:34    10.12.0.2       FastEthernet1/0
```

The adjacency later reached:

```text
FULL
```

R1's OSPF configuration showed:

```text
Routing for Networks:
  10.14.0.0 0.0.0.255 area 1
  10.12.0.0 0.0.0.255 area 1
```

The missing Loopback0 advertisement was identified and corrected.

### R1 Correction

```cisco
R1(config)# router ospf 1
R1(config-router)# network 1.1.1.1 0.0.0.0 area 1
R1(config-router)# passive-interface loopback0
```

The configuration was then saved:

```cisco
R1(config-router)# do write memory
```

---

# 2. Troubleshooting R2

R2 was inspected using:

```cisco
R2# show ip ospf neighbor
R2# show ip protocols
R2# show ip interface brief
R2# show running-config
```

Initially, R2's `FastEthernet2/0` interface was administratively down:

```text
FastEthernet2/0   10.23.0.2   YES manual   administratively down   down
```

This prevented R2 from forming an OSPF adjacency with R3.

### R2 Correction

The interface was enabled:

```cisco
R2(config)# interface FastEthernet2/0
R2(config-if)# no shutdown
```

The interface then transitioned to:

```text
%LINK-5-CHANGED: Interface FastEthernet2/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet2/0,
changed state to up
```

Shortly afterward, R2 formed an OSPF adjacency with R3:

```text
%OSPF-5-ADJCHG: Process 1, Nbr 3.3.3.3 on FastEthernet2/0
from LOADING to FULL
```

---

# 3. Verify R2 OSPF Configuration

R2 correctly has two OSPF areas:

```text
Number of areas in this router is 2.
2 normal 0 stub 0 nssa
```

The network statements are:

```text
10.23.0.0 0.0.0.255 area 0
2.2.2.2 0.0.0.0 area 0
10.12.0.0 0.0.0.255 area 1
```

This makes R2 an **ABR between Area 1 and Area 0**.

R2 also had the summary configuration:

```cisco
area 0 range 10.0.0.0 255.0.0.0
```

The OSPF reference bandwidth was correctly configured as:

```cisco
auto-cost reference-bandwidth 100000
```

The Loopback0 interface was also correctly configured as passive:

```cisco
passive-interface Loopback0
```

---

# 4. Troubleshooting R3

R3 was the most significant source of the problem.

The OSPF neighbor table initially showed only:

```text
Neighbor ID     Pri   State       Dead Time   Address       Interface
5.5.5.5           1   FULL/DR     00:00:34    10.35.0.5     FastEthernet3/0
```

R3 was **not forming an adjacency with R2**.

More importantly, R3 was generating the following error:

```text
%OSPF-4-ERRRCV: Received invalid packet: mismatch area ID,
from backbone area must be virtual-link but not found
from 10.23.0.3, FastEthernet1/0
```

This indicated an **OSPF area mismatch** on the R2-R3 link.

---

# 5. Identify the R3 Misconfiguration

R3's OSPF configuration showed:

```text
Routing for Networks:
  10.35.0.0 0.0.0.255 area 2
  10.23.0.0 0.0.0.255 area 2
  3.3.3.3 0.0.0.0 area 0
```

The problem was:

```text
10.23.0.0/24 → Area 2
```

The R2-R3 link is supposed to be part of the OSPF backbone:

```text
10.23.0.0/24 → Area 0
```

R2 already had:

```text
10.23.0.0 0.0.0.255 area 0
```

Therefore, R3 had an area mismatch.

---

# 6. Correct R3's Area Assignment

The incorrect Area 2 network statement was corrected by assigning the R2-R3 link to Area 0:

```cisco
R3(config)# router ospf 1
R3(config-router)# network 10.23.0.0 0.0.0.255 area 0
```

IOS confirmed the change:

```text
%OSPF-6-AREACHG: 10.23.0.0/0 changed from area 2 to area 0
```

R3 then successfully formed an adjacency with R2:

```text
Neighbor ID     Pri   State       Dead Time   Address       Interface

5.5.5.5           1   FULL/DR     00:00:34    10.35.0.5     FastEthernet3/0
2.2.2.2           1   FULL/BDR    00:00:33    10.23.0.2     FastEthernet1/0
```

---

# 7. Correct R3 Route Summarization

R3 initially had the summary configured under Area 2:

```cisco
area 2 range 10.0.0.0 255.0.0.0
```

This was incorrect for the intended topology.

The incorrect summary was removed:

```cisco
R3(config-router)# no area 2 range 10.0.0.0 255.0.0.0
```

The summary was then configured under Area 0:

```cisco
R3(config-router)# area 0 range 10.0.0.0 255.0.0.0
```

The configuration was saved:

```cisco
R3(config-router)# do write memory
```

---

# 8. Troubleshooting R4

R4's OSPF configuration showed:

```text
Passive Interface(s):
  GigabitEthernet0/0
  Loopback0
```

The problem was that the physical interface connecting R4 to R1 was configured as passive.

Therefore, R4 could not form an OSPF neighbor relationship with R1.

### R4 Correction

The passive setting was removed from the active OSPF interface:

```cisco
R4(config)# router ospf 1
R4(config-router)# no passive-interface GigabitEthernet0/0
```

R4 subsequently formed a FULL adjacency with R1:

```text
Neighbor ID     Pri   State       Dead Time   Address       Interface

1.1.1.1           1   FULL/BDR    00:00:34    10.14.0.1     GigabitEthernet0/0
```

Loopback0 remained passive, which is the desired configuration.

---

# 9. R5 Verification

R5 already had a healthy OSPF adjacency with R3:

```text
Neighbor ID     Pri   State       Dead Time   Address       Interface

3.3.3.3           1   FULL/BDR    00:00:37    10.35.0.3     FastEthernet0/0
```

Before the fix, R5 was receiving the individual networks:

```text
O IA 10.12.0.0
O IA 10.14.0.0
O IA 10.23.0.0
```

but not the required summary.

After correcting the R3 area assignment and summary configuration, R5 received:

```text
10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks

O IA    10.0.0.0/8 [110/2000] via 10.35.0.3,
                   FastEthernet0/0
```

This confirms that the required summary route is now being advertised to R5.

---

# 10. Final R5 Routing Table

The important portion of the final routing table is:

```text
R5# show ip route

     1.0.0.0/32 is subnetted, 1 subnets
O IA    1.1.1.1 [110/3012] via 10.35.0.3

     2.0.0.0/32 is subnetted, 1 subnets
O IA    2.2.2.2 [110/2012] via 10.35.0.3

     3.0.0.0/32 is subnetted, 1 subnets
O IA    3.3.3.3 [110/1012] via 10.35.0.3

     4.0.0.0/32 is subnetted, 1 subnets
O IA    4.4.4.4 [110/3112] via 10.35.0.3

     5.0.0.0/32 is subnetted, 1 subnets
C       5.5.5.5 is directly connected, Loopback0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
O IA    10.0.0.0/8 [110/2000] via 10.35.0.3

C       10.35.0.0/24 is directly connected, FastEthernet0/0
```

The key result is:

```text
O IA 10.0.0.0/8 via 10.35.0.3
```

---

# 11. Final OSPF Neighbor Relationships

After troubleshooting, the expected OSPF adjacencies are:

| Router | Neighbor | Interface | Area | State |
|---|---|---|---:|---|
| R1 | R2 | Fa1/0 | 1 | FULL |
| R1 | R4 | G0/0 | 1 | FULL |
| R2 | R1 | Fa1/0 | 1 | FULL |
| R2 | R3 | Fa2/0 | 0 | FULL |
| R3 | R2 | Fa1/0 | 0 | FULL |
| R3 | R5 | Fa3/0 | 2 | FULL |
| R4 | R1 | G0/0 | 1 | FULL |
| R5 | R3 | Fa0/0 | 2 | FULL |

---

# 12. Key Faults Identified

| Device | Problem | Correction |
|---|---|---|
| R1 | Loopback0 was not advertised | Added `network 1.1.1.1 0.0.0.0 area 1` |
| R2 | Fa2/0 was shutdown | Configured `no shutdown` |
| R3 | R2-R3 link incorrectly placed in Area 2 | Changed `10.23.0.0/24` to Area 0 |
| R3 | Summary configured under wrong area | Removed Area 2 summary and configured Area 0 summary |
| R4 | G0/0 incorrectly configured as passive | Used `no passive-interface g0/0` |
| R5 | No direct configuration fault identified | Verified after upstream corrections |

**Important:** Not every router was misconfigured. The lab required troubleshooting rather than blindly changing every device.

---

# 13. Useful Troubleshooting Commands

### Check OSPF neighbors

```cisco
show ip ospf neighbor
```

### Check OSPF configuration

```cisco
show ip protocols
```

### Check OSPF routes

```cisco
show ip route ospf
```

### Check complete routing table

```cisco
show ip route
```

### Check OSPF interfaces

```cisco
show ip ospf interface
```

### Check running configuration

```cisco
show running-config
```

### Check interface status

```cisco
show ip interface brief
```

### Check OSPF process

```cisco
show ip ospf
```

---

# 14. Troubleshooting Methodology

The lab followed a structured troubleshooting process:

```text
1. Identify the missing route
           ↓
2. Check the affected router's routing table
           ↓
3. Check OSPF neighbors
           ↓
4. Check OSPF network statements
           ↓
5. Verify OSPF area assignments
           ↓
6. Check interface status
           ↓
7. Correct only the misconfigured devices
           ↓
8. Verify OSPF adjacencies
           ↓
9. Verify route summarization
           ↓
10. Confirm 10.0.0.0/8 on R5
```

---

# 15. Final Verification Checklist

- [x] R1 advertises its required interfaces into OSPF.
- [x] R2 has an active connection to R3.
- [x] R2-R3 operates through Area 0.
- [x] R3 connects Area 0 and Area 2.
- [x] R3's `10.23.0.0/24` interface is in Area 0.
- [x] R3's `10.35.0.0/24` interface remains in Area 2.
- [x] R3's Loopback0 remains in Area 0.
- [x] R4's active interface is not passive.
- [x] Loopback interfaces remain passive.
- [x] OSPF neighbors reach `FULL`.
- [x] R2 and R3 operate as ABRs.
- [x] R3 has the `10.0.0.0/8` summary configured correctly.
- [x] R5 receives `10.0.0.0/8`.
- [x] R5 sees the summary as an `O IA` route.
- [x] All configurations are saved.

---

# 🧠 Key Concepts Practiced

This lab provides practical experience with:

- Multi-Area OSPF
- OSPF Area 0
- Area Border Routers (ABRs)
- OSPF neighbor adjacency
- OSPF area mismatches
- Passive interfaces
- OSPF network statements
- OSPF route summarization
- Inter-area routes (`O IA`)
- Interface troubleshooting
- Routing-table analysis
- Systematic network troubleshooting

---

# 🏁 Conclusion

The primary issue preventing R5 from receiving the `10.0.0.0/8` summary was caused by incorrect OSPF configuration in the network.

The most important fault was on **R3**, where the `10.23.0.0/24` link to R2 was incorrectly assigned to **Area 2 instead of Area 0**. This prevented R2 and R3 from forming the required backbone adjacency.

Additional issues were identified on R1, R2, and R4, including a missing loopback advertisement, a shutdown interface, and an incorrectly configured passive interface.

After correcting the problems and configuring the route summary under the correct OSPF area, R5 successfully received:

```text
O IA 10.0.0.0/8 via 10.35.0.3
```

The lab was therefore successfully completed, with the Multi-Area OSPF topology operating correctly and route summarization functioning as required.
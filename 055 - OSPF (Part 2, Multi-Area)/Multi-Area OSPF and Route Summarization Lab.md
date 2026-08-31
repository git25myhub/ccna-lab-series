# Multi-Area OSPF and Route Summarization Lab

## 📌 Lab Overview

This lab focuses on configuring **Multi-Area OSPF (Open Shortest Path First)** across a five-router topology and implementing **OSPF route summarization on Area Border Routers (ABRs)**.

The lab demonstrates how OSPF can divide a large network into multiple areas to improve scalability and reduce routing information exchanged between areas.

---

## 🎯 Objectives

The objectives of this lab are:

1. Configure **Multi-Area OSPF** according to the network topology.
2. Advertise all configured router interfaces into OSPF.
3. Place the **R2 and R3 loopback interfaces in Area 0**.
4. Configure all loopback interfaces as **passive OSPF interfaces**.
5. Configure a consistent OSPF reference bandwidth of **100,000 Mbps**.
6. Configure the reference bandwidth so that a **100-Gigabit interface has an OSPF cost of 1**.
7. Configure **OSPF route summarization on the ABRs**.
8. Verify OSPF neighbor relationships and routing tables.
9. Verify that inter-area routes are being learned correctly.

---

## 🗺️ OSPF Area Design

The topology uses three OSPF areas:

| Router | Interface/Network | OSPF Area |
|---|---|---:|
| R1 | 10.12.0.0/24 | Area 1 |
| R1 | 10.14.0.0/24 | Area 1 |
| R1 | Loopback0 | Area 1 |
| R2 | 10.12.0.0/24 | Area 1 |
| R2 | 10.23.0.0/24 | Area 0 |
| R2 | Loopback0 | Area 0 |
| R3 | 10.23.0.0/24 | Area 0 |
| R3 | 10.35.0.0/24 | Area 2 |
| R3 | Loopback0 | Area 0 |
| R4 | 10.14.0.0/24 | Area 1 |
| R4 | Loopback0 | Area 1 |
| R5 | 10.35.0.0/24 | Area 2 |
| R5 | Loopback0 | Area 2 |

### Area Layout

```text
                    AREA 1
              ┌─────────────────┐
              │                 │
             R1────────────────R4
              │
              │ 10.12.0.0/24
              │
              R2
              │
              │ 10.23.0.0/24
              │
           ┌──R3──┐
           │      │
      AREA 0      │ AREA 2
                  │
                  R5
```

### Important Area Requirement

The loopbacks of **R2 and R3 must belong to Area 0**.

Therefore:

- R2 Loopback0 → Area 0
- R3 Loopback0 → Area 0

This is important because **Area 0 is the OSPF backbone area**, and the topology uses R2 and R3 as part of that backbone.

---

# 1. Multi-Area OSPF Configuration

## R1

R1 participates in **Area 1**.

All interfaces are advertised using a wildcard network statement, and Loopback0 is configured as passive.

```cisco
R1(config)# router ospf 1
R1(config-router)# network 0.0.0.0 255.255.255.255 area 1
R1(config-router)# passive-interface loopback0
```

### OSPF Reference Bandwidth

```cisco
R1(config-router)# auto-cost reference-bandwidth 100000
```

The value is specified in Mbps.

Therefore:

```text
100,000 Mbps = 100 Gbps
```

With this reference bandwidth, a 100-Gigabit interface has:

```text
OSPF Cost = 100,000 / 100,000 = 1
```

---

# 2. R2 OSPF Configuration

R2 is an **ABR** because it connects Area 1 and Area 0.

```cisco
R2(config)# router ospf 1

R2(config-router)# network 10.12.0.0 0.0.0.255 area 1
R2(config-router)# network 10.23.0.0 0.0.0.255 area 0
R2(config-router)# network 2.2.2.2 0.0.0.0 area 0

R2(config-router)# passive-interface loopback0

R2(config-router)# auto-cost reference-bandwidth 100000
```

### Route Summarization on R2

R2 summarizes routes from Area 0 toward other areas:

```cisco
R2(config-router)# area 0 range 10.0.0.0 255.0.0.0
```

This summarizes the Area 0 routes into:

```text
10.0.0.0/8
```

---

# 3. R3 OSPF Configuration

R3 is another **ABR**, connecting Area 0 and Area 2.

```cisco
R3(config)# router ospf 1

R3(config-router)# network 10.23.0.0 0.0.0.255 area 0
R3(config-router)# network 10.35.0.0 0.0.0.255 area 2
R3(config-router)# network 3.3.3.3 0.0.0.0 area 0

R3(config-router)# passive-interface loopback0

R3(config-router)# auto-cost reference-bandwidth 100000
```

### Route Summarization on R3

R3 summarizes the Area 0 routes:

```cisco
R3(config-router)# area 0 range 10.0.0.0 255.0.0.0
```

This causes the multiple `10.x.x.x` networks learned through the backbone to be represented by the summary:

```text
10.0.0.0/8
```

---

# 4. R4 OSPF Configuration

R4 belongs entirely to **Area 1**.

```cisco
R4(config)# router ospf 1
R4(config-router)# network 0.0.0.0 255.255.255.255 area 1
R4(config-router)# passive-interface loopback0
R4(config-router)# auto-cost reference-bandwidth 100000
```

---

# 5. R5 OSPF Configuration

R5 belongs entirely to **Area 2**.

```cisco
R5(config)# router ospf 1
R5(config-router)# network 0.0.0.0 255.255.255.255 area 2
R5(config-router)# passive-interface loopback0
R5(config-router)# auto-cost reference-bandwidth 100000
```

---

# 6. OSPF Passive Interfaces

Loopback interfaces were configured as passive interfaces on every router:

```cisco
passive-interface loopback0
```

A passive OSPF interface is still advertised into OSPF, but the router does not send OSPF Hello packets through that interface.

This is appropriate for loopbacks because they do not need to form OSPF neighbor adjacencies.

The loopback addresses remain reachable through OSPF.

---

# 7. OSPF Reference Bandwidth

The reference bandwidth was configured as:

```cisco
auto-cost reference-bandwidth 100000
```

on **all routers**.

This is important because OSPF calculates interface cost using:

```text
OSPF Cost = Reference Bandwidth / Interface Bandwidth
```

With a reference bandwidth of:

```text
100,000 Mbps
```

a 100-Gigabit interface has:

```text
100,000 / 100,000 = 1
```

Therefore:

| Interface | Bandwidth | OSPF Cost |
|---|---:|---:|
| 100 Gbps | 100,000 Mbps | 1 |
| 10 Gbps | 10,000 Mbps | 10 |
| 1 Gbps | 1,000 Mbps | 100 |
| 100 Mbps | 100 Mbps | 1000 |
| 10 Mbps | 10 Mbps | 10000 |

### Important

The reference bandwidth must be consistent on **all OSPF routers**.

Cisco IOS displays the following warning when the value is changed:

```text
% OSPF: Reference bandwidth is changed.

Please ensure reference bandwidth is consistent across all routers.
```

This warning is expected.

---

# 8. OSPF Route Summarization

Route summarization reduces the number of routes advertised between OSPF areas.

In this lab, the ABRs use:

```cisco
area 0 range 10.0.0.0 255.0.0.0
```

The command summarizes multiple networks into:

```text
10.0.0.0/8
```

### ABRs in This Lab

| ABR | Connected Areas | Summary |
|---|---|---|
| R2 | Area 1 ↔ Area 0 | 10.0.0.0/8 |
| R3 | Area 0 ↔ Area 2 | 10.0.0.0/8 |

The purpose is to reduce the amount of detailed routing information advertised between areas.

---

# 9. OSPF Neighbor Verification

After configuration, verify that OSPF adjacencies have formed.

Use:

```cisco
show ip ospf neighbor
```

Expected neighbor relationships include:

### R1

```text
R1# show ip ospf neighbor
```

R1 should have neighbors:

```text
2.2.2.2
4.4.4.4
```

### R2

R2 should have neighbors:

```text
1.1.1.1
3.3.3.3
```

### R3

R3 should have neighbors:

```text
2.2.2.2
5.5.5.5
```

### R5

R5 should have neighbor:

```text
3.3.3.3
```

A healthy OSPF adjacency should reach:

```text
FULL
```

---

# 10. Routing Table Verification

Use:

```cisco
show ip route
```

OSPF routes are identified with:

```text
O
```

while inter-area OSPF routes are identified with:

```text
O IA
```

For example, R1's routing table showed:

```text
O IA    2.2.2.2 [110/1012] via 10.12.0.2
O IA    3.3.3.3 [110/2012] via 10.12.0.2
O IA    5.5.5.5 [110/3012] via 10.12.0.2
```

This confirms that R1 is learning routes from other OSPF areas.

---

# 11. Verification Commands

Use the following commands to verify the configuration.

### Check OSPF neighbors

```cisco
show ip ospf neighbor
```

### Check OSPF configuration

```cisco
show ip protocols
```

### Display OSPF routes

```cisco
show ip route ospf
```

### Display the complete routing table

```cisco
show ip route
```

### Display OSPF process information

```cisco
show ip ospf
```

### Display OSPF interfaces

```cisco
show ip ospf interface
```

### Check the running configuration

```cisco
show running-config
```

---

# 12. Connectivity Testing

Test connectivity between loopback interfaces using:

```cisco
ping 1.1.1.1
ping 2.2.2.2
ping 3.3.3.3
ping 4.4.4.4
ping 5.5.5.5
```

The routers should be able to reach the loopback addresses across the different OSPF areas.

For example, from R5:

```cisco
R5# ping 1.1.1.1
R5# ping 2.2.2.2
R5# ping 3.3.3.3
R5# ping 4.4.4.4
```

Successful replies confirm end-to-end OSPF connectivity.

---

# 13. Troubleshooting

If OSPF neighbors do not reach `FULL`, check:

### Verify interfaces are up

```cisco
show ip interface brief
```

### Verify OSPF configuration

```cisco
show running-config | section router ospf
```

### Verify neighbors

```cisco
show ip ospf neighbor
```

### Verify OSPF interfaces

```cisco
show ip ospf interface
```

### Check the routing table

```cisco
show ip route
```

### Common Problems

- Incorrect OSPF area assignment
- Incorrect wildcard masks
- Missing network statements
- Loopback assigned to the wrong area
- OSPF reference bandwidth inconsistent between routers
- Interface shutdown
- Incorrect IP addressing
- Missing passive-interface configuration
- Incorrect route summarization command
- OSPF process not enabled on an interface

---

# 14. Configuration Summary

| Router | Area(s) | Role |
|---|---|---|
| R1 | Area 1 | Internal Router |
| R2 | Area 1, Area 0 | ABR |
| R3 | Area 0, Area 2 | ABR |
| R4 | Area 1 | Internal Router |
| R5 | Area 2 | Internal Router |

### Loopback Areas

```text
R1 Loopback0 → Area 1
R2 Loopback0 → Area 0
R3 Loopback0 → Area 0
R4 Loopback0 → Area 1
R5 Loopback0 → Area 2
```

### Reference Bandwidth

```text
100,000 Mbps
```

### 100-Gigabit Interface Cost

```text
1
```

### Route Summary

```text
10.0.0.0/8
```

### ABRs

```text
R2 → Area 1 / Area 0
R3 → Area 0 / Area 2
```

---

# ✅ Lab Completion Criteria

The lab is successfully completed when:

- [x] Multi-area OSPF is configured according to the topology.
- [x] All configured interfaces are advertised into OSPF.
- [x] R2 Loopback0 is in Area 0.
- [x] R3 Loopback0 is in Area 0.
- [x] All loopback interfaces are passive.
- [x] OSPF reference bandwidth is set to `100000 Mbps` on every router.
- [x] A 100-Gigabit interface would have an OSPF cost of `1`.
- [x] R2 and R3 operate as OSPF ABRs.
- [x] Route summarization is configured on the ABRs.
- [x] OSPF neighbor relationships reach `FULL`.
- [x] Inter-area routes appear as `O IA`.
- [x] All router loopbacks are reachable through OSPF.
- [x] The configuration is saved using:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

---

## 🧠 Key Concepts Practiced

This lab provides practical experience with:

- Multi-Area OSPF
- OSPF Area 0 backbone
- Area Border Routers (ABRs)
- OSPF network statements
- OSPF wildcard masks
- Passive interfaces
- OSPF reference bandwidth
- OSPF interface cost calculation
- OSPF inter-area routes
- OSPF route summarization
- OSPF neighbor adjacency
- Routing-table verification
- Network troubleshooting

---

## 🏁 Conclusion

This lab demonstrates how **Multi-Area OSPF** can be used to divide an OSPF domain into smaller logical areas while maintaining connectivity through the **Area 0 backbone**.

R2 and R3 function as ABRs between the backbone and the surrounding areas. Loopback interfaces are advertised but configured as passive, while the OSPF reference bandwidth is increased to `100,000 Mbps` to support modern high-speed interfaces and ensure a **100-Gigabit interface has an OSPF cost of 1**.

Finally, route summarization is implemented on the ABRs to reduce routing-table information exchanged between areas and improve the scalability of the OSPF network.
# EIGRP Unequal-Cost Load Balancing and Route Summarization Lab

## 📌 Lab Overview

This Cisco Packet Tracer lab focuses on configuring **EIGRP (Enhanced Interior Gateway Routing Protocol)** across a five-router topology.

The lab demonstrates:

- Configuring loopback interfaces as stable router identifiers
- Configuring EIGRP AS 100
- Advertising all required interfaces into EIGRP
- Configuring loopback interfaces as passive interfaces
- Disabling EIGRP auto-summary
- Configuring unequal-cost load balancing using EIGRP variance
- Configuring EIGRP route summarization
- Verifying EIGRP neighbors, routes, and topology information

---

## 🎯 Lab Objectives

The lab requirements were:

1. Configure a loopback address on each router:
   - R1 → `1.1.1.1/32`
   - R2 → `2.2.2.2/32`
   - R3 → `3.3.3.3/32`
   - R4 → `4.4.4.4/32`
   - R5 → `5.5.5.5/32`

2. Configure EIGRP on every router and advertise all configured interfaces.

3. Configure all loopback interfaces as **passive interfaces**.

4. Disable EIGRP **auto-summary**.

5. Configure R1 to perform **unequal-cost load balancing** when sending traffic toward R5.

6. Configure R3 to advertise a `10.0.0.0/8` summary network toward R5.

---

# 🗺️ Topology Overview

The topology contains five routers:

```text
                 R4
                /  \
               /    \
             R1      R3 -------- R5
              \     /            |
               \   /             |
                 R2
```

The exact physical topology should be referenced from the Packet Tracer file. The supplied configurations show the following important links:

| Link | Network |
|------|---------|
| R1 - R2 | `10.12.0.0/24` |
| R1 - R4 | `10.14.0.0/24` |
| R2 - R3 | `10.23.0.0/24` |
| R3 - R4 | `10.34.0.0/24` |
| R3 - R5 | `10.35.0.0/24` |

---

# 1. Configure Loopback Interfaces

Loopback interfaces provide stable `/32` addresses for each router.

## R1

```cisco
R1(config)# interface loopback0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
R1(config-if)# exit
```

The configuration successfully created R1's loopback:

```text
1.1.1.1/32
```

---

## R2

```cisco
R2(config)# interface loopback0
R2(config-if)# ip address 2.2.2.2 255.255.255.255
R2(config-if)# exit
```

R2 uses:

```text
2.2.2.2/32
```

---

## R3

```cisco
R3(config)# interface loopback0
R3(config-if)# ip address 3.3.3.3 255.255.255.255
R3(config-if)# exit
```

R3 uses:

```text
3.3.3.3/32
```

---

## R4

```cisco
R4(config)# interface loopback0
R4(config-if)# ip address 4.4.4.4 255.255.255.255
R4(config-if)# exit
```

R4 uses:

```text
4.4.4.4/32
```

---

## R5

```cisco
R5(config)# interface loopback0
R5(config-if)# ip address 5.5.5.5 255.255.255.255
R5(config-if)# exit
```

R5 uses:

```text
5.5.5.5/32
```

The final R5 routing table confirms that `5.5.5.5/32` is directly connected through Loopback0.

---

# 2. Configure EIGRP

The EIGRP autonomous system used throughout the topology is:

```text
AS 100
```

The general configuration is:

```cisco
router eigrp 100
 network 10.0.0.0
 network <loopback-address> 0.0.0.0
 passive-interface loopback0
 no auto-summary
```

Using the broad `network 10.0.0.0` statement allows the configured `10.x.x.x` interfaces to participate in EIGRP.

---

# 3. R1 EIGRP Configuration

R1 was configured with:

```cisco
R1(config)# router eigrp 100
R1(config-router)# network 10.0.0.0
R1(config-router)# network 1.1.1.1 0.0.0.0
R1(config-router)# passive-interface loopback0
R1(config-router)# no auto-summary
```

After configuration, R1 established EIGRP adjacencies with:

```text
10.12.0.2
10.14.0.4
```

The EIGRP neighbor messages confirm both neighbors reached the `up` state.

---

# 4. R2 EIGRP Configuration

R2 was configured with:

```cisco
R2(config)# router eigrp 100
R2(config-router)# network 10.0.0.0
R2(config-router)# network 2.2.2.2 0.0.0.0
R2(config-router)# passive-interface loopback0
R2(config-router)# no auto-summary
```

R2 established EIGRP adjacencies with:

```text
R1 → 10.12.0.1
R3 → 10.23.0.3
```

The neighbor-change messages confirm successful EIGRP adjacency formation.

---

# 5. R3 EIGRP Configuration

R3 was configured with:

```cisco
R3(config)# router eigrp 100
R3(config-router)# network 10.0.0.0
R3(config-router)# network 3.3.3.3 0.0.0.0
R3(config-router)# passive-interface loopback0
R3(config-router)# no auto-summary
```

R3 established EIGRP relationships with:

```text
10.23.0.2
10.34.0.4
10.35.0.5
```

This makes R3 the router responsible for advertising the summarized `10.0.0.0/8` network toward R5.

---

# 6. R4 EIGRP Configuration

R4 was configured with:

```cisco
R4(config)# router eigrp 100
R4(config-router)# network 10.0.0.0
R4(config-router)# network 4.4.4.4 0.0.0.0
R4(config-router)# passive-interface loopback0
R4(config-router)# no auto-summary
```

R4 established EIGRP adjacencies with:

```text
R1 → 10.14.0.1
R3 → 10.34.0.3
```

---

# 7. R5 EIGRP Configuration

R5 was configured with:

```cisco
R5(config)# router eigrp 100
R5(config-router)# network 10.0.0.0
R5(config-router)# network 5.5.5.5 0.0.0.0
R5(config-router)# passive-interface loopback0
R5(config-router)# no auto-summary
```

R5 established an EIGRP adjacency with R3:

```text
10.35.0.3
```

The final R5 routing table confirms that routes to the other routers were learned through R3.

---

# 8. Configure Loopbacks as Passive Interfaces

The loopback interfaces were configured as passive:

```cisco
passive-interface loopback0
```

For example, on R1:

```cisco
R1(config-router)# passive-interface loopback0
```

This allows the loopback network to be advertised by EIGRP without attempting to form an EIGRP neighbor relationship through the loopback interface.

The same configuration was applied to the loopback interfaces on R2, R3, R4, and R5.

---

# 9. Disable EIGRP Auto-Summary

EIGRP auto-summary was disabled on every router:

```cisco
no auto-summary
```

This is important because the topology uses multiple `10.x.x.x` subnetworks.

Without auto-summary, EIGRP advertises the actual subnet information rather than automatically summarizing routes at their classful network boundaries.

The configuration was applied on the routers as shown in the supplied command logs.

---

# 10. Configure Unequal-Cost Load Balancing on R1

One of the main objectives of this lab is to make R1 use **unequal-cost load balancing** toward R5.

Before applying variance, R1's EIGRP topology showed two possible paths to `5.5.5.5/32`:

```text
via 10.14.0.4
via 10.12.0.2
```

However, only the path through R4 was installed as the successor.

The topology table showed:

```text
P 5.5.5.5/32, 1 successors
FD is 156672

via 10.14.0.4 (156672/156416)
via 10.12.0.2 (158976/156416)
```

The second path has a higher metric and therefore was not initially installed as an active forwarding path.

---

## Apply EIGRP Variance

On R1:

```cisco
R1(config)# router eigrp 100
R1(config-router)# variance 2
```

The command:

```text
variance 2
```

allows EIGRP to install feasible paths whose metrics are within two times the minimum feasible distance.

---

# 11. Verify Unequal-Cost Load Balancing

After applying:

```cisco
variance 2
```

R1's routing table changed.

The route to R3 became:

```text
D 3.3.3.3/32
    [90/156416] via 10.14.0.4
    [90/158720] via 10.12.0.2
```

More importantly, the route to R5 became:

```text
D 5.5.5.5/32
    [90/156672] via 10.14.0.4
    [90/158976] via 10.12.0.2
```

This confirms that R1 is now using **two paths with different EIGRP metrics** toward R5.

The path through R4 remains the preferred path because it has the lower metric, while the path through R2 is also installed because it falls within the configured variance.

---

# 12. Configure EIGRP Route Summarization on R3

The second major routing requirement is to advertise a summary route:

```text
10.0.0.0/8
```

from R3 toward R5.

The summary is configured on the interface connecting R3 to R5.

On R3:

```cisco
R3(config)# interface gigabitEthernet0/0
R3(config-if)# ip summary-address eigrp 100 10.0.0.0 255.0.0.0
```

This summarizes the multiple `10.x.x.x` networks behind R3 into:

```text
10.0.0.0/8
```

The summary is advertised toward the EIGRP neighbor on that interface.

---

# 13. Verify the Summary Route on R5

After R3 advertises the summary, R5's routing table contains:

```text
10.0.0.0/8
```

The supplied R5 output shows:

```text
10.0.0.0/8 is variably subnetted, 3 subnets, 3 masks

D       10.0.0.0/8 [90/3072]
        via 10.35.0.3, GigabitEthernet0/0
```

This confirms that R5 learned the `10.0.0.0/8` summary through R3.

---

# 14. Important EIGRP Verification Commands

## Check EIGRP Neighbors

```cisco
show ip eigrp neighbors
```

Expected result:

```text
Neighbor relationships should be in an active/up state.
```

---

## Check EIGRP Routes

```cisco
show ip route eigrp
```

EIGRP routes are identified with:

```text
D
```

For example:

```text
D 5.5.5.5/32
```

---

## Check the EIGRP Topology

```cisco
show ip eigrp topology
```

This is particularly important when troubleshooting unequal-cost load balancing.

On R1, the topology table showed both paths to R5:

```text
via 10.14.0.4
via 10.12.0.2
```

After configuring variance, both paths were installed in the routing table.

---

## Check the Complete Routing Table

```cisco
show ip route
```

On R5, the final routing table confirmed the summarized route:

```text
D 10.0.0.0/8
    via 10.35.0.3
```



---

## Check EIGRP Protocol Configuration

```cisco
show ip protocols
```

This can be used to verify:

- EIGRP AS number
- Advertised networks
- Passive interfaces
- Auto-summary status
- EIGRP parameters

---

# 🔍 Troubleshooting Notes

## Issue 1 — Incorrect command entered on R2

The following command was initially entered:

```text
R2#conf t\
```

which produced:

```text
% Invalid input detected at '^' marker.
```

The command was subsequently entered correctly:

```cisco
R2# conf t
```

and configuration continued successfully.

---

## Issue 2 — Incorrect loopback command on R3

An incomplete command was initially entered:

```text
R3(config-if)#3.3.3.3 255.255.255.255
```

Cisco returned:

```text
% Invalid input detected at '^' marker.
```

The correct command was then entered:

```cisco
R3(config-if)# ip address 3.3.3.3 255.255.255.255
```

The loopback was successfully configured afterward.

---

# 🧠 Key Concepts Learned

## EIGRP Feasible Distance

EIGRP uses the **feasible distance (FD)** to determine the best path.

The routing information includes:

```text
FD / Reported Distance
```

For example:

```text
via 10.14.0.4 (156672/156416)
```

The first value represents the feasible distance, while the second represents the reported distance from the neighbor.

---

## EIGRP Variance

The command:

```cisco
variance 2
```

allows EIGRP to install additional feasible paths whose metrics are within the configured multiplier of the best path.

This enables **unequal-cost load balancing**.

---

## Passive Interfaces

The command:

```cisco
passive-interface loopback0
```

prevents EIGRP from sending hello packets and forming neighbor relationships through the loopback interface while still allowing the loopback network to be advertised.

---

## EIGRP Auto-Summary

The command:

```cisco
no auto-summary
```

prevents automatic classful summarization.

This allows EIGRP to maintain the actual subnet information in the topology.

---

## EIGRP Manual Summarization

The command:

```cisco
ip summary-address eigrp 100 10.0.0.0 255.0.0.0
```

creates a manual summary route.

In this lab, R3 uses it to advertise:

```text
10.0.0.0/8
```

toward R5.

---

# ✅ Completion Checklist

- [x] R1 loopback `1.1.1.1/32` configured
- [x] R2 loopback `2.2.2.2/32` configured
- [x] R3 loopback `3.3.3.3/32` configured
- [x] R4 loopback `4.4.4.4/32` configured
- [x] R5 loopback `5.5.5.5/32` configured
- [x] EIGRP AS 100 configured
- [x] Required interfaces advertised into EIGRP
- [x] Loopback interfaces configured as passive
- [x] EIGRP auto-summary disabled
- [x] EIGRP neighbors successfully established
- [x] R1 configured with `variance 2`
- [x] R1 uses unequal-cost paths toward R5
- [x] R3 configured with `10.0.0.0/8` summary
- [x] R5 learns `10.0.0.0/8` through R3
- [x] EIGRP routes verified using `show ip route`
- [x] EIGRP topology verified using `show ip eigrp topology`

---

# 🏁 Final Result

The lab successfully demonstrates advanced EIGRP functionality.

All five routers participate in **EIGRP AS 100**, advertise their configured networks, and use passive loopback interfaces. Auto-summary is disabled to maintain the actual subnet structure.

R1 is configured with:

```cisco
variance 2
```

allowing it to perform **unequal-cost load balancing** toward R5 using both the R4 and R2 paths.

R3 performs manual EIGRP summarization toward R5:

```cisco
ip summary-address eigrp 100 10.0.0.0 255.0.0.0
```

As a result, R5 learns the summarized:

```text
10.0.0.0/8
```

route through R3.

This lab provides practical experience with **EIGRP neighbor formation, passive interfaces, auto-summary, feasible paths, variance, unequal-cost load balancing, and manual route summarization**.
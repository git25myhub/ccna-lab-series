# OSPF Routing and Path Manipulation Lab

## 📌 Lab Overview

This lab focuses on configuring and troubleshooting **Open Shortest Path First (OSPF)** in a four-router topology.

The main objectives are to:

- Configure loopback interfaces on all routers.
- Advertise all router interfaces using OSPF.
- Make OSPF loopback interfaces passive.
- Configure a consistent OSPF reference bandwidth.
- Modify OSPF interface costs to influence the routing path.
- Ensure **R1 reaches R3's loopback through R2 rather than R4**.

---

## 🖧 Network Topology

The topology consists of four routers connected in a square topology:

```text
                 R1
              /      \
        12.0.0.0     14.0.0.0
           /            \
         R2              R4
           \            /
        23.0.0.0     34.0.0.0
              \      /
                 R3
```

### Router Loopbacks

| Router | Loopback Address |
|--------|------------------|
| R1 | `1.1.1.1/32` |
| R2 | `2.2.2.2/32` |
| R3 | `3.3.3.3/32` |
| R4 | `4.4.4.4/32` |

### Inter-Router Networks

| Link | Network |
|------|---------|
| R1 ↔ R2 | `12.0.0.0/24` |
| R1 ↔ R4 | `14.0.0.0/24` |
| R2 ↔ R3 | `23.0.0.0/24` |
| R3 ↔ R4 | `34.0.0.0/24` |

---

# 🎯 Lab Objectives

The following tasks must be completed:

1. Configure a loopback address on each router.
2. Configure OSPF on every router.
3. Advertise all interfaces, including loopbacks.
4. Configure the loopback interfaces as passive OSPF interfaces.
5. Configure the OSPF reference bandwidth so that a 10-GigabitEthernet interface has an OSPF cost of `1`.
6. Modify OSPF interface costs so that traffic from R1 to R3's loopback uses **R1 → R2 → R3**.
7. Verify OSPF neighbor relationships and routing tables.
8. Save the configuration.

---

# ⚙️ Configuration

## 1. Configure Loopback Interfaces

Each router receives a unique `/32` loopback address.

### R1

```cisco
R1(config)#interface loopback0
R1(config-if)#ip address 1.1.1.1 255.255.255.255
```

### R2

```cisco
R2(config)#interface loopback0
R2(config-if)#ip address 2.2.2.2 255.255.255.255
```

### R3

```cisco
R3(config)#interface loopback0
R3(config-if)#ip address 3.3.3.3 255.255.255.255
```

### R4

```cisco
R4(config)#interface loopback0
R4(config-if)#ip address 4.4.4.4 255.255.255.255
```

Loopbacks provide stable router identifiers and are commonly used as OSPF router IDs.

---

# 2. Configure OSPF

OSPF process IDs are locally significant, so different process IDs can be used on different routers while still forming an OSPF adjacency.

The configuration used in this lab is:

| Router | OSPF Process |
|--------|--------------|
| R1 | `1` |
| R2 | `2` |
| R3 | `3` |
| R4 | `4` |

All interfaces are placed into **Area 0**.

### R1

```cisco
R1(config)#router ospf 1
R1(config-router)#network 12.0.0.0 0.255.255.255 area 0
R1(config-router)#network 14.0.0.0 0.255.255.255 area 0
R1(config-router)#network 1.1.1.1 0.0.0.0 area 0
```

### R2

```cisco
R2(config)#router ospf 2
R2(config-router)#network 0.0.0.0 255.255.255.255 area 0
```

### R3

```cisco
R3(config)#router ospf 3
R3(config-router)#network 0.0.0.0 255.255.255.255 area 0
```

### R4

```cisco
R4(config)#router ospf 4
R4(config-router)#network 0.0.0.0 255.255.255.255 area 0
```

Using:

```cisco
network 0.0.0.0 255.255.255.255 area 0
```

causes all interfaces whose addresses match the statement to participate in OSPF.

---

# 3. Make Loopback Interfaces Passive

Loopback interfaces do not need to form OSPF neighbor relationships.

Therefore, they should be configured as passive interfaces.

### R1

```cisco
R1(config-router)#passive-interface loopback0
```

### R2

```cisco
R2(config-router)#passive-interface loopback0
```

### R3

```cisco
R3(config-router)#passive-interface loopback0
```

### R4

```cisco
R4(config-router)#passive-interface loopback0
```

A passive interface continues to have its network advertised through OSPF, but OSPF does not send Hello packets out of that interface.

---

# 4. Configure OSPF Reference Bandwidth

By default, OSPF uses a reference bandwidth of `100 Mbps`.

The lab requires the reference bandwidth to be increased so that a **10-GigabitEthernet interface has an OSPF cost of 1**.

The required reference bandwidth is:

```text
10,000 Mbps
```

Configure this on **every router**.

### R1

```cisco
R1(config)#router ospf 1
R1(config-router)#auto-cost reference-bandwidth 10000
```

### R2

```cisco
R2(config)#router ospf 2
R2(config-router)#auto-cost reference-bandwidth 10000
```

### R3

```cisco
R3(config)#router ospf 3
R3(config-router)#auto-cost reference-bandwidth 10000
```

### R4

```cisco
R4(config)#router ospf 4
R4(config-router)#auto-cost reference-bandwidth 10000
```

The command produces a warning similar to:

```text
% OSPF: Reference bandwidth is changed.
Please ensure reference bandwidth is consistent across all routers.
```

This is expected.

The reference bandwidth **must be consistent across all routers** for predictable OSPF cost calculations.

---

# 5. OSPF Cost Calculation

OSPF calculates interface cost using:

```text
OSPF Cost = Reference Bandwidth / Interface Bandwidth
```

With a reference bandwidth of `10,000 Mbps`:

| Interface Speed | OSPF Cost |
|----------------|-----------|
| 10 Gbps | `1` |
| 1 Gbps | `10` |
| 100 Mbps | `100` |
| 10 Mbps | `1000` |

This is important because the topology contains both FastEthernet and GigabitEthernet links.

---

# 6. Manipulate OSPF Cost

Initially, R1 has two possible paths to R3's loopback:

```text
R1 → R2 → R3
```

or

```text
R1 → R4 → R3
```

After changing the reference bandwidth to `10,000 Mbps`, the GigabitEthernet links become significantly more attractive than FastEthernet links.

As a result, R1 initially preferred the path through R4:

```text
R1 → R4 → R3
```

The routing table showed:

```text
3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/111] via 14.0.0.4, GigabitEthernet0/0
```

To force R1 to use R2 instead, the OSPF cost of R1's link toward R4 was increased.

### R1

```cisco
R1(config)#interface gigabitEthernet0/0
R1(config-if)#ip ospf cost 10000
```

This makes the R1 → R4 link extremely expensive from OSPF's perspective.

After the cost adjustment, R1 selected:

```text
R1 → R2 → R3
```

The routing table then showed:

```text
3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/201] via 12.0.0.2, FastEthernet1/0
```

This confirms that R1 reaches R3's loopback through R2.

---

# 7. Additional Cost Adjustment

The same principle can be applied to R4.

The configuration used was:

```cisco
R4(config)#interface gigabitEthernet0/0
R4(config-if)#ip ospf cost 10000
```

This increases the cost of the R4 → R1 link and helps influence the overall OSPF path selection.

> **Note:** The lab allows multiple valid solutions. The important requirement is that the final OSPF routing decision causes R1 to reach `3.3.3.3/32` through R2.

---

# 🔍 Verification

## Verify OSPF Neighbors

Use:

```cisco
show ip ospf neighbor
```

R1 should have OSPF neighbors including:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           1   FULL/BDR        ...         14.0.0.4        GigabitEthernet0/0
2.2.2.2           1   FULL/BDR        ...         12.0.0.2        FastEthernet1/0
```

The important state is:

```text
FULL
```

A `FULL` adjacency indicates that the OSPF neighbor relationship has successfully reached the fully adjacent state.

---

## Verify the Routing Table

Use:

```cisco
show ip route
```

R1 should learn the remote loopbacks through OSPF.

For example:

```text
O       2.2.2.2 [110/101] via 12.0.0.2
O       3.3.3.3 [110/201] via 12.0.0.2
O       4.4.4.4 [110/301] via 12.0.0.2
```

Most importantly:

```text
O       3.3.3.3 [110/201] via 12.0.0.2
```

confirms that R1 is using **R2 as the next hop** to reach R3's loopback.

---

# 🧪 Connectivity Testing

From R1:

```cisco
R1#ping 2.2.2.2
```

```cisco
R1#ping 3.3.3.3
```

```cisco
R1#ping 4.4.4.4
```

All three should succeed.

You can also use:

```cisco
R1#traceroute 3.3.3.3
```

The path should demonstrate that traffic toward R3's loopback goes through R2 rather than R4.

Expected logical path:

```text
R1 → R2 → R3
```

---

# 🔎 Useful OSPF Verification Commands

The following commands are useful when troubleshooting the lab:

```cisco
show ip ospf neighbor
```

Displays OSPF neighbor relationships.

```cisco
show ip ospf interface
```

Displays OSPF parameters and interface costs.

```cisco
show ip ospf interface brief
```

Provides a concise summary of OSPF-enabled interfaces.

```cisco
show ip protocols
```

Displays routing protocol configuration.

```cisco
show ip route ospf
```

Displays routes learned through OSPF.

```cisco
show ip ospf database
```

Displays the OSPF link-state database.

```cisco
show running-config
```

Displays the current configuration.

---

# ✅ Final Configuration Summary

| Requirement | Configuration |
|------------|---------------|
| R1 Loopback | `1.1.1.1/32` |
| R2 Loopback | `2.2.2.2/32` |
| R3 Loopback | `3.3.3.3/32` |
| R4 Loopback | `4.4.4.4/32` |
| OSPF Area | Area `0` |
| R1 OSPF Process | `1` |
| R2 OSPF Process | `2` |
| R3 OSPF Process | `3` |
| R4 OSPF Process | `4` |
| Loopbacks | Passive |
| Reference Bandwidth | `10000 Mbps` |
| R1 → R4 OSPF Cost | `10000` |
| Desired R1 → R3 Path | **R1 → R2 → R3** |

---

# 💾 Save Configuration

After completing the lab, save the configuration on each router:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

Expected result:

```text
Building configuration...
[OK]
```

---

# 🏁 Lab Completion Criteria

The lab is successfully completed when:

- [x] R1 has loopback `1.1.1.1/32`.
- [x] R2 has loopback `2.2.2.2/32`.
- [x] R3 has loopback `3.3.3.3/32`.
- [x] R4 has loopback `4.4.4.4/32`.
- [x] All router interfaces are advertised through OSPF.
- [x] All routers operate in OSPF Area 0.
- [x] All loopback interfaces are passive.
- [x] OSPF reference bandwidth is `10,000 Mbps` on every router.
- [x] OSPF adjacencies reach the `FULL` state.
- [x] R1 learns R3's loopback through OSPF.
- [x] R1 reaches `3.3.3.3` through **R2 rather than R4**.
- [x] Configuration is saved.

---

## 🧠 Key Concepts Learned

This lab demonstrates several important OSPF concepts:

1. **Loopback interfaces** provide stable logical addresses for routers.
2. **OSPF network statements** determine which interfaces participate in OSPF.
3. **Passive interfaces** advertise networks without forming OSPF neighbor relationships.
4. **OSPF reference bandwidth** affects how interface costs are calculated.
5. **OSPF cost manipulation** can be used to influence path selection.
6. **Lower cumulative OSPF cost** normally determines the preferred path.
7. OSPF process IDs are **locally significant** and do not have to match between neighboring routers.
8. Consistent reference bandwidth across all OSPF routers is important for predictable routing decisions.

---

## 📚 Commands Used

```cisco
interface loopback0
ip address <address> 255.255.255.255

router ospf <process-id>
network <network> <wildcard-mask> area 0

passive-interface loopback0

auto-cost reference-bandwidth 10000

interface <interface>
ip ospf cost 10000

show ip ospf neighbor
show ip route
show ip route ospf
show ip ospf interface
show ip ospf interface brief
show ip protocols
show ip ospf database

ping <destination>
traceroute <destination>

copy running-config startup-config
```

---

## 🎓 Conclusion

The lab demonstrates how OSPF dynamically calculates routes based on interface costs and how an administrator can manipulate those costs to influence traffic flow.

By increasing the OSPF cost of the R1–R4 link, R1 is prevented from choosing the lower-cost GigabitEthernet path through R4 and instead selects the R2 path to reach R3's loopback:

```text
R1
 |
 | FastEthernet
 v
R2
 |
 | FastEthernet
 v
R3
```

The final routing decision confirms successful OSPF configuration and path manipulation.
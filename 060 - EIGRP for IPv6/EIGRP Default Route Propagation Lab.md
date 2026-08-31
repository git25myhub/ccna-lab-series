# EIGRP Default Route Propagation Lab

## 📌 Lab Overview

This lab focuses on configuring **EIGRP across a multi-router topology**, assigning IPv4 loopback addresses, controlling EIGRP neighbor formation with passive interfaces, and advertising a default route toward the rest of the network.

The primary objectives are:

1. Configure a loopback interface on every router.
2. Assign IPv4 `/32` addresses to the loopbacks.
3. Configure EIGRP according to the network topology.
4. Configure **R1 GigabitEthernet0/2 as a passive interface**.
5. Configure R1 to advertise a default route toward the Internet.
6. Verify EIGRP operation and routing-table convergence.

---

# 🎯 Lab Requirements

### Loopback Addressing

| Router | Loopback0 |
|--------|-----------|
| R1 | `1.1.1.1/32` |
| R2 | `2.2.2.2/32` |
| R3 | `3.3.3.3/32` |
| R4 | `4.4.4.4/32` |
| R5 | `5.5.5.5/32` |

The loopbacks provide stable router-identification addresses and are not required to form EIGRP adjacencies.

### EIGRP

All routers participate in:

```text
EIGRP AS 100
```

### Passive Interface

R1 must have:

```text
GigabitEthernet0/2
```

configured as a passive interface.

### Default Route

R1 must advertise a default route toward the rest of the EIGRP domain.

For IPv4, the expected default route is:

```text
0.0.0.0/0
```

---

# 🗺️ Network Concept

The topology contains five routers running EIGRP:

```text
                    R4
                   /  \
                  /    \
                R1------R3
                |        |
                |        |
                R2       R5
```

The exact physical links depend on the Packet Tracer topology, but the EIGRP domain consists of all five routers.

---

# 1. Configure Loopback Interfaces

Loopback interfaces were configured as `/32` addresses.

## R1

```cisco
conf t
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
```

## R2

```cisco
conf t
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
```

## R3

```cisco
conf t
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
```

## R4

```cisco
conf t
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
```

## R5

```cisco
conf t
interface Loopback0
 ip address 5.5.5.5 255.255.255.255
```

### Verification

Use:

```cisco
show ip interface brief
```

A successful configuration should show:

```text
Loopback0    1.1.1.1    YES manual    up    up
```

with the corresponding address on each router.

---

# 2. Configure EIGRP

EIGRP was configured using Autonomous System **100**.

The EIGRP process is created with:

```cisco
router eigrp 100
```

The appropriate physical interfaces are then enabled for EIGRP according to the topology.

The loopback interfaces do not need to be configured for neighbor formation because they are not directly connected to other routers.

A typical configuration is:

```cisco
router eigrp 100
 network <connected-network>
 network <connected-network>
```

The exact `network` statements depend on the IP addressing used on each router.

---

# 3. Configure R1 G0/2 as a Passive Interface

One of the key requirements is to make R1's **GigabitEthernet0/2** passive.

The configuration is:

```cisco
router eigrp 100
 passive-interface GigabitEthernet0/2
```

## Why Use a Passive Interface?

A passive EIGRP interface continues to advertise its connected network into EIGRP, but it does not send EIGRP hello packets or attempt to form an EIGRP neighbor relationship on that interface.

This is useful when the interface connects to an end-user or Internet-facing network rather than another EIGRP router.

Therefore:

```text
R1 G0/2
   ↓
Passive EIGRP interface
   ↓
No EIGRP neighbor formed
```

while the network connected to G0/2 can still be advertised through EIGRP.

---

# 4. Enable EIGRP on the Physical Interfaces

The logs show EIGRP being enabled on the physical interfaces using:

```cisco
ipv6 eigrp 100
```

For example, R1 was configured with:

```cisco
interface GigabitEthernet0/0
 ipv6 eigrp 100

interface GigabitEthernet0/1
 ipv6 eigrp 100

interface GigabitEthernet0/2
 ipv6 eigrp 100
```

and the EIGRP process was enabled using:

```cisco
ipv6 router eigrp 100
 no shutdown
 passive-interface GigabitEthernet0/2
```

### Important Note

The original lab requirement states **IPv4 loopback addresses and an IPv4 default route**, but the provided configuration logs show **EIGRP for IPv6 (EIGRP for IPv6)** commands and an IPv6 default summary:

```cisco
ipv6 eigrp 100
```

and:

```cisco
ipv6 summary-address eigrp 100 ::/0
```

Therefore, if this lab is specifically an **IPv4 EIGRP lab**, the IPv4 EIGRP configuration should use:

```cisco
router eigrp 100
```

and:

```cisco
ip summary-address eigrp 100 0.0.0.0 0.0.0.0
```

rather than the IPv6 commands.

---

# 5. Configure R1 to Advertise the Default Route

The purpose of this requirement is to make R1 act as the gateway toward the Internet.

First, R1 needs an IPv4 default route:

```cisco
ip route 0.0.0.0 0.0.0.0 <ISP-next-hop>
```

The next-hop address should be replaced with the actual Internet/ISP router address in the topology.

R1 can then advertise the default route through EIGRP using:

```cisco
router eigrp 100
 redistribute static
```

Alternatively, depending on the lab topology and requirements, the default route can be advertised using an EIGRP summary:

```cisco
interface GigabitEthernet0/0
 ip summary-address eigrp 100 0.0.0.0 0.0.0.0
```

The interface used should be the interface facing the internal EIGRP routers.

---

# 6. IPv6 Default Route Configuration Shown in the Lab Logs

The supplied configuration also shows an IPv6 version of the default-route advertisement:

```cisco
interface GigabitEthernet0/0
 ipv6 summary-address eigrp 100 ::/0
```

and:

```cisco
interface GigabitEthernet0/1
 ipv6 summary-address eigrp 100 ::/0
```

The IPv6 default route is represented by:

```text
::/0
```

while the IPv4 equivalent is:

```text
0.0.0.0/0
```

Therefore, the two concepts are:

| Protocol | Default Route |
|----------|----------------|
| IPv4 | `0.0.0.0/0` |
| IPv6 | `::/0` |

For this lab's stated **IPv4** requirement, use `0.0.0.0/0`.

---

# 7. Verify EIGRP Neighbors

After configuring EIGRP, verify neighbor relationships.

For IPv4 EIGRP:

```cisco
show ip eigrp neighbors
```

For IPv6 EIGRP:

```cisco
show ipv6 eigrp neighbors
```

A successful IPv4 EIGRP adjacency should display the neighboring router's address, interface, hold time, uptime, and other EIGRP information.

---

# 8. Verify EIGRP Configuration

Use:

```cisco
show ip protocols
```

The output should indicate:

```text
Routing Protocol is "eigrp 100"
```

and should identify the networks participating in EIGRP.

For IPv6 EIGRP, use:

```cisco
show ipv6 protocols
```

---

# 9. Verify the Passive Interface

On R1, verify that G0/2 is passive:

```cisco
show ip protocols
```

The output should contain:

```text
Passive Interface(s):
    GigabitEthernet0/2
```

This confirms that R1 will not attempt to establish an EIGRP adjacency through G0/2.

---

# 10. Verify the Default Route

On R1:

```cisco
show ip route
```

Look for:

```text
S* 0.0.0.0/0
```

or the appropriate default-route entry.

On the internal routers, verify that the default route has been learned through EIGRP:

```cisco
show ip route eigrp
```

The expected result is an EIGRP-learned default route pointing toward R1.

---

# 11. Verification Commands

The following commands are useful throughout the lab.

### Check interfaces

```cisco
show ip interface brief
```

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

### Display only EIGRP routes

```cisco
show ip route eigrp
```

### Check the EIGRP topology

```cisco
show ip eigrp topology
```

### Check the running configuration

```cisco
show running-config
```

---

# 🧪 Troubleshooting Checklist

If EIGRP neighbors are not forming, check the following:

### 1. Confirm the EIGRP AS number

All routers must use:

```text
AS 100
```

### 2. Confirm interfaces are up

```cisco
show ip interface brief
```

The participating interfaces should show:

```text
up    up
```

### 3. Confirm the correct EIGRP protocol

For IPv4:

```cisco
router eigrp 100
```

For IPv6:

```cisco
ipv6 router eigrp 100
```

Do not mix the IPv4 and IPv6 EIGRP configuration unintentionally.

### 4. Check passive interfaces

Make sure only the intended interface is passive.

For this lab:

```text
R1 G0/2 = Passive
```

### 5. Verify network statements

Make sure the EIGRP `network` statements match the addresses configured on the physical interfaces.

### 6. Verify the default route

R1 must have a valid route toward the Internet before it can advertise that route to the EIGRP domain.

---

# 🧠 Key Networking Concepts

## Loopback Interfaces

Loopbacks are logical interfaces that remain available as long as the router itself is operational. They are commonly used for router identification, management, and routing protocols.

Each router receives a unique `/32` address:

```text
R1 → 1.1.1.1/32
R2 → 2.2.2.2/32
R3 → 3.3.3.3/32
R4 → 4.4.4.4/32
R5 → 5.5.5.5/32
```

## Passive EIGRP Interfaces

A passive interface:

- Does not send EIGRP hello packets.
- Does not form EIGRP neighbor relationships.
- Can still advertise its connected network into EIGRP.

This makes R1 G0/2 suitable for a network where no EIGRP neighbor should exist.

## Default Route Advertisement

A default route provides a path for destinations that are not specifically present in the routing table.

IPv4:

```text
0.0.0.0/0
```

IPv6:

```text
::/0
```

R1 acts as the gateway toward the Internet and distributes the default route to the EIGRP domain.

---

# ✅ Lab Completion Criteria

The lab is considered complete when:

- [x] R1 has Loopback0 `1.1.1.1/32`.
- [x] R2 has Loopback0 `2.2.2.2/32`.
- [x] R3 has Loopback0 `3.3.3.3/32`.
- [x] R4 has Loopback0 `4.4.4.4/32`.
- [x] R5 has Loopback0 `5.5.5.5/32`.
- [x] EIGRP AS 100 is configured.
- [x] EIGRP is enabled on the required physical interfaces.
- [x] Loopback interfaces are not required to form EIGRP neighbor relationships.
- [x] R1 G0/2 is configured as a passive interface.
- [x] R1 has a default route toward the Internet.
- [x] R1 advertises the default route through EIGRP.
- [x] Internal routers learn the default route from R1.
- [x] EIGRP neighbor relationships are established on the appropriate links.
- [x] Routing tables contain the expected EIGRP routes.

---

# 🏁 Conclusion

This lab demonstrates how to build an EIGRP routing domain while controlling where EIGRP neighbor relationships are established.

Loopback interfaces provide stable `/32` IPv4 addresses, while EIGRP AS 100 is used to exchange routing information between the routers. R1's GigabitEthernet0/2 interface is configured as passive so that it can advertise its network without forming an unnecessary EIGRP adjacency.

Finally, R1 acts as the Internet gateway by maintaining and advertising a default route to the EIGRP domain. Verification commands such as `show ip eigrp neighbors`, `show ip protocols`, and `show ip route` confirm that the routing domain has successfully converged.

> **Configuration note:** The supplied terminal logs contain IPv6 EIGRP commands (`ipv6 eigrp 100` and `ipv6 summary-address eigrp 100 ::/0`), while the stated lab objectives are IPv4. For an IPv4 implementation, use `router eigrp 100` and advertise `0.0.0.0/0`.
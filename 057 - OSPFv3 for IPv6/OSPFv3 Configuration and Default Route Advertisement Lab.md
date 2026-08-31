# OSPFv3 Configuration and Default Route Advertisement Lab

## 📌 Lab Overview

This Cisco Packet Tracer lab focuses on configuring **OSPFv3 for IPv6 routing** across multiple routers and advertising a default route from the Internet-facing router.

The lab demonstrates:

- Configuring IPv4 loopback interfaces for OSPFv3 router IDs
- Configuring OSPFv3 on IPv6-enabled interfaces
- Assigning routers to the correct OSPFv3 areas
- Configuring a static IPv6 default route toward the Internet
- Advertising the default route through OSPFv3
- Verifying OSPFv3 neighbor relationships and IPv6 routing
- Troubleshooting an OSPFv3 area mismatch

---

## 🎯 Lab Objectives

The lab requirements were:

1. Configure a loopback interface on each router with an IPv4 address:
   - R1 → `1.1.1.1/32`
   - R2 → `2.2.2.2/32`
   - R3 → `3.3.3.3/32`
   - R4 → `4.4.4.4/32`
   - R5 → `5.5.5.5/32`

2. Configure **OSPFv3** on each router according to the topology.

3. OSPFv3 does **not** need to be configured on the loopback interfaces.

4. Configure a default static IPv6 route to the Internet on R1.

5. Advertise the default route into OSPFv3 so that other routers can learn the Internet route dynamically.

---

## 🗺️ OSPFv3 Area Design

Based on the configuration and troubleshooting output, the topology uses multiple OSPFv3 areas.

| Router | Loopback / Router ID | OSPFv3 Area(s) |
|--------|----------------------|----------------|
| R1 | `1.1.1.1` | Area 1 |
| R2 | `2.2.2.2` | Area 1, Area 0 |
| R3 | `3.3.3.3` | Area 0 / Area 2* |
| R4 | `4.4.4.4` | Area 1 |
| R5 | `5.5.5.5` | Area 2 |

> **Note:** The provided logs show an area mismatch involving R2/R3. The exact intended R3 interface-to-area assignment should follow the Packet Tracer topology.

---

# 1. Configure Loopback Interfaces

Loopback interfaces provide stable IPv4 addresses that can be used as OSPFv3 router IDs.

### R1

```cisco
R1(config)# interface loopback0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
R1(config-if)# exit
```

### R2

```cisco
R2(config)# interface loopback0
R2(config-if)# ip address 2.2.2.2 255.255.255.255
R2(config-if)# exit
```

The provided configuration confirms that R2 was assigned `2.2.2.2/32`.

### R3

```cisco
R3(config)# interface loopback0
R3(config-if)# ip address 3.3.3.3 255.255.255.255
R3(config-if)# exit
```

### R4

```cisco
R4(config)# interface loopback0
R4(config-if)# ip address 4.4.4.4 255.255.255.255
R4(config-if)# exit
```

### R5

```cisco
R5(config)# interface loopback0
R5(config-if)# ip address 5.5.5.5 255.255.255.255
R5(config-if)# exit
```

The R5 verification output shows OSPFv3 using `5.5.5.5` as its router ID.

---

# 2. Configure OSPFv3

OSPFv3 is used to dynamically exchange IPv6 routes.

The OSPFv3 process used in the lab is:

```text
OSPFv3 Process ID: 1
```

The loopback interfaces are not configured for OSPFv3, as required by the lab.

---

## R1 OSPFv3 Configuration

R1 participates in Area 1 through its connected interfaces.

```cisco
R1(config)# ipv6 unicast-routing

R1(config)# ipv6 router ospf 1
R1(config-rtr)# router-id 1.1.1.1
R1(config-rtr)# exit

R1(config)# interface g0/0
R1(config-if)# ipv6 ospf 1 area 1
R1(config-if)# exit

R1(config)# interface g0/1
R1(config-if)# ipv6 ospf 1 area 1
R1(config-if)# exit
```

The provided R1 configuration shows OSPFv3 process 1 applied to the interfaces in Area 1.

---

# 3. Configure OSPFv3 on R2

R2 connects Area 1 to Area 0, making it an important router in the OSPFv3 topology.

```cisco
R2(config)# ipv6 unicast-routing

R2(config)# ipv6 router ospf 1
R2(config-rtr)# router-id 2.2.2.2
R2(config-rtr)# exit

R2(config)# interface g0/0
R2(config-if)# ipv6 ospf 1 area 1
R2(config-if)# exit

R2(config)# interface g0/1
R2(config-if)# ipv6 ospf 1 area 0
R2(config-if)# exit
```

The lab output confirms that R2 formed a FULL adjacency with R1 on G0/0 and with R3 on G0/1.

---

# 4. Configure OSPFv3 on R4

R4 participates in Area 1.

```cisco
R4(config)# ipv6 unicast-routing

R4(config)# ipv6 router ospf 1
R4(config-rtr)# router-id 4.4.4.4
R4(config-rtr)# exit

R4(config)# interface g0/0
R4(config-if)# ipv6 ospf 1 area 1
R4(config-if)# exit
```

The provided R4 running configuration confirms that G0/0 is assigned to OSPFv3 Area 1.

---

# 5. Configure OSPFv3 on R5

R5 participates in Area 2.

```cisco
R5(config)# ipv6 unicast-routing

R5(config)# ipv6 router ospf 1
R5(config-rtr)# router-id 5.5.5.5
R5(config-rtr)# exit

R5(config)# interface g0/0
R5(config-if)# ipv6 ospf 1 area 2
R5(config-if)# exit
```

The verification output confirms that R5 has one OSPFv3 interface in Area 2 and uses `5.5.5.5` as its router ID.

---

# 6. Configure the IPv6 Default Route on R1

R1 is the Internet-facing router, so it requires a static IPv6 default route.

The default route is:

```cisco
R1(config)# ipv6 route ::/0 2001:DB8:1:1::2
```

This sends all unknown IPv6 destinations toward the Internet next hop:

```text
2001:DB8:1:1::2
```

Verification on R1:

```cisco
R1# show ipv6 route
```

Expected output includes:

```text
S   ::/0 [1/0]
     via 2001:DB8:1:1::2
```

The provided routing table confirms that R1 installed the static IPv6 default route successfully.

---

# 7. Advertise the Default Route into OSPFv3

A static default route alone only affects R1.

To allow the other routers to use R1 as their path to the Internet, R1 must advertise the default route into OSPFv3.

Enter OSPFv3 router configuration mode:

```cisco
R1(config)# ipv6 router ospf 1
```

Then configure:

```cisco
R1(config-rtr)# default-information originate
```

Save the configuration:

```cisco
R1(config-rtr)# end
R1# write memory
```

or:

```cisco
R1# copy running-config startup-config
```

---

# 8. Troubleshooting OSPFv3 Area Mismatch

During the lab, the following error repeatedly appeared:

```text
%OSPFv3-4-AREA_MISMATCH:
Received packet with incorrect area
```

For example:

```text
GigabitEthernet0/1, area 0.0.0.0,
packet area 0.0.0.2
```

This indicates that the two OSPFv3 neighbors on the link were configured for **different OSPF areas**.

OSPF neighbors on the same link must agree on the area assigned to that interface.

### Problem

One side was using:

```text
Area 0
```

while the other side was sending OSPFv3 packets for:

```text
Area 2
```

The repeated error messages in the R2 output clearly identify this mismatch.

### Troubleshooting Approach

Check the OSPFv3 configuration on both ends of the affected link:

```cisco
show ipv6 ospf interface
```

or:

```cisco
show running-config
```

Then verify that both interfaces use the intended area.

For example:

```cisco
interface g0/1
 ipv6 ospf 1 area 0
```

must match the area configured on the router at the other end of that link.

If the topology requires Area 2 instead, change the interface to:

```cisco
interface g0/1
 ipv6 ospf 1 area 2
```

After correcting the mismatch, the OSPFv3 adjacency should transition to FULL.

---

# 9. Verification Commands

## Verify IPv6 Interfaces

```cisco
show ipv6 interface brief
```

Confirm that the required interfaces have IPv6 addresses and are operational.

---

## Verify OSPFv3 Neighbors

```cisco
show ipv6 ospf neighbor
```

A healthy adjacency should show:

```text
FULL
```

For example, the lab output showed R2 forming FULL adjacencies with R1 and R3.

---

## Verify OSPFv3 Configuration

```cisco
show ipv6 protocols
```

This displays the interfaces participating in OSPFv3 and their assigned areas.

The R5 output, for example, confirms:

```text
IPv6 Routing Protocol is "ospf 1"

Interfaces (Area 2)
    GigabitEthernet0/0
```



---

## Verify OSPFv3 Process

```cisco
show ipv6 ospf
```

This displays:

- OSPFv3 process ID
- Router ID
- Areas
- Number of interfaces
- LSA information
- SPF calculations

---

## Verify IPv6 Routing Table

```cisco
show ipv6 route
```

Look for routes marked:

```text
O
```

for OSPF intra-area routes,

```text
OI
```

for OSPF inter-area routes, and

```text
OE2
```

for OSPF external routes.

The R2 routing table in the lab shows the default route learned as:

```text
OE2 ::/0 [110/1]
```

via its OSPFv3 neighbor.

---

# 10. Testing Connectivity

After configuration and troubleshooting, test connectivity between router loopbacks.

For example:

```cisco
R2# ping 1.1.1.1
R2# ping 4.4.4.4
R2# ping 5.5.5.5
```

For IPv6 connectivity, test the remote IPv6 interface addresses:

```cisco
R2# ping 2001:DB8:14:14::4
```

The important test is that routers can reach remote IPv6 networks through OSPFv3 and that non-local routers learn the Internet default route through R1.

---

# 🔍 Key Troubleshooting Lessons

### 1. OSPFv3 uses IPv6

OSPFv3 is used for IPv6 routing. Therefore, IPv6 must be enabled on the routers:

```cisco
ipv6 unicast-routing
```

### 2. Loopbacks provide stable router IDs

The IPv4 loopback addresses provide predictable router IDs:

```text
R1 → 1.1.1.1
R2 → 2.2.2.2
R3 → 3.3.3.3
R4 → 4.4.4.4
R5 → 5.5.5.5
```

### 3. OSPF area numbers must match on a link

An OSPFv3 adjacency cannot form correctly when the two sides of a link use different area numbers.

The lab demonstrated this through repeated:

```text
%OSPFv3-4-AREA_MISMATCH
```

messages.

### 4. A static default route must exist before advertising it

R1 first needs:

```cisco
ipv6 route ::/0 2001:DB8:1:1::2
```

Then OSPFv3 can advertise that default route using:

```cisco
ipv6 router ospf 1
 default-information originate
```

### 5. Always verify the routing table

The final verification should confirm that downstream routers learn:

```text
::/0
```

through OSPFv3.

The lab's R2 routing table demonstrates this with an OSPF external default route:

```text
OE2 ::/0 [110/1]
```



---

# ✅ Completion Criteria

The lab is successfully completed when:

- [x] Loopback interfaces are configured on the routers.
- [x] R1 uses `1.1.1.1` as its router ID.
- [x] R2 uses `2.2.2.2` as its router ID.
- [x] R4 uses `4.4.4.4` as its router ID.
- [x] R5 uses `5.5.5.5` as its router ID.
- [x] OSPFv3 process 1 is configured.
- [x] OSPFv3 is enabled on the required physical interfaces.
- [x] Loopback interfaces are not required to participate in OSPFv3.
- [x] OSPFv3 neighbors reach the `FULL` state.
- [x] OSPFv3 area mismatches are corrected.
- [x] R1 has a static IPv6 default route.
- [x] R1 originates the default route into OSPFv3.
- [x] Other routers learn `::/0` through OSPFv3.
- [x] IPv6 connectivity across the topology is successful.

---

## 🧠 Skills Practiced

This lab provides hands-on practice with:

- IPv6 addressing
- IPv6 static routing
- OSPFv3
- OSPF router IDs
- OSPF areas
- OSPFv3 neighbor adjacencies
- Default route propagation
- IPv6 routing-table interpretation
- OSPFv3 troubleshooting
- Area mismatch troubleshooting
- Cisco IOS verification commands

---

## 📚 Useful Cisco IOS Commands

```cisco
show ipv6 interface brief
show ipv6 route
show ipv6 protocols
show ipv6 ospf
show ipv6 ospf neighbor
show ipv6 ospf interface
show running-config
ping <IPv6-address>
traceroute <IPv6-address>
```

---

## 🏁 Final Result

The completed topology uses **OSPFv3 to dynamically exchange IPv6 routes** while R1 provides connectivity toward the Internet through a static IPv6 default route.

R1 originates the default route into OSPFv3, allowing the rest of the OSPFv3 domain to learn the Internet path dynamically.

The main troubleshooting issue encountered during the lab was an **OSPFv3 area mismatch**, where an interface expected OSPFv3 packets from one area but received packets belonging to another area. Correcting the area assignment restores proper OSPFv3 neighbor formation and routing.
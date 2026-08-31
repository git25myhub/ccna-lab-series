# Enterprise Routing, GRE, OSPF, EIGRP, PPP/CHAP & EBGP Troubleshooting Lab

## 📌 Lab Overview

This lab focuses on configuring and troubleshooting multiple enterprise routing technologies across two separate enterprise networks.

The lab covers:

- Static default routes
- GRE tunneling
- OSPF
- PPP with CHAP authentication
- EIGRP
- EBGP
- Route propagation
- Default routing
- End-to-end connectivity troubleshooting

The objective is to build connectivity between the different enterprise networks while using the appropriate routing protocol for each section.

---

# 🏢 Network Design

The lab consists of two independent enterprise environments.

## Enterprise A

Enterprise A contains:

- **R1**
  - LAN: `192.168.1.0/24`
  - Internet-facing network: `203.0.113.0/30`
  - GRE tunnel endpoint: `192.168.12.1/30`

- **R2**
  - LAN: `192.168.2.0/24`
  - Internet-facing network: `203.0.113.4/30`
  - GRE tunnel endpoint: `192.168.12.2/30`

R1 and R2 communicate using a GRE tunnel.

OSPF runs across the GRE tunnel and advertises the internal LAN networks.

### Enterprise A Addressing

| Device | Interface | IP Address | Network |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1/24` | `192.168.1.0/24` |
| R1 | G0/0/0 | `203.0.113.2/30` | Internet |
| R1 | Tunnel0 | `192.168.12.1/30` | GRE |
| R2 | G0/0 | `192.168.2.1/24` | `192.168.2.0/24` |
| R2 | G0/0/0 | `203.0.113.6/30` | Internet |
| R2 | Tunnel0 | `192.168.12.2/30` | GRE |

---

# 🌐 Enterprise A Configuration

## 1. Configure Static Routes to the Internet

R1 requires a default route pointing toward its Internet-facing interface.

### R1

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 g0/0/0
```

R2 also requires a default route.

### R2

```cisco
R2(config)# ip route 0.0.0.0 0.0.0.0 g0/0/0
```

### Verification

```cisco
R1# show ip route
R2# show ip route
```

The routing table should contain:

```text
S* 0.0.0.0/0 is directly connected, GigabitEthernet0/0/0
```

---

# 2. Configure the GRE Tunnel

The GRE tunnel uses the `192.168.12.0/30` network.

- R1 Tunnel0: `192.168.12.1/30`
- R2 Tunnel0: `192.168.12.2/30`

The tunnel uses the Internet-facing interfaces as the source and destination.

## R1

```cisco
R1(config)# interface tunnel 0
R1(config-if)# ip address 192.168.12.1 255.255.255.252
R1(config-if)# tunnel source g0/0/0
R1(config-if)# tunnel destination 203.0.113.6
```

## R2

```cisco
R2(config)# interface tunnel 0
R2(config-if)# ip address 192.168.12.2 255.255.255.252
R2(config-if)# tunnel source g0/0/0
R2(config-if)# tunnel destination 203.0.113.2
```

### GRE Verification

From R1:

```cisco
R1# ping 192.168.12.2
```

Expected result:

```text
Success rate is 100 percent (5/5)
```

From R2:

```cisco
R2# ping 192.168.12.1
```

The tunnel interfaces should also be operational:

```cisco
R1# show ip interface brief
R2# show ip interface brief
```

Expected:

```text
Tunnel0    192.168.12.x    YES manual    up    up
```

---

# 3. Configure OSPF

OSPF is used to exchange the internal routes between R1 and R2 through the GRE tunnel.

OSPF Process ID:

```text
1
```

Area:

```text
Area 0
```

## R1 OSPF Configuration

```cisco
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 192.168.12.0 0.0.0.3 area 0
```

## R2 OSPF Configuration

```cisco
R2(config)# router ospf 1
R2(config-router)# network 192.168.2.0 0.0.0.255 area 0
R2(config-router)# network 192.168.12.0 0.0.0.3 area 0
```

### OSPF Verification

```cisco
R1# show ip ospf neighbor
R2# show ip ospf neighbor
```

The expected neighbor relationship is:

```text
Neighbor ID      State
203.0.113.6      FULL
```

on R1, and:

```text
Neighbor ID      State
203.0.113.2      FULL
```

on R2.

### Verify Learned Routes

On R1:

```cisco
R1# show ip route
```

R1 should learn:

```text
O 192.168.2.0/24 via 192.168.12.2, Tunnel0
```

R2 should learn:

```text
O 192.168.1.0/24 via 192.168.12.1, Tunnel0
```

This confirms that OSPF is successfully operating across the GRE tunnel.

---

# 🏢 Enterprise B

Enterprise B contains:

- **R3**
  - LAN: `192.168.3.0/24`
  - R3-R4 link: `192.168.34.0/24`
  - ISP link: `203.0.113.8/30`

- **R4**
  - LAN: `192.168.4.0/24`
  - R3-R4 link: `192.168.34.0/24`

- **ISP**
  - R3-ISP connection
  - BGP Autonomous System: `65001`

### Enterprise B Addressing

| Device | Interface | IP Address | Network |
|---|---|---|---|
| R3 | LAN | `192.168.3.1/24` | `192.168.3.0/24` |
| R3 | R3-R4 | `192.168.34.1/24` | `192.168.34.0/24` |
| R3 | S0/0/0 | `203.0.113.10/30` | `203.0.113.8/30` |
| R4 | R3-R4 | `192.168.34.2/24` | `192.168.34.0/24` |
| R4 | LAN | `192.168.4.1/24` | `192.168.4.0/24` |
| ISP | R3-facing | `203.0.113.9/30` | `203.0.113.8/30` |

---

# 4. Configure PPP with CHAP Authentication

R3 connects to the ISP using PPP.

The authentication credentials are:

```text
Username: ISP
Password: CCNA
```

## R3 Configuration

Create the ISP username:

```cisco
R3(config)# username ISP password CCNA
```

Configure the serial interface:

```cisco
R3(config)# interface s0/0/0
R3(config-if)# encapsulation ppp
R3(config-if)# ppp authentication chap
R3(config-if)# no shutdown
```

### Verification

First verify the serial interface:

```cisco
R3# show ip interface brief
```

The interface should show:

```text
Serial0/0/0    ...    up    up
```

Test connectivity to the ISP:

```cisco
R3# ping 203.0.113.9
```

Expected:

```text
Success rate is 100 percent (5/5)
```

Successful pinging confirms that the PPP/CHAP connection is operational.

---

# 5. Configure EIGRP

EIGRP Autonomous System:

```text
100
```

EIGRP is used between R3 and R4 to advertise their connected networks.

## R3

```cisco
R3(config)# router eigrp 100
R3(config-router)# network 203.0.113.8 0.0.0.3
R3(config-router)# network 192.168.3.0 0.0.0.255
R3(config-router)# network 192.168.34.0 0.0.0.255
```

The serial interface must be passive:

```cisco
R3(config-router)# passive-interface s0/0/0
```

The passive interface prevents EIGRP neighbor formation toward the ISP while still allowing the connected network to be advertised.

## R4

```cisco
R4(config)# router eigrp 100
R4(config-router)# network 192.168.34.0 0.0.0.255
R4(config-router)# network 192.168.4.0 0.0.0.255
```

### Verify EIGRP Neighbors

On R3:

```cisco
R3# show ip eigrp neighbors
```

Expected neighbor:

```text
192.168.34.2
```

On R4:

```cisco
R4# show ip eigrp neighbors
```

Expected neighbor:

```text
192.168.34.1
```

### Verify EIGRP Routes

On R4:

```cisco
R4# show ip route
```

R4 should learn:

```text
D 192.168.3.0/24 via 192.168.34.1
D 203.0.113.8/30 via 192.168.34.1
```

---

# 6. Configure EBGP

R3 peers with the ISP using external BGP.

R3:

```text
AS 65000
```

ISP:

```text
AS 65001
```

## R3 BGP Configuration

```cisco
R3(config)# router bgp 65000
R3(config-router)# neighbor 203.0.113.9 remote-as 65001
```

Advertise the Enterprise B networks:

```cisco
R3(config-router)# network 192.168.3.0 mask 255.255.255.0
R3(config-router)# network 192.168.34.0 mask 255.255.255.0
R3(config-router)# network 192.168.4.0 mask 255.255.255.0
```

> **Important:** The BGP `network` command only advertises a route if that exact network exists in the routing table. Because `192.168.4.0/24` is learned from R4 through EIGRP, it can be advertised by R3 once EIGRP has installed the route.

### Verify BGP

```cisco
R3# show ip bgp summary
```

The neighbor should be in an established state:

```text
203.0.113.9    ...    Established
```

The lab output confirms that the BGP neighbor successfully came up:

```text
%BGP-5-ADJCHANGE: neighbor 203.0.113.9 Up
```

---

# 🔧 Troubleshooting: Why Can't PC1 Ping PC4?

The important clue in the lab is:

> **HINT: Look on R4.**

The problem is related to the route/default route configuration on R4.

Initially, R4's routing table contained the internal EIGRP routes:

```text
D 192.168.3.0/24 via 192.168.34.1
D 203.0.113.8/30 via 192.168.34.1
```

and its directly connected network:

```text
C 192.168.4.0/24
```

However, R4 did not have a default route.

Therefore, traffic originating from PC4 toward networks outside its directly connected and EIGRP-learned networks could not be forwarded correctly.

The fix was to configure a default route on R4 pointing toward R3:

```cisco
R4(config)# ip route 0.0.0.0 0.0.0.0 192.168.34.1
```

This makes R3 the next hop for destinations that are not otherwise present in R4's routing table.

### Verify the Fix

```cisco
R4# show ip route
```

The routing table should now contain:

```text
S* 0.0.0.0/0 via 192.168.34.1
```

---

# 🧪 Connectivity Testing

After completing the configurations, test connectivity from the PCs.

## PC1 → PC3

```text
PC1> ping 192.168.3.100
```

Expected:

```text
Reply from 192.168.3.100
```

The first packet may fail because of ARP/routing convergence. Subsequent packets should succeed.

## PC1 → PC4

```text
PC1> ping 192.168.4.100
```

Expected:

```text
Reply from 192.168.4.100
```

This confirms end-to-end routing through:

```text
PC1
 ↓
R1
 ↓
Enterprise routing
 ↓
R3
 ↓
R4
 ↓
PC4
```

---

# 🔍 Useful Verification Commands

## Interface Status

```cisco
show ip interface brief
```

## Routing Table

```cisco
show ip route
```

## OSPF Neighbors

```cisco
show ip ospf neighbor
```

## OSPF Routes

```cisco
show ip route ospf
```

## EIGRP Neighbors

```cisco
show ip eigrp neighbors
```

## EIGRP Routes

```cisco
show ip route eigrp
```

## BGP Summary

```cisco
show ip bgp summary
```

## BGP Routes

```cisco
show ip route bgp
```

## GRE Tunnel

```cisco
show interfaces tunnel 0
```

## PPP Configuration

```cisco
show interfaces serial 0/0/0
```

## Configuration Verification

```cisco
show running-config
```

---

# 💾 Save the Configuration

After completing each router configuration:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

Expected:

```text
[OK]
```

---

# ✅ Final Verification Checklist

### Enterprise A

- [x] R1 has a static default route.
- [x] R2 has a static default route.
- [x] GRE Tunnel0 configured between R1 and R2.
- [x] Tunnel network uses `192.168.12.0/30`.
- [x] R1 Tunnel0 uses `192.168.12.1`.
- [x] R2 Tunnel0 uses `192.168.12.2`.
- [x] GRE tunnel is operational.
- [x] OSPF process 1 configured.
- [x] OSPF Area 0 configured.
- [x] R1 learns `192.168.2.0/24`.
- [x] R2 learns `192.168.1.0/24`.
- [x] OSPF neighbor reaches `FULL` state.

### Enterprise B

- [x] R3 username `ISP` configured.
- [x] CHAP password `CCNA` configured.
- [x] PPP encapsulation configured.
- [x] R3 can reach the ISP.
- [x] EIGRP AS 100 configured.
- [x] R3 advertises its connected routes.
- [x] R4 advertises its connected routes.
- [x] R3 S0/0/0 configured as passive.
- [x] R3 and R4 form an EIGRP adjacency.
- [x] EBGP configured between R3 and ISP.
- [x] R3 uses AS 65000.
- [x] ISP uses AS 65001.
- [x] BGP neighbor reaches established/up state.
- [x] R4 has a default route via `192.168.34.1`.
- [x] PC1 can ping PC4.

---

# 🧠 Key Concepts Practiced

| Technology | Purpose |
|---|---|
| Static Route | Provides a default path toward the Internet |
| GRE | Creates a logical tunnel between R1 and R2 |
| OSPF | Dynamically exchanges Enterprise A internal routes |
| PPP | Provides encapsulation on the serial WAN connection |
| CHAP | Authenticates R3 with the ISP |
| EIGRP | Exchanges routes between R3 and R4 |
| Passive Interface | Prevents EIGRP neighbor formation on R3's ISP-facing interface |
| EBGP | Exchanges routes between Enterprise B and the ISP |
| Default Route | Provides R4 with a path toward unknown destinations |

---

# 🏁 Lab Completion

The lab is successfully completed when:

1. R1 and R2 can communicate through the GRE tunnel.
2. OSPF forms a `FULL` adjacency between R1 and R2.
3. R1 learns the `192.168.2.0/24` network through OSPF.
4. R2 learns the `192.168.1.0/24` network through OSPF.
5. R3 successfully authenticates with the ISP using PPP/CHAP.
6. R3 and R4 establish an EIGRP adjacency.
7. R3's ISP-facing serial interface is passive under EIGRP.
8. R3 establishes an EBGP session with the ISP.
9. R4 has a default route pointing to R3.
10. PC1 can successfully ping PC4.

The main troubleshooting lesson is that **successful routing depends not only on having routes to remote networks, but also on having a return path**. In this lab, the missing default route on R4 prevented traffic from being returned correctly toward networks outside Enterprise B.
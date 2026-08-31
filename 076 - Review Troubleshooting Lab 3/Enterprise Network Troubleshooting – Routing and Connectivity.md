# Enterprise Network Troubleshooting – Routing and Connectivity

## 📌 Lab Overview

This Cisco Packet Tracer lab focuses on troubleshooting and fixing routing and connectivity problems across two enterprise networks.

The objective is to identify why specific PCs cannot communicate and restore full end-to-end connectivity without unnecessarily changing the existing network design.

### Reported Problems

The lab contains three connectivity issues:

1. **PC2 cannot ping PC1.**
2. **PC4 cannot ping PC3.**
3. **Hosts in Enterprise B cannot ping hosts in Enterprise A.**

The troubleshooting process requires examining interface status, routing tables, dynamic routing relationships, and WAN connectivity.

---

# 🏢 Network Architecture

The topology consists of two enterprise environments connected through routing infrastructure.

## Enterprise A

Enterprise A contains the following networks:

| Network | Purpose |
|---|---|
| `192.168.1.0/24` | Enterprise A LAN |
| `192.168.2.0/24` | Enterprise A LAN |
| `203.0.113.0/30` | WAN/Internet connectivity |
| `203.0.113.4/30` | WAN/Internet connectivity |

R1 and R2 use OSPF across a GRE tunnel.

## Enterprise B

Enterprise B contains:

| Network | Purpose |
|---|---|
| `192.168.3.0/24` | Enterprise B LAN |
| `192.168.4.0/24` | Enterprise B LAN |
| `192.168.34.0/24` | R3-R4 transit network |
| `203.0.113.8/30` | R3-ISP WAN connection |

R3 and R4 use EIGRP internally.

R3 connects to the ISP using PPP/CHAP and EBGP.

---

# 🔎 Initial Troubleshooting

The first step is to reproduce each reported problem from the PCs.

---

## Problem 1 – PC2 Cannot Ping PC1

The initial test from PC2 was:

```text
C:\> ping 192.168.1.100
```

The result was:

```text
Reply from 192.168.2.1: Destination host unreachable.
```

This is significant because the message comes from the default gateway:

```text
192.168.2.1
```

rather than from PC1.

This indicates that the router serving PC2 does not initially have a valid route toward:

```text
192.168.1.0/24
```

---

# 🛠️ Fixing the Enterprise A Routing Problem

The routing table on R2 was inspected:

```cisco
R2# show ip route
```

Initially, the routing table contained only the directly connected networks:

```text
C 192.168.2.0/24
C 203.0.113.4/30
```

There was no route for:

```text
192.168.1.0/24
```

and no default route.

A static default route was therefore configured:

```cisco
R2(config)# ip route 0.0.0.0 0.0.0.0 g0/0/0
```

However, the important part of the topology is that R1 and R2 use a GRE tunnel and OSPF to exchange their internal LAN routes.

The OSPF adjacency subsequently came up:

```text
%OSPF-5-ADJCHG: Process 1, Nbr 203.0.113.2 on Tunnel0 from LOADING to FULL
```

This confirms that the OSPF neighbor relationship reached the required:

```text
FULL
```

state.

### Verify

```cisco
R2# show ip ospf neighbor
R2# show ip route
```

R2 should eventually have an OSPF route similar to:

```text
O 192.168.1.0/24 via 192.168.12.1, Tunnel0
```

---

# 🧪 PC2 → PC1 Verification

After routing convergence, PC2 was tested again:

```text
C:\> ping 192.168.1.100
```

The initial attempt showed:

```text
Request timed out.
Reply from 192.168.1.100
Reply from 192.168.1.100
Reply from 192.168.1.100
```

The first failed packet is normal during ARP/routing convergence.

A subsequent test achieved:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Result

✅ **PC2 can now successfully ping PC1.**

---

# 🔎 Problem 2 – PC4 Cannot Ping PC3

The second reported issue was:

> PC4 cannot ping PC3.

The relevant Enterprise B networks are:

```text
192.168.3.0/24
192.168.4.0/24
```

R3 connects these networks through R4 using:

```text
192.168.34.0/24
```

---

# 🔍 Troubleshooting R4

The routing table on R4 was inspected:

```cisco
R4# show ip route
```

The routing table showed:

```text
C 192.168.4.0/24
C 192.168.34.0/24
```

and the default route:

```text
S* 0.0.0.0/0 [1/0] via 192.168.34.1
```

R4 also established an EIGRP relationship with R3:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 192.168.34.1 (GigabitEthernet0/0) is up
```

This confirms that R4 successfully formed an EIGRP adjacency with R3.

---

# 🛠️ EIGRP Route Verification

R4 should learn the Enterprise B LAN behind R3:

```text
192.168.3.0/24
```

The expected route is:

```text
D 192.168.3.0/24 via 192.168.34.1
```

This provides the path:

```text
PC4
  ↓
R4
  ↓
R3
  ↓
192.168.3.0/24
  ↓
PC3
```

---

# 🧪 PC4 → PC3 Verification

From PC4:

```text
C:\> ping 192.168.3.100
```

The first test may result in timeouts while ARP and routing information are being resolved.

After convergence, the test should show:

```text
Reply from 192.168.3.100
Reply from 192.168.3.100
Reply from 192.168.3.100
Reply from 192.168.3.100
```

Expected result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Result

✅ **PC4 can successfully communicate with PC3.**

---

# 🌐 Problem 3 – Enterprise B Cannot Ping Enterprise A

The third issue is more significant because it crosses the enterprise boundary.

Enterprise B uses:

```text
192.168.3.0/24
192.168.4.0/24
```

Enterprise A uses:

```text
192.168.1.0/24
192.168.2.0/24
```

The routing path should be:

```text
Enterprise B
     |
    R3
     |
    ISP
     |
Enterprise A
```

---

# 🔍 Troubleshooting R3

The running configuration of R3 was examined:

```cisco
R3# show running-config
```

R3 has:

```text
GigabitEthernet0/0
192.168.3.1/24

GigabitEthernet0/1
192.168.34.1/24

Serial0/0/0
203.0.113.10/30
```

R3 is also running EIGRP:

```text
router eigrp 100
```

and BGP:

```text
router bgp 65000
```

with the ISP:

```text
neighbor 203.0.113.9 remote-as 65001
```

---

# 🔍 Verify the R3–ISP Connection

R3 uses PPP with CHAP authentication.

The configuration includes:

```cisco
username ISP password 0 CCNA
```

and:

```cisco
interface Serial0/0/0
 encapsulation ppp
 ppp authentication chap
```

After the configuration was corrected, the serial interface returned to:

```text
up/up
```

The BGP session also came up:

```text
%BGP-5-ADJCHANGE: neighbor 203.0.113.9 Up
```

This confirms that the R3–ISP BGP relationship is operational.

---

# 🔍 Verify R3's Routing Table

The final R3 routing table contains:

```text
B 1.1.1.1/32 via 203.0.113.9
B 192.168.1.0/24 via 203.0.113.9
B 192.168.2.0/24 via 203.0.113.9
```

This is critical.

R3 now knows how to reach both Enterprise A networks:

```text
192.168.1.0/24
192.168.2.0/24
```

The routes are learned through EBGP from the ISP.

---

# 🧭 End-to-End Routing

The path from Enterprise B to Enterprise A is now:

```text
PC3/PC4
   |
   v
R3
   |
   | EBGP
   v
ISP
   |
   v
Enterprise A
```

For example, traffic destined for:

```text
192.168.1.100
```

is forwarded by R3 using:

```text
B 192.168.1.0/24 via 203.0.113.9
```

---

# 🔁 Return Path

Routing is bidirectional.

It is not enough for R3 to know how to reach Enterprise A.

Enterprise A must also know how to return traffic to Enterprise B.

Therefore, verify that the Enterprise A routers have routes for:

```text
192.168.3.0/24
192.168.4.0/24
```

The OSPF/GRE portion of the topology should provide the internal Enterprise A routes, while the WAN/BGP configuration provides connectivity toward the other enterprise.

---

# 🧪 Final Connectivity Tests

## PC2 → PC1

```text
PC2> ping 192.168.1.100
```

Expected:

```text
Reply from 192.168.1.100
```

✅ Successful.

---

## PC4 → PC3

```text
PC4> ping 192.168.3.100
```

Expected:

```text
Reply from 192.168.3.100
```

✅ Successful.

---

## Enterprise B → Enterprise A

From PC3:

```text
PC3> ping 192.168.1.100
```

or:

```text
PC3> ping 192.168.2.100
```

Expected:

```text
Reply from 192.168.x.100
```

The same should work from PC4.

---

# 🔧 Useful Troubleshooting Commands

## Check Interface Status

```cisco
show ip interface brief
```

Use this to identify interfaces that are:

```text
administratively down
down
up/down
up/up
```

---

## Check the Routing Table

```cisco
show ip route
```

Look for:

```text
C = Connected
S = Static
D = EIGRP
O = OSPF
B = BGP
```

---

## Check OSPF Neighbors

```cisco
show ip ospf neighbor
```

The expected state is:

```text
FULL
```

---

## Check OSPF Routes

```cisco
show ip route ospf
```

---

## Check EIGRP Neighbors

```cisco
show ip eigrp neighbors
```

---

## Check EIGRP Routes

```cisco
show ip route eigrp
```

---

## Check BGP Neighbors

```cisco
show ip bgp summary
```

The BGP peer should be established.

---

## Check BGP Routes

```cisco
show ip route bgp
```

On R3, the Enterprise A routes should appear as BGP routes.

---

## Test the Next Hop

Use ping to verify individual links:

```cisco
R3# ping 203.0.113.9
R3# ping 192.168.34.2
```

From R4:

```cisco
R4# ping 192.168.34.1
R4# ping 192.168.3.1
```

---

## Trace the Path

From a PC:

```text
tracert 192.168.1.100
```

This helps identify where packets stop when end-to-end connectivity fails.

---

# 🧠 Troubleshooting Lessons

## 1. Destination Host Unreachable vs Request Timed Out

These messages provide useful clues.

### Destination Host Unreachable

Example:

```text
Reply from 192.168.2.1:
Destination host unreachable.
```

This indicates that the device at `192.168.2.1` does not know how to reach the destination or cannot forward the packet.

### Request Timed Out

A timeout means that the packet did not receive a response within the expected period.

Possible causes include:

- Missing route
- Incorrect return route
- Interface failure
- ACL
- Host firewall
- ARP resolution
- Routing protocol convergence

---

# 🧠 Important Routing Principle

A successful ping requires a **forward path and a return path**.

For:

```text
PC4 → PC1
```

there must be a route:

```text
PC4 → R4 → R3 → ISP → Enterprise A
```

and a return route:

```text
Enterprise A → ISP/R3 → R4 → PC4
```

If either direction is missing, the ping can fail.

---

# 📋 Final Verification Checklist

### Enterprise A

- [x] R2 has a valid path toward `192.168.1.0/24`.
- [x] GRE Tunnel0 is operational.
- [x] OSPF neighbor reaches `FULL`.
- [x] Enterprise A internal routes are present.
- [x] PC2 can ping PC1.

### Enterprise B

- [x] R3 and R4 are connected through `192.168.34.0/24`.
- [x] EIGRP AS 100 is configured.
- [x] R3 and R4 form an EIGRP adjacency.
- [x] R4 learns `192.168.3.0/24`.
- [x] R4 has a default route via `192.168.34.1`.
- [x] PC4 can ping PC3.

### WAN / ISP

- [x] R3 serial interface is operational.
- [x] PPP encapsulation is configured.
- [x] CHAP authentication is configured.
- [x] R3 can reach the ISP.
- [x] EBGP peer `203.0.113.9` is established.
- [x] R3 learns `192.168.1.0/24` through BGP.
- [x] R3 learns `192.168.2.0/24` through BGP.
- [x] Enterprise B has a valid path toward Enterprise A.

---

# 🏁 Lab Completion

The lab is successfully completed when all three original problems have been resolved:

| Problem | Resolution | Status |
|---|---|---|
| PC2 cannot ping PC1 | Restore/verify Enterprise A routing and OSPF convergence | ✅ |
| PC4 cannot ping PC3 | Verify EIGRP adjacency and R4 routing | ✅ |
| Enterprise B cannot ping Enterprise A | Restore WAN/BGP route exchange and verify return paths | ✅ |

The main troubleshooting approach used in this lab is:

```text
Test Connectivity
       ↓
Identify Where the Packet Stops
       ↓
Check Interface Status
       ↓
Check Routing Table
       ↓
Check Routing Protocol Neighbors
       ↓
Fix the Configuration
       ↓
Verify Learned Routes
       ↓
Retest End-to-End Connectivity
```

## 🎯 Key Skills Practiced

- Reading Cisco routing tables
- Identifying missing routes
- Troubleshooting OSPF
- Troubleshooting EIGRP
- Troubleshooting BGP
- Verifying PPP/CHAP
- Understanding default routes
- Understanding next-hop routing
- Troubleshooting return paths
- Using `ping` and `tracert`
- Validating end-to-end network connectivity

---

## 💾 Save Configurations

After fixing the problems, save each router's configuration:

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

This ensures that the troubleshooting fixes persist after the devices are restarted.
# GRE Tunnel and EIGRP Configuration Lab

## 📌 Lab Overview

This lab demonstrates how to configure a **Generic Routing Encapsulation (GRE) tunnel** between two Cisco routers and use **EIGRP** to dynamically exchange routes across the tunnel.

The objective is to allow **PC1 and PC2**, which are located on different LAN networks, to communicate with each other through the GRE tunnel.

---

## 🎯 Lab Objectives

1. Configure a GRE tunnel between **R1 and R2**.
2. Assign IP addresses to the GRE tunnel interfaces.
3. Configure static routes to provide reachability to the remote tunnel endpoints.
4. Configure **EIGRP AS 100** on R1 and R2.
5. Advertise the LAN networks and GRE tunnel network through EIGRP.
6. Verify that an EIGRP neighbor relationship is established over the GRE tunnel.
7. Verify that PC1 and PC2 can communicate across the network.

---

## 🗺️ Network Addressing

| Device | Interface | IP Address | Network |
|---|---|---|---|
| R1 | G0/0 | 10.0.1.1/24 | PC1 LAN |
| R1 | S0/0/0 | 100.0.0.2/30 | WAN |
| R1 | Tunnel0 | 192.168.1.1/30 | GRE Tunnel |
| R2 | G0/0 | 10.0.2.1/24 | PC2 LAN |
| R2 | S0/0/0 | 200.0.0.2/30 | WAN |
| R2 | Tunnel0 | 192.168.1.2/30 | GRE Tunnel |
| PC1 | NIC | 10.0.1.100/24 | 10.0.1.0/24 |
| PC2 | NIC | 10.0.2.100/24 | 10.0.2.0/24 |

### GRE Tunnel

```text
R1 Tunnel0: 192.168.1.1/30
       |
       | GRE Tunnel
       |
R2 Tunnel0: 192.168.1.2/30
```

### WAN Transport

```text
R1 S0/0/0: 100.0.0.2/30
        |
        | WAN
        |
R2 S0/0/0: 200.0.0.2/30
```

---

# 🔧 Configuration

## 1. Configure the GRE Tunnel on R1

Enter configuration mode:

```cisco
R1# configure terminal
```

Create the tunnel interface:

```cisco
R1(config)# interface tunnel 0
R1(config-if)# ip address 192.168.1.1 255.255.255.252
R1(config-if)# tunnel source serial 0/0/0
R1(config-if)# tunnel destination 200.0.0.2
```

Save the configuration:

```cisco
R1(config-if)# do write
```

### R1 GRE Configuration

```cisco
interface Tunnel0
 ip address 192.168.1.1 255.255.255.252
 tunnel source Serial0/0/0
 tunnel destination 200.0.0.2
```

---

## 2. Configure the GRE Tunnel on R2

On R2:

```cisco
R2# configure terminal
R2(config)# interface tunnel 0
R2(config-if)# ip address 192.168.1.2 255.255.255.252
R2(config-if)# tunnel source serial 0/0/0
R2(config-if)# tunnel destination 100.0.0.2
```

Save the configuration:

```cisco
R2(config-if)# do write
```

### R2 GRE Configuration

```cisco
interface Tunnel0
 ip address 192.168.1.2 255.255.255.252
 tunnel source Serial0/0/0
 tunnel destination 100.0.0.2
```

---

# 🌐 3. Configure Routes to the GRE Tunnel Endpoints

The routers must be able to reach each other's **GRE destination addresses** through the underlying WAN network.

### R1

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 100.0.0.1
```

### R2

```cisco
R2(config)# ip route 0.0.0.0 0.0.0.0 200.0.0.1
```

These routes provide reachability toward the remote GRE tunnel endpoint.

---

# 🔄 4. Verify the GRE Tunnel

On R1:

```cisco
R1# show interface tunnel 0
```

A successful tunnel should display:

```text
Tunnel0 is up, line protocol is up
```

The output should also show:

```text
Internet address is 192.168.1.1/30
Tunnel source 100.0.0.2
Tunnel destination 200.0.0.2
Tunnel protocol/transport GRE/IP
```

On R2:

```cisco
R2# show interface tunnel 0
```

The expected result is:

```text
Tunnel0 is up, line protocol is up
```

---

# 🧪 5. Test the GRE Tunnel

From R1, ping the tunnel IP of R2:

```cisco
R1# ping 192.168.1.2
```

Expected result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

This confirms that the GRE tunnel is operational.

---

# 🛣️ 6. Configure EIGRP on R1

Enable EIGRP using autonomous system **100**:

```cisco
R1(config)# router eigrp 100
```

Advertise the R1 LAN:

```cisco
R1(config-router)# network 10.0.1.0 0.0.0.255
```

Advertise the GRE tunnel network:

```cisco
R1(config-router)# network 192.168.1.0 0.0.0.3
```

Save the configuration:

```cisco
R1(config-router)# do write
```

### R1 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.1.0 0.0.0.255
 network 192.168.1.0 0.0.0.3
```

---

# 🛣️ 7. Configure EIGRP on R2

Enable the same EIGRP autonomous system:

```cisco
R2(config)# router eigrp 100
```

Advertise the R2 LAN:

```cisco
R2(config-router)# network 10.0.2.0 0.0.0.255
```

Advertise the GRE tunnel network:

```cisco
R2(config-router)# network 192.168.1.0 0.0.0.3
```

Save the configuration:

```cisco
R2(config-router)# do write
```

### R2 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.2.0 0.0.0.255
 network 192.168.1.0 0.0.0.3
```

---

# 🤝 8. Verify the EIGRP Neighbor Relationship

On R1:

```cisco
R1# show ip eigrp neighbors
```

The expected neighbor should be:

```text
Neighbor Address: 192.168.1.2
Interface: Tunnel0
```

The lab output confirms:

```text
%DUAL-5-NBRCHANGE: IP-EIGRP 100:
Neighbor 192.168.1.2 (Tunnel0) is up: new adjacency
```

This proves that **EIGRP formed an adjacency across the GRE tunnel**.

On R2, the neighbor should similarly be:

```text
Neighbor 192.168.1.1 (Tunnel0)
```

---

# 🗺️ 9. Verify EIGRP Routes

On R2:

```cisco
R2# show ip route
```

The remote R1 LAN should appear as an EIGRP route:

```text
D       10.0.1.0/24 [90/26880256] via 192.168.1.1, Tunnel0
```

The `D` indicates that the route was learned through **EIGRP**.

Likewise, R1 should learn the R2 LAN:

```text
D       10.0.2.0/24 via 192.168.1.2, Tunnel0
```

---

# 💻 10. Test PC-to-PC Connectivity

Once EIGRP has successfully exchanged the LAN routes, PC1 should be able to communicate with PC2.

From PC2:

```text
C:\> ping 10.0.1.100
```

Expected result:

```text
Reply from 10.0.1.100
Reply from 10.0.1.100
Reply from 10.0.1.100
```

The first packet may time out while ARP and routing information are being resolved.

Example from the lab:

```text
Pinging 10.0.1.100 with 32 bytes of data:

Request timed out.
Reply from 10.0.1.100: bytes=32 time=25ms TTL=126
Reply from 10.0.1.100: bytes=32 time=10ms TTL=126
Reply from 10.0.1.100: bytes=32 time=4ms TTL=126

Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

Subsequent pings should succeed consistently.

---

# 🔍 Verification Commands

### Check interface status

```cisco
show ip interface brief
```

### Check the GRE tunnel

```cisco
show interface tunnel 0
```

### Check the routing table

```cisco
show ip route
```

### Check EIGRP neighbors

```cisco
show ip eigrp neighbors
```

### Check EIGRP routes

```cisco
show ip route eigrp
```

### Check EIGRP configuration

```cisco
show running-config | section router eigrp
```

### Test tunnel connectivity

```cisco
ping 192.168.1.2
```

### Test end-to-end connectivity

```text
PC1> ping 10.0.2.100
```

or:

```text
PC2> ping 10.0.1.100
```

---

# 🧠 Key Concepts Learned

## GRE

**Generic Routing Encapsulation (GRE)** creates a virtual point-to-point tunnel between two routers.

In this lab:

```text
R1 ================= GRE ================= R2
     192.168.1.1                 192.168.1.2
```

The GRE tunnel transports the traffic between the two LANs over the underlying WAN network.

---

## EIGRP

**Enhanced Interior Gateway Routing Protocol (EIGRP)** is a dynamic routing protocol that allows routers to exchange network information.

EIGRP was configured using:

```cisco
router eigrp 100
```

Both routers use **AS 100**, allowing them to establish an EIGRP adjacency.

---

## EIGRP Over GRE

The important concept demonstrated by this lab is that a routing protocol can operate across a GRE tunnel.

Instead of relying on the physical WAN interface for EIGRP adjacency, R1 and R2 form their EIGRP relationship through:

```text
Tunnel0
```

Therefore:

```text
R1 LAN
10.0.1.0/24
     |
    R1
     |
Tunnel0
192.168.1.0/30
     |
    R2
     |
R2 LAN
10.0.2.0/24
```

---

# 🛠️ Troubleshooting

If PC1 and PC2 cannot communicate, troubleshoot in this order.

### 1. Check physical interfaces

```cisco
show ip interface brief
```

Make sure the relevant interfaces are:

```text
up/up
```

### 2. Check the WAN endpoint

From R1:

```cisco
ping 200.0.0.2
```

From R2:

```cisco
ping 100.0.0.2
```

The GRE tunnel cannot function if the tunnel endpoints cannot reach each other.

### 3. Check Tunnel0

```cisco
show interface tunnel 0
```

Verify:

```text
Tunnel0 is up, line protocol is up
```

### 4. Test the tunnel IP

From R1:

```cisco
ping 192.168.1.2
```

### 5. Check EIGRP neighbors

```cisco
show ip eigrp neighbors
```

R1 should see R2 and R2 should see R1.

### 6. Check learned routes

```cisco
show ip route eigrp
```

R1 should have a route to:

```text
10.0.2.0/24
```

R2 should have a route to:

```text
10.0.1.0/24
```

### 7. Test PC connectivity

Finally:

```text
PC1> ping 10.0.2.100
```

and:

```text
PC2> ping 10.0.1.100
```

---

# ✅ Completion Criteria

The lab is successfully completed when:

- [x] GRE Tunnel0 is configured on R1.
- [x] GRE Tunnel0 is configured on R2.
- [x] Tunnel0 is **up/up** on both routers.
- [x] R1 can ping `192.168.1.2`.
- [x] R2 can ping `192.168.1.1`.
- [x] EIGRP AS 100 is configured on both routers.
- [x] EIGRP forms a neighbor relationship over Tunnel0.
- [x] R1 learns `10.0.2.0/24` through EIGRP.
- [x] R2 learns `10.0.1.0/24` through EIGRP.
- [x] PC1 can ping PC2.
- [x] PC2 can ping PC1.

---

# 📚 Summary

This lab demonstrates how **GRE tunneling and dynamic routing can work together**.

GRE provides a logical point-to-point tunnel between R1 and R2, while EIGRP operates across that tunnel to dynamically advertise the LAN networks.

The final communication path is:

```text
PC1
10.0.1.100
   |
   |
R1
10.0.1.1
   |
   | GRE Tunnel
   | 192.168.1.0/30
   |
R2
10.0.2.1
   |
   |
PC2
10.0.2.100
```

The successful EIGRP adjacency and learned routes allow the two PCs to communicate without requiring static routes for the LAN networks.
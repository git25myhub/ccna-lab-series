# Cisco BGP – eBGP Route Advertisement and Troubleshooting Lab

## 📌 Lab Overview

This lab focuses on configuring **External Border Gateway Protocol (eBGP)** between an internal router and a service provider router, advertising internal networks through BGP, and verifying end-to-end connectivity to a remote server.

The lab also demonstrates how a **Null0 static route** can be used to satisfy BGP's requirement that a network must exist in the routing table before it can be advertised with a `network` statement.

### Topology

```text
                         External Network
                              |
                              |
                         +---------+
                         |  SPR1   |
                         | AS 65001|
                         +----+----+
                              |
                       100.0.0.0/30
                              |
                         +----+----+
                         |   R1   |
                         | AS65000|
                         +----+----+
                         /       \
                 10.0.12.0/30   10.0.13.0/30
                       /             \
                  +---+---+       +---+---+
                  |  R2   |-------|  R3   |
                  +-------+ 10.0.23.0/30
```

R1, R2, and R3 use OSPF internally, while R1 establishes an eBGP peering session with SPR1.

---

# 🎯 Lab Objectives

The objectives of this lab are:

1. Test connectivity to the remote server at `15.0.0.1`.
2. Configure eBGP between R1 and SPR1.
3. Advertise each router's loopback interface in BGP.
4. Advertise the following networks using a **single BGP network command with a `/16` mask**:
   - `10.0.12.0/30`
   - `10.0.23.0/30`
   - `10.0.13.0/30`
5. Create a static route to `Null0` so that the `10.0.0.0/16` network exists in R1's routing table.
6. Verify BGP routes and restore end-to-end connectivity to `15.0.0.1`.

---

# 🧪 Initial Connectivity Test

Before configuring BGP, connectivity to the remote server was tested from R2.

```text
R2#ping 15.0.0.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 15.0.0.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

The initial ping failed because the required BGP connectivity and route advertisements had not yet been completed.

---

# 🔎 Initial Routing Information

R2 already had OSPF routes to the internal routers and networks.

Important routes included:

```text
O       1.1.1.1/32 via 10.0.12.1
C       2.2.2.2/32 directly connected
O       3.3.3.3/32 via 10.0.23.2

C       10.0.12.0/30
C       10.0.23.0/30
O       10.0.13.0/30
```

R2 also had a default route toward R1:

```text
O*E2 0.0.0.0/0 via 10.0.12.1
```

However, the remote network was not yet reachable.

---

# 1. Configure eBGP Between R1 and SPR1

SPR1 was already pre-configured.

R1 belongs to **AS 65000**, while SPR1 belongs to **AS 65001**.

The eBGP neighbor was configured using SPR1's address:

```text
R1(config)#router bgp 65000
R1(config-router)#neighbor 100.0.0.1 remote-as 65001
```

After configuring the neighbor, the following message confirmed that the BGP adjacency was successfully established:

```text
%BGP-5-ADJCHANGE: neighbor 100.0.0.1 Up
```

This confirms that R1 successfully established an eBGP session with SPR1.

---

# 2. Verify the BGP Routes

After establishing the BGP relationship, R1 received routes from SPR1.

The routing table showed:

```text
B    15.0.0.0/8 [20/0] via 100.0.0.1
B    67.56.22.0/24 [20/0] via 100.0.0.1
B    111.0.0.0/24 [20/0] via 100.0.0.1
B    130.0.2.0/24 [20/0] via 100.0.0.1
B    220.0.0.0/24 [20/0] via 100.0.0.1
```

Most importantly, R1 learned the remote `15.0.0.0/8` network through SPR1:

```text
B    15.0.0.0/8 [20/0] via 100.0.0.1
```

The BGP administrative distance of an external BGP route is shown as `20`.

---

# 3. Advertise the Loopback Interfaces

The loopback addresses needed to be advertised through BGP.

The following network statements were configured on R1:

```text
R1(config-router)#network 1.1.1.1 mask 255.255.255.255
R1(config-router)#network 2.2.2.2 mask 255.255.255.255
R1(config-router)#network 3.3.3.3 mask 255.255.255.255
```

These statements advertise:

| Router | Loopback | Prefix |
|---|---|---|
| R1 | Loopback0 | `1.1.1.1/32` |
| R2 | Loopback0 | `2.2.2.2/32` |
| R3 | Loopback0 | `3.3.3.3/32` |

The routes for R2 and R3's loopbacks subsequently appeared in R1's routing table as BGP routes:

```text
B 2.2.2.2/32 [20/0] via 100.0.0.1
B 3.3.3.3/32 [20/0] via 100.0.0.1
```

This confirms that the loopback prefixes were being exchanged through BGP.

> **Note:** Because the `network` command requires the specified prefix to exist in the routing table, the loopback routes must already be present before they can be advertised.

---

# 4. Advertise the 10.0.0.0/16 Network

The lab specifically requires the following networks to be advertised:

```text
10.0.12.0/30
10.0.23.0/30
10.0.13.0/30
```

However, they must be advertised using **one BGP network statement with a `/16` mask**.

The command used was:

```text
R1(config-router)#network 10.0.0.0 mask 255.255.0.0
```

This advertises the aggregate:

```text
10.0.0.0/16
```

The individual `/30` networks fall within this `/16` address space:

```text
10.0.12.0/30
10.0.13.0/30
10.0.23.0/30
```

---

# 5. Create a Null0 Route

Initially, R1 did not have an exact `10.0.0.0/16` route in its routing table.

Because BGP's `network` command requires a matching route in the routing table, a static route to `Null0` was created.

```text
R1(config)#ip route 10.0.0.0 255.255.0.0 null0
```

This creates a summary route:

```text
S 10.0.0.0/16 is directly connected, Null0
```

The resulting routing table confirmed that the route was installed:

```text
S       10.0.0.0/16 is directly connected, Null0
```

This allowed R1 to successfully originate the `10.0.0.0/16` prefix into BGP. 
---

# 6. Why Null0 Is Used

The Null0 route is used as a **discard route** for traffic destined for addresses within `10.0.0.0/16` that do not have a more specific route.

The routing table contains more-specific routes such as:

```text
10.0.12.0/30
10.0.13.0/30
10.0.23.0/30
```

These more-specific routes take precedence over the `/16` Null0 route.

Therefore:

```text
10.0.12.0/30 → specific route
10.0.13.0/30 → specific route
10.0.23.0/30 → specific route
```

while unmatched addresses within the `/16` can be discarded by Null0.

This technique allows R1 to advertise the aggregate `10.0.0.0/16` with a single BGP `network` command.

---

# 7. Verify the Final Routing Table

After the Null0 route was added, R1's routing table contained:

```text
S       10.0.0.0/16 is directly connected, Null0
C       10.0.12.0/30 is directly connected, GigabitEthernet0/1
C       10.0.13.0/30 is directly connected, GigabitEthernet0/2
O       10.0.23.0/30 via 10.0.12.2
                     via 10.0.13.2
```

The remote BGP network was also present:

```text
B    15.0.0.0/8 [20/0] via 100.0.0.1
```

This confirmed that both the internal aggregate and external BGP routes were present.

---

# 8. Final Connectivity Test

After the BGP configuration was completed, the remote server was tested again from R2.

```text
R2#ping 15.0.0.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 15.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```

The final result was:

```text
Success rate is 100 percent (5/5)
```

This confirms successful end-to-end connectivity from R2 to the remote server.

---

# 🔧 Final R1 Configuration

The important BGP and routing configuration on R1 was:

```text
router bgp 65000
 neighbor 100.0.0.1 remote-as 65001
 network 1.1.1.1 mask 255.255.255.255
 network 2.2.2.2 mask 255.255.255.255
 network 3.3.3.3 mask 255.255.255.255
 network 10.0.0.0 mask 255.255.0.0

ip route 10.0.0.0 255.255.0.0 Null0
```

The configuration was saved using:

```text
R1(config-router)#do wr
```

---

# 🧰 Verification Commands

The following commands are useful when troubleshooting this lab:

### Check the routing table

```text
show ip route
```

### Check BGP summary

```text
show ip bgp summary
```

### View BGP routes

```text
show ip bgp
```

### View a specific BGP route

```text
show ip bgp 15.0.0.0
```

### Check BGP neighbors

```text
show ip bgp neighbors
```

### Verify the Null0 route

```text
show ip route 10.0.0.0
```

### Test connectivity

```text
ping 15.0.0.1
```

---

# ✅ Verification Checklist

| Requirement | Status |
|---|---|
| Initial ping to `15.0.0.1` tested | ✅ |
| eBGP configured between R1 and SPR1 | ✅ |
| R1 AS = `65000` | ✅ |
| SPR1 AS = `65001` | ✅ |
| BGP neighbor `100.0.0.1` established | ✅ |
| `1.1.1.1/32` advertised | ✅ |
| `2.2.2.2/32` advertised | ✅ |
| `3.3.3.3/32` advertised | ✅ |
| `10.0.0.0/16` advertised with one network command | ✅ |
| Null0 route configured | ✅ |
| `15.0.0.0/8` learned through BGP | ✅ |
| Final ping to `15.0.0.1` successful | ✅ |

---

# 🧠 Key Concepts Learned

## eBGP

eBGP is BGP peering between routers belonging to different autonomous systems.

In this lab:

```text
R1  → AS 65000
SPR1 → AS 65001
```

Therefore, their BGP relationship is an eBGP relationship.

## BGP Network Advertisement

A BGP `network` command tells BGP to originate a prefix, but the corresponding route must exist in the routing table.

For example:

```text
network 10.0.0.0 mask 255.255.0.0
```

requires a matching `10.0.0.0/16` route.

## Null0 Summary Route

The command:

```text
ip route 10.0.0.0 255.255.0.0 Null0
```

creates the required `/16` route without requiring an actual interface configured with that exact network.

## Longest Prefix Match

More-specific routes take precedence over the `/16` summary route.

For example:

```text
10.0.12.0/30
```

is preferred over:

```text
10.0.0.0/16
```

because `/30` is more specific than `/16`.

---

# 🏁 Lab Completion

The lab is successfully completed when:

- R1 has an established eBGP session with SPR1.
- The required loopback prefixes are advertised.
- The `10.0.0.0/16` aggregate is advertised using a single BGP `network` command.
- The `10.0.0.0/16` Null0 route exists in R1's routing table.
- R1 learns the remote `15.0.0.0/8` network from SPR1.
- R2/R3 can successfully reach the remote server at `15.0.0.1`.

The final successful ping demonstrates that the BGP configuration and routing changes restored end-to-end connectivity.
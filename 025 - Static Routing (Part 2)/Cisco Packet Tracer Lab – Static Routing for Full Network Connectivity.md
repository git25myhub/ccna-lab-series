# Static Routing for Full Network Connectivity

## Lab Overview

In this Cisco Packet Tracer lab, all IP addresses and interface configurations are **preconfigured**. The objective is to configure **static routes on each router** so that all networks throughout the topology can communicate with one another.

The network consists of four routers connected through multiple point-to-point networks, with each router also providing connectivity to a local LAN.

The lab is successfully completed when **every PC can ping every other PC and reachable network endpoint**.

---

## Objectives

By completing this lab, you will:

- Configure static routes on multiple Cisco routers.
- Identify directly connected and remote networks.
- Determine the correct next-hop address for each remote network.
- Verify static routes using the routing table.
- Test end-to-end connectivity using `ping`.
- Troubleshoot routing problems when connectivity fails.
- Save router configurations to NVRAM.

---

## Topology Addressing

The topology uses the following networks:

| Network | Purpose |
|---|---|
| `10.0.1.0/24` | R1 LAN |
| `10.0.2.0/24` | R2 LAN |
| `10.0.3.0/24` | R3 LAN |
| `10.0.4.0/24` | R4 LAN |
| `192.168.12.0/24` | R1–R2 link |
| `192.168.13.0/24` | R1–R3 link |
| `192.168.24.0/24` | R2–R4 link |
| `192.168.34.0/24` | R3–R4 link |

All IP addresses are already configured, so **do not modify the existing IP addressing**.

---

## Routing Concept

A router automatically knows about networks that are directly connected to its interfaces. However, it does not automatically know how to reach remote networks.

Static routes manually tell the router:

> "To reach this destination network, forward the packet to this next-hop router."

The general Cisco IOS syntax is:

```text
ip route <destination-network> <subnet-mask> <next-hop-ip>
```

For example:

```text
ip route 192.168.24.0 255.255.255.0 192.168.12.2
```

This tells R1 to send traffic destined for `192.168.24.0/24` to R2 at `192.168.12.2`.

---

# Configuration

## R1 Static Routes

R1 has the following networks directly connected:

- `10.0.1.0/24`
- `192.168.12.0/24`
- `192.168.13.0/24`

Therefore, R1 needs routes to the remaining remote networks.

```text
R1> enable
R1# configure terminal

R1(config)# ip route 192.168.24.0 255.255.255.0 192.168.12.2
R1(config)# ip route 192.168.34.0 255.255.255.0 192.168.13.3
R1(config)# ip route 10.0.2.0 255.255.255.0 192.168.12.2
R1(config)# ip route 10.0.3.0 255.255.255.0 192.168.13.3
R1(config)# ip route 10.0.4.0 255.255.255.0 192.168.12.2
```

Save the configuration:

```text
R1(config)# do write
```

### R1 Next-Hop Logic

| Destination | Next Hop | Reason |
|---|---|---|
| `10.0.2.0/24` | `192.168.12.2` | R2 |
| `10.0.3.0/24` | `192.168.13.3` | R3 |
| `10.0.4.0/24` | `192.168.12.2` | R2 → R4 |
| `192.168.24.0/24` | `192.168.12.2` | R2 |
| `192.168.34.0/24` | `192.168.13.3` | R3 |

---

## R2 Static Routes

R2 is directly connected to:

- `10.0.2.0/24`
- `192.168.12.0/24`
- `192.168.24.0/24`

Configure routes to the remaining networks:

```text
R2> enable
R2# configure terminal

R2(config)# ip route 192.168.13.0 255.255.255.0 192.168.12.1
R2(config)# ip route 192.168.34.0 255.255.255.0 192.168.24.4
R2(config)# ip route 10.0.1.0 255.255.255.0 192.168.12.1
R2(config)# ip route 10.0.3.0 255.255.255.0 192.168.12.1
R2(config)# ip route 10.0.4.0 255.255.255.0 192.168.24.4
```

Save the configuration:

```text
R2(config)# do write
```

### R2 Next-Hop Logic

| Destination | Next Hop | Reason |
|---|---|---|
| `10.0.1.0/24` | `192.168.12.1` | R1 |
| `10.0.3.0/24` | `192.168.12.1` | R1 → R3 |
| `10.0.4.0/24` | `192.168.24.4` | R4 |
| `192.168.13.0/24` | `192.168.12.1` | R1 |
| `192.168.34.0/24` | `192.168.24.4` | R4 |

---

## R3 Static Routes

R3 is directly connected to:

- `10.0.3.0/24`
- `192.168.13.0/24`
- `192.168.34.0/24`

Configure the following routes:

```text
R3> enable
R3# configure terminal

R3(config)# ip route 192.168.12.0 255.255.255.0 192.168.13.1
R3(config)# ip route 192.168.24.0 255.255.255.0 192.168.34.4
R3(config)# ip route 10.0.1.0 255.255.255.0 192.168.13.1
R3(config)# ip route 10.0.2.0 255.255.255.0 192.168.13.1
R3(config)# ip route 10.0.4.0 255.255.255.0 192.168.34.4
```

Save the configuration:

```text
R3(config)# do write
```

### R3 Next-Hop Logic

| Destination | Next Hop | Reason |
|---|---|---|
| `10.0.1.0/24` | `192.168.13.1` | R1 |
| `10.0.2.0/24` | `192.168.13.1` | R1 → R2 |
| `10.0.4.0/24` | `192.168.34.4` | R4 |
| `192.168.12.0/24` | `192.168.13.1` | R1 |
| `192.168.24.0/24` | `192.168.34.4` | R4 |

---

## R4 Static Routes

R4 is directly connected to:

- `10.0.4.0/24`
- `192.168.24.0/24`
- `192.168.34.0/24`

Configure routes to the remaining networks:

```text
R4> enable
R4# configure terminal

R4(config)# ip route 192.168.12.0 255.255.255.0 192.168.24.2
R4(config)# ip route 192.168.13.0 255.255.255.0 192.168.34.3
R4(config)# ip route 10.0.1.0 255.255.255.0 192.168.24.2
R4(config)# ip route 10.0.2.0 255.255.255.0 192.168.24.2
R4(config)# ip route 10.0.3.0 255.255.255.0 192.168.34.3
```

Save the configuration:

```text
R4(config)# do write
```

### R4 Next-Hop Logic

| Destination | Next Hop | Reason |
|---|---|---|
| `10.0.1.0/24` | `192.168.24.2` | R2 → R1 |
| `10.0.2.0/24` | `192.168.24.2` | R2 |
| `10.0.3.0/24` | `192.168.34.3` | R3 |
| `192.168.12.0/24` | `192.168.24.2` | R2 |
| `192.168.13.0/24` | `192.168.34.3` | R3 |

---

# Verification

After configuring all routers, verify their routing tables.

Use:

```text
show ip route
```

Static routes should appear with an **`S`** beside them.

For example, R1 should contain entries similar to:

```text
S 10.0.2.0 [1/0] via 192.168.12.2
S 10.0.3.0 [1/0] via 192.168.13.3
S 10.0.4.0 [1/0] via 192.168.12.2
S 192.168.24.0 [1/0] via 192.168.12.2
S 192.168.34.0 [1/0] via 192.168.13.3
```

You can also verify a specific route with:

```text
show ip route 10.0.4.0
```

---

# Connectivity Testing

From a PC, use the `ping` command to test connectivity to other LANs.

Example:

```text
C:\> ping 10.0.4.144
```

Expected successful output:

```text
Reply from 10.0.4.144: bytes=32 time=2ms TTL=125
Reply from 10.0.4.144: bytes=32 time=2ms TTL=125
Reply from 10.0.4.144: bytes=32 time=2ms TTL=125
```

Test another remote network:

```text
C:\> ping 10.0.2.122
```

The first ping may occasionally time out while ARP information is being resolved. Subsequent packets should succeed.

---

# Troubleshooting

If a ping fails, check the following in order.

### 1. Check the PC IP configuration

On the PC:

```text
ipconfig
```

Verify:

- IP address
- Subnet mask
- Default gateway

### 2. Check router interfaces

On each router:

```text
show ip interface brief
```

All required interfaces should show:

```text
Status: up
Protocol: up
```

### 3. Check the routing table

```text
show ip route
```

Look for the destination network. If it is missing, the router does not have a route to that network.

### 4. Verify the next-hop address

Make sure the next-hop address belongs to the directly connected neighboring router.

For example, R1 should use:

```text
192.168.12.2
```

to reach networks through R2.

### 5. Test the next-hop router

From R1:

```text
ping 192.168.12.2
```

From R3:

```text
ping 192.168.34.4
```

If the next hop cannot be reached, investigate the directly connected link before troubleshooting the static route.

### 6. Check the configured static routes

Use:

```text
show running-config
```

or:

```text
show ip route
```

Look for incorrect network addresses, subnet masks, or next-hop addresses.

---

# Important Cisco Commands

| Command | Purpose |
|---|---|
| `enable` | Enter privileged EXEC mode |
| `configure terminal` | Enter global configuration mode |
| `ip route` | Configure a static route |
| `show ip route` | Display the routing table |
| `show ip interface brief` | Check interface status and IP addresses |
| `show running-config` | Display the active configuration |
| `ping` | Test Layer 3 connectivity |
| `traceroute` | Identify the path packets take |
| `do write` | Save the configuration |

---

# Completion Criteria

The lab is considered **successfully completed** when:

- [x] All IP addresses remain correctly configured.
- [x] R1 has routes to all remote networks.
- [x] R2 has routes to all remote networks.
- [x] R3 has routes to all remote networks.
- [x] R4 has routes to all remote networks.
- [x] Static routes appear in the routing tables with the `S` code.
- [x] Router-to-router connectivity works.
- [x] PCs can reach remote LANs.
- [x] Each PC can ping any other PC in the topology.
- [x] Configurations have been saved using `write` or `copy running-config startup-config`.

---

# Key Learning Points

This lab demonstrates an important principle of Layer 3 networking:

> **Routers only know about directly connected networks unless routes to remote networks are provided.**

Static routing is simple and predictable, making it useful for small networks and learning environments. However, configuring many static routes becomes difficult to maintain as a network grows. Larger networks typically use dynamic routing protocols such as **OSPF, EIGRP, or BGP**.

The key skill practiced in this lab is learning to look at a topology, identify **remote networks**, determine the correct **next-hop router**, and install the appropriate static route.
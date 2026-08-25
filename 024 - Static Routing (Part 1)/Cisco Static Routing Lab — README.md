# Cisco Static Routing Lab

## Lab Overview

In this lab, you will configure two Cisco routers connected through an inter-router network. You will first investigate connectivity between the two LANs, identify which destinations are reachable before routing is configured, and then configure **static routes** to enable communication between PC1 and PC2.

The key objective is to understand that routers know about their **directly connected networks**, but they do not automatically know how to reach remote networks.

---

## Topology

The lab consists of:

- **PC1** — Located on the `192.168.1.0/24` LAN
- **R1** — Connects PC1's LAN to R2
- **R2** — Connects R1 to PC2's LAN
- **PC2** — Located on the `192.168.2.0/24` LAN

### IP Addressing

| Device | Interface | IP Address | Subnet Mask | Network |
|---|---|---|---|---|
| R1 | G0/0 | `10.0.0.1` | `255.255.255.0` | `10.0.0.0/24` |
| R1 | G0/1 | `192.168.1.1` | `255.255.255.0` | `192.168.1.0/24` |
| R2 | G0/0 | `10.0.0.2` | `255.255.255.0` | `10.0.0.0/24` |
| R2 | G0/1 | `192.168.2.1` | `255.255.255.0` | `192.168.2.0/24` |

The PC addressing should correspond to the LANs shown in the network diagram.

---

## Objectives

By completing this lab, you will learn how to:

- Configure router Gigabit Ethernet interfaces.
- Enable router interfaces using `no shutdown`.
- Test connectivity using `ping`.
- Identify directly connected networks.
- Understand why remote networks are initially unreachable.
- Configure static routes.
- Route traffic between two remote LANs.
- Use `ping` and `tracert` to verify end-to-end connectivity.
- Interpret the routing table using `show ip route`.

---

# Part 1 — Configure Router Interfaces

Configure R1 according to the topology.

### R1

```cisco
enable
configure terminal

interface g0/0
 ip address 10.0.0.1 255.255.255.0
 no shutdown
exit

interface g0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

end
write memory
```

Configure R2:

### R2

```cisco
enable
configure terminal

interface g0/0
 ip address 10.0.0.2 255.255.255.0
 no shutdown
exit

interface g0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

end
write memory
```

---

## Verify Interface Status

On both routers, run:

```cisco
show ip interface brief
```

The interfaces should show:

```text
Status: up
Protocol: up
```

For R1:

```text
GigabitEthernet0/0    10.0.0.1       up    up
GigabitEthernet0/1    192.168.1.1    up    up
```

For R2:

```text
GigabitEthernet0/0    10.0.0.2       up    up
GigabitEthernet0/1    192.168.2.1    up    up
```

---

# Part 2 — Test Connectivity from PC1

From PC1, progressively test connectivity toward PC2.

Test R1's G0/1:

```text
ping 192.168.1.1
```

Test R1's G0/0:

```text
ping 10.0.0.1
```

Test R2's G0/0:

```text
ping 10.0.0.2
```

Test R2's G0/1:

```text
ping 192.168.2.1
```

Finally, test PC2:

```text
ping <PC2-IP>
```

---

## Expected Results Before Static Routing

PC1 should be able to reach:

- R1 G0/1 — `192.168.1.1`
- R1 G0/0 — `10.0.0.1`
- R2 G0/0 — `10.0.0.2`

However, communication with the remote LAN will initially fail:

- R2 G0/1 — `192.168.2.1`
- PC2

### Why?

R1 knows about these networks because they are directly connected:

```text
192.168.1.0/24
10.0.0.0/24
```

R1 does **not** initially know how to reach:

```text
192.168.2.0/24
```

Therefore, when R1 receives traffic destined for PC2's network, it does not have a route for that destination.

---

# Part 3 — Test Connectivity from PC2

Perform the same progressive test from PC2.

First ping R2's G0/1:

```text
ping 192.168.2.1
```

Then ping R2's G0/0:

```text
ping 10.0.0.2
```

Then ping R1's G0/0:

```text
ping 10.0.0.1
```

Then ping R1's G0/1:

```text
ping 192.168.1.1
```

Finally, ping PC1:

```text
ping <PC1-IP>
```

Before static routing is configured, PC2 should be unable to reach the remote `192.168.1.0/24` network.

---

# Part 4 — Examine the Routing Tables

On R1:

```cisco
show ip route
```

R1 should have directly connected routes for:

```text
10.0.0.0/24
192.168.1.0/24
```

On R2:

```cisco
show ip route
```

R2 should have directly connected routes for:

```text
10.0.0.0/24
192.168.2.0/24
```

Neither router automatically learns the LAN on the opposite side.

This is the problem that the static routes will solve.

---

# Part 5 — Configure Static Routes

The instructions specifically require routes to the **subnets**, not individual PCs.

Therefore:

- R1 needs a route to `192.168.2.0/24`.
- R2 needs a route to `192.168.1.0/24`.

## R1 Static Route

R1 must forward traffic for the `192.168.2.0/24` network to R2.

R2's address on the transit network is `10.0.0.2`.

Configure:

```cisco
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

Save the configuration:

```cisco
R1(config)# end
R1# write memory
```

---

## R2 Static Route

R2 must forward traffic for the `192.168.1.0/24` network to R1.

R1's address on the transit network is `10.0.0.1`.

Configure:

```cisco
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

Save the configuration:

```cisco
R2(config)# end
R2# write memory
```

---

# Verify the Static Routes

On R1:

```cisco
show ip route
```

You should see:

```text
S    192.168.2.0/24 [1/0] via 10.0.0.2
```

The `S` indicates a **static route**.

On R2:

```cisco
show ip route
```

You should see:

```text
S    192.168.1.0/24 [1/0] via 10.0.0.1
```

---

# Part 6 — Test End-to-End Connectivity

After configuring both static routes, test from PC1:

```text
ping <PC2-IP>
```

The ping should now succeed.

Test from PC2:

```text
ping <PC1-IP>
```

This should also succeed.

You can also test the router interfaces again.

From PC1:

```text
ping 192.168.1.1
ping 10.0.0.1
ping 10.0.0.2
ping 192.168.2.1
ping <PC2-IP>
```

All destinations should now be reachable.

---

# Part 7 — Use Tracert

The `tracert` command can be used to see the path traffic takes through the network.

From PC1:

```text
tracert <PC2-IP>
```

The path should travel approximately:

```text
PC1
  |
  v
R1 - 192.168.1.1
  |
  v
R2 - 10.0.0.2
  |
  v
PC2
```

From PC2:

```text
tracert <PC1-IP>
```

The reverse path should travel through R2 and then R1.

---

# Important Routing Concept

This lab demonstrates a fundamental routing principle:

> A router can only forward packets to a remote network if it has a route to that network.

R1 initially knows:

```text
192.168.1.0/24
10.0.0.0/24
```

R2 initially knows:

```text
192.168.2.0/24
10.0.0.0/24
```

After static routing:

```text
R1 ---> 192.168.2.0/24 via 10.0.0.2
R2 ---> 192.168.1.0/24 via 10.0.0.1
```

This creates a complete routing path between the two LANs.

---

# Troubleshooting

If the final PC-to-PC ping fails, check the following.

### 1. Verify interfaces

```cisco
show ip interface brief
```

All required interfaces should be `up/up`.

### 2. Verify R1's routing table

```cisco
show ip route
```

Look for:

```text
S    192.168.2.0/24 via 10.0.0.2
```

### 3. Verify R2's routing table

```cisco
show ip route
```

Look for:

```text
S    192.168.1.0/24 via 10.0.0.1
```

### 4. Test the router-to-router connection

From R1:

```cisco
ping 10.0.0.2
```

From R2:

```cisco
ping 10.0.0.1
```

Both should succeed.

### 5. Check PC default gateways

PC1 should use:

```text
Default Gateway: 192.168.1.1
```

PC2 should use:

```text
Default Gateway: 192.168.2.1
```

Without the correct default gateway, PCs cannot send traffic destined for remote networks to their routers.

---

# Key Commands

| Purpose | Command |
|---|---|
| Check interfaces | `show ip interface brief` |
| View routing table | `show ip route` |
| Test connectivity | `ping <IP>` |
| Configure static route | `ip route <network> <mask> <next-hop>` |
| Save configuration | `write memory` |
| Trace packet path | `tracert <IP>` |

---

# Lab Completion Checklist

- [ ] Configure R1 G0/0 as `10.0.0.1/24`.
- [ ] Configure R1 G0/1 as `192.168.1.1/24`.
- [ ] Enable both R1 interfaces.
- [ ] Configure R2 G0/0 as `10.0.0.2/24`.
- [ ] Configure R2 G0/1 as `192.168.2.1/24`.
- [ ] Enable both R2 interfaces.
- [ ] Test progressive connectivity from PC1.
- [ ] Test progressive connectivity from PC2.
- [ ] Identify which destinations initially fail.
- [ ] Configure R1's route to `192.168.2.0/24`.
- [ ] Configure R2's route to `192.168.1.0/24`.
- [ ] Verify the static routes with `show ip route`.
- [ ] Successfully ping PC2 from PC1.
- [ ] Successfully ping PC1 from PC2.
- [ ] Use `tracert` to verify the path between the PCs.
- [ ] Save the router configurations.

---

## Skills Practiced

This lab reinforces the following CCNA concepts:

- IPv4 addressing
- Router interface configuration
- Interface activation
- Connected routes
- Static routing
- Next-hop addresses
- Routing table interpretation
- Default gateways
- ICMP troubleshooting
- `ping` testing
- `tracert` path analysis

---

## Final Result

The lab is successfully completed when **PC1 and PC2 can communicate in both directions**.

The final routing design should contain:

```text
PC1 LAN
192.168.1.0/24
      |
      |
    R1
192.168.1.1
10.0.0.1
      |
      | 10.0.0.0/24
      |
10.0.0.2
    R2
192.168.2.1
      |
      |
PC2 LAN
192.168.2.0/24
```

R1 reaches the PC2 subnet through `10.0.0.2`, while R2 reaches the PC1 subnet through `10.0.0.1`.
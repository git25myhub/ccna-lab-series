# Cisco RIP Version 1 and Version 2 Routing Lab

## Objective

The objective of this lab is to configure and observe **RIP (Routing Information Protocol)** using its default behavior (RIP Version 1), then upgrade the configuration to **RIP Version 2** with **automatic summarization disabled**.

This lab demonstrates how RIP handles classful and classless network advertisements and how enabling RIPv2 with `no auto-summary` changes the routes learned by the routers.

---

## Lab Tasks

### Step 1: Configure RIP Version 1

Configure RIP on **R1 and R2**.

> Do **not** manually configure `version 2` at this stage. Cisco IOS uses RIP Version 1 by default.

Advertise all networks connected to the interfaces of each router.

### R1

```cisco
R1#configure terminal
R1(config)#router rip
R1(config-router)#network 192.168.1.0
```

Save the configuration:

```cisco
R1(config-router)#do write
```

### R2

Configure RIP and advertise the networks connected to R2:

```cisco
R2#configure terminal
R2(config)#router rip
R2(config-router)#network 192.168.1.0
```

Save the configuration:

```cisco
R2(config-router)#do write
```

> **Important:** Make sure every network connected to each router is included in the RIP configuration according to the addressing shown in the Packet Tracer topology.

---

## Step 2: Examine R1's Routing Table

Allow RIP some time to exchange routing information and converge.

On R1, use:

```cisco
R1#show ip route
```

The routing table displays connected routes using `C` and RIP-learned routes using `R`.

### Observation

In the provided output, R1 initially shows:

```text
C    192.168.1.0/24 is directly connected, Serial2/0
```

No RIP-learned route is visible in the captured output.

This indicates that, at the time the routing table was checked, R1 had only its directly connected `192.168.1.0/24` network and had not yet learned a remote network through RIP.

### Question

**What route has R1 learned?**

Based strictly on the provided output:

**R1 has not learned a RIP route yet.** The only displayed route is the directly connected:

```text
192.168.1.0/24
```

If the lab topology contains additional networks behind R2, they should appear with an `R` code after RIP successfully converges and the correct networks are advertised.

---

# Step 3: Enable RIP Version 2

After completing the RIPv1 observation, upgrade RIP on **both R1 and R2** to Version 2.

RIPv2 supports classless routing information and includes the subnet mask in routing updates.

### R1

```cisco
R1#configure terminal
R1(config)#router rip
R1(config-router)#version 2
R1(config-router)#no auto-summary
```

Save the configuration:

```cisco
R1(config-router)#do write
```

### R2

```cisco
R2#configure terminal
R2(config)#router rip
R2(config-router)#version 2
R2(config-router)#no auto-summary
```

Save the configuration:

```cisco
R2(config-router)#do write
```

---

## Why Use `no auto-summary`?

By default, older RIP behavior can automatically summarize routes at major network boundaries.

The command:

```cisco
no auto-summary
```

prevents this behavior.

This is particularly important when the topology uses subnetted networks or when different subnets of the same major network exist on different interfaces.

With RIPv2 and `no auto-summary`, routers can advertise the **specific subnet and mask** rather than relying on classful summarization.

---

# Step 4: Examine R1's Routing Table Again

Allow RIPv2 time to converge.

Then check R1:

```cisco
R1#show ip route
```

Look for entries beginning with:

```text
R
```

These entries represent routes learned through RIP.

You can also view only RIP routes with:

```cisco
R1#show ip route rip
```

---

## Expected Result

After RIPv2 has successfully converged, R1 should learn the remote networks advertised by R2.

The routing table should contain:

- `C` routes for networks directly connected to R1.
- `R` routes for remote networks learned from R2.

The exact RIP routes depend on the network addresses configured in the Packet Tracer topology.

---

# Verification Commands

Use the following commands to verify the configuration and operation of RIP.

### Display the routing table

```cisco
show ip route
```

### Display only RIP routes

```cisco
show ip route rip
```

### Display the RIP configuration

```cisco
show running-config | section router rip
```

### Display RIP protocol information

```cisco
show ip protocols
```

### Test connectivity

```cisco
ping <remote-ip-address>
```

---

# Important RIP Concepts

| Feature | RIP Version 1 | RIP Version 2 |
|---|---|---|
| Routing protocol | Distance vector | Distance vector |
| Maximum hop count | 15 | 15 |
| Metric | Hop count | Hop count |
| Multicast updates | No | Yes, 224.0.0.9 |
| Supports VLSM | No | Yes |
| Supports CIDR | No | Yes |
| Automatic summarization | Yes | Yes, unless disabled |
| `no auto-summary` | Not applicable to this lab | Recommended |

---

# Troubleshooting

If R1 does not learn the expected routes, check the following:

### 1. Verify RIP is configured

```cisco
show running-config | section router rip
```

Confirm that both routers have:

```text
router rip
```

and, after Step 3:

```text
version 2
no auto-summary
```

### 2. Verify advertised networks

Check that every required connected network is included under RIP:

```cisco
router rip
network <network-address>
```

### 3. Verify interfaces

Use:

```cisco
show ip interface brief
```

All required interfaces should be **up/up**.

### 4. Verify RIP operation

Use:

```cisco
show ip protocols
```

This helps confirm the RIP version and networks being advertised.

### 5. Check the routing table

```cisco
show ip route
```

Look specifically for routes marked with:

```text
R
```

---

# Lab Questions

### Question 1

What routing protocol was configured?

**Answer:** RIP.

### Question 2

Which RIP version is used before explicitly configuring `version 2`?

**Answer:** RIP Version 1.

### Question 3

What does the `R` code represent in the routing table?

**Answer:** A route learned through RIP.

### Question 4

What does `version 2` do?

**Answer:** It enables RIP Version 2, which supports classless routing and includes subnet-mask information in routing updates.

### Question 5

What does `no auto-summary` do?

**Answer:** It disables automatic classful route summarization, allowing specific subnet routes to be advertised.

### Question 6

Why might R1 initially show no RIP-learned routes?

**Answer:** RIP may not have converged yet, or the required networks may not have been correctly advertised/configured on both routers.

---

# Final Configuration Summary

### R1

```cisco
router rip
 network 192.168.1.0
 version 2
 no auto-summary
```

### R2

```cisco
router rip
 network 192.168.1.0
 version 2
 no auto-summary
```

> Add the other directly connected networks from the topology to the appropriate router's RIP configuration.

---

# Key Takeaways

- RIP Version 1 is the default RIP version when `version 2` is not configured.
- RIP uses **hop count** as its routing metric.
- A maximum of **15 hops** is supported; 16 is considered unreachable.
- RIPv1 is classful and does not support VLSM/CIDR.
- RIPv2 supports classless routing.
- `no auto-summary` prevents automatic summarization at major network boundaries.
- `show ip route` is used to identify learned routes.
- Routes marked with **`R`** are learned through RIP.
- `show ip protocols` is useful for verifying RIP version and advertised networks.
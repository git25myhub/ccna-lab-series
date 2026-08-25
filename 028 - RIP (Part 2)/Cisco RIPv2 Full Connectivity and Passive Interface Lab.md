# Cisco RIPv2 Full Connectivity and Passive Interface Lab

## Objective

The network has been preconfigured according to the provided topology.

The objective of this lab is to:

1. Configure **RIPv2** on all routers.
2. Advertise all directly connected networks.
3. Disable automatic route summarization.
4. Disable RIP routing updates on interfaces connected to switches using **passive interfaces**.
5. Allow full connectivity between all networks in the topology.
6. Verify the learned RIP routes and end-to-end connectivity.

---

# Network Overview

The topology consists of four routers:

- **R1**
- **R2**
- **R3**
- **R4**

The routers are connected using the following networks:

| Network | Purpose |
|---|---|
| `10.0.0.0/24` | R1 LAN |
| `20.0.0.0/24` | R2 LAN |
| `30.0.0.0/24` | R3 LAN |
| `40.0.0.0/24` | R4 LAN |
| `12.0.0.0/24` | R1–R2 connection |
| `13.0.0.0/24` | R1–R3 connection |
| `24.0.0.0/24` | R2–R4 connection |
| `34.0.0.0/24` | R3–R4 connection |

The interfaces connected to switches are the LAN-facing interfaces, and these should be configured as **passive interfaces**.

---

# Step 1: Configure RIPv2 on R1

Enter RIP configuration mode:

```cisco
R1#configure terminal
R1(config)#router rip
```

Enable RIPv2:

```cisco
R1(config-router)#version 2
```

Disable automatic summarization:

```cisco
R1(config-router)#no auto-summary
```

Advertise R1's connected networks:

```cisco
R1(config-router)#network 10.0.0.0
R1(config-router)#network 12.0.0.0
R1(config-router)#network 13.0.0.0
```

The LAN interface connected to the switch should not send RIP updates:

```cisco
R1(config-router)#passive-interface gigabitEthernet 0/2
```

Save the configuration:

```cisco
R1(config-router)#do write
```

### R1 Final RIP Configuration

```cisco
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 12.0.0.0
 network 13.0.0.0
 passive-interface GigabitEthernet0/2
```

---

# Step 2: Configure RIPv2 on R2

Enter RIP configuration mode:

```cisco
R2#configure terminal
R2(config)#router rip
```

Configure RIPv2:

```cisco
R2(config-router)#version 2
R2(config-router)#no auto-summary
```

Advertise R2's networks:

```cisco
R2(config-router)#network 20.0.0.0
R2(config-router)#network 12.0.0.0
R2(config-router)#network 24.0.0.0
```

Make the LAN-facing interface passive:

```cisco
R2(config-router)#passive-interface gigabitEthernet 0/2
```

Save:

```cisco
R2(config-router)#do write
```

### R2 Final RIP Configuration

```cisco
router rip
 version 2
 no auto-summary
 network 20.0.0.0
 network 12.0.0.0
 network 24.0.0.0
 passive-interface GigabitEthernet0/2
```

---

# Step 3: Configure RIPv2 on R3

Enter RIP configuration mode:

```cisco
R3#configure terminal
R3(config)#router rip
```

Configure RIPv2:

```cisco
R3(config-router)#version 2
R3(config-router)#no auto-summary
```

Advertise R3's networks:

```cisco
R3(config-router)#network 30.0.0.0
R3(config-router)#network 13.0.0.0
R3(config-router)#network 34.0.0.0
```

Make the LAN-facing interface passive:

```cisco
R3(config-router)#passive-interface gigabitEthernet 0/2
```

Save:

```cisco
R3(config-router)#do write
```

### R3 Final RIP Configuration

```cisco
router rip
 version 2
 no auto-summary
 network 30.0.0.0
 network 13.0.0.0
 network 34.0.0.0
 passive-interface GigabitEthernet0/2
```

---

# Step 4: Configure RIPv2 on R4

Enter RIP configuration mode:

```cisco
R4#configure terminal
R4(config)#router rip
```

Configure RIPv2:

```cisco
R4(config-router)#version 2
R4(config-router)#no auto-summary
```

Advertise R4's networks:

```cisco
R4(config-router)#network 40.0.0.0
R4(config-router)#network 24.0.0.0
R4(config-router)#network 34.0.0.0
```

Make the LAN-facing interface passive:

```cisco
R4(config-router)#passive-interface gigabitEthernet 0/2
```

Save:

```cisco
R4(config-router)#do write
```

### R4 Final RIP Configuration

```cisco
router rip
 version 2
 no auto-summary
 network 40.0.0.0
 network 24.0.0.0
 network 34.0.0.0
 passive-interface GigabitEthernet0/2
```

---

# Why Use Passive Interfaces?

A passive interface prevents RIP routing updates from being **sent out** of that interface.

The interface still advertises its connected network through RIP to the other routers.

For example, on R4:

```cisco
passive-interface GigabitEthernet0/2
```

prevents R4 from sending RIP updates toward the LAN/switch while still allowing the `40.0.0.0/24` network to be advertised to neighboring routers through the active RIP interfaces.

### Interfaces in This Lab

| Router | LAN Interface | Passive Interface |
|---|---|---|
| R1 | G0/2 | G0/2 |
| R2 | G0/2 | G0/2 |
| R3 | G0/2 | G0/2 |
| R4 | G0/2 | G0/2 |

---

# Step 5: Verify RIPv2

Use the following command on each router:

```cisco
show ip protocols
```

A correctly configured router should display:

```text
Routing Protocol is "rip"
Default version control: send version 2, receive 2
Automatic network summarization is not in effect
```

It should also list the configured networks and the passive interface.

For example, R4 produced:

```text
Routing Protocol is "rip"
Default version control: send version 2, receive 2
Automatic network summarization is not in effect

Routing for Networks:
    24.0.0.0
    34.0.0.0
    40.0.0.0

Passive Interface(s):
    GigabitEthernet0/2
```

This confirms that R4 is correctly running RIPv2 with automatic summarization disabled.

---

# Step 6: Verify the Routing Table

Use:

```cisco
show ip route
```

or:

```cisco
show ip route rip
```

RIP-learned routes are identified by the letter:

```text
R
```

For example, R4 learned:

```text
R 10.0.0.0/24
R 12.0.0.0/24
R 13.0.0.0/24
R 20.0.0.0/24
R 30.0.0.0/24
```

The routing table showed:

```text
R       10.0.0.0/24 [120/2] via 34.0.0.3
                    [120/2] via 24.0.0.2

R       12.0.0.0/24 [120/1] via 24.0.0.2

R       13.0.0.0/24 [120/1] via 34.0.0.3

R       20.0.0.0/24 [120/1] via 24.0.0.2

R       30.0.0.0/24 [120/1] via 34.0.0.3
```

This demonstrates successful RIP convergence.

---

# Understanding the R4 Routing Table

The route:

```text
R 10.0.0.0/24 [120/2]
```

means:

- `R` = learned through RIP
- `120` = RIP administrative distance
- `2` = RIP metric/hop count

R4 has two possible paths to `10.0.0.0/24`:

```text
R4 → R2 → R1
```

and:

```text
R4 → R3 → R1
```

Both paths have a metric of **2**, so RIP installs both equal-cost paths.

This provides a degree of redundancy in the topology.

---

# Step 7: Test End-to-End Connectivity

From the PC connected to the R4 LAN, test the remote networks.

### Test R2's LAN

```text
C:\>ping 20.0.0.10
```

Observed result:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The first packet timed out, but subsequent packets succeeded.

### Test R3's LAN

```text
C:\>ping 30.0.0.10
```

Observed:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

Again, the first packet timed out and the remaining packets succeeded.

### Test R4's LAN

```text
C:\>ping 40.0.0.10
```

Observed:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The first packet timed out while subsequent packets succeeded.

---

# Why Did the First Ping Time Out?

The initial ping timeout is normal in Packet Tracer environments.

The first packet may be lost while the devices perform **ARP resolution** to discover the Layer 2 MAC address associated with the destination or next-hop IP address.

Once ARP information is learned, subsequent packets can successfully reach the destination.

Therefore, the observed results:

```text
Request timed out.
Reply from ...
Reply from ...
Reply from ...
```

indicate that connectivity is working.

---

# Useful Verification Commands

## Check interfaces

```cisco
show ip interface brief
```

All required interfaces should normally show:

```text
Status       Protocol
up           up
```

## Check routing table

```cisco
show ip route
```

## Show RIP-learned routes

```cisco
show ip route rip
```

## Verify RIP configuration

```cisco
show ip protocols
```

## Display the running configuration

```cisco
show running-config
```

## Test connectivity

```cisco
ping <destination-ip>
```

## Trace the path

```cisco
traceroute <destination-ip>
```

---

# Troubleshooting

If full connectivity is not achieved, check the following.

### 1. Verify RIPv2 is enabled

```cisco
show ip protocols
```

Confirm:

```text
Default version control: send version 2, receive 2
```

### 2. Verify automatic summarization is disabled

The output should show:

```text
Automatic network summarization is not in effect
```

If necessary:

```cisco
router rip
no auto-summary
```

### 3. Verify all networks are advertised

Each router must advertise all of its directly connected networks.

For example, R1 should advertise:

```text
10.0.0.0
12.0.0.0
13.0.0.0
```

### 4. Verify passive interfaces

Use:

```cisco
show ip protocols
```

The LAN interface should appear under:

```text
Passive Interface(s):
```

### 5. Verify RIP neighbors/routes

Check:

```cisco
show ip route
```

Look for routes beginning with:

```text
R
```

### 6. Verify physical interfaces

Use:

```cisco
show ip interface brief
```

Check that the required router-to-router interfaces are `up/up`.

---

# Final Verification Checklist

- [x] RIPv2 configured on R1.
- [x] RIPv2 configured on R2.
- [x] RIPv2 configured on R3.
- [x] RIPv2 configured on R4.
- [x] `no auto-summary` configured on all routers.
- [x] All directly connected networks advertised.
- [x] LAN interfaces configured as passive interfaces.
- [x] RIP-learned routes appear in the routing tables.
- [x] R4 has learned remote networks through both R2 and R3 where applicable.
- [x] Remote LAN connectivity verified with ping.
- [x] Full network convergence achieved.

---

# Key Takeaways

1. **RIPv2** provides dynamic routing between the four routers.
2. Every router must advertise its directly connected networks.
3. `no auto-summary` allows the routers to maintain specific subnet information.
4. `passive-interface` prevents unnecessary RIP updates from being sent toward end devices/switches.
5. RIP routes appear in the routing table with the **`R`** code.
6. RIP has an administrative distance of **120**.
7. RIP uses **hop count** as its metric.
8. Equal-cost RIP paths can provide redundancy, as demonstrated by R4 having two paths toward `10.0.0.0/24`.
9. Successful pings after an initial timeout confirm end-to-end connectivity.

## Lab Completion

The lab is successfully completed when RIPv2 is configured on all four routers, RIP updates are disabled on switch-facing interfaces, the routers have learned the required remote networks, and PCs across the topology can communicate successfully.
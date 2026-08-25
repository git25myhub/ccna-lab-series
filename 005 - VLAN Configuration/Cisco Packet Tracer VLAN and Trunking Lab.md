# VLAN and Trunking Lab

## Lab Objective

The objective of this lab is to understand how **VLANs** separate broadcast domains and how **trunk links** allow multiple VLANs to travel between switches.

In this lab:

- PC1 and PC3 are assigned to **VLAN 1**.
- PC2 and PC4 are assigned to **VLAN 2**.
- The link between SW1 and SW2 is initially an access/default link.
- The link between SW1 and SW2 is then configured as a **trunk**.
- Connectivity is tested before and after trunk configuration.

---

## Network Topology

```text
        PC1                         PC3
     10.0.0.1                    10.0.0.3
       VLAN 1                      VLAN 1
         |                           |
       SW1=========================SW2
         |       Trunk Link          |
         |                           |
     PC2                         PC4
   10.0.0.2                    10.0.0.4
     VLAN 2                      VLAN 2
```

### VLAN Assignment

| Device | IP Address | VLAN | Switch Port |
|---|---|---:|---|
| PC1 | 10.0.0.1 | VLAN 1 | SW1 Fa0/2 |
| PC2 | 10.0.0.2 | VLAN 2 | SW1 Fa0/3 |
| PC3 | 10.0.0.3 | VLAN 1 | SW2 Fa0/2 |
| PC4 | 10.0.0.4 | VLAN 2 | SW2 Fa0/3 |
| SW1 ↔ SW2 | — | Trunk | Fa0/1 |

> **Note:** The exact PC-to-port mapping is based on the configurations provided in the lab output.

---

# Part 1 — Test Initial Connectivity

Before making VLAN changes, ping between the PCs to verify basic connectivity.

Example:

```text
C:\>ping 10.0.0.2
```

The initial tests showed successful connectivity, with replies received from the destination PCs.

This establishes that the PCs are physically connected and initially able to communicate.

---

# Part 2 — Assign PCs to VLANs

## SW1 Configuration

PC1 is placed in VLAN 1 using the default VLAN configuration, while PC2 is assigned to VLAN 2.

```cisco
SW1# configure terminal

SW1(config)# interface fastethernet 0/2
SW1(config-if)# switchport mode access

SW1(config-if)# interface fastethernet 0/3
SW1(config-if)# switchport access vlan 2
SW1(config-if)# switchport mode access

SW1(config-if)# end
SW1# write memory
```

### SW1 VLAN Assignment

```text
Fa0/2 → VLAN 1 → PC1
Fa0/3 → VLAN 2 → PC2
```

---

## SW2 Configuration

PC3 is assigned to VLAN 1 and PC4 is assigned to VLAN 2.

```cisco
SW2# configure terminal

SW2(config)# interface fastethernet 0/2
SW2(config-if)# switchport mode access
```

Fa0/2 remains in VLAN 1.

PC4 is assigned to VLAN 2:

```cisco
SW2(config-if)# interface fastethernet 0/3
SW2(config-if)# switchport access vlan 2
SW2(config-if)# switchport mode access

SW2(config-if)# end
SW2# write memory
```

### SW2 VLAN Assignment

```text
Fa0/2 → VLAN 1 → PC3
Fa0/3 → VLAN 2 → PC4
```

---

# Part 3 — Test Connectivity Before Trunking

After assigning the PCs to VLANs, test communication between PCs belonging to the same VLAN.

### PC1 → PC3

PC1 and PC3 are both members of VLAN 1.

```text
PC1 → PC3
10.0.0.1 → 10.0.0.3
```

The ping succeeds because both PCs belong to VLAN 1.

### PC2 → PC4

PC2 and PC4 are both members of VLAN 2.

```text
PC2 → PC4
10.0.0.2 → 10.0.0.4
```

The ping fails at this stage.

## Why?

The problem is the connection between **SW1 and SW2**.

Although PC2 and PC4 are both in VLAN 2, VLAN 2 traffic cannot properly cross the inter-switch link while that link is operating as an access/non-trunk connection.

VLAN 1 works because VLAN 1 is the default/native VLAN on the switches. VLAN 2 requires the inter-switch connection to carry VLAN 2 traffic.

Therefore:

```text
PC1 ─ VLAN 1 ─ SW1 ─ SW2 ─ VLAN 1 ─ PC3
                         ✓

PC2 ─ VLAN 2 ─ SW1 ─ SW2 ─ VLAN 2 ─ PC4
                         ✗
```

---

# Part 4 — Configure the SW1/SW2 Link as a Trunk

The interfaces connecting SW1 and SW2 are **FastEthernet0/1**.

## SW1

```cisco
SW1# configure terminal

SW1(config)# interface fastethernet 0/1
SW1(config-if)# switchport mode trunk

SW1(config-if)# end
SW1# write memory
```

The resulting configuration is:

```cisco
interface FastEthernet0/1
 switchport mode trunk
```

---

## SW2

```cisco
SW2# configure terminal

SW2(config)# interface fastethernet 0/1
SW2(config-if)# switchport mode trunk

SW2(config-if)# end
SW2# write memory
```

The resulting configuration is:

```cisco
interface FastEthernet0/1
 switchport mode trunk
```

The trunk allows VLAN traffic to cross between the two switches.

---

# Important Spanning Tree Message

Before SW2's Fa0/1 was configured as a trunk, the switch produced:

```text
%SPANTREE-2-RECV_PVID_ERR:
Received 802.1Q BPDU on non trunk FastEthernet0/1 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL:
Blocking FastEthernet0/1 on VLAN0001. Inconsistent port type.
```

This message indicates a **trunk mismatch**.

SW1 was sending 802.1Q trunk traffic while SW2's Fa0/1 was still operating as a non-trunk interface.

After configuring SW2 Fa0/1 as a trunk, the mismatch was resolved.

---

# Part 5 — Test Connectivity After Trunking

After configuring Fa0/1 on both switches as trunk interfaces, test the PCs again.

## PC1 → PC3

```text
10.0.0.1 → 10.0.0.3
```

**Result: SUCCESS**

Both PCs are in VLAN 1.

---

## PC2 → PC4

```text
10.0.0.2 → 10.0.0.4
```

**Result: SUCCESS**

Both PCs are in VLAN 2, and VLAN 2 can now cross the trunk link between SW1 and SW2.

---

## Cross-VLAN Tests

PCs in different VLANs should still be unable to communicate:

```text
PC1 → PC2
VLAN 1 → VLAN 2
FAIL

PC1 → PC4
VLAN 1 → VLAN 2
FAIL

PC3 → PC2
VLAN 1 → VLAN 2
FAIL

PC3 → PC4
VLAN 1 → VLAN 2
FAIL
```

This is expected because there is **no router or Layer 3 switch configured to perform inter-VLAN routing**.

---

# Final Connectivity Results

| Source | Destination | VLANs | Expected Result |
|---|---|---|---|
| PC1 | PC3 | VLAN 1 → VLAN 1 | **SUCCESS** |
| PC2 | PC4 | VLAN 2 → VLAN 2 | **SUCCESS after trunking** |
| PC1 | PC2 | VLAN 1 → VLAN 2 | **FAIL** |
| PC1 | PC4 | VLAN 1 → VLAN 2 | **FAIL** |
| PC3 | PC2 | VLAN 1 → VLAN 2 | **FAIL** |
| PC3 | PC4 | VLAN 1 → VLAN 2 | **FAIL** |

---

# Verification Commands

Use the following commands to verify the configuration.

### View VLAN Membership

```cisco
SW1# show vlan brief
SW2# show vlan brief
```

Expected result:

```text
VLAN 1:
PC1 / PC3

VLAN 2:
PC2 / PC4
```

### Verify Trunk Interfaces

```cisco
SW1# show interfaces trunk
SW2# show interfaces trunk
```

Fa0/1 should appear as a trunk.

### View Running Configuration

```cisco
SW1# show running-config
SW2# show running-config
```

The important configuration should include:

```cisco
interface FastEthernet0/1
 switchport mode trunk
```

and:

```cisco
interface FastEthernet0/3
 switchport access vlan 2
 switchport mode access
```

---

# Key Concepts Learned

### 1. VLANs Provide Layer 2 Segmentation

Devices in different VLANs are separated into different broadcast domains.

```text
VLAN 1
PC1 ───── PC3

VLAN 2
PC2 ───── PC4
```

### 2. Access Ports

PC-facing interfaces are configured as access ports:

```cisco
switchport mode access
```

An access port normally belongs to a single VLAN.

### 3. Trunk Ports

The link between switches must be configured as a trunk when multiple VLANs need to cross the connection:

```cisco
switchport mode trunk
```

### 4. Trunks Carry Multiple VLANs

The trunk between SW1 and SW2 carries traffic for both:

```text
VLAN 1
VLAN 2
```

This allows:

```text
PC1 ─ VLAN 1 ─ SW1 ═══ SW2 ─ VLAN 1 ─ PC3
                         ↑
                      Trunk

PC2 ─ VLAN 2 ─ SW1 ═══ SW2 ─ VLAN 2 ─ PC4
                         ↑
                      Trunk
```

### 5. Same VLAN vs Different VLAN

Devices in the **same VLAN** can communicate at Layer 2 if the VLAN is correctly carried across the network.

Devices in **different VLANs** require a Layer 3 device such as a router or Layer 3 switch for communication.

---

# Conclusion

This lab demonstrated why VLAN traffic must be properly carried across an inter-switch connection.

Initially, communication between **PC1 and PC3** worked because both PCs were in VLAN 1, while communication between **PC2 and PC4** failed because VLAN 2 was not being carried correctly across the SW1-SW2 link.

After configuring **FastEthernet0/1 on both SW1 and SW2 as trunk interfaces**, VLAN 2 traffic was successfully transported between the switches.

The final result was:

```text
Same VLAN  →  Communication succeeds
Different VLAN  →  Communication fails without inter-VLAN routing
```

The main configuration required to resolve the issue was:

```cisco
SW1(config)# interface fastethernet 0/1
SW1(config-if)# switchport mode trunk

SW2(config)# interface fastethernet 0/1
SW2(config-if)# switchport mode trunk
```

This demonstrates the fundamental difference between **access ports** and **trunk ports** in a switched VLAN environment.
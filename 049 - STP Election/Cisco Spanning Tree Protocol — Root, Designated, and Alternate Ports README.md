# Cisco Spanning Tree Protocol — Root, Designated, and Alternate Ports

## Lab Overview

This lab focuses on understanding how **Spanning Tree Protocol (STP)** determines the roles and states of switch interfaces in a Layer 2 topology.

The main objective is to analyze the STP topology **without initially using the CLI** and determine:

- The root bridge for VLAN 1
- Root ports
- Designated ports
- Alternate/blocked ports

After making the predictions, Cisco `show` commands are used to verify the answers.

---

# 1. Lab Objectives

By completing this lab, you should be able to:

1. Identify the STP root bridge by comparing bridge IDs.
2. Determine the root port on non-root switches.
3. Identify designated ports.
4. Identify alternate/blocked ports.
5. Understand how STP path cost influences port selection.
6. Verify STP predictions using Cisco IOS commands.

---

# 2. Hardware and Software

All switches are Cisco Catalyst 2960 switches.

| Device | Model | IOS Version | STP Protocol |
|---|---|---|---|
| SW1 | WS-C2960-24TT | 12.2(25)FX | PVST |
| SW2 | WS-C2960-24TT | 12.2(25)FX | PVST |
| SW3 | WS-C2960-24TT | 12.2(25)FX | PVST |
| SW4 | WS-C2960-24TT | 12.2(25)FX | PVST |

The STP output shows:

```text
Spanning tree enabled protocol ieee
```

This indicates that the switches are using the default PVST implementation.

---

# 3. Important STP Concepts

## Root Bridge

The root bridge is the switch with the **lowest Bridge ID**.

The Bridge ID is determined by:

1. STP priority
2. Switch MAC address

When all switches have the default priority, the switch with the **lowest MAC address** becomes the root bridge.

---

## Root Port

Every non-root switch selects one root port.

The root port is the interface that provides the **best path toward the root bridge**.

STP considers:

- Root path cost
- Sender Bridge ID
- Sender Port ID

The root port is placed into the forwarding state.

---

## Designated Port

A designated port is the interface selected to forward traffic for a particular Layer 2 segment.

Designated ports are normally:

```text
Forwarding
```

and have the role:

```text
Desg
```

---

## Alternate / Blocked Port

An alternate port provides a redundant path toward the root bridge.

To prevent a Layer 2 loop, STP places the redundant interface into a blocking state.

The output identifies this as:

```text
Altn BLK
```

The interface remains available as a backup path if the active STP path fails.

---

# 4. Determine the Root Bridge Without Using the CLI

Before checking the switches with `show spanning-tree`, examine the information available in the topology.

The four switches have the following base MAC addresses:

| Switch | Base MAC Address |
|---|---|
| SW1 | `0001.4258.19EA` |
| SW2 | `00D0.97E6.E169` |
| SW3 | `0001.4368.9B1B` |
| SW4 | `00E0.8FA4.BA54` |

All switches use the default STP priority:

```text
32768
```

For VLAN 1, the effective priority is:

```text
32769
```

Because the priority is equal, the switch with the lowest MAC address becomes the root bridge.

Comparing the addresses:

```text
0001.4258.19EA
0001.4368.9B1B
00D0.97E6.E169
00E0.8FA4.BA54
```

The lowest MAC address belongs to **SW1**.

Therefore:

### Root Bridge

```text
SW1
```

---

# 5. Determine the Root Ports

SW1 is the root bridge, so it does **not** have a root port.

The other switches must select their best path toward SW1.

## SW1

```text
Root Port: None
```

SW1 is already the root bridge.

---

## SW2

SW2 has two FastEthernet interfaces:

```text
Fa0/1
Fa0/2
```

The path toward the root uses:

```text
Fa0/2
```

Therefore:

```text
SW2 Root Port: Fa0/2
```

The STP verification confirms:

```text
Fa0/2    Root FWD    19
```

---

## SW3

SW3 has:

```text
Fa0/1
Gi0/1
```

Its path toward the root uses:

```text
Fa0/1
```

Therefore:

```text
SW3 Root Port: Fa0/1
```

The verification output confirms:

```text
Fa0/1    Root FWD    19
```

---

## SW4

SW4 has:

```text
Fa0/1
Gi0/1
```

The GigabitEthernet interface provides the lower-cost path toward the root.

Therefore:

```text
SW4 Root Port: Gi0/1
```

The verification output confirms:

```text
Gi0/1    Root FWD    4
```

---

# 6. Root Port Summary

| Switch | Root Port |
|---|---|
| SW1 | None — Root Bridge |
| SW2 | Fa0/2 |
| SW3 | Fa0/1 |
| SW4 | Gi0/1 |

---

# 7. Determine the Designated Ports

Designated ports are forwarding interfaces that provide the best path for their respective network segments.

Based on the topology and the verified STP output:

## SW1

SW1 is the root bridge, so its active interfaces are designated ports.

```text
SW1:
Fa0/1
Fa0/2
```

Both interfaces are shown as:

```text
Desg FWD
```

---

## SW2

SW2 has:

```text
Fa0/1
```

as its designated forwarding interface.

Therefore:

```text
SW2:
Fa0/1
```

---

## SW3

SW3 has:

```text
Gi0/1
```

as its designated forwarding interface.

Therefore:

```text
SW3:
Gi0/1
```

---

## SW4

SW4 does not have a designated forwarding interface in the provided output.

Its two interfaces are:

```text
Gi0/1    Root FWD
Fa0/1    Altn BLK
```

Therefore:

```text
SW4:
None
```

---

# 8. Designated Port Summary

| Switch | Designated Ports |
|---|---|
| SW1 | Fa0/1, Fa0/2 |
| SW2 | Fa0/1 |
| SW3 | Gi0/1 |
| SW4 | None |

---

# 9. Determine the Alternate / Blocked Ports

STP blocks redundant paths to prevent switching loops.

## SW1

SW1 is the root bridge and its shown interfaces are forwarding designated ports.

```text
SW1:
None
```

---

## SW2

Both interfaces are forwarding:

```text
Fa0/1    Desg FWD
Fa0/2    Root FWD
```

Therefore:

```text
SW2:
None
```

---

## SW3

Both interfaces are forwarding:

```text
Fa0/1    Root FWD
Gi0/1    Desg FWD
```

Therefore:

```text
SW3:
None
```

---

## SW4

SW4 has an alternate interface:

```text
Fa0/1    Altn BLK
```

Therefore:

```text
SW4:
Fa0/1
```

---

# 10. Alternate / Blocked Port Summary

| Switch | Alternate / Blocked Ports |
|---|---|
| SW1 | None |
| SW2 | None |
| SW3 | None |
| SW4 | Fa0/1 |

---

# 11. Complete Predicted Answer

Before using the CLI, the expected answers are:

## VLAN 1 Root Bridge

```text
SW1
```

## Root Ports

```text
SW1: None
SW2: Fa0/2
SW3: Fa0/1
SW4: Gi0/1
```

## Designated Ports

```text
SW1: Fa0/1, Fa0/2
SW2: Fa0/1
SW3: Gi0/1
SW4: None
```

## Alternate / Blocked Ports

```text
SW1: None
SW2: None
SW3: None
SW4: Fa0/1
```

---

# 12. Verify Using the CLI

After completing your predictions, use:

```bash
show spanning-tree
```

on each switch.

For VLAN 1, the command displays:

- Root ID
- Root bridge status
- Root port
- Designated ports
- Alternate ports
- Port cost
- Port state

---

## SW1 Verification

The output shows:

```text
Root ID
Priority 32769
Address 0001.4258.19EA
This bridge is the root
```

Therefore, SW1 is confirmed as the root bridge.

Its interfaces are:

```text
Fa0/1    Desg FWD    Cost 19
Fa0/2    Desg FWD    Cost 19
```

---

## SW2 Verification

The output shows:

```text
Root ID
Address 0001.4258.19EA
```

and:

```text
Fa0/1    Desg FWD    Cost 19
Fa0/2    Root FWD    Cost 19
```

Therefore:

```text
Root Port = Fa0/2
Designated Port = Fa0/1
```

---

## SW3 Verification

The output shows:

```text
Root ID
Address 0001.4258.19EA
```

and:

```text
Fa0/1    Root FWD    Cost 19
Gi0/1    Desg FWD    Cost 4
```

Therefore:

```text
Root Port = Fa0/1
Designated Port = Gi0/1
```

---

## SW4 Verification

The output shows:

```text
Root ID
Address 0001.4258.19EA
Cost 23
Port 25(GigabitEthernet0/1)
```

Its interfaces are:

```text
Fa0/1    Altn BLK    Cost 19
Gi0/1    Root FWD    Cost 4
```

Therefore:

```text
Root Port = Gi0/1
Alternate/Blocked Port = Fa0/1
```

---

# 13. Why Is SW4 Fa0/1 Blocked?

SW4 has two possible paths toward the root bridge:

- `Gi0/1` with a cost of **4**
- `Fa0/1` with a cost of **19**

The GigabitEthernet path has the lower STP cost.

Therefore, STP selects:

```text
Gi0/1 → Root Port
```

The FastEthernet path is redundant and has the higher cost, so STP places:

```text
Fa0/1 → Alternate / Blocking
```

This prevents a Layer 2 loop while keeping the redundant path available.

---

# 14. Useful Verification Commands

### View complete STP information

```bash
show spanning-tree
```

### View a specific VLAN

```bash
show spanning-tree vlan 1
```

### View STP summary

```bash
show spanning-tree summary
```

### View STP information for a specific interface

```bash
show spanning-tree interface fa0/1 detail
```

or:

```bash
show spanning-tree interface gi0/1 detail
```

---

# 15. Key Lessons

This lab demonstrates several important STP concepts:

### 1. Lowest Bridge ID wins

When switches have equal priorities, the switch with the lowest MAC address becomes the root bridge.

```text
SW1 = 0001.4258.19EA
```

is lower than the MAC addresses of SW2, SW3, and SW4.

Therefore:

```text
SW1 = Root Bridge
```

### 2. Every non-root switch needs a root port

```text
SW2 → Fa0/2
SW3 → Fa0/1
SW4 → Gi0/1
```

### 3. Root bridge interfaces are designated

SW1's active interfaces are designated ports because SW1 is the root bridge.

### 4. STP uses path cost

GigabitEthernet has a lower default STP cost than FastEthernet:

```text
GigabitEthernet = 4
FastEthernet = 19
```

This explains why SW4 selects `Gi0/1` as its root port.

### 5. STP blocks redundant paths

SW4's `Fa0/1` is placed into:

```text
Altn BLK
```

to prevent a Layer 2 loop.

---

# 16. Final Lab Answer

```text
VLAN 1 Root Bridge:
SW1

Root Ports:
SW1: None
SW2: Fa0/2
SW3: Fa0/1
SW4: Gi0/1

Designated Ports:
SW1: Fa0/1, Fa0/2
SW2: Fa0/1
SW3: Gi0/1
SW4: None

Alternate/Blocked Ports:
SW1: None
SW2: None
SW3: None
SW4: Fa0/1
```

The topology is successfully understood when you can determine these STP roles from the topology and bridge information **before** using `show spanning-tree` to verify your predictions.
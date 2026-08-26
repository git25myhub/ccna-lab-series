# Cisco STP and Rapid-PVST Configuration Lab

## Lab Overview

This lab focuses on understanding and configuring **Spanning Tree Protocol (STP)** on a three-switch Cisco topology.

The objectives are to:

1. Identify the default STP version running on the switches.
2. Examine bridge IDs, root bridges, interface costs, and blocked interfaces.
3. Change the STP mode from PVST to **Rapid-PVST+ (RPVST)**.
4. Configure primary and secondary root bridges for VLANs 10, 20, and 30.
5. Enable **PortFast** and **BPDU Guard** on host-facing interfaces.
6. Verify the resulting STP topology and root bridge placement.

---

## Devices and Hardware

| Device | Model | IOS Version | STP Mode |
|---|---|---|---|
| SW1 | WS-C2960-24TT | 12.2(25)FX | Rapid-PVST |
| SW2 | WS-C2960-24TT | 12.2(25)FX | Rapid-PVST |
| SW3 | WS-C2960-24TT | 12.2(25)FX | Rapid-PVST |

### Switch Specifications

- 24 FastEthernet interfaces
- 2 GigabitEthernet interfaces
- Cisco IOS Software: `C2960-LANBASE-M`
- IOS Version: `12.2(25)FX`
- Extended System ID: Enabled
- Path Cost Method: Short

---

# 1. Initial STP Investigation

Before making any configuration changes, inspect the STP state on each switch.

Use:

```bash
show spanning-tree summary
show spanning-tree
```

## Default STP Version

The switches were initially running:

```text
PVST
```

The detailed output shows:

```text
Spanning tree enabled protocol ieee
```

This represents the default **Per-VLAN Spanning Tree (PVST)** operation on these switches.

---

## Initial Bridge IDs

The bridge ID consists primarily of the switch's STP priority and MAC address.

### SW1

```text
Bridge ID:
Priority: 32769 for VLAN 1
MAC: 0060.3E50.6B2C
```

For the configured VLANs:

```text
VLAN 10: Priority 32778
VLAN 20: Priority 32788
VLAN 30: Priority 32798
```

### SW2

```text
MAC: 0001.435C.8630
```

### SW3

```text
MAC: 0004.9A73.4E46
```

The Extended System ID causes the VLAN ID to be added to the base priority.

---

# 2. Initial Root Bridge

Before configuring the desired root placement, STP selected the switch with the lowest bridge ID as the root bridge.

From the initial output, SW2 was the root bridge for VLANs 10, 20, and 30.

For example, SW1 reported:

```text
Root ID
Priority 32778
Address 0001.435C.8630
```

The MAC address corresponds to SW2.

Therefore:

| VLAN | Initial Root Bridge |
|---|---|
| VLAN 10 | SW2 |
| VLAN 20 | SW2 |
| VLAN 30 | SW2 |

This is changed later so that the root bridge is distributed across the three switches.

---

# 3. Initial STP Interface Costs

The topology uses both FastEthernet and GigabitEthernet interfaces.

The observed STP costs were:

| Interface Type | STP Cost |
|---|---:|
| FastEthernet | 19 |
| GigabitEthernet | 4 |

For example, on SW1:

```text
Fa0/1    Cost 19
Fa0/2    Cost 19
Fa0/3    Cost 19
Gi0/1    Cost 4
Gi0/2    Cost 4
```

The lower cost of the GigabitEthernet links makes them preferable for reaching the root bridge.

---

# 4. Initially Blocked Interface

On SW1, the following interface was initially blocked:

```text
Gi0/1    Altn BLK    Cost 4
```

This interface was placed into the **Alternate/Blocking** role to prevent a Layer 2 switching loop.

SW1 instead used:

```text
Gi0/2    Root FWD    Cost 4
```

as its root path.

The blocked interface is not necessarily a failed interface. STP intentionally blocks redundant paths so that the Layer 2 topology remains loop-free.

---

# 5. Configure Rapid-PVST+

The next objective is to change the STP mode on all switches from PVST to **Rapid-PVST+**.

## SW1

```bash
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```

## SW2

```bash
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```

## SW3

```bash
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```

Verify:

```bash
show spanning-tree summary
```

Expected result:

```text
Switch is in rapid-pvst mode
```

The detailed STP output should show:

```text
Spanning tree enabled protocol rstp
```

---

# 6. Configure STP Root Bridges

The required root bridge design is:

| VLAN | Primary Root | Secondary Root |
|---|---|---|
| VLAN 10 | SW1 | SW2 |
| VLAN 20 | SW2 | SW3 |
| VLAN 30 | SW3 | SW1 |

This provides a distributed STP design rather than placing all VLANs on the same root switch.

---

## SW1 Configuration

SW1 must be:

- Primary root for VLAN 10
- Secondary root for VLAN 30

```bash
enable
configure terminal

spanning-tree vlan 10 root primary
spanning-tree vlan 30 root secondary

end
write memory
```

---

## SW2 Configuration

SW2 must be:

- Primary root for VLAN 20
- Secondary root for VLAN 10

```bash
enable
configure terminal

spanning-tree vlan 20 root primary
spanning-tree vlan 10 root secondary

end
write memory
```

---

## SW3 Configuration

SW3 must be:

- Primary root for VLAN 30
- Secondary root for VLAN 20

```bash
enable
configure terminal

spanning-tree vlan 30 root primary
spanning-tree vlan 20 root secondary

end
write memory
```

---

# 7. Enable PortFast

PortFast should be enabled on interfaces connected directly to end devices such as PCs.

In this topology, the host-facing FastEthernet interfaces are:

```text
Fa0/1
Fa0/2
Fa0/3
```

Configure them as follows on each switch:

```bash
configure terminal
interface range fastethernet 0/1 - 3
spanning-tree portfast
end
```

### Important

PortFast should **not** normally be enabled on switch-to-switch trunk links.

Cisco warns that enabling PortFast on a port connected to another switch can create temporary bridging loops.

---

# 8. Enable BPDU Guard

BPDU Guard protects PortFast-enabled access ports.

If a BPDU is received on a BPDU Guard-enabled port, the switch can place the interface into an error-disabled state.

Configure BPDU Guard on the same host-facing interfaces:

```bash
configure terminal
interface range fastethernet 0/1 - 3
spanning-tree bpduguard enable
end
write memory
```

Apply this configuration to:

- SW1
- SW2
- SW3

---

# 9. Complete Configuration Summary

## SW1

```bash
enable
configure terminal

spanning-tree mode rapid-pvst

spanning-tree vlan 10 root primary
spanning-tree vlan 30 root secondary

interface range fastethernet 0/1 - 3
spanning-tree portfast
spanning-tree bpduguard enable

end
write memory
```

---

## SW2

```bash
enable
configure terminal

spanning-tree mode rapid-pvst

spanning-tree vlan 20 root primary
spanning-tree vlan 10 root secondary

interface range fastethernet 0/1 - 3
spanning-tree portfast
spanning-tree bpduguard enable

end
write memory
```

---

## SW3

```bash
enable
configure terminal

spanning-tree mode rapid-pvst

spanning-tree vlan 30 root primary
spanning-tree vlan 20 root secondary

interface range fastethernet 0/1 - 3
spanning-tree portfast
spanning-tree bpduguard enable

end
write memory
```

---

# 10. Verification

After completing the configuration, verify the STP mode:

```bash
show spanning-tree summary
```

Expected:

```text
Switch is in rapid-pvst mode
```

Verify the root bridge for each VLAN:

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
```

---

## Expected Root Bridge Design

### VLAN 10

SW1 should report:

```text
This bridge is the root
```

SW2 should identify SW1 as the root.

SW2 is the configured secondary root.

### VLAN 20

SW2 should report:

```text
This bridge is the root
```

SW3 is the configured secondary root.

### VLAN 30

SW3 should report:

```text
This bridge is the root
```

SW1 is the configured secondary root.

---

# 11. Verify PortFast and BPDU Guard

Use:

```bash
show spanning-tree interface fastethernet 0/1 detail
show spanning-tree interface fastethernet 0/2 detail
show spanning-tree interface fastethernet 0/3 detail
```

Look for indicators that PortFast is enabled.

You can also check the configuration directly:

```bash
show running-config
```

Look for:

```text
spanning-tree portfast
spanning-tree bpduguard enable
```

---

# 12. Verification Checklist

- [x] Identified the default STP mode.
- [x] Identified the bridge IDs.
- [x] Identified the initial root bridge.
- [x] Identified STP interface costs.
- [x] Identified the initially blocked interface.
- [x] Changed SW1 to Rapid-PVST.
- [x] Changed SW2 to Rapid-PVST.
- [x] Changed SW3 to Rapid-PVST.
- [x] Configured SW1 as VLAN 10 primary root.
- [x] Configured SW2 as VLAN 10 secondary root.
- [x] Configured SW2 as VLAN 20 primary root.
- [x] Configured SW3 as VLAN 20 secondary root.
- [x] Configured SW3 as VLAN 30 primary root.
- [x] Configured SW1 as VLAN 30 secondary root.
- [x] Enabled PortFast on host-facing interfaces.
- [x] Enabled BPDU Guard on host-facing interfaces.
- [x] Saved the configurations.

---

## Final Result

The lab demonstrates how **Rapid-PVST+** can be used to provide rapid Layer 2 convergence while allowing the network administrator to control the root bridge on a per-VLAN basis.

The final STP topology should distribute the root bridge responsibilities as follows:

```text
VLAN 10 → SW1
VLAN 20 → SW2
VLAN 30 → SW3
```

The secondary root assignments provide redundancy:

```text
VLAN 10 → Secondary SW2
VLAN 20 → Secondary SW3
VLAN 30 → Secondary SW1
```

PortFast and BPDU Guard are enabled on the end-device interfaces to allow hosts to transition quickly to forwarding and to protect the STP topology from unexpected BPDUs.
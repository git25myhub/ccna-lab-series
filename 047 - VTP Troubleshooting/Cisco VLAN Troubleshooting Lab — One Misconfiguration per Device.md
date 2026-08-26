# Cisco VLAN Troubleshooting Lab — One Misconfiguration per Device

## Lab Overview

This lab is a **Cisco Packet Tracer troubleshooting exercise** involving three Cisco Catalyst 2960 switches.

The network contains connectivity problems caused by **one misconfiguration on each networking device**. The objective is to identify the incorrect configuration, correct it, and verify that the resulting network behavior matches the requirements.

The final requirement is:

> **PCs in the same VLAN must be able to ping each other, while PCs in different VLANs must not be able to communicate.**

This lab reinforces troubleshooting skills involving **VLANs, trunking, VTP, switchport configuration, and Layer 2 connectivity**.

---

## Network Devices

| Device | Model | IOS | Role |
|---|---|---|---|
| SW1 | WS-C2960-24TT | 12.2(25)FX | Access/Distribution Switch |
| SW2 | WS-C2960-24TT | 12.2(25)FX | Access Switch |
| SW3 | WS-C2960-24TT | 12.2(25)FX | Access Switch |

---

## VLAN Configuration

The network uses the following VLANs:

| VLAN | Name | Purpose |
|---:|---|---|
| 10 | HR | Human Resources |
| 20 | Sales | Sales |
| 30 | Engineering | Engineering |
| 40 | Accounting | Accounting |

The troubleshooting output shows that SW1 and SW3 have VLANs 10, 20, and 30, while SW2 also has VLAN 40.

---

# Initial Problems Identified

The lab contains one configuration problem per switch.

Based on the supplied configurations and troubleshooting commands, the following misconfigurations were identified and corrected.

## SW1 — GigabitEthernet0/1

### Problem

SW1's `GigabitEthernet0/1` was configured as an **access port** even though it was being used as a switch-to-switch connection.

The original configuration contained:

```cisco
interface GigabitEthernet0/1
 switchport trunk allowed vlan 1,10,20
 switchport mode access
```

This is a configuration mismatch.

The interface had a trunk VLAN list configured, but its operational mode was:

```text
switchport mode access
```

The switch also reported:

```text
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU on non trunk GigabitEthernet0/1 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/1 on VLAN0001. Inconsistent port type.
```

These messages are strong indicators of a trunk mismatch.

### Fix

Configure the interface as a trunk:

```cisco
enable
configure terminal

interface GigabitEthernet0/1
 switchport mode trunk
exit

end
write memory
```

### Verification

```cisco
show interfaces trunk
```

Expected result:

```text
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1
```

The configured VLANs allowed on the trunk are:

```text
1,10,20
```

---

# SW2 — FastEthernet0/2

## Problem

SW2 had VLAN 40 configured correctly on `FastEthernet0/1`, but `FastEthernet0/2` was incorrectly assigned to **VLAN 50**.

Initial VLAN output:

```text
40   Accounting                       active    Fa0/1
50   VLAN0050                         active    Fa0/2
```

This means the host connected to Fa0/2 was placed into the wrong VLAN.

### Fix

Move Fa0/2 into VLAN 40:

```cisco
enable
configure terminal

interface FastEthernet0/2
 switchport access vlan 40
exit

no vlan 50

end
write memory
```

### Verification

```cisco
show vlan brief
```

Expected result:

```text
40   Accounting                       active    Fa0/1, Fa0/2
```

VLAN 50 should no longer appear in the VLAN database.

---

# SW3 — VTP Domain

## Problem

SW3 was configured as a VTP client, but its VTP domain did not initially match the rest of the network.

Initial VTP status showed:

```text
VTP Operating Mode              : Client
VTP Domain Name                 : CCNP
```

The network should use the `CCNA` VTP domain.

Because SW3 was a VTP client, the following command also demonstrated the effect of the incorrect configuration:

```cisco
vlan 10
```

produced:

```text
VTP VLAN configuration not allowed when device is in CLIENT mode.
```

This is expected behavior for a VTP client; VLAN creation should come from the VTP server.

### Fix

Change the VTP domain to `CCNA`:

```cisco
enable
configure terminal

vtp domain CCNA

end
write memory
```

### Verification

```cisco
show vtp status
```

Expected:

```text
VTP Version                     : 2
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP V2 Mode                     : Enabled
```

After the correction, SW3 learned the VLAN information and displayed:

```text
10   HR                               active    Fa0/1
20   Sales                            active    Fa0/4
30   Engineering                      active    Fa0/2, Fa0/3
```

---

# Correct VLAN-to-Port Assignments

After troubleshooting, the relevant host ports should be assigned as follows.

## SW1

| Interface | VLAN | Department |
|---|---:|---|
| Fa0/1 | 10 | HR |
| Fa0/2 | 10 | HR |
| Fa0/3 | 20 | Sales |
| Gi0/1 | Trunk | VLANs 1, 10, 20 |

SW1 host-port configuration:

```cisco
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
```

---

## SW2

| Interface | VLAN | Department |
|---|---:|---|
| Fa0/1 | 40 | Accounting |
| Fa0/2 | 40 | Accounting |

Correct configuration:

```cisco
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 40
```

---

## SW3

| Interface | VLAN | Department |
|---|---:|---|
| Fa0/1 | 10 | HR |
| Fa0/2 | 30 | Engineering |
| Fa0/3 | 30 | Engineering |
| Fa0/4 | 20 | Sales |
| Gi0/1 | Trunk | VLANs 1, 10, 20 |

The existing SW3 access-port configuration is:

```cisco
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
 switchport nonegotiate

interface FastEthernet0/2
 switchport access vlan 30
 switchport mode access
 switchport nonegotiate

interface FastEthernet0/3
 switchport access vlan 30
 switchport mode access
 switchport nonegotiate

interface FastEthernet0/4
 switchport access vlan 20
 switchport mode access
 switchport nonegotiate
```

---

# Troubleshooting Methodology

A systematic troubleshooting process was used rather than changing configurations randomly.

## Step 1 — Check VLANs

Run:

```cisco
show vlan brief
```

This verifies:

- Which VLANs exist
- Which ports belong to each VLAN
- Whether a host has been assigned to the wrong VLAN

This identified the incorrect **VLAN 50 assignment on SW2 Fa0/2**.

---

## Step 2 — Check Trunk Ports

Run:

```cisco
show interfaces trunk
```

If a switch-to-switch connection does not appear as a trunk, inspect the interface configuration.

On SW1, the command initially returned no trunk information because Gi0/1 was configured as an access port.

The configuration showed:

```cisco
switchport mode access
```

This was corrected to:

```cisco
switchport mode trunk
```

---

## Step 3 — Check the Running Configuration

Use:

```cisco
show running-config
```

Look specifically at:

```cisco
interface GigabitEthernet0/1
```

and host-facing interfaces.

Compare the configuration against the expected topology.

---

## Step 4 — Check VTP Status

Use:

```cisco
show vtp status
```

Important fields include:

```text
VTP Version
VTP Operating Mode
VTP Domain Name
Configuration Revision
```

SW3 initially had:

```text
VTP Domain Name : CCNP
```

The correct domain was:

```text
CCNA
```

After correction, SW3 successfully learned the VLAN information.

---

# Connectivity Testing

The purpose of the final connectivity tests is not to achieve unrestricted connectivity.

Instead, the lab requires **VLAN isolation**.

## Same-VLAN Connectivity

PCs belonging to the same VLAN should be able to communicate.

For example, a successful ping was observed:

```text
C:\>ping 10.0.0.2

Reply from 10.0.0.2: bytes=32 time=4ms TTL=128
Reply from 10.0.0.2: bytes=32 time<1ms TTL=128
Reply from 10.0.0.2: bytes=32 time<1ms TTL=128
Reply from 10.0.0.2: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms Layer 2 connectivity between the appropriate hosts.

---

## Different-VLAN Connectivity

PCs in different VLANs should **not** be able to communicate because there is no Layer 3 routing configured between the VLANs.

For example:

```text
C:\>ping 10.0.0.5

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

This is an **expected result** when the destination is in a different VLAN.

---

# Why Different VLANs Cannot Communicate

VLANs create separate Layer 2 broadcast domains.

For example:

```text
VLAN 10 — HR
VLAN 20 — Sales
VLAN 30 — Engineering
VLAN 40 — Accounting
```

A switch can forward frames within the same VLAN, but it does not route traffic between VLANs.

Therefore:

```text
VLAN 10  ──X──  VLAN 20
VLAN 10  ──X──  VLAN 30
VLAN 20  ──X──  VLAN 30
```

A router or Layer 3 switch would be required to provide communication between the VLANs.

Since this troubleshooting lab does not configure inter-VLAN routing, failed cross-VLAN pings are expected and indicate correct VLAN isolation.

---

# Useful Troubleshooting Commands

### VLAN information

```cisco
show vlan brief
```

### Trunk information

```cisco
show interfaces trunk
```

### Interface configuration

```cisco
show interfaces switchport
```

### Running configuration

```cisco
show running-config
```

### VTP information

```cisco
show vtp status
```

### Spanning Tree information

```cisco
show spanning-tree
```

### Interface status

```cisco
show interfaces status
```

---

# Key Lessons

## 1. A trunk VLAN list does not make a port a trunk

This configuration:

```cisco
switchport trunk allowed vlan 1,10,20
```

does not replace:

```cisco
switchport mode trunk
```

Both settings serve different purposes.

---

## 2. VLAN assignment must match the intended host

A host connected to an Accounting port must be assigned to VLAN 40.

Incorrect:

```cisco
switchport access vlan 50
```

Correct:

```cisco
switchport access vlan 40
```

---

## 3. VTP clients cannot create VLANs locally

On a VTP client:

```cisco
vlan 10
```

can produce:

```text
VTP VLAN configuration not allowed when device is in CLIENT mode.
```

The VLAN should instead be learned from the VTP server.

---

## 4. VTP domains must match

SW3 initially had:

```text
CCNP
```

while the network used:

```text
CCNA
```

Changing the domain to `CCNA` allowed SW3 to synchronize with the VTP domain.

---

## 5. Failed cross-VLAN pings are expected

The completion criteria specifically require:

- Same VLAN → **Ping succeeds**
- Different VLAN → **Ping fails**

Therefore, a failed ping is not automatically evidence of a problem. Always determine whether the source and destination belong to the same VLAN before diagnosing the result.

---

# Final Verification Checklist

- [x] SW1 Gi0/1 configured as a trunk
- [x] SW1 trunk allows VLANs 1, 10, and 20
- [x] SW2 Fa0/2 moved from VLAN 50 to VLAN 40
- [x] VLAN 50 removed from SW2
- [x] SW3 VTP domain changed from `CCNP` to `CCNA`
- [x] SW3 remains a VTP client
- [x] VLAN 10 exists on SW3
- [x] VLAN 20 exists on SW3
- [x] VLAN 30 exists on SW3
- [x] HR ports are assigned to VLAN 10
- [x] Sales ports are assigned to VLAN 20
- [x] Engineering ports are assigned to VLAN 30
- [x] Accounting ports are assigned to VLAN 40
- [x] Same-VLAN connectivity verified
- [x] Different-VLAN connectivity blocked
- [x] Configurations saved with `write memory`

---

# Completion Criteria

The troubleshooting lab is successfully completed when:

1. Each of the three networking devices has its incorrect configuration corrected.
2. VLAN assignments match the intended network design.
3. SW1's switch-to-switch interface operates as a trunk.
4. SW3 has the correct VTP domain and receives the required VLAN information.
5. SW2's Accounting hosts are both in VLAN 40.
6. PCs in the **same VLAN can successfully ping each other**.
7. PCs in **different VLANs cannot ping each other**.
8. The network maintains proper Layer 2 VLAN separation.

**Final result: the network provides same-VLAN connectivity while maintaining isolation between different VLANs.**
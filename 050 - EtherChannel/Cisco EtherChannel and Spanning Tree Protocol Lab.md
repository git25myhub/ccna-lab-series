# Cisco EtherChannel & Spanning Tree Protocol Lab

## 📌 Lab Overview

This lab focuses on analyzing **Spanning Tree Protocol (STP)** and configuring multiple **EtherChannel** implementations using different negotiation methods.

The topology contains four Cisco switches:

- **SW1**
- **SW2**
- **SW3**
- **SW4**

The lab demonstrates three EtherChannel configurations:

1. **Layer 2 EtherChannel between SW1 and SW2** using Cisco **PAgP**
2. **Layer 3 EtherChannel between SW2 and SW3** using a **static EtherChannel**
3. **Layer 2 EtherChannel between SW3 and SW4** using IEEE **LACP**

The lab also requires analysis of STP roles, including the **Root Bridge, Root Ports, Designated Ports, and Alternate Ports**.

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

- Analyze an STP topology.
- Identify the STP Root Bridge.
- Identify Root, Designated, and Alternate ports.
- Understand how STP selects port roles.
- Configure a Layer 2 EtherChannel using **PAgP**.
- Configure a Layer 3 EtherChannel using a **static `mode on`** EtherChannel.
- Configure a Layer 2 EtherChannel using **LACP**.
- Configure EtherChannels as trunk links.
- Configure IP addressing on a Layer 3 Port-Channel.
- Verify EtherChannel operation using Cisco IOS commands.
- Troubleshoot EtherChannel negotiation and trunking problems.

---

# 🧠 Part 1 — Spanning Tree Protocol Analysis

## Root Bridge

The STP output from SW1 shows:

```text
Root ID    Priority    32769
           Address     0004.9A9D.E23D
           This bridge is the root
```

Therefore:

**SW1 is the Root Bridge.**

SW1 has the same Bridge ID as the Root ID:

```text
Bridge ID  Priority    32769
           Address     0004.9A9D.E23D
```

This confirms that SW1 is the root bridge for VLAN 1.

---

## STP Port Roles

Because SW1 is the Root Bridge, its active STP interfaces toward the topology operate as **Designated Ports**.

The observed output is:

```text
Interface        Role Sts Cost      Prio.Nbr Type
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

Therefore:

| Port Role | Switch | Interfaces |
|---|---|---|
| Root Bridge | SW1 | — |
| Designated Ports | SW1 | Fa0/1–Fa0/4 |
| Root Ports | Non-root switches | Lowest-cost path toward SW1 |
| Alternate Ports | Non-root switches | Redundant paths placed into blocking state |

> **Important:** After EtherChannel formation, STP treats a Port-Channel as a single logical link rather than treating every physical member independently. Therefore, the final STP roles should be verified after all EtherChannels have been configured.

---

## How STP Determines Port Roles

STP uses the **Bridge ID** and path costs to construct a loop-free Layer 2 topology.

The selection process is generally:

1. **Elect the Root Bridge**
   - The switch with the lowest Bridge ID becomes the Root Bridge.
   - Bridge ID consists primarily of:
     - Bridge priority
     - VLAN/system ID
     - MAC address

2. **Select Root Ports**
   - Every non-root switch selects one Root Port.
   - The Root Port provides the lowest-cost path toward the Root Bridge.

3. **Select Designated Ports**
   - Each network segment has one Designated Port.
   - The Designated Port provides the best path toward the Root Bridge.

4. **Place Redundant Paths into Alternate/Blocking State**
   - If a redundant path would create a Layer 2 loop, STP prevents it from forwarding.
   - Such a port may appear as an **Alternate** port on supported STP implementations.

### STP Decision Factors

When selecting the best path, STP considers:

1. Lowest Root Bridge ID
2. Lowest Root Path Cost
3. Lowest Sender Bridge ID
4. Lowest Sender Port ID

This process allows STP to maintain redundancy while preventing Layer 2 switching loops.

---

# 🔗 Part 2 — Layer 2 EtherChannel: SW1 ↔ SW2

## Protocol

The requirement is to use a **Cisco proprietary EtherChannel protocol**.

The Cisco proprietary protocol is:

**PAgP — Port Aggregation Protocol**

SW1 was configured using:

```text
channel-group 1 mode desirable
```

`desirable` actively attempts to form a PAgP EtherChannel.

The resulting configuration on SW1 includes:

```text
interface Port-channel1
 switchport mode trunk

interface FastEthernet0/1
 switchport mode trunk
 channel-group 1 mode desirable

interface FastEthernet0/2
 switchport mode trunk
 channel-group 1 mode desirable

interface FastEthernet0/3
 switchport mode trunk
 channel-group 1 mode desirable

interface FastEthernet0/4
 switchport mode trunk
 channel-group 1 mode desirable
```

### Required SW2 Configuration

The SW2 side should use a compatible PAgP mode, for example:

```text
interface range FastEthernet0/1 - 4
 channel-group 2 mode auto

interface Port-channel2
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

`auto` waits for a PAgP negotiation request from the remote switch, while SW1's `desirable` mode actively initiates the negotiation.

### Verification

Use:

```text
show etherchannel summary
```

A successful EtherChannel should resemble:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-------------------------------
1      Po1(SU)       PAgP        Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

Important flags:

- `S` = Layer 2
- `U` = In use
- `P` = Port is bundled in the Port-Channel

---

# 🌐 Part 3 — Layer 3 EtherChannel: SW2 ↔ SW3

## EtherChannel Type

The SW2–SW3 EtherChannel is a **Layer 3 EtherChannel**.

It is configured as a **static EtherChannel**, meaning no negotiation protocol such as PAgP or LACP is used.

The required EtherChannel mode is:

```text
channel-group 1 mode on
```

---

## SW2 Configuration

The physical interfaces are converted to Layer 3 interfaces:

```text
ip routing

interface range GigabitEthernet0/1 - 2
 no switchport
 channel-group 1 mode on
```

The logical Port-Channel receives the IP address:

```text
interface Port-channel1
 no switchport
 ip address 23.0.0.1 255.255.255.0
```

---

## SW3 Configuration

SW3 is configured similarly:

```text
ip routing

interface range GigabitEthernet0/1 - 2
 no switchport
 channel-group 1 mode on
```

The Port-Channel receives:

```text
interface Port-channel1
 no switchport
 ip address 23.0.0.2 255.255.255.0
```

### Addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| SW2 | Port-channel1 | 23.0.0.1 | 255.255.255.0 |
| SW3 | Port-channel1 | 23.0.0.2 | 255.255.255.0 |

---

## Verification

On both switches:

```text
show etherchannel summary
```

Expected result:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+---------------------------
1      Po1(RU)       -           Gig0/1(P) Gig0/2(P)
```

The important flags are:

- `R` = Layer 3
- `U` = In use
- `P` = Port is bundled

Connectivity can be tested from SW3:

```text
ping 23.0.0.1
```

A successful result confirms Layer 3 connectivity across the Port-Channel.

---

# 🔗 Part 4 — Layer 2 EtherChannel: SW3 ↔ SW4

## Protocol

The requirement is to use an **IEEE standard EtherChannel protocol**.

The IEEE standard protocol is:

**LACP — Link Aggregation Control Protocol**

LACP uses:

- `active`
- `passive`

For this lab, the interfaces are configured with:

```text
channel-group 2 mode active
```

---

## SW3 Configuration

```text
interface range FastEthernet0/1 - 4
 channel-group 2 mode active

interface Port-channel2
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

The physical interfaces should also operate as trunks:

```text
interface range FastEthernet0/1 - 4
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 2 mode active
```

---

## SW4 Configuration

SW4 was configured using LACP active mode:

```text
interface range FastEthernet0/1 - 4
 channel-group 1 mode active

interface Port-channel1
 switchport mode trunk
```

The resulting configuration includes:

```text
interface Port-channel1
 switchport mode trunk

interface FastEthernet0/1
 switchport mode trunk
 channel-group 1 mode active

interface FastEthernet0/2
 switchport mode trunk
 channel-group 1 mode active

interface FastEthernet0/3
 switchport mode trunk
 channel-group 1 mode active

interface FastEthernet0/4
 switchport mode trunk
 channel-group 1 mode active
```

### Verification

SW4 produced:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(SU)       LACP        Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

This confirms that:

- Port-Channel 1 exists.
- The EtherChannel is Layer 2.
- The protocol is LACP.
- All four physical interfaces are successfully bundled.
- The Port-Channel is operational.

---

# ⚠️ Troubleshooting Observations

Several configuration mistakes and expected EtherChannel messages appeared during the lab.

## 1. Incorrect Interface Range Syntax

The following command was rejected:

```text
int f0/1 - 4
```

The correct syntax is:

```text
interface range f0/1 - 4
```

or:

```text
int range f0/1 - 4
```

---

## 2. PAgP Negotiation Failure

SW1 initially reported:

```text
Fa0/1 suspended: PAGP currently not enabled on the remote port.
```

This occurred because the remote SW2 ports were not yet configured with a compatible PAgP mode.

The solution was to configure SW2 with:

```text
channel-group 2 mode auto
```

while SW1 uses:

```text
channel-group 1 mode desirable
```

A compatible combination is required on both sides.

---

## 3. LACP Negotiation Failure

SW4 initially reported:

```text
Fa0/1 suspended: LACP currently not enabled on the remote port.
```

This occurred because LACP had not yet been enabled on the SW3 side.

The solution was to configure both sides with compatible LACP modes.

For example:

```text
SW3:
channel-group 2 mode active
```

and:

```text
SW4:
channel-group 1 mode active
```

---

## 4. Trunk Encapsulation Error

SW2 initially displayed:

```text
Command rejected: An interface whose trunk encapsulation is "Auto"
can not be configured to "trunk" mode.
```

The solution was to explicitly configure IEEE 802.1Q:

```text
switchport trunk encapsulation dot1q
switchport mode trunk
```

This is especially relevant on older Cisco platforms that support multiple trunk encapsulation types.

---

## 5. STP Inconsistent Port Type

SW2 generated:

```text
%SPANTREE-2-RECV_PVID_ERR:
Received 802.1Q BPDU on non trunk FastEthernet0/1 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL:
Blocking FastEthernet0/1 on VLAN0001. Inconsistent port type.
```

This indicates a trunk mismatch.

One side was sending tagged 802.1Q BPDUs while the local interface was not operating as a trunk.

The solution is to ensure both sides of the EtherChannel are consistently configured as trunks.

---

# 🔍 Useful Verification Commands

## STP

```text
show spanning-tree
```

```text
show spanning-tree vlan 1
```

```text
show spanning-tree root
```

Use these commands to identify:

- Root Bridge
- Root Ports
- Designated Ports
- Alternate/blocked ports
- Root path cost

---

## EtherChannel

```text
show etherchannel summary
```

```text
show etherchannel port-channel
```

```text
show etherchannel detail
```

---

## Port-Channel Interfaces

```text
show interfaces port-channel 1
```

```text
show interfaces port-channel 2
```

---

## Trunk Verification

```text
show interfaces trunk
```

This confirms whether the Port-Channel is operating as a trunk and which VLANs are allowed.

---

## Configuration Verification

```text
show running-config
```

For a specific interface:

```text
show running-config interface port-channel 1
```

---

## Layer 3 Verification

On SW2 and SW3:

```text
show ip interface brief
```

```text
ping 23.0.0.2
```

or from SW3:

```text
ping 23.0.0.1
```

---

# 📊 Final EtherChannel Summary

| Link | Layer | Protocol | Mode | Member Interfaces | Purpose |
|---|---|---|---|---|---|
| SW1 ↔ SW2 | Layer 2 | PAgP | Desirable / Auto | Fa0/1–Fa0/4 | Trunk |
| SW2 ↔ SW3 | Layer 3 | None | Static `on` | Gi0/1–Gi0/2 | Routed link |
| SW3 ↔ SW4 | Layer 2 | LACP | Active / Active | Fa0/1–Fa0/4 | Trunk |

---

# 🧪 Final Verification Checklist

- [ ] SW1 is identified as the STP Root Bridge.
- [ ] STP Root, Designated, and Alternate ports are identified.
- [ ] SW1–SW2 Layer 2 EtherChannel uses PAgP.
- [ ] SW1–SW2 Port-Channel operates as a trunk.
- [ ] SW2–SW3 is configured as a Layer 3 EtherChannel.
- [ ] SW2–SW3 uses static `channel-group mode on`.
- [ ] SW2 Port-channel1 uses `23.0.0.1/24`.
- [ ] SW3 Port-channel1 uses `23.0.0.2/24`.
- [ ] SW2 and SW3 can ping across the Layer 3 EtherChannel.
- [ ] SW3–SW4 Layer 2 EtherChannel uses LACP.
- [ ] SW3–SW4 Port-Channel operates as a trunk.
- [ ] All expected member interfaces show `(P)` in `show etherchannel summary`.
- [ ] Port-Channels show `SU` for Layer 2 EtherChannels.
- [ ] The Layer 3 Port-Channel shows `RU`.
- [ ] No EtherChannel member interfaces remain suspended.
- [ ] No STP trunk/PVID inconsistency errors remain.
- [ ] Configurations are saved with `write memory` or `copy running-config startup-config`.

---

# 🏁 Lab Completion Criteria

The lab is considered successfully completed when:

1. **SW1 is confirmed as the STP Root Bridge.**
2. STP port roles are correctly identified and explained.
3. **SW1–SW2** operates as a Layer 2 PAgP EtherChannel trunk.
4. **SW2–SW3** operates as a static Layer 3 EtherChannel with IP connectivity.
5. **SW3–SW4** operates as a Layer 2 LACP EtherChannel trunk.
6. All EtherChannel member interfaces are successfully bundled.
7. STP no longer reports trunk/PVID inconsistencies.
8. The final configurations are saved.

## Key Concepts Practiced

> **STP** prevents Layer 2 loops by selectively forwarding or blocking redundant paths.

> **EtherChannel** combines multiple physical links into one logical link, increasing available bandwidth and providing redundancy.

> **PAgP** is Cisco proprietary.

> **LACP** is the IEEE 802.3ad/802.1AX standard for link aggregation.

> **Static EtherChannel (`mode on`)** does not use PAgP or LACP negotiation.

> **Layer 2 EtherChannels** can carry VLAN traffic and operate as trunks.

> **Layer 3 EtherChannels** operate as routed interfaces and receive their IP address on the logical Port-Channel interface.
# EtherChannel Troubleshooting Lab — PAgP, Static EtherChannel & LACP

## 📌 Lab Overview

The EtherChannels in the network are not functioning properly. Your task is to **troubleshoot the configuration errors and restore all EtherChannels to an operational state**.

The network contains three separate EtherChannel configurations:

| Link | EtherChannel Type | Layer | Expected Port-Channel |
|---|---|---|---|
| SW1 ↔ SW2 | PAgP | Layer 2 | Po1 / Po2 |
| SW2 ↔ SW3 | Static EtherChannel | Layer 3 | Po1 |
| SW3 ↔ SW4 | LACP | Layer 2 | Po1 |

The goal is to identify the misconfigurations, correct them, and verify that all member interfaces successfully bundle into their respective EtherChannels.

---

## 🎯 Objectives

By completing this lab, you should be able to:

- Troubleshoot EtherChannel configuration problems.
- Identify mismatched EtherChannel negotiation protocols.
- Configure a Layer 2 EtherChannel using **PAgP**.
- Configure a Layer 3 static EtherChannel using **`mode on`**.
- Configure a Layer 2 EtherChannel using **LACP**.
- Identify interface compatibility problems such as speed mismatches.
- Verify EtherChannel operation using Cisco IOS verification commands.
- Verify Layer 3 connectivity across a routed EtherChannel.

---

## 🗺️ Network Topology

```text
       Layer 2 PAgP
   =====================
      SW1  ========  SW2
           Po1/Po2

                      Layer 3 Static
                  =====================
                     SW2 ======== SW3
                         Po1
                     23.0.0.0/24

                                      Layer 2 LACP
                                  =====================
                                      SW3 ======== SW4
                                           Po2/Po1
```

### EtherChannel Requirements

#### SW1 ↔ SW2

- Layer 2 EtherChannel
- PAgP
- Four FastEthernet links
- Interfaces:
  - SW1: `Fa0/1 - Fa0/4`
  - SW2: `Fa0/1 - Fa0/4`
- Links configured as trunks

#### SW2 ↔ SW3

- Layer 3 EtherChannel
- Static EtherChannel
- Four? **GigabitEthernet links shown in the supplied configuration use two interfaces**
- Interfaces:
  - SW2: `Gi0/1 - Gi0/2`
  - SW3: `Gi0/1 - Gi0/2`
- `no switchport`
- SW2: `23.0.0.1/24`
- SW3: `23.0.0.2/24`
- EtherChannel must use `mode on`

#### SW3 ↔ SW4

- Layer 2 EtherChannel
- LACP
- Four FastEthernet links
- Interfaces:
  - SW3: `Fa0/1 - Fa0/4`
  - SW4: `Fa0/1 - Fa0/4`
- Links configured as trunks
- LACP should be operational

---

# 🔎 Initial Troubleshooting

## 1. SW1 — SW2 PAgP Problem

The initial EtherChannel summary on SW1 showed:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SD)           LACP   Fa0/1(D) Fa0/2(I) Fa0/3(I) Fa0/4(I)
```

This immediately indicates a problem.

The requirement is **PAgP**, but SW1 was initially operating with **LACP**.

The correct PAgP configuration on SW1 is:

```text
interface range fa0/1 - 4
 channel-group 1 mode desirable
```

PAgP uses:

- `desirable`
- `auto`

At least one side must actively negotiate PAgP.

After changing SW1 to PAgP, the switch reported:

```text
%EC-5-L3DONTBNDL2: Fa0/1 suspended: PAGP currently not enabled on the remote port.
```

This message confirms that the remote side, SW2, also needed a compatible PAgP configuration.

### Verification

The corrected SW1 output became:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           PAgP   Fa0/1(D) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

The important indicators are:

- `PAgP` = correct negotiation protocol.
- `SU` = Layer 2 port-channel that is up and in use.
- `P` = interface successfully bundled.

---

# 2. SW2 — SW3 Layer 3 Static EtherChannel

The SW2 configuration showed:

```text
interface Port-channel1
 no switchport
 ip address 23.0.0.1 255.255.255.0
```

and:

```text
interface GigabitEthernet0/1
 no switchport
 no ip address
 channel-group 1 mode on
```

```text
interface GigabitEthernet0/2
 no switchport
 no ip address
 channel-group 1 mode on
```

This is the correct approach for a **Layer 3 static EtherChannel**.

The `mode on` command creates a static EtherChannel without using PAgP or LACP negotiation.

SW3 similarly has:

```text
interface Port-channel1
 no switchport
 ip address 23.0.0.2 255.255.255.0
```

with:

```text
channel-group 1 mode on
```

on both GigabitEthernet interfaces.

### Verification

SW2 showed:

```text
1      Po1(RU)           -      Gig0/1(P) Gig0/2(P)
```

SW3 showed:

```text
1      Po1(RU)           -      Gig0/1(P) Gig0/2(P)
```

The important indicators are:

- `R` = Layer 3 port-channel.
- `U` = port-channel is in use.
- `P` = physical interface is successfully bundled.
- `-` = no PAgP/LACP protocol because this is a static EtherChannel.

### Layer 3 Connectivity Test

The connection between SW2 and SW3 was tested with:

```text
ping 23.0.0.2
```

Result:

```text
Success rate is 80 percent (4/5)
```

This demonstrates that Layer 3 communication across the routed EtherChannel is functioning.

---

# 3. SW2 — SW1 Speed Mismatch

SW2 initially reported:

```text
%EC-5-CANNOT_BUNDLE2: Fa0/4 is not compatible with Fa0/1 and will be suspended
(speed of Fa0/4 is 10M, Fa0/1 is 100M)
```

This is an **EtherChannel member compatibility problem**.

EtherChannel member interfaces must have compatible physical characteristics. In this case:

```text
Fa0/1 = 100 Mbps
Fa0/2 = 100 Mbps
Fa0/3 = 100 Mbps
Fa0/4 = 10 Mbps
```

Fa0/4 therefore could not join the EtherChannel.

The interface was corrected with:

```text
interface fa0/4
 speed 100
```

The interface configuration should also maintain consistent EtherChannel settings with the other member interfaces.

### Important Observation

The supplied configuration showed:

```text
interface FastEthernet0/4
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 2 mode auto
 speed 100
 bandwidth 10000
```

The physical **speed** is now 100 Mbps, but the configured bandwidth remains different from the other member interfaces.

For a clean EtherChannel configuration, member interfaces should have consistent operational characteristics wherever the platform/lab requires them.

---

# 4. SW3 — SW4 LACP Problem

SW4 initially showed:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           LACP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

However, the configuration showed:

```text
interface FastEthernet0/1
 switchport mode trunk
 channel-group 1 mode passive
```

and the same passive configuration was applied to all four interfaces.

The problem is that **both sides cannot be passive** for LACP to form an EtherChannel.

LACP requires at least one side to actively negotiate.

The correction on SW4 was:

```text
interface range fa0/1 - 4
 channel-group 1 mode active
```

After the correction, SW4 showed:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           LACP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

This confirms that all four interfaces successfully joined the LACP EtherChannel.

---

# 🛠️ EtherChannel Negotiation Modes

| Protocol | Active/Negotiating Modes | Passive/Non-Initiating Mode |
|---|---|---|
| PAgP | `desirable` | `auto` |
| LACP | `active` | `passive` |
| Static | `on` | N/A |

### PAgP

For PAgP:

```text
desirable + auto
desirable + desirable
```

will form an EtherChannel.

```text
auto + auto
```

will not form one because neither side initiates negotiation.

### LACP

For LACP:

```text
active + passive
active + active
```

will form an EtherChannel.

```text
passive + passive
```

will not form one because neither side initiates negotiation.

### Static EtherChannel

Static EtherChannel uses:

```text
channel-group 1 mode on
```

No negotiation protocol is used.

Both sides must be configured consistently because there is no protocol to detect or negotiate the bundle.

---

# 🔧 Useful Troubleshooting Commands

## Check EtherChannel Summary

```text
show etherchannel summary
```

This is the primary command used in this lab.

Example:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           PAgP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

---

## Check Running Configuration

```text
show running-config
```

Look for:

```text
channel-group
```

and verify:

- Correct channel-group number
- Correct negotiation mode
- Correct switchport configuration
- Correct Layer 2 or Layer 3 configuration

---

## Check Individual Interfaces

```text
show interfaces fa0/1
```

or:

```text
show interfaces gigabitEthernet0/1
```

Check:

- Speed
- Duplex
- Interface status
- Errors
- Bandwidth
- Operational state

---

## Check Port-Channel Interface

```text
show interfaces port-channel 1
```

This verifies whether the logical EtherChannel interface is operational.

---

## Check Trunking

For Layer 2 EtherChannels:

```text
show interfaces trunk
```

Verify that the Port-channel is operating as a trunk.

---

## Check Layer 3 Interfaces

For the SW2 ↔ SW3 routed EtherChannel:

```text
show ip interface brief
```

Expected addresses include:

```text
SW2  Po1  23.0.0.1
SW3  Po1  23.0.0.2
```

---

## Test Layer 3 Connectivity

From SW2:

```text
ping 23.0.0.2
```

From SW3:

```text
ping 23.0.0.1
```

Successful replies confirm Layer 3 connectivity across the routed EtherChannel.

---

# ✅ Final Verification Checklist

Use the following checklist when completing the lab:

- [ ] SW1 ↔ SW2 is configured as a Layer 2 PAgP EtherChannel.
- [ ] SW1 uses PAgP `desirable`.
- [ ] SW2 uses a compatible PAgP mode on the SW1-facing interfaces.
- [ ] All four SW1 ↔ SW2 interfaces are bundled.
- [ ] SW1 ↔ SW2 Port-channel is operational.
- [ ] SW2 ↔ SW3 is configured as a Layer 3 EtherChannel.
- [ ] SW2 and SW3 use `no switchport` on the routed member interfaces.
- [ ] SW2 and SW3 use `channel-group 1 mode on`.
- [ ] SW2 Po1 has `23.0.0.1/24`.
- [ ] SW3 Po1 has `23.0.0.2/24`.
- [ ] Both SW2 ↔ SW3 GigabitEthernet interfaces are bundled.
- [ ] SW2 can ping `23.0.0.2`.
- [ ] SW3 ↔ SW4 is configured as a Layer 2 LACP EtherChannel.
- [ ] At least one side of the LACP link uses `active`.
- [ ] SW4 uses LACP `active`.
- [ ] All four SW3 ↔ SW4 interfaces are bundled.
- [ ] All Layer 2 EtherChannels operate as trunks.
- [ ] `show etherchannel summary` shows the expected protocol and bundled ports.
- [ ] Configurations are saved with `write memory` or `copy running-config startup-config`.

---

# 📊 Expected Final EtherChannel Status

A successful lab should produce results similar to:

### SW1

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           PAgP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

### SW2

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(RU)           -      Gi0/1(P) Gi0/2(P)
2      Po2(SU)           PAgP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

### SW3

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(RU)           -      Gi0/1(P) Gi0/2(P)
2      Po2(SU)           LACP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

### SW4

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           LACP   Fa0/1(P) Fa0/2(P) Fa0/3(P) Fa0/4(P)
```

> **Note:** Port-channel numbering is locally significant. SW1/SW2 and SW3/SW4 do not have to use the same Port-channel number for the EtherChannel to function.

---

# 🧠 Key Lessons

This lab demonstrates several common EtherChannel troubleshooting scenarios:

1. **Protocol mismatch**  
   PAgP, LACP, and static EtherChannel must be configured consistently between connected switches.

2. **Negotiation mode mismatch**  
   Two passive/non-initiating interfaces cannot establish an LACP or PAgP bundle.

3. **Speed mismatch**  
   An EtherChannel member operating at 10 Mbps cannot join a bundle whose other members operate at 100 Mbps.

4. **Layer 2 vs. Layer 3 EtherChannel**  
   Layer 2 EtherChannels use `switchport` configuration, while Layer 3 EtherChannels require `no switchport` and IP addressing on the Port-channel interface.

5. **Verification is essential**  
   `show etherchannel summary` quickly reveals whether interfaces are bundled, suspended, down, or operating under the wrong protocol.

---

## 🏁 Completion Criteria

The lab is successfully completed when:

```text
SW1 ↔ SW2 = Layer 2 PAgP EtherChannel UP
SW2 ↔ SW3 = Layer 3 Static EtherChannel UP
SW3 ↔ SW4 = Layer 2 LACP EtherChannel UP
```

All intended member interfaces should display `(P)` in:

```text
show etherchannel summary
```

and the routed SW2 ↔ SW3 EtherChannel should provide successful connectivity between:

```text
23.0.0.1 ↔ 23.0.0.2
```

Finally, save the corrected configurations on all switches:

```text
copy running-config startup-config
```

or:

```text
write memory
```
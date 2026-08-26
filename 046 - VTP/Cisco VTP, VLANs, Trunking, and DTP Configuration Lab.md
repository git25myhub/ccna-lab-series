# Cisco VTP, VLANs, Trunking, and DTP Configuration Lab

## Lab Overview

This lab focuses on configuring **VTP Version 2**, VLANs, trunk links, access ports, and **DTP** on three Cisco Catalyst 2960 switches.

The topology consists of:

- **SW1** — VTP Server
- **SW2** — VTP Transparent
- **SW3** — VTP Client

The lab also demonstrates how to manually configure trunk links and restrict the VLANs allowed across those trunks.

---

## Objectives

By completing this lab, you will:

1. Configure switch-to-switch links as trunk ports.
2. Disable Dynamic Trunking Protocol (DTP) on trunk ports.
3. Configure SW1 as a **VTP Server using Version 2**.
4. Configure SW2 as a **VTP Transparent switch using Version 2**.
5. Create VLAN 40 (Accounting) locally on SW2.
6. Configure SW3 as a **VTP Client**.
7. Create VLANs 10, 20, and 30 on SW1.
8. Assign host-facing switchports to the appropriate VLANs.
9. Disable DTP on host-facing access ports.
10. Restrict trunk links to VLANs **1, 10, and 20**.

---

## VLAN Assignment

| VLAN | Name | Purpose |
|---:|---|---|
| 1 | Default | Native/default VLAN |
| 10 | HR | Human Resources |
| 20 | Sales | Sales department |
| 30 | Engineering | Engineering department |
| 40 | Accounting | Accounting department |

> **Important:** Although VLAN 30 and VLAN 40 are configured on the switches, the trunk links are restricted to VLANs **1, 10, and 20**. Therefore, VLAN 30 and VLAN 40 traffic is not permitted across the configured trunks.

---

## VTP Configuration

| Switch | VTP Mode | VTP Version | Domain |
|---|---|---:|---|
| SW1 | Server | 2 | CCNA |
| SW2 | Transparent | 2 | CCNA |
| SW3 | Client | 2 | CCNA |

### SW1 — VTP Server

SW1 is configured as the VTP server and is responsible for creating the main VTP VLAN database.

Configured VLANs:

- VLAN 10 — HR
- VLAN 20 — Sales
- VLAN 30 — Engineering

### SW2 — VTP Transparent

SW2 operates in VTP transparent mode. VLAN 40 is created locally on SW2:

- VLAN 40 — Accounting

### SW3 — VTP Client

SW3 operates as a VTP client and receives VLAN information through VTP advertisements from the VTP domain.

---

## SW1 Configuration

### Configure the trunk

```cisco
enable
configure terminal

interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
 switchport trunk allowed vlan 1,10,20
exit
```

### Configure VTP Version 2

```cisco
vtp domain CCNA
vtp version 2
vtp mode server
```

### Create VLANs

```cisco
vlan 10
 name HR

vlan 20
 name Sales

vlan 30
 name Engineering
```

### Configure host-facing ports

#### HR — Fa0/1–Fa0/2

```cisco
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate
exit
```

#### Sales — Fa0/3

```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
exit
```

### Save the configuration

```cisco
end
write memory
```

---

## SW2 Configuration

### Configure both switch-to-switch links as trunks

```cisco
enable
configure terminal

interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 switchport nonegotiate
 switchport trunk allowed vlan 1,10,20
exit
```

### Configure VTP Transparent Mode

```cisco
vtp domain CCNA
vtp version 2
vtp mode transparent
```

### Create VLAN 40

```cisco
vlan 40
 name Accounting
```

### Configure Accounting host ports

```cisco
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 40
 switchport nonegotiate
exit
```

### Save the configuration

```cisco
end
write memory
```

---

## SW3 Configuration

### Configure the trunk

```cisco
enable
configure terminal

interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
 switchport trunk allowed vlan 1,10,20
exit
```

### Configure VTP Client Mode

```cisco
vtp domain CCNA
vtp version 2
vtp mode client
```

### Configure host-facing ports

#### HR — Fa0/1

```cisco
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate
exit
```

#### Engineering — Fa0/2–Fa0/3

```cisco
interface range FastEthernet0/2 - 3
 switchport mode access
 switchport access vlan 30
 switchport nonegotiate
exit
```

#### Sales — Fa0/4

```cisco
interface FastEthernet0/4
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
exit
```

### Save the configuration

```cisco
end
write memory
```

---

## Verification Commands

### Verify VTP status

Run on each switch:

```cisco
show vtp status
```

Expected results:

**SW1:**

```text
VTP Version                     : 2
VTP Operating Mode              : Server
VTP Domain Name                 : CCNA
VTP V2 Mode                     : Enabled
```

**SW2:**

```text
VTP Version                     : 2
VTP Operating Mode              : Transparent
VTP Domain Name                 : CCNA
VTP V2 Mode                     : Enabled
```

**SW3:**

```text
VTP Version                     : 2
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP V2 Mode                     : Enabled
```

The completed lab output on SW3 showed:

```text
VTP Version                     : 2
Configuration Revision          : 7
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP V2 Mode                     : Enabled
```

---

### Verify VLANs

```cisco
show vlan brief
```

Check that the appropriate VLANs and access ports are present.

---

### Verify trunk configuration

```cisco
show interfaces trunk
```

The trunk should show:

```text
Vlans allowed on trunk
1,10,20
```

This confirms that VLANs other than **1, 10, and 20** are filtered from the trunk.

---

### Verify individual interfaces

```cisco
show interfaces switchport
```

This can be used to verify:

- Administrative mode
- Operational mode
- Access VLAN
- Trunk VLANs
- DTP status

---

### Verify DTP status

```cisco
show interfaces switchport
```

Look for:

```text
Negotiation of Trunking: Off
```

This confirms that DTP has been disabled with:

```cisco
switchport nonegotiate
```

---

## Important Troubleshooting Note

During the initial configuration, the following messages appeared:

```text
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU on non trunk GigabitEthernet0/1 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/1 on VLAN0001. Inconsistent port type.
```

These messages indicate a **trunk mismatch**. One side of the link was operating as a trunk while the other side was not yet configured as a trunk.

The issue is resolved by ensuring that **both ends of every switch-to-switch link are manually configured with:**

```cisco
switchport mode trunk
switchport nonegotiate
```

Once both sides use the same trunk configuration, the VLAN/PVID inconsistency should clear.

---

## Key Concepts Learned

### 1. VTP Server

A VTP server can create, modify, and delete VLANs within the VTP domain.

In this lab:

```text
SW1 = VTP Server
```

---

### 2. VTP Transparent

A VTP transparent switch maintains its own local VLAN database and does not synchronize its VLAN database with VTP advertisements.

In this lab:

```text
SW2 = VTP Transparent
VLAN 40 = Accounting
```

---

### 3. VTP Client

A VTP client learns VLAN information from VTP advertisements and does not normally create VLANs locally.

In this lab:

```text
SW3 = VTP Client
```

---

### 4. Trunk Ports

Trunks carry traffic for multiple VLANs between switches.

The lab requires the trunks to carry only:

```text
VLAN 1
VLAN 10
VLAN 20
```

Configuration:

```cisco
switchport mode trunk
switchport trunk allowed vlan 1,10,20
```

---

### 5. DTP

DTP allows Cisco switches to dynamically negotiate trunking.

This lab requires DTP to be disabled:

```cisco
switchport nonegotiate
```

Trunking is therefore configured manually rather than negotiated dynamically.

---

## Final Verification Checklist

- [x] SW1 configured as VTP Server
- [x] SW1 configured for VTP Version 2
- [x] VTP domain configured as `CCNA`
- [x] VLAN 10 HR created on SW1
- [x] VLAN 20 Sales created on SW1
- [x] VLAN 30 Engineering created on SW1
- [x] SW2 configured as VTP Transparent
- [x] SW2 configured for VTP Version 2
- [x] VLAN 40 Accounting created on SW2
- [x] SW3 configured as VTP Client
- [x] Switch-to-switch links configured as trunks
- [x] DTP disabled on trunk ports
- [x] Host-facing ports configured as access ports
- [x] DTP disabled on host-facing ports
- [x] Trunks restricted to VLANs 1, 10, and 20
- [x] Configurations saved with `write memory`
- [x] VTP status verified
- [x] Trunk configuration verified

## Completion Criteria

The lab is successfully completed when:

1. SW1 operates as a **VTP Version 2 Server**.
2. SW2 operates as a **VTP Version 2 Transparent switch**.
3. SW3 operates as a **VTP Version 2 Client**.
4. VLANs 10, 20, and 30 are configured on SW1.
5. VLAN 40 is configured locally on SW2.
6. All switch-to-switch links operate as manually configured trunks.
7. DTP is disabled on the required interfaces.
8. Host ports are assigned to their correct VLANs.
9. Trunk links allow only **VLANs 1, 10, and 20**.
10. The trunk/PVID inconsistency errors are resolved.

## Platform

- **Device:** Cisco WS-C2960-24TT
- **IOS:** Cisco IOS Software, C2960-LANBASE-M
- **IOS Version:** 12.2(25)FX
- **Simulation:** Cisco Packet Tracer
- **VTP Version:** 2

---

### Useful Commands Summary

```cisco
show vtp status
show vlan brief
show interfaces trunk
show interfaces switchport
show running-config
show spanning-tree
```

These commands provide the main information required to verify the VLAN, VTP, trunking, DTP, and Spanning Tree configuration.
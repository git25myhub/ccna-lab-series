# Cisco Packet Tracer – VLAN and DTP Troubleshooting Lab

## Lab Overview

This lab focuses on configuring VLANs, manually configuring switchport modes, and disabling Dynamic Trunking Protocol (DTP) negotiation.

The network contains two VLANs:

- **VLAN 13:** PC1 and PC3
- **VLAN 24:** PC2 and PC4

The switches must use explicitly configured access and trunk ports rather than relying on DTP to negotiate port roles.

The lab is successfully completed when:

- DTP negotiation is disabled on all required switchports.
- Trunk links are manually configured.
- PC-facing ports are manually configured as access ports.
- PCs are assigned to the correct VLANs.
- VLAN traffic can traverse the trunk between switches.
- Hosts in the same VLAN have full connectivity across the network.

---

# 1. VLAN Assignment

The required VLAN membership is:

| VLAN | VLAN ID | Hosts |
|---|---:|---|
| VLAN 13 | 13 | PC1, PC3 |
| VLAN 24 | 24 | PC2, PC4 |

The expected topology logically resembles:

```text
                 Trunk
          G0/1 = 802.1Q
       ┌──────────────────┐
       │                  │
     SW1                 SW2
   ┌───────┐            ┌───────┐
   │       │            │       │
 F0/1    F0/2          F0/1    F0/2
   │       │            │       │
  PC1     PC2          PC3     PC4
 VLAN 13 VLAN 24      VLAN 13 VLAN 24
```

The trunk must carry both VLAN 13 and VLAN 24 between SW1 and SW2.

---

# 2. Disable DTP Negotiation

## What is DTP?

Dynamic Trunking Protocol (DTP) is a Cisco proprietary protocol that allows switchports to negotiate whether a link should become a trunk.

In this lab, DTP must be disabled and port modes must be manually configured.

The key command is:

```cisco
switchport nonegotiate
```

However, `switchport nonegotiate` cannot be used while the port is operating in a dynamic mode such as:

```text
dynamic auto
dynamic desirable
```

Therefore, the port must first be manually configured as either:

```cisco
switchport mode trunk
```

or:

```cisco
switchport mode access
```

---

# 3. Configure SW1

## Step 1 – Configure the Trunk Ports

The GigabitEthernet interfaces used for the switch-to-switch connections must be manually configured as trunks.

Example:

```cisco
SW1#configure terminal

SW1(config)#interface GigabitEthernet0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport nonegotiate

SW1(config-if)#interface GigabitEthernet0/2
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport nonegotiate
```

The important configuration is:

```cisco
switchport mode trunk
switchport nonegotiate
```

### Why `switchport mode trunk` comes first

Initially, the interface was configured as:

```text
Administrative Mode: dynamic auto
Negotiation of Trunking: On
```

Attempting:

```cisco
switchport nonegotiate
```

while the interface remained dynamic produced:

```text
Command rejected: Conflict between 'nonegotiate' and 'dynamic' status.
```

Changing the port to:

```cisco
switchport mode trunk
```

removes the dynamic mode and allows DTP negotiation to be disabled.

---

# 4. Configure SW1 Access Ports

The PC-facing ports should be manually configured as access ports.

For PC1 and PC2:

```cisco
SW1(config)#interface range FastEthernet0/1 - 2
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport nonegotiate
```

Assign PC1 to VLAN 13:

```cisco
SW1(config)#interface FastEthernet0/1
SW1(config-if)#switchport access vlan 13
```

Assign PC2 to VLAN 24:

```cisco
SW1(config)#interface FastEthernet0/2
SW1(config-if)#switchport access vlan 24
```

Packet Tracer automatically created the VLANs when the access VLANs were assigned:

```text
% Access VLAN does not exist. Creating vlan 13
```

and:

```text
% Access VLAN does not exist. Creating vlan 24
```

For a cleaner production-style configuration, the VLANs can also be created explicitly:

```cisco
SW1(config)#vlan 13
SW1(config-vlan)#name VLAN13
SW1(config-vlan)#exit

SW1(config)#vlan 24
SW1(config-vlan)#name VLAN24
SW1(config-vlan)#exit
```

---

# 5. Configure SW2

## Step 1 – Configure the Trunk

Configure the switch-to-switch interface as a static trunk:

```cisco
SW2#configure terminal

SW2(config)#interface GigabitEthernet0/1
SW2(config-if)#switchport mode trunk
SW2(config-if)#switchport nonegotiate
```

This ensures that the trunk does not rely on DTP negotiation.

---

# 6. Configure SW2 Access Ports

Configure the PC-facing interfaces as static access ports:

```cisco
SW2(config)#interface range FastEthernet0/1 - 2
SW2(config-if-range)#switchport mode access
SW2(config-if-range)#switchport nonegotiate
```

Assign PC3 to VLAN 13:

```cisco
SW2(config)#interface FastEthernet0/1
SW2(config-if)#switchport access vlan 13
```

Assign PC4 to VLAN 24:

```cisco
SW2(config)#interface FastEthernet0/2
SW2(config-if)#switchport access vlan 24
```

Create the VLANs explicitly if necessary:

```cisco
SW2(config)#vlan 13
SW2(config-vlan)#name VLAN13
SW2(config-vlan)#exit

SW2(config)#vlan 24
SW2(config-vlan)#name VLAN24
SW2(config-vlan)#exit
```

---

# 7. Save the Configuration

Save the configuration on both switches:

```cisco
SW1#write memory
SW2#write memory
```

or:

```cisco
copy running-config startup-config
```

Expected output:

```text
Building configuration...
[OK]
```

---

# 8. Verify VLAN Configuration

On both switches:

```cisco
show vlan brief
```

Expected VLAN membership should resemble:

### SW1

```text
VLAN  Name       Status    Ports
----  ---------  --------  -------------------------
13    VLAN13     active    Fa0/1
24    VLAN24     active    Fa0/2
```

### SW2

```text
VLAN  Name       Status    Ports
----  ---------  --------  -------------------------
13    VLAN13     active    Fa0/1
24    VLAN24     active    Fa0/2
```

The important point is:

```text
PC1 → VLAN 13
PC2 → VLAN 24
PC3 → VLAN 13
PC4 → VLAN 24
```

---

# 9. Verify Trunk Configuration

On SW1 and SW2:

```cisco
show interfaces trunk
```

The trunk interface should appear as a trunk.

For example:

```text
Port        Mode         Encapsulation  Status        Native vlan
Gi0/1       on           802.1q         trunking      1
```

The trunk should carry VLANs 13 and 24.

---

# 10. Verify DTP Is Disabled

Use:

```cisco
show interfaces GigabitEthernet0/1 switchport
```

The relevant output should show:

```text
Administrative Mode: trunk
Operational Mode: trunk
Negotiation of Trunking: Off
```

The key requirement is:

```text
Negotiation of Trunking: Off
```

For an access port, verify:

```cisco
show interfaces FastEthernet0/1 switchport
```

The port should show:

```text
Administrative Mode: static access
Operational Mode: static access
Negotiation of Trunking: Off
```

---

# 11. Understanding the Spanning-Tree Error

During the configuration process, SW2 displayed:

```text
%SPANTREE-2-RECV_PVID_ERR:
Received 802.1Q BPDU on non trunk GigabitEthernet0/1 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL:
Blocking GigabitEthernet0/1 on VLAN0001.
Inconsistent port type.
```

This occurred because one side of the switch-to-switch link was temporarily configured differently from the other side.

One switch was operating the link as a trunk while the other side was still operating as a non-trunk/dynamic port.

Once both sides were manually configured consistently as trunks:

```cisco
switchport mode trunk
```

and DTP was disabled:

```cisco
switchport nonegotiate
```

the trunk could operate correctly.

### Troubleshooting lesson

When configuring a trunk between two switches, always make sure both ends agree:

```text
SW1                          SW2
----                         ----
mode trunk      <-------->   mode trunk
nonegotiate                  nonegotiate
```

Do not leave one side as an access port while the other side is configured as a trunk.

---

# 12. Verify Host Connectivity

The final requirement is full connectivity throughout the network.

## VLAN 13

PC1 and PC3 belong to VLAN 13.

They should be able to communicate with each other.

From PC1:

```text
C:\>ping <PC3-IP-address>
```

Expected:

```text
Reply from <PC3-IP-address>
Reply from <PC3-IP-address>
Reply from <PC3-IP-address>
Reply from <PC3-IP-address>
```

---

## VLAN 24

PC2 and PC4 belong to VLAN 24.

They should be able to communicate with each other.

From PC2:

```text
C:\>ping <PC4-IP-address>
```

Expected:

```text
Reply from <PC4-IP-address>
Reply from <PC4-IP-address>
Reply from <PC4-IP-address>
Reply from <PC4-IP-address>
```

---

# 13. VLAN Connectivity Expectations

The expected Layer 2 connectivity is:

| Source | Destination | Expected |
|---|---|---|
| PC1 | PC3 | Successful |
| PC2 | PC4 | Successful |
| PC1 | PC2 | Not expected at Layer 2 |
| PC1 | PC4 | Not expected at Layer 2 |
| PC3 | PC2 | Not expected at Layer 2 |
| PC3 | PC4 | Not expected at Layer 2 |

PC1 and PC3 share VLAN 13.

PC2 and PC4 share VLAN 24.

Communication between VLAN 13 and VLAN 24 would require a Layer 3 device such as a router or multilayer switch. In this lab, the primary objective is correct VLAN segmentation and same-VLAN connectivity.

---

# 14. Verification Commands

Use the following commands when troubleshooting the lab:

### Check VLANs

```cisco
show vlan brief
```

### Check trunk status

```cisco
show interfaces trunk
```

### Check detailed switchport configuration

```cisco
show interfaces switchport
```

or:

```cisco
show interfaces GigabitEthernet0/1 switchport
```

### Check DTP status

```cisco
show dtp interface
```

### Check interface status

```cisco
show interfaces status
```

### Check spanning tree

```cisco
show spanning-tree
```

### Check configuration

```cisco
show running-config
```

---

# 15. Final Switch Configuration Example

## SW1

The relevant configuration should resemble:

```cisco
vlan 13
 name VLAN13
!
vlan 24
 name VLAN24
!
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 13
 switchport nonegotiate
!
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 24
 switchport nonegotiate
!
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet0/2
 switchport mode trunk
 switchport nonegotiate
```

## SW2

The relevant configuration should resemble:

```cisco
vlan 13
 name VLAN13
!
vlan 24
 name VLAN24
!
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 13
 switchport nonegotiate
!
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 24
 switchport nonegotiate
!
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
```

---

# 16. Final Troubleshooting Checklist

- [ ] VLAN 13 exists on SW1.
- [ ] VLAN 13 exists on SW2.
- [ ] VLAN 24 exists on SW1.
- [ ] VLAN 24 exists on SW2.
- [ ] PC1 is assigned to VLAN 13.
- [ ] PC2 is assigned to VLAN 24.
- [ ] PC3 is assigned to VLAN 13.
- [ ] PC4 is assigned to VLAN 24.
- [ ] PC-facing interfaces are manually configured as access ports.
- [ ] Switch-to-switch interfaces are manually configured as trunks.
- [ ] DTP negotiation is disabled with `switchport nonegotiate`.
- [ ] Both ends of every trunk agree on trunk mode.
- [ ] VLAN 13 is allowed across the trunk.
- [ ] VLAN 24 is allowed across the trunk.
- [ ] PC1 can ping PC3.
- [ ] PC2 can ping PC4.
- [ ] No trunk inconsistency or PVID errors remain.
- [ ] Configurations are saved.

---

# Key Takeaways

This lab demonstrates three important Cisco switching concepts:

### Static Access Ports

```cisco
switchport mode access
```

Used for end devices such as PCs.

### Static Trunk Ports

```cisco
switchport mode trunk
```

Used for links that carry multiple VLANs between switches.

### Disable DTP

```cisco
switchport nonegotiate
```

Prevents the switchport from using DTP to negotiate trunking.

The correct approach is therefore:

```text
PC Port
   ↓
switchport mode access
switchport nonegotiate
   ↓
Correct VLAN

Switch-to-Switch Link
   ↓
switchport mode trunk
switchport nonegotiate
   ↓
Carries VLAN 13 + VLAN 24
```

The lab is complete when **DTP is disabled, all switchports are manually configured, VLAN membership is correct, the trunk is operational, and same-VLAN hosts have full connectivity.**
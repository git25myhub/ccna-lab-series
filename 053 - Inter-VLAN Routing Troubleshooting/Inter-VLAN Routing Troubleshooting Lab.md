# Inter-VLAN Routing Troubleshooting Lab

## Lab Overview

Inter-VLAN routing is not functioning properly in the provided networks.

The objective of this lab is to **identify and correct one configuration error on each network device** and restore connectivity between hosts in different VLANs.

The lab contains two separate inter-VLAN routing environments:

- **SW1** — Multilayer switch providing inter-VLAN routing using SVIs.
- **R2** — Router providing inter-VLAN routing using Router-on-a-Stick.
- **SW2** — Layer 2 switch providing VLAN segmentation and trunk connectivity to R2.

The troubleshooting process involves examining interface status, VLAN assignments, routing configuration, router subinterfaces, and trunk operation.

---

# Lab Objectives

By completing this lab, you will learn how to:

- Troubleshoot inter-VLAN routing problems.
- Verify SVI status on a multilayer switch.
- Verify VLAN membership.
- Identify incorrect access VLAN assignments.
- Verify Layer 3 routing on a Cisco multilayer switch.
- Troubleshoot router-on-a-stick configurations.
- Verify 802.1Q encapsulation.
- Verify switch trunk configuration.
- Correct incorrect IP addressing on router subinterfaces.
- Test end-to-end connectivity using `ping`.

---

# Network Addressing

The networks use the following addressing scheme:

| VLAN | Network | Default Gateway | Routing Device |
|---|---|---|---|
| VLAN 10 | 10.0.1.0/24 | 10.0.1.1 | SW1 |
| VLAN 20 | 10.0.2.0/24 | 10.0.2.1 | SW1 |
| VLAN 30 | 10.0.3.0/24 | 10.0.3.1 | R2 |
| VLAN 40 | 10.0.4.0/24 | 10.0.4.1 | R2 |

Hosts should use the `.1` address in their subnet as their default gateway.

---

# Topology Concept

The lab uses two different methods of inter-VLAN routing.

```text
             SW1
       Multilayer Switch
       SVI Inter-VLAN Routing
          /           \
     VLAN 10         VLAN 20
   10.0.1.0/24     10.0.2.0/24


             R2
          Router
      Router-on-a-Stick
             |
          802.1Q
           Trunk
             |
            SW2
          /      \
     VLAN 30    VLAN 40
   10.0.3.0/24 10.0.4.0/24
```

---

# Part 1 — Troubleshooting SW1

SW1 is a Cisco Catalyst 3560 multilayer switch.

The switch is responsible for routing between:

- VLAN 10 — `10.0.1.0/24`
- VLAN 20 — `10.0.2.0/24`

The first troubleshooting command used was:

```cisco
SW1# show ip interface brief
```

The output showed:

```text
Vlan10    10.0.1.1    YES manual    up    down
Vlan20    10.0.2.1    YES manual    up    up
```

The important observation is that **VLAN 10 was `up/down`**, while VLAN 20 was `up/up`.

This indicated a problem affecting VLAN 10.

---

## Check VLAN Membership

The VLAN database was examined using:

```cisco
SW1# show vlan brief
```

The output showed:

```text
10   VLAN0010                         active
11   VLAN0011                         active    Fa0/1
20   VLAN0020                         active    Fa0/2
```

The problem was identified:

> `Fa0/1`, which should belong to VLAN 10, was incorrectly assigned to VLAN 11.

Because VLAN 10 had no active member port, the VLAN 10 SVI could not remain operational.

---

## SW1 Misconfiguration

The incorrect configuration was:

```text
Fa0/1 → VLAN 11
```

Instead, the interface needed to belong to:

```text
Fa0/1 → VLAN 10
```

---

## Fix SW1

Enter configuration mode:

```cisco
SW1# configure terminal
```

Select the affected interface:

```cisco
SW1(config)# interface fastEthernet 0/1
```

Configure it as an access port in VLAN 10:

```cisco
SW1(config-if)# switchport access vlan 10
```

The VLAN 10 SVI then changed to an operational state:

```text
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan10, changed state to up
```

The incorrect VLAN 11 was subsequently removed:

```cisco
SW1(config)# no vlan 11
```

---

## Verify SW1

Run:

```cisco
SW1# show vlan brief
```

The corrected configuration showed:

```text
10   VLAN0010                         active    Fa0/1
20   VLAN0020                         active    Fa0/2
```

The incorrect VLAN 11 was no longer present.

Verify the SVIs:

```cisco
SW1# show ip interface brief
```

The expected state is:

```text
Vlan10    10.0.1.1    YES manual    up    up
Vlan20    10.0.2.1    YES manual    up    up
```

---

# SW1 Troubleshooting Summary

| Item | Original State | Correct State |
|---|---|---|
| Fa0/1 | VLAN 11 | VLAN 10 |
| VLAN 10 SVI | Up/Down | Up/Up |
| VLAN 20 SVI | Up/Up | Up/Up |
| VLAN 11 | Incorrect VLAN | Removed |

### Root Cause

**Fa0/1 was incorrectly assigned to VLAN 11 instead of VLAN 10.**

### Fix

```cisco
interface fastEthernet 0/1
 switchport access vlan 10
```

---

# Part 2 — Troubleshooting R2

R2 is a Cisco 2911 router providing inter-VLAN routing for:

- VLAN 30
- VLAN 40

The router uses the **Router-on-a-Stick** method.

The physical interface is:

```text
GigabitEthernet0/0
```

The required subinterfaces are:

```text
G0/0.30 → VLAN 30 → 10.0.3.1/24
G0/0.40 → VLAN 40 → 10.0.4.1/24
```

---

## Examine the Running Configuration

The router configuration was inspected using:

```cisco
R2# show running-config
```

The relevant configuration initially showed:

```cisco
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.0.1 255.255.255.0

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.0.4.1 255.255.255.0
```

The problem was immediately visible.

The VLAN 30 subinterface had:

```text
10.0.0.1
```

but the VLAN 30 network should be:

```text
10.0.3.0/24
```

Therefore, the correct gateway must be:

```text
10.0.3.1
```

---

## R2 Misconfiguration

The incorrect configuration was:

```cisco
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.0.1 255.255.255.0
```

The correct configuration is:

```cisco
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.3.1 255.255.255.0
```

---

## Fix R2

Enter configuration mode:

```cisco
R2# configure terminal
```

Select the VLAN 30 subinterface:

```cisco
R2(config)# interface gigabitEthernet 0/0.30
```

Correct the IP address:

```cisco
R2(config-subif)# ip address 10.0.3.1 255.255.255.0
```

Save the configuration:

```cisco
R2(config-subif)# do write
```

---

## Verify R2

Run:

```cisco
R2# show ip interface brief
```

The expected configuration is:

```text
GigabitEthernet0/0.30    10.0.3.1    up    up
GigabitEthernet0/0.40    10.0.4.1    up    up
```

Verify the subinterface configuration:

```cisco
R2# show running-config
```

The relevant section should now be:

```cisco
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.3.1 255.255.255.0

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.0.4.1 255.255.255.0
```

---

# R2 Troubleshooting Summary

| Interface | Original IP | Correct IP | VLAN |
|---|---|---|---|
| G0/0.30 | 10.0.0.1/24 | 10.0.3.1/24 | VLAN 30 |
| G0/0.40 | 10.0.4.1/24 | 10.0.4.1/24 | VLAN 40 |

### Root Cause

**The VLAN 30 router subinterface had the wrong IP address.**

### Fix

```cisco
interface gigabitEthernet 0/0.30
 ip address 10.0.3.1 255.255.255.0
```

---

# Part 3 — Troubleshooting SW2

SW2 is a Cisco Catalyst 2960 Layer 2 switch.

It provides access connectivity for:

- VLAN 30
- VLAN 40

The switch connects to R2 using:

```text
GigabitEthernet0/1
```

This interface must be a trunk because it needs to carry both VLAN 30 and VLAN 40.

---

## Examine VLAN Configuration

The VLAN configuration was checked using:

```cisco
SW2# show vlan brief
```

The output confirmed:

```text
30   VLAN0030                         active    Fa0/1
40   VLAN0040                         active    Fa0/2
```

The access ports were correctly assigned:

```text
Fa0/1 → VLAN 30
Fa0/2 → VLAN 40
```

---

## Check the Trunk

The trunk configuration was inspected using:

```cisco
SW2# show interfaces trunk
```

Initially, no trunk information was displayed.

The running configuration showed:

```cisco
interface GigabitEthernet0/1
 switchport mode access
```

This was the problem.

The interface connecting SW2 to R2 was configured as an **access port**, but it needed to be a **trunk**.

---

## SW2 Misconfiguration

Incorrect:

```cisco
interface GigabitEthernet0/1
 switchport mode access
```

Correct:

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
```

---

## Fix SW2

Enter configuration mode:

```cisco
SW2# configure terminal
```

Select the interface:

```cisco
SW2(config)# interface gigabitEthernet 0/1
```

Configure it as a trunk:

```cisco
SW2(config-if)# switchport mode trunk
```

Save the configuration:

```cisco
SW2(config-if)# do write
```

---

## Verify the Trunk

Run:

```cisco
SW2# show interfaces trunk
```

The corrected output showed:

```text
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      1-1005

Port        Vlans allowed and active in management domain
Gig0/1      1,30,40

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      1,30,40
```

This confirms that:

- GigabitEthernet0/1 is trunking.
- 802.1Q encapsulation is being used.
- VLAN 30 is allowed.
- VLAN 40 is allowed.
- Both VLANs are active and forwarding.

---

# SW2 Troubleshooting Summary

| Item | Original State | Correct State |
|---|---|---|
| Fa0/1 | VLAN 30 | VLAN 30 |
| Fa0/2 | VLAN 40 | VLAN 40 |
| G0/1 | Access | Trunk |
| Encapsulation | — | 802.1Q |
| VLANs carried | — | 30, 40 |

### Root Cause

**The switch-to-router interface G0/1 was incorrectly configured as an access port.**

### Fix

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
```

---

# Complete Troubleshooting Summary

There was **one misconfiguration per network device**.

| Device | Misconfiguration | Fix |
|---|---|---|
| SW1 | Fa0/1 assigned to VLAN 11 instead of VLAN 10 | `switchport access vlan 10` |
| R2 | G0/0.30 had IP `10.0.0.1` instead of `10.0.3.1` | Corrected IP address |
| SW2 | G0/1 configured as an access port | `switchport mode trunk` |

---

# Final Correct Configuration

## SW1

```cisco
ip routing

interface FastEthernet0/1
 switchport access vlan 10

interface FastEthernet0/2
 switchport access vlan 20

interface Vlan10
 ip address 10.0.1.1 255.255.255.0
 no shutdown

interface Vlan20
 ip address 10.0.2.1 255.255.255.0
 no shutdown
```

---

## R2

```cisco
interface GigabitEthernet0/0
 no shutdown

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.3.1 255.255.255.0

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.0.4.1 255.255.255.0
```

---

## SW2

```cisco
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 30

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 40

interface GigabitEthernet0/1
 switchport mode trunk
```

---

# Verification Commands

After fixing the devices, the following commands can be used to verify the network.

### SW1

```cisco
show vlan brief
show ip interface brief
show ip route
```

### R2

```cisco
show ip interface brief
show running-config
show ip route
```

### SW2

```cisco
show vlan brief
show interfaces trunk
show running-config
```

---

# Connectivity Testing

Connectivity was tested from Packet Tracer PCs using `ping`.

For example:

```text
C:\> ping 10.0.2.10
```

After the network was corrected, successful replies were received:

```text
Reply from 10.0.2.10: bytes=32 time<1ms TTL=127
Reply from 10.0.2.10: bytes=32 time<1ms TTL=127
Reply from 10.0.2.10: bytes=32 time<1ms TTL=127
```

Connectivity to a VLAN 40 host was also tested:

```text
C:\> ping 10.0.4.10
```

Successful replies were received:

```text
Reply from 10.0.4.10: bytes=32 time<1ms TTL=127
Reply from 10.0.4.10: bytes=32 time=12ms TTL=127
Reply from 10.0.4.10: bytes=32 time<1ms TTL=127
```

The first ping may time out because devices are resolving ARP information. Subsequent packets succeeding indicates that connectivity has been established.

---

# Troubleshooting Methodology

This lab demonstrates a systematic approach to troubleshooting inter-VLAN routing.

The troubleshooting process was:

```text
1. Check interface status
        ↓
2. Check VLAN assignments
        ↓
3. Check SVI configuration
        ↓
4. Check router subinterfaces
        ↓
5. Check IP addressing
        ↓
6. Check trunk configuration
        ↓
7. Correct the identified misconfiguration
        ↓
8. Verify configuration
        ↓
9. Test end-to-end connectivity
```

This approach prevents random configuration changes and makes it easier to isolate the actual cause of a connectivity problem.

---

# Key Lessons

## 1. SVI Status Depends on VLAN Activity

An SVI can have an IP address and still fail to operate correctly if the associated VLAN does not have an active Layer 2 path.

In this lab:

```text
VLAN 10 SVI = 10.0.1.1
```

was initially:

```text
up/down
```

because the intended access port was assigned to the wrong VLAN.

---

## 2. Router Subinterface IP Addresses Must Match the VLAN Subnet

For VLAN 30:

```text
Network: 10.0.3.0/24
Gateway: 10.0.3.1
```

Using `10.0.0.1` as the gateway places the router interface in the wrong subnet and prevents proper communication.

---

## 3. Router-on-a-Stick Requires a Trunk

The switch interface connected to the router must carry multiple VLANs.

Therefore:

```cisco
switchport mode trunk
```

is required on the switch-side interface.

Without the trunk, VLAN-tagged traffic cannot properly reach the corresponding router subinterfaces.

---

## 4. Default Gateways Must Be Correct

Each PC must point to the Layer 3 interface responsible for its VLAN.

| VLAN | Gateway |
|---|---|
| VLAN 10 | `10.0.1.1` |
| VLAN 20 | `10.0.2.1` |
| VLAN 30 | `10.0.3.1` |
| VLAN 40 | `10.0.4.1` |

---

# Completion Criteria

The lab is successfully completed when:

- SW1 correctly assigns the appropriate access ports to VLAN 10 and VLAN 20.
- SW1's VLAN 10 and VLAN 20 SVIs are operational.
- R2 has the correct IP address on each router subinterface.
- R2 uses the correct 802.1Q VLAN IDs.
- SW2's G0/1 interface operates as an 802.1Q trunk.
- VLAN 30 and VLAN 40 are carried across the SW2-R2 trunk.
- PCs use the correct default gateways.
- Inter-VLAN communication succeeds.
- Ping tests between the appropriate hosts receive successful replies.

---

# Conclusion

This troubleshooting lab demonstrates how a single configuration error can prevent inter-VLAN communication.

Three devices contained one misconfiguration each:

1. **SW1** — incorrect VLAN assignment on `Fa0/1`.
2. **R2** — incorrect IP address on `G0/0.30`.
3. **SW2** — incorrect switchport mode on `G0/1`.

By systematically examining VLAN membership, SVI status, IP addressing, router subinterfaces, and trunk operation, all three problems were identified and corrected.

The successful ping tests confirmed that **inter-VLAN routing was restored** and that the network was operating as intended.
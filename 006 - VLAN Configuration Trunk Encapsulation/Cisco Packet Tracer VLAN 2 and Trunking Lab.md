# VLAN 2 and Trunking Lab

## Lab Objective

The objective of this lab is to understand how VLAN membership affects connectivity between PCs connected to different switches and how a **trunk link** allows VLAN traffic to cross between switches.

In this lab:

- PC2 and PC3 are assigned to **VLAN 2**.
- The connection between SW1 and SW2 is configured as an **802.1Q trunk**.
- Connectivity is tested before and after the VLAN and trunk configurations.

---

## Network Topology

```text
             SW1 ================= SW2
                  Trunk Link
                Fa0/1 ↔ Fa0/1
                /              \
              PC2              PC3
          10.0.0.2          10.0.0.3
             VLAN 2             VLAN 2
```

### Device Addressing

| Device | IP Address | VLAN | Switch Port |
|---|---|---:|---|
| PC2 | 10.0.0.2 | VLAN 2 | SW1 Fa0/3 |
| PC3 | 10.0.0.3 | VLAN 2 | SW2 Fa0/2 |
| SW1 ↔ SW2 | — | Trunk | Fa0/1 ↔ Fa0/1 |

---

# Part 1 — Test Initial Connectivity

Before making any VLAN changes, test connectivity between the PCs.

From a PC, use:

```text
C:\>ping 10.0.0.2
```

and:

```text
C:\>ping 10.0.0.3
```

The initial tests were successful.

### Initial PC2 Test

```text
Pinging 10.0.0.2 with 32 bytes of data:

Reply from 10.0.0.2: bytes=32 time=4ms TTL=128
Reply from 10.0.0.2: bytes=32 time<1ms TTL=128
Reply from 10.0.0.2: bytes=32 time<1ms TTL=128
Reply from 10.0.0.2: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Initial PC3 Test

```text
Pinging 10.0.0.3 with 32 bytes of data:

Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time=1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

At this stage, the PCs can communicate because the switch ports are initially operating in the default VLAN environment.

---

# Part 2 — Assign PC2 and PC3 to VLAN 2

PC2 is connected to SW1 Fa0/3.

Configure the interface as an access port in VLAN 2:

```cisco
SW1# configure terminal

SW1(config)# interface fastethernet 0/3
SW1(config-if)# switchport access vlan 2
SW1(config-if)# switchport mode access

SW1(config-if)# end
SW1# write memory
```

The resulting configuration on SW1 is:

```cisco
interface FastEthernet0/3
 switchport access vlan 2
 switchport mode access
```

---

## Configure PC3 on SW2

PC3 is connected to SW2 Fa0/2.

```cisco
SW2# configure terminal

SW2(config)# interface fastethernet 0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 2

SW2(config-if)# end
SW2# write memory
```

Packet Tracer reported:

```text
% Access VLAN does not exist. Creating vlan 2
```

This means the switch automatically created VLAN 2 when the interface was assigned to it.

The resulting configuration is:

```cisco
interface FastEthernet0/2
 switchport access vlan 2
 switchport mode access
```

---

# Part 3 — Test Connectivity Before Creating the Trunk

After assigning the PCs to VLAN 2, test connectivity again.

The ping from PC2 to PC3 initially failed:

```text
C:\>ping 10.0.0.3

Pinging 10.0.0.3 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

## Why Did the Ping Fail?

PC2 and PC3 are now both members of VLAN 2, but VLAN 2 has not yet been allowed to cross the connection between SW1 and SW2.

The inter-switch connection must be configured as a **trunk** so that VLAN 2 traffic can travel between the switches.

```text
PC2
 |
VLAN 2
 |
SW1
 |
 |  ❌ VLAN 2 not being carried
 |
SW2
 |
VLAN 2
 |
PC3
```

---

# Part 4 — Create a Trunk Between SW1 and SW2

The connection between the switches uses:

```text
SW1 Fa0/1 ↔ SW2 Fa0/1
```

Because the switches in this lab use an IOS version that supports configurable trunk encapsulation, **802.1Q encapsulation must first be selected**.

---

## SW1 Trunk Configuration

Initially, the following command was attempted:

```cisco
SW1(config-if)# switchport mode trunk
```

However, Packet Tracer returned:

```text
Command rejected: An interface whose trunk encapsulation is "Auto"
can not be configured to "trunk" mode.
```

This occurs because the switch is using **Auto** trunk encapsulation.

Configure 802.1Q first:

```cisco
SW1# configure terminal

SW1(config)# interface fastethernet 0/1
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk

SW1(config-if)# end
SW1# write memory
```

The final SW1 configuration is:

```cisco
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

---

## SW2 Trunk Configuration

The same issue occurred on SW2 when attempting to configure Fa0/1 as a trunk.

Configure 802.1Q encapsulation first:

```cisco
SW2# configure terminal

SW2(config)# interface fastethernet 0/1
SW2(config-if)# switchport trunk encapsulation dot1q
SW2(config-if)# switchport mode trunk

SW2(config-if)# end
SW2# write memory
```

The final configuration is:

```cisco
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

---

# Spanning Tree Warning

While SW1 was configured as a trunk and SW2 was still operating as a non-trunk interface, SW2 reported:

```text
%SPANTREE-2-RECV_PVID_ERR:
Received 802.1Q BPDU on non trunk FastEthernet0/1 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL:
Blocking FastEthernet0/1 on VLAN0001. Inconsistent port type.
```

This is a **trunk mismatch**.

SW1 was sending 802.1Q trunk traffic while SW2 Fa0/1 was not yet configured as a trunk.

Once SW2 was configured with:

```cisco
switchport trunk encapsulation dot1q
switchport mode trunk
```

the mismatch was resolved.

---

# Part 5 — Verify the Trunk

Use the following command on both switches:

```cisco
SW1# show interfaces trunk
SW2# show interfaces trunk
```

Fa0/1 should be displayed as a trunk interface.

You can also verify the running configuration:

```cisco
SW1# show running-config
SW2# show running-config
```

The important configuration should be:

### SW1

```cisco
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface FastEthernet0/3
 switchport access vlan 2
 switchport mode access
```

### SW2

```cisco
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface FastEthernet0/2
 switchport access vlan 2
 switchport mode access
```

---

# Part 6 — Test Connectivity After Trunking

After configuring the trunk on both switches, test connectivity again.

From PC2:

```text
C:\>ping 10.0.0.3
```

The final test was successful:

```text
Pinging 10.0.0.3 with 32 bytes of data:

Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Final Result

```text
PC2 → PC3
VLAN 2 → VLAN 2
SUCCESS
```

The trunk now carries VLAN 2 traffic between SW1 and SW2.

---

# Final Configuration Summary

## SW1

```cisco
hostname SW1

interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface FastEthernet0/3
 switchport access vlan 2
 switchport mode access
```

## SW2

```cisco
hostname SW2

interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface FastEthernet0/2
 switchport access vlan 2
 switchport mode access
```

---

# Connectivity Results

| Test | Before VLAN Configuration | After VLAN 2 Assignment | After Trunk |
|---|---|---|---|
| PC2 → PC3 | **SUCCESS** | **FAIL** | **SUCCESS** |
| VLAN 2 across SW1/SW2 | Not applicable | **Not working** | **Working** |

The important observation is that **putting two PCs in the same VLAN is not enough when they are connected to different switches**. The VLAN must also be carried across the link connecting those switches.

---

# Key Concepts Learned

## 1. Access Ports

PC-facing ports are configured as access ports:

```cisco
switchport mode access
```

An access port belongs to one VLAN.

---

## 2. VLAN Assignment

The command:

```cisco
switchport access vlan 2
```

places an access port into VLAN 2.

In this lab:

```text
SW1 Fa0/3 → VLAN 2 → PC2
SW2 Fa0/2 → VLAN 2 → PC3
```

---

## 3. Trunk Ports

The link between the switches must be configured as a trunk:

```cisco
switchport mode trunk
```

The trunk allows VLAN 2 traffic to pass between SW1 and SW2.

---

## 4. 802.1Q Encapsulation

On this particular switch IOS version, the trunk encapsulation had to be explicitly configured:

```cisco
switchport trunk encapsulation dot1q
```

Only after selecting 802.1Q could the interface be placed into trunk mode.

---

## 5. Trunk Mismatch

If one side is configured as a trunk while the other side is not, problems can occur.

The following messages demonstrated the mismatch:

```text
RECV_PVID_ERR
BLOCK_PVID_LOCAL
Inconsistent port type
```

Both sides of the inter-switch link should therefore be configured consistently.

---

# Conclusion

This lab demonstrated how to extend a VLAN across two switches.

Initially, PC2 and PC3 could communicate. After both PCs were assigned to VLAN 2, connectivity was lost because VLAN 2 traffic could not cross the inter-switch link.

After configuring **Fa0/1 on both SW1 and SW2 as 802.1Q trunk interfaces**, connectivity was restored.

The final communication path was:

```text
PC2
 |
Access Port
VLAN 2
 |
SW1
 |
 |========== 802.1Q Trunk ==========|
 |
SW2
 |
Access Port
VLAN 2
 |
PC3
```

The key lesson is:

> **Access ports connect end devices to a VLAN, while trunk ports carry multiple VLANs between network devices.**

For this lab, the essential commands were:

```cisco
switchport access vlan 2
switchport mode access
```

for the PC-facing interfaces, and:

```cisco
switchport trunk encapsulation dot1q
switchport mode trunk
```

for the SW1-SW2 inter-switch link.
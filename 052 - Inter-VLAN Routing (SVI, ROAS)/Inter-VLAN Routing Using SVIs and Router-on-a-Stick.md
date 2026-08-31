# Inter-VLAN Routing Using SVIs and Router-on-a-Stick

## Lab Overview

This Cisco Packet Tracer lab demonstrates two different methods of **Inter-VLAN Routing**:

1. **Switch Virtual Interfaces (SVIs)** on a multilayer switch for VLAN 10 and VLAN 20.
2. **Router-on-a-Stick** using 802.1Q subinterfaces on a router for VLAN 30 and VLAN 40.

The objective is to configure the network so that devices in different VLANs can communicate with each other through their respective Layer 3 gateways.

---

## Lab Objectives

By completing this lab, you will learn how to:

- Configure VLANs on Cisco switches.
- Configure SVIs on a multilayer switch.
- Enable Layer 3 IP routing on a Cisco multilayer switch.
- Configure default gateways for hosts in different VLANs.
- Configure a router-on-a-stick topology.
- Configure 802.1Q encapsulation on router subinterfaces.
- Configure a switch interface as an 802.1Q trunk.
- Verify inter-VLAN connectivity using `ping`.

---

## Network Requirements

The lab uses the following VLANs and addressing scheme:

| VLAN | Network | Default Gateway | Routing Method |
|---|---|---|---|
| VLAN 10 | 10.0.1.0/24 | 10.0.1.1 | SVI on SW1 |
| VLAN 20 | 10.0.2.0/24 | 10.0.2.1 | SVI on SW1 |
| VLAN 30 | 10.0.3.0/24 | 10.0.3.1 | Router-on-a-Stick |
| VLAN 40 | 10.0.4.0/24 | 10.0.4.1 | Router-on-a-Stick |

Each PC should use `.1` of its subnet as its default gateway.

For example:

- VLAN 10 hosts → `10.0.1.1`
- VLAN 20 hosts → `10.0.2.1`
- VLAN 30 hosts → `10.0.3.1`
- VLAN 40 hosts → `10.0.4.1`

---

# Part 1 — Inter-VLAN Routing Using SVIs

## SW1 Configuration

SW1 is a Cisco Catalyst 3560 multilayer switch. Unlike a Layer 2 switch, the 3560 can perform Layer 3 routing between VLANs.

First, Layer 3 routing was enabled:

```cisco
SW1> enable
SW1# configure terminal
SW1(config)# ip routing
```

### Configure VLAN 10 SVI

The VLAN 10 SVI acts as the default gateway for hosts in the `10.0.1.0/24` network.

```cisco
SW1(config)# interface vlan 10
SW1(config-if)# ip address 10.0.1.1 255.255.255.0
SW1(config-if)# no shutdown
```

### Configure VLAN 20 SVI

The VLAN 20 SVI acts as the default gateway for hosts in the `10.0.2.0/24` network.

```cisco
SW1(config)# interface vlan 20
SW1(config-if)# ip address 10.0.2.1 255.255.255.0
SW1(config-if)# no shutdown
```

### Verify VLANs

The VLAN database was verified using:

```cisco
SW1# show vlan brief
```

The resulting configuration showed:

```text
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    ...
10   VLAN0010                         active    Fa0/1
20   VLAN0020                         active    Fa0/2
```

This confirms that VLAN 10 and VLAN 20 are active and that the appropriate switch ports have been assigned.

---

## How SVI Routing Works

The multilayer switch uses the SVIs as Layer 3 interfaces.

```text
VLAN 10
10.0.1.0/24
     |
     | Gateway: 10.0.1.1
     |
    SW1
     |
     | Gateway: 10.0.2.1
     |
VLAN 20
10.0.2.0/24
```

When a host in VLAN 10 communicates with a host in VLAN 20, the traffic is sent to the VLAN 10 SVI (`10.0.1.1`).

SW1 routes the packet between the two directly connected networks and forwards it toward the VLAN 20 host.

No external router is required for communication between VLAN 10 and VLAN 20.

---

# Part 2 — Inter-VLAN Routing Using Router-on-a-Stick

VLAN 30 and VLAN 40 use a different inter-VLAN routing method.

A single physical router interface is divided into multiple **logical subinterfaces**.

This technique is known as **Router-on-a-Stick**.

---

## R2 Configuration

The physical interface connected to the switch was enabled first:

```cisco
R2> enable
R2# configure terminal
R2(config)# interface gigabitEthernet 0/0
R2(config-if)# no shutdown
```

---

## Configure VLAN 30 Subinterface

The first subinterface was configured for VLAN 30:

```cisco
R2(config)# interface gigabitEthernet 0/0.30
R2(config-subif)# encapsulation dot1Q 30
R2(config-subif)# ip address 10.0.3.1 255.255.255.0
```

The router now provides the default gateway:

```text
VLAN 30 Gateway = 10.0.3.1
```

---

## Configure VLAN 40 Subinterface

The VLAN 40 subinterface was configured as follows:

```cisco
R2(config)# interface gigabitEthernet 0/0.40
R2(config-subif)# encapsulation dot1Q 40
R2(config-subif)# ip address 10.0.4.1 255.255.255.0
```

The router now provides the default gateway:

```text
VLAN 40 Gateway = 10.0.4.1
```

---

# Switch Trunk Configuration

The switch interface connected to R2 must operate as a trunk so that traffic from multiple VLANs can travel over the same physical link.

On SW2, the appropriate interface was configured as a trunk:

```cisco
SW2> enable
SW2# configure terminal
SW2(config)# interface gigabitEthernet 0/1
SW2(config-if)# switchport mode trunk
```

The command initially attempted was:

```cisco
SW2(config-if)# int mode trunk
```

This generated an invalid-input error because the correct command is:

```cisco
switchport mode trunk
```

The trunk configuration was then successfully applied.

---

# How Router-on-a-Stick Works

The physical connection between SW2 and R2 carries traffic for both VLAN 30 and VLAN 40.

```text
                    R2
             GigabitEthernet0/0
                    |
          +---------+---------+
          |                   |
     G0/0.30              G0/0.40
   VLAN 30                  VLAN 40
 10.0.3.1/24              10.0.4.1/24
          |                   |
          +---------+---------+
                    |
                 Trunk
                    |
                   SW2
                /       \
           VLAN 30     VLAN 40
```

802.1Q VLAN tagging allows R2 to determine which VLAN each incoming frame belongs to.

For VLAN 30:

```text
VLAN ID = 30
Gateway = 10.0.3.1
```

For VLAN 40:

```text
VLAN ID = 40
Gateway = 10.0.4.1
```

The router performs the Layer 3 routing between the VLANs.

---

# Verification

## Verify SW1 Routing

Use:

```cisco
SW1# show ip route
```

The VLAN networks should appear as directly connected routes.

You can also verify the SVI status with:

```cisco
SW1# show ip interface brief
```

Expected interfaces include:

```text
Vlan10    10.0.1.1    up    up
Vlan20    10.0.2.1    up    up
```

---

## Verify R2 Subinterfaces

Use:

```cisco
R2# show ip interface brief
```

Expected results include:

```text
GigabitEthernet0/0.30    10.0.3.1    up    up
GigabitEthernet0/0.40    10.0.4.1    up    up
```

The physical interface `GigabitEthernet0/0` must also be operational.

---

## Verify the Trunk

On SW2:

```cisco
SW2# show interfaces trunk
```

The interface connected to R2 should appear as a trunk.

---

## Test Connectivity

From a PC in VLAN 20, connectivity to a host in VLAN 20 was tested:

```text
C:\> ping 10.0.2.10
```

The initial ping may show a timeout while ARP entries are being learned:

```text
Request timed out.
Reply from 10.0.2.10: bytes=32 time<1ms TTL=127
Reply from 10.0.2.10: bytes=32 time<1ms TTL=127
Reply from 10.0.2.10: bytes=32 time<1ms TTL=127
```

This is normal in Packet Tracer because the first packet may be lost while the devices resolve the destination MAC address.

A VLAN 40 connectivity test was also performed:

```text
C:\> ping 10.0.4.10
```

After ARP resolution, successful replies were received:

```text
Reply from 10.0.4.10: bytes=32 time<1ms TTL=127
Reply from 10.0.4.10: bytes=32 time=12ms TTL=127
Reply from 10.0.4.10: bytes=32 time<1ms TTL=127
```

---

# Troubleshooting Notes

If inter-VLAN communication does not work, check the following.

### 1. Verify VLAN membership

On the switch:

```cisco
show vlan brief
```

Make sure the correct access ports belong to the correct VLANs.

### 2. Verify SVI configuration

On SW1:

```cisco
show ip interface brief
```

Ensure VLAN 10 and VLAN 20 are `up/up`.

### 3. Verify Layer 3 routing

On SW1:

```cisco
show running-config | include ip routing
```

Make sure:

```cisco
ip routing
```

is configured.

### 4. Verify PC default gateways

Each PC must use the `.1` address of its subnet.

| VLAN | Default Gateway |
|---|---|
| VLAN 10 | `10.0.1.1` |
| VLAN 20 | `10.0.2.1` |
| VLAN 30 | `10.0.3.1` |
| VLAN 40 | `10.0.4.1` |

### 5. Verify the router subinterfaces

On R2:

```cisco
show ip interface brief
```

Confirm:

```text
G0/0.30    10.0.3.1    up    up
G0/0.40    10.0.4.1    up    up
```

### 6. Verify 802.1Q encapsulation

Use:

```cisco
show running-config interface gigabitEthernet 0/0.30
show running-config interface gigabitEthernet 0/0.40
```

The configurations should contain:

```cisco
encapsulation dot1Q 30
```

and:

```cisco
encapsulation dot1Q 40
```

### 7. Verify the trunk

On SW2:

```cisco
show interfaces trunk
```

The link toward R2 must be operating as a trunk.

---

# Configuration Summary

## SW1 — SVI Inter-VLAN Routing

```cisco
ip routing

interface vlan 10
 ip address 10.0.1.1 255.255.255.0
 no shutdown

interface vlan 20
 ip address 10.0.2.1 255.255.255.0
 no shutdown
```

## R2 — Router-on-a-Stick

```cisco
interface gigabitEthernet 0/0
 no shutdown

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 10.0.3.1 255.255.255.0

interface gigabitEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 10.0.4.1 255.255.255.0
```

## SW2 — Trunk

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
```

---

# Key Concepts Learned

### SVI-Based Inter-VLAN Routing

A multilayer switch can route between VLANs by creating an SVI for each VLAN.

```text
VLAN 10 → SVI 10.0.1.1
VLAN 20 → SVI 10.0.2.1
```

The switch performs the routing internally.

### Router-on-a-Stick

A router can route between multiple VLANs using subinterfaces over a single physical interface.

```text
G0/0.30 → VLAN 30 → 10.0.3.1
G0/0.40 → VLAN 40 → 10.0.4.1
```

The switch-to-router connection must be configured as a trunk.

### Default Gateway

Every PC must have the appropriate Layer 3 interface configured as its default gateway.

For this lab:

```text
VLAN 10 → 10.0.1.1
VLAN 20 → 10.0.2.1
VLAN 30 → 10.0.3.1
VLAN 40 → 10.0.4.1
```

---

# Completion Criteria

The lab is considered successfully completed when:

- VLAN 10 and VLAN 20 are configured on SW1.
- SW1 has working SVIs for VLAN 10 and VLAN 20.
- IP routing is enabled on SW1.
- VLAN 30 and VLAN 40 are configured for router-on-a-stick routing.
- R2 has correctly configured 802.1Q subinterfaces.
- The SW2-to-R2 connection operates as a trunk.
- All PCs have the correct default gateway.
- Hosts can successfully communicate across their respective VLANs.
- Inter-VLAN connectivity is verified using `ping`.

---

## Conclusion

This lab demonstrates two fundamental approaches to inter-VLAN routing.

**VLAN 10 and VLAN 20** use **SVIs on SW1**, allowing the multilayer switch to perform routing directly.

**VLAN 30 and VLAN 40** use **Router-on-a-Stick**, where R2 uses 802.1Q subinterfaces to provide gateways and route traffic between VLANs.

Understanding both methods is important when working with Cisco networks because the appropriate solution depends on the capabilities of the network devices and the design requirements of the network.
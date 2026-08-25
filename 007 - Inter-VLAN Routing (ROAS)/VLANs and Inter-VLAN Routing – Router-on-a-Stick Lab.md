# VLANs and Inter-VLAN Routing – Router-on-a-Stick

## Lab Overview

This lab demonstrates how to configure VLANs on Cisco switches and enable communication between different VLANs using **inter-VLAN routing with router-on-a-stick**.

The topology consists of:

- **R1** – Cisco 2901 Router
- **SW1** – Cisco 2960 Switch
- **SW2** – Cisco 2960 Switch
- **PC1**
- **PC2**
- **PC3**
- **PC4**

Two VLANs are used:

| VLAN | VLAN Name | Network | Gateway |
|---|---|---|---|
| VLAN 13 | VLAN 13 | 10.0.0.0/25 | 10.0.0.1 |
| VLAN 24 | VLAN 24 | 10.0.0.128/25 | 10.0.0.129 |

---

# Lab Objectives

1. Test connectivity between the PCs before VLAN configuration.
2. Assign PC1 and PC3 to VLAN 13.
3. Assign PC2 and PC4 to VLAN 24.
4. Configure trunk links between the switches.
5. Configure a trunk connection between SW1 and R1.
6. Configure router-on-a-stick using subinterfaces on R1.
7. Configure gateways for VLAN 13 and VLAN 24.
8. Test connectivity between PCs in the same and different VLANs.

---

# Initial Connectivity Test

Before configuring the VLANs and inter-VLAN routing, ping tests were performed between the PCs.

The initial results demonstrated that connectivity was not fully established between all devices.

For example:

```text
C:\>ping 10.0.0.130

Request timed out.
Reply from 10.0.0.130: bytes=32 time<1ms TTL=127
Reply from 10.0.0.130: bytes=32 time<1ms TTL=127
Reply from 10.0.0.130: bytes=32 time=2ms TTL=127

Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The first packet timeout can also occur in Packet Tracer when ARP resolution is taking place.

---

# VLAN Assignment

The PCs were divided into two VLANs.

### VLAN 13

- PC1 → VLAN 13
- PC3 → VLAN 13

### VLAN 24

- PC2 → VLAN 24
- PC4 → VLAN 24

This creates two separate Layer 2 broadcast domains.

---

# SW1 Configuration

## Configure PC1 for VLAN 13

```cisco
SW1> enable
SW1# configure terminal

SW1(config)# interface fastethernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 13
```

Packet Tracer automatically created VLAN 13 when it was assigned to the interface:

```text
% Access VLAN does not exist. Creating vlan 13
```

---

## Configure PC2 for VLAN 24

```cisco
SW1(config)# interface fastethernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 24
```

Packet Tracer automatically created VLAN 24:

```text
% Access VLAN does not exist. Creating vlan 24
```

---

# SW1 Trunk Configuration

The connection between SW1 and SW2 was configured as a trunk.

The correct interface used for the trunk was **GigabitEthernet0/1**.

```cisco
SW1(config)# interface gigabitethernet0/1
SW1(config-if)# switchport mode trunk
```

The final SW1 configuration showed:

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
```

### Important troubleshooting note

An initial attempt was made to configure FastEthernet0/2 as a trunk:

```cisco
SW1(config)# interface fastethernet0/2
SW1(config-if)# switchport mode trunk
```

This was incorrect because FastEthernet0/2 was being used as an access port for PC2.

The resulting configuration caused a spanning-tree inconsistency:

```text
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU on non trunk GigabitEthernet0/2 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/2 on VLAN0001. Inconsistent port type.
```

The incorrect trunk configuration was removed and the appropriate GigabitEthernet interface was configured as the trunk.

---

# SW1 Final Configuration

Relevant portions of the final configuration:

```cisco
interface FastEthernet0/1
 switchport access vlan 13
 switchport mode access

interface FastEthernet0/2
 switchport access vlan 24
 switchport mode access

interface GigabitEthernet0/1
 switchport mode trunk

interface GigabitEthernet0/2
 switchport mode trunk
```

The required access ports are:

- Fa0/1 → VLAN 13
- Fa0/2 → VLAN 24

The trunk toward the other network device is configured on Gi0/1.

---

# SW2 Configuration

## Configure PC3 for VLAN 13

```cisco
SW2> enable
SW2# configure terminal

SW2(config)# interface fastethernet0/1
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 13
```

---

## Configure PC4 for VLAN 24

```cisco
SW2(config)# interface fastethernet0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 24
```

---

# SW2 Trunk Configuration

The connection between SW2 and SW1 was configured on GigabitEthernet0/1:

```cisco
SW2(config)# interface gigabitethernet0/1
SW2(config-if)# switchport mode trunk
```

The final configuration confirmed:

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
```

---

# SW2 Final Configuration

Relevant portions:

```cisco
interface FastEthernet0/1
 switchport access vlan 13
 switchport mode access

interface FastEthernet0/2
 switchport access vlan 24
 switchport mode access

interface GigabitEthernet0/1
 switchport mode trunk
```

---

# Router-on-a-Stick Configuration

R1 was configured to perform inter-VLAN routing using subinterfaces on GigabitEthernet0/0.

The physical interface was enabled first:

```cisco
R1> enable
R1# configure terminal

R1(config)# interface gigabitethernet0/0
R1(config-if)# no shutdown
```

The physical interface does not receive an IP address because the IP addresses are configured on the VLAN subinterfaces.

---

# VLAN 13 Subinterface

The VLAN 13 subinterface was configured with:

- VLAN ID: 13
- Gateway: 10.0.0.1
- Subnet mask: 255.255.255.128

```cisco
R1(config)# interface gigabitethernet0/0.13
R1(config-subif)# encapsulation dot1Q 13
R1(config-subif)# ip address 10.0.0.1 255.255.255.128
```

The resulting configuration:

```cisco
interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.128
```

---

# VLAN 24 Subinterface

The VLAN 24 subinterface was configured with:

- VLAN ID: 24
- Gateway: 10.0.0.129
- Subnet mask: 255.255.255.128

```cisco
R1(config)# interface gigabitethernet0/0.24
R1(config-subif)# encapsulation dot1Q 24
R1(config-subif)# ip address 10.0.0.129 255.255.255.128
```

The resulting configuration:

```cisco
interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.0.129 255.255.255.128
```

---

# Important Router Configuration Issue

Initially, the IP address was configured on the VLAN 13 subinterface before configuring 802.1Q encapsulation:

```cisco
R1(config-subif)# ip address 10.0.0.1 255.255.255.128
```

The router returned:

```text
% Configuring IP routing on a LAN subinterface is only allowed if that
subinterface is already configured as part of an IEEE 802.10,
IEEE 802.1Q, or ISL vLAN.
```

The problem was corrected by configuring the encapsulation first:

```cisco
R1(config-subif)# encapsulation dot1q 13
R1(config-subif)# ip address 10.0.0.1 255.255.255.128
```

This demonstrates that an 802.1Q encapsulation must be associated with the subinterface before assigning the Layer 3 IP address.

---

# R1 Final Configuration

The important portion of R1's final running configuration was:

```cisco
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto

interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.128

interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.0.129 255.255.255.128
```

---

# PC IP Addressing

The PCs should use addresses within their respective VLAN subnets.

### VLAN 13

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC1 | 10.0.0.2 | 255.255.255.128 | 10.0.0.1 |
| PC3 | 10.0.0.3 | 255.255.255.128 | 10.0.0.1 |

### VLAN 24

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC2 | 10.0.0.130 | 255.255.255.128 | 10.0.0.129 |
| PC4 | 10.0.0.131 | 255.255.255.128 | 10.0.0.129 |

---

# Connectivity Verification

After completing the VLAN and router-on-a-stick configuration, connectivity was tested using ping.

## Same-VLAN Test

A ping to:

```text
10.0.0.3
```

was successful:

```text
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms connectivity between devices in VLAN 13.

---

## Inter-VLAN Test

A ping to:

```text
10.0.0.130
```

was also successful after the router-on-a-stick configuration was completed.

The first attempt produced one timeout followed by successful replies:

```text
Request timed out.

Reply from 10.0.0.130: bytes=32 time<1ms TTL=127
Reply from 10.0.0.130: bytes=32 time<1ms TTL=127
Reply from 10.0.0.130: bytes=32 time=2ms TTL=127
```

The final result was:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The initial timeout is consistent with ARP resolution in Packet Tracer. Repeating the ping should result in successful replies once the ARP tables have been populated.

---

# Verification Commands

The following commands can be used to verify the configuration.

### Check VLANs

```cisco
show vlan brief
```

Expected result:

```text
VLAN 13
  Fa0/1

VLAN 24
  Fa0/2
```

Run this command on both SW1 and SW2.

---

### Check Trunk Ports

```cisco
show interfaces trunk
```

This should confirm that the appropriate GigabitEthernet interface is operating as a trunk.

---

### Check R1 Subinterfaces

```cisco
show ip interface brief
```

Expected interfaces:

```text
GigabitEthernet0/0.13    10.0.0.1       up    up
GigabitEthernet0/0.24    10.0.0.129     up    up
```

---

### Check R1 Running Configuration

```cisco
show running-config
```

Verify:

```cisco
interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.128

interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.0.129 255.255.255.128
```

---

### Save the Configuration

On each Cisco device:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

---

# Troubleshooting Lessons

Several useful troubleshooting scenarios occurred during this lab.

### 1. Do not configure a PC access port as a trunk

Fa0/2 was initially configured as a trunk on SW1, but the port was supposed to connect to a PC in VLAN 24.

The correct configuration is:

```cisco
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 24
```

### 2. Use the correct interface for the switch-to-switch trunk

The switch-to-switch connection should use the appropriate GigabitEthernet interface rather than a PC access port.

### 3. Configure 802.1Q before the subinterface IP address

For router-on-a-stick:

```cisco
interface g0/0.13
 encapsulation dot1q 13
 ip address 10.0.0.1 255.255.255.128
```

The encapsulation identifies which VLAN's traffic should be handled by the subinterface.

### 4. The physical router interface must be enabled

The parent interface must be active:

```cisco
interface g0/0
 no shutdown
```

### 5. Check the default gateway on every PC

VLAN 13 devices should use:

```text
10.0.0.1
```

VLAN 24 devices should use:

```text
10.0.0.129
```

---

# Key Concepts Learned

## VLANs

A VLAN logically separates devices into different Layer 2 broadcast domains.

## Access Ports

Access ports connect end devices such as PCs to a specific VLAN.

Example:

```cisco
switchport mode access
switchport access vlan 13
```

## Trunk Ports

Trunk ports carry traffic from multiple VLANs over a single physical link using VLAN tagging.

Example:

```cisco
switchport mode trunk
```

## Router-on-a-Stick

Router-on-a-stick allows a single physical router interface to route traffic between multiple VLANs by using multiple subinterfaces.

In this lab:

```text
G0/0.13 → VLAN 13 → 10.0.0.1/25
G0/0.24 → VLAN 24 → 10.0.0.129/25
```

The router receives tagged VLAN traffic from the switch, routes between the VLANs, and sends the traffic back through the trunk.

---

# Final Topology Logic

```text
             R1
        G0/0 / 802.1Q
             |
             | Trunk
             |
            SW1
          /     \
       VLAN 13  VLAN 24
        PC1      PC2
          |
          | Trunk
          |
          SW2
        /     \
     VLAN 13  VLAN 24
      PC3      PC4
```

Traffic between PC1 and PC3 stays within **VLAN 13**.

Traffic between PC2 and PC4 stays within **VLAN 24**.

Traffic between VLAN 13 and VLAN 24 must pass through **R1**, where the router performs inter-VLAN routing.

---

# Conclusion

This lab successfully demonstrated VLAN segmentation, trunking, and inter-VLAN routing using the router-on-a-stick method.

The final configuration placed PC1 and PC3 in VLAN 13 and PC2 and PC4 in VLAN 24. Trunk links were configured to carry multiple VLANs, while R1 used 802.1Q subinterfaces to provide the default gateways and route traffic between the two VLANs.

The successful ping tests confirmed that the VLANs and inter-VLAN routing were functioning correctly.
# VLAN Inter-VLAN Routing Troubleshooting Lab

## Lab Overview

This lab focuses on troubleshooting **VLANs, trunking, and inter-VLAN routing** in a Cisco network.

The network contains two VLANs:

- **VLAN 13:** PC1, PC3
- **VLAN 24:** PC2, PC4

PC1's user reported that they were unable to communicate with the other PCs on the network. The objective was to identify and correct **one misconfiguration on each networking device** without changing the intended VLAN membership.

## Topology

The lab consists of:

- **1 Router:** R1
- **2 Switches:** SW1 and SW2
- **4 PCs:** PC1, PC2, PC3, PC4

### VLAN Assignment

| VLAN | Devices |
|---|---|
| VLAN 13 | PC1, PC3 |
| VLAN 24 | PC2, PC4 |

### IP Addressing

R1 provides the default gateways for both VLANs using router-on-a-stick:

| Interface | VLAN | IP Address | Subnet Mask |
|---|---:|---|---|
| G0/0.13 | 13 | 10.0.0.1 | 255.255.255.128 |
| G0/0.24 | 24 | 10.0.0.129 | 255.255.255.128 |

## Objectives

By completing this lab, you will:

- Troubleshoot VLAN connectivity problems.
- Verify access-port VLAN assignments.
- Verify trunk configurations.
- Troubleshoot router-on-a-stick configurations.
- Identify misconfigurations on multiple networking devices.
- Restore full connectivity between all PCs.
- Verify connectivity using ICMP ping tests.

## Initial Problem

PC1 was unable to communicate with other PCs on the network.

The lab contained **one misconfiguration per networking device**, requiring troubleshooting of:

- R1
- SW1
- SW2

The VLAN membership of the PCs had to remain unchanged.

## Troubleshooting Process

The troubleshooting process involved checking the configuration of each device and verifying the following:

### 1. Router Configuration

On R1, the router-on-a-stick configuration was examined.

```text
interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.128

interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.0.129 255.255.255.128
```

Both VLAN subinterfaces were configured with the appropriate 802.1Q VLAN tags and gateway addresses.

### 2. Switch Access Ports

The access ports were checked to ensure that PCs remained in their assigned VLANs.

Expected configuration:

```text
PC1 → VLAN 13
PC3 → VLAN 13

PC2 → VLAN 24
PC4 → VLAN 24
```

The access ports were configured using:

```text
switchport mode access
switchport access vlan <VLAN-ID>
```

### 3. Trunk Links

The trunk links between the networking devices were inspected.

Trunking is required so that traffic belonging to VLAN 13 and VLAN 24 can traverse the links between the switches and router.

The relevant configuration uses:

```text
switchport mode trunk
```

The troubleshooting process identified and corrected the incorrect configurations on the networking devices.

## Verification

After correcting the misconfigurations, connectivity was tested between all PCs.

The successful result should allow:

```text
PC1 → PC2
PC1 → PC3
PC1 → PC4
```

and communication in the opposite direction as well.

The lab is considered complete when **PC1 can successfully ping every other PC in the network**.

## Useful Verification Commands

### On R1

```text
show running-config
show ip interface brief
show interfaces
```

### On SW1 and SW2

```text
show running-config
show vlan brief
show interfaces trunk
show interfaces status
```

### From the PCs

Use the `ping` command to test connectivity:

```text
ping <destination-IP>
```

## Key Concepts Practiced

- VLAN configuration
- Access ports
- 802.1Q trunking
- Router-on-a-stick
- VLAN subinterfaces
- Default gateways
- Inter-VLAN routing
- Network troubleshooting
- ICMP connectivity testing
- Configuration verification

## Completion Criteria

The lab was successfully completed when:

- VLAN 13 contained PC1 and PC3.
- VLAN 24 contained PC2 and PC4.
- The required trunk links were operational.
- R1 correctly provided inter-VLAN routing.
- The misconfiguration on each networking device was corrected.
- PC1 successfully pinged PC2, PC3, and PC4.
- All PCs could communicate across the network.

## Skills Demonstrated

This lab demonstrates practical troubleshooting skills involving **Layer 2 VLANs and Layer 3 inter-VLAN routing**. It reinforces the importance of systematically checking VLAN assignments, trunk links, router subinterfaces, and gateway configuration when troubleshooting end-to-end connectivity problems.
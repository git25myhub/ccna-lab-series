# Cisco CCNA Troubleshooting Lab – VLANs, Trunks, Port Security & Inter-VLAN Routing

## Lab Objective

In this lab, **R1, SW1, and SW2** contain one or two configuration errors each.

Your task is to troubleshoot the network, identify the misconfigurations, and correct them without changing the intended network design.

The lab is successfully completed when **every PC can ping every other PC**, including communication between different VLANs.

---

## Network Configuration Requirements

The devices were originally configured according to the previous lab **#020**.

### 1. Configure Hostnames

Configure the device hostnames according to the network diagram:

- Router: `R1`
- Switch: `SW1`
- Switch: `SW2`

### 2. Configure Enable Secret

Each networking device should use:

```text
CCNA
```

as the enable secret.

---

## 3. Configure PC Access Ports

PC-facing switchports must be configured as access ports in the correct VLANs.

| VLAN | Devices |
|---|---|
| VLAN 13 | PC1, PC3 |
| VLAN 24 | PC2, PC4 |

The switchports connected to PCs should therefore have:

```text
switchport mode access
switchport access vlan <VLAN>
```

---

## 4. Configure the SW1–SW2 Trunk

Use **CDP** to determine which interfaces connect SW1 and SW2.

The link between the two switches must operate as an **802.1Q trunk**.

Useful commands include:

```text
show cdp neighbors
show interfaces trunk
show interfaces <interface> switchport
```

A trunking error can prevent VLAN traffic from crossing between SW1 and SW2.

---

## 5. Configure Port Security

All switchports connected to PCs must use port security with the following requirements:

- Port security enabled
- Sticky MAC address learning enabled
- Violation mode set to `restrict`

Expected configuration:

```text
switchport port-security
switchport port-security mac-address sticky
switchport port-security violation restrict
```

Use the following command to verify port security:

```text
show port-security
```

You can also inspect individual interfaces with:

```text
show port-security interface <interface>
```

### Troubleshooting Clue

SW1 currently reports a port-security violation on `FastEthernet0/2`:

```text
%PORT_SECURITY-2-PSECURE_VIOLATION
```

The running configuration also shows:

```text
switchport port-security mac-address AAAA.AAAA.AAAA
```

This is inconsistent with the requirement for **sticky MAC address learning**.

The incorrect manually configured secure MAC address should be removed and sticky learning enabled.

---

## 6. Configure Inter-VLAN Routing

R1 must provide inter-VLAN routing using the **router-on-a-stick** method.

The required gateway addresses are:

| VLAN | Gateway |
|---|---|
| VLAN 13 | `10.0.0.1/24` |
| VLAN 24 | `10.0.1.1/24` |

The router should use subinterfaces on `GigabitEthernet0/0`.

Expected configuration:

```text
interface GigabitEthernet0/0
 no ip address

interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.0

interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.1.1 255.255.255.0
```

### Troubleshooting Clues

The original R1 configuration contains:

```text
interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.2 255.255.255.0
```

The required gateway for VLAN 13 is:

```text
10.0.0.1
```

The VLAN 24 subinterface also contains:

```text
encapsulation dot1Q 2
```

The required VLAN is **24**, so the encapsulation should identify VLAN 24.

Correct these errors and save the configuration.

---

## 7. Verify Connectivity

After correcting the misconfigurations, test connectivity between all PCs.

Example tests include:

```text
ping 10.0.0.12
ping 10.0.0.13
ping 10.0.1.12
ping 10.0.1.14
```

The exact addresses should be verified from the network diagram.

Test both:

### Same-VLAN Connectivity

- PC1 → PC3
- PC2 → PC4

### Inter-VLAN Connectivity

- PC1 → PC2
- PC1 → PC4
- PC3 → PC2
- PC3 → PC4

Every PC should successfully ping every other PC.

---

## Troubleshooting Commands

Useful Cisco IOS commands for this lab include:

```text
show running-config
show vlan brief
show interfaces trunk
show interfaces switchport
show cdp neighbors
show port-security
show port-security interface <interface>
show mac address-table
show ip interface brief
show interfaces
```

On R1, also verify the router-on-a-stick configuration:

```text
show running-config interface gigabitEthernet0/0.13
show running-config interface gigabitEthernet0/0.24
```

---

## Misconfigurations Identified During Troubleshooting

The supplied troubleshooting output reveals several important errors to investigate:

### SW1 – FastEthernet0/2

Incorrect:

```text
switchport port-security mac-address AAAA.AAAA.AAAA
```

Required:

```text
switchport port-security mac-address sticky
```

Remove the incorrect static secure MAC address before enabling sticky learning.

### SW2 – FastEthernet0/1

The switch reports:

```text
%SPANTREE-2-RECV_PVID_ERR
%SPANTREE-2-BLOCK_PVID_LOCAL
```

The interface was configured as an access port while receiving a trunk BPDU.

The interface should be configured as a trunk:

```text
interface FastEthernet0/1
 switchport mode trunk
```

### SW2 – FastEthernet0/2

The port was initially not assigned to the correct VLAN.

It should be an access port in VLAN 13:

```text
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 13
```

### R1 – GigabitEthernet0/0.13

Incorrect gateway:

```text
ip address 10.0.0.2 255.255.255.0
```

Required:

```text
ip address 10.0.0.1 255.255.255.0
```

### R1 – GigabitEthernet0/0.24

Incorrect VLAN encapsulation:

```text
encapsulation dot1Q 2
```

Required:

```text
encapsulation dot1Q 24
```

---

## Verification Checklist

- [ ] R1 hostname is correct.
- [ ] SW1 hostname is correct.
- [ ] SW2 hostname is correct.
- [ ] Enable secret is `CCNA` on all networking devices.
- [ ] PC access ports are assigned to the correct VLANs.
- [ ] VLAN 13 contains PC1 and PC3.
- [ ] VLAN 24 contains PC2 and PC4.
- [ ] SW1–SW2 link is configured as a trunk.
- [ ] PC-facing ports have port security enabled.
- [ ] Sticky MAC learning is enabled.
- [ ] Port-security violation mode is `restrict`.
- [ ] R1 VLAN 13 subinterface uses `10.0.0.1/24`.
- [ ] R1 VLAN 13 uses `dot1Q 13`.
- [ ] R1 VLAN 24 subinterface uses `10.0.1.1/24`.
- [ ] R1 VLAN 24 uses `dot1Q 24`.
- [ ] Same-VLAN PC pings succeed.
- [ ] Inter-VLAN PC pings succeed.
- [ ] All configurations are saved with `copy running-config startup-config` or `write memory`.

---

## Completion Criteria

The lab is complete when:

```text
PC1 can ping PC2
PC1 can ping PC3
PC1 can ping PC4
PC2 can ping PC3
PC2 can ping PC4
PC3 can ping PC4
```

with successful replies and no persistent connectivity failures.

The objective is to **troubleshoot and correct the existing configuration**, not to redesign the network or change VLAN membership.
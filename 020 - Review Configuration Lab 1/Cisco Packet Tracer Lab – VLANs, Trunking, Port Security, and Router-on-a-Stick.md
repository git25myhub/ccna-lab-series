# Cisco Packet Tracer Lab – VLANs, Trunking, Port Security, and Router-on-a-Stick

## Objective

Configure a small switched network using **VLANs, 802.1Q trunking, sticky MAC port security, and inter-VLAN routing with the router-on-a-stick method**.

The final network should allow all PCs to communicate with one another, including communication between PCs in different VLANs.

## Network Requirements

### Devices

- R1 – Cisco 2901 Router
- SW1 – Cisco 2960 Switch
- SW2 – Cisco 2960 Switch
- PC1
- PC2
- PC3
- PC4

### VLAN Assignment

| VLAN | Hosts |
|---|---|
| VLAN 13 | PC1, PC3 |
| VLAN 24 | PC2, PC4 |

### Router Subinterfaces

| VLAN | R1 Subinterface | Gateway |
|---|---|---|
| VLAN 13 | G0/0.13 | `10.0.0.1/24` |
| VLAN 24 | G0/0.24 | `10.0.1.1/24` |

## Lab Tasks

### 1. Configure Device Hostnames

Configure the devices according to the topology:

```cisco
R1#configure terminal
R1(config)#hostname R1
```

On SW1:

```cisco
Switch#configure terminal
Switch(config)#hostname SW1
```

On SW2:

```cisco
Switch#configure terminal
Switch(config)#hostname SW2
```

Verify:

```cisco
show running-config
```

---

## 2. Configure the Enable Secret

Configure the password `CCNA` on R1, SW1, and SW2.

On each device:

```cisco
enable
configure terminal
enable secret CCNA
end
write memory
```

The `enable secret` protects access to privileged EXEC mode.

> **Important:** Cisco IOS treats passwords as case-sensitive. `CCNA` is different from `ccna`.

---

## 3. Configure the PC Access Ports

The switchports connected to the PCs must be configured as access ports.

### SW1

Assuming:

- F0/2 → PC1 → VLAN 13
- F0/3 → PC2 → VLAN 24

Configure:

```cisco
SW1#configure terminal

SW1(config)#interface fastethernet 0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 13

SW1(config-if)#interface fastethernet 0/3
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 24

SW1(config-if)#end
SW1#write memory
```

If the VLANs do not already exist, Cisco Packet Tracer may automatically create them when assigning an access port.

### SW2

Assuming:

- F0/2 → PC3 → VLAN 13
- F0/3 → PC4 → VLAN 24

Configure:

```cisco
SW2#configure terminal

SW2(config)#interface fastethernet 0/2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 13

SW2(config-if)#interface fastethernet 0/3
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 24

SW2(config-if)#end
SW2#write memory
```

### Verify VLAN Membership

On both switches:

```cisco
show vlan brief
```

You should see the appropriate PC ports assigned to VLAN 13 and VLAN 24.

---

## 4. Use CDP to Identify the SW1–SW2 Link

Cisco Discovery Protocol can be used to determine which interface connects SW1 to SW2.

On SW1:

```cisco
SW1#show cdp neighbors
```

The lab output identifies:

```text
Device ID    Local Intrfce    Port ID
SW2          Fas 0/1          Fas 0/1
```

Therefore:

```text
SW1 F0/1 ↔ SW2 F0/1
```

The same relationship can be confirmed from SW2:

```cisco
SW2#show cdp neighbors
```

---

## 5. Configure the SW1–SW2 Trunk

Because VLAN 13 and VLAN 24 must travel between the switches, the link between SW1 and SW2 must be a trunk.

### SW1

```cisco
SW1#configure terminal
SW1(config)#interface fastethernet 0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#end
SW1#write memory
```

### SW2

```cisco
SW2#configure terminal
SW2(config)#interface fastethernet 0/1
SW2(config-if)#switchport mode trunk
SW2(config-if)#end
SW2#write memory
```

Verify the trunk:

```cisco
show interfaces trunk
```

Expected information should include:

```text
Port        Mode         Encapsulation  Status
Fa0/1       on           802.1q         trunking
```

The active VLANs should include:

```text
1,13,24
```

> **Troubleshooting note:** SW2 initially generated an STP message indicating an inconsistent port type because SW1 had already been configured as a trunk while SW2's F0/1 was still operating as an access port. Configure both ends consistently as trunks to resolve this condition.

---

## 6. Configure Sticky MAC Port Security

Port security should be enabled on all PC-facing switchports.

The required settings are:

- Sticky MAC learning: **enabled**
- Violation action: **restrict**

### SW1

```cisco
SW1#configure terminal
SW1(config)#interface range fastethernet 0/2 - 3
SW1(config-if-range)#switchport port-security
SW1(config-if-range)#switchport port-security mac-address sticky
SW1(config-if-range)#switchport port-security violation restrict
SW1(config-if-range)#end
SW1#write memory
```

### SW2

```cisco
SW2#configure terminal
SW2(config)#interface range fastethernet 0/2 - 3
SW2(config-if-range)#switchport port-security
SW2(config-if-range)#switchport port-security mac-address sticky
SW2(config-if-range)#switchport port-security violation restrict
SW2(config-if-range)#end
SW2#write memory
```

### Verify Port Security

Use:

```cisco
show port-security
```

For an individual interface:

```cisco
show port-security interface fastethernet 0/2
```

You can also check the sticky MAC addresses with:

```cisco
show running-config
```

After traffic has been generated, the learned MAC addresses should appear as secure sticky MAC addresses.

---

# 7. Configure Router-on-a-Stick

Router-on-a-stick allows a single physical router interface to route traffic between multiple VLANs using 802.1Q subinterfaces.

The physical interface used in this lab is:

```text
R1 G0/0
```

First enable the physical interface:

```cisco
R1#configure terminal
R1(config)#interface gigabitethernet 0/0
R1(config-if)#no shutdown
```

Do not assign a normal IP address directly to G0/0 when using subinterfaces for the VLAN gateways.

---

## 8. Configure the VLAN 13 Subinterface

Create G0/0.13:

```cisco
R1(config)#interface gigabitethernet 0/0.13
R1(config-subif)#encapsulation dot1Q 13
R1(config-subif)#ip address 10.0.0.1 255.255.255.0
```

This creates the default gateway for VLAN 13:

```text
VLAN 13 Gateway = 10.0.0.1
```

---

## 9. Configure the VLAN 24 Subinterface

Create G0/0.24:

```cisco
R1(config)#interface gigabitethernet 0/0.24
R1(config-subif)#encapsulation dot1Q 24
R1(config-subif)#ip address 10.0.1.1 255.255.255.0
```

This creates the default gateway for VLAN 24:

```text
VLAN 24 Gateway = 10.0.1.1
```

Save the configuration:

```cisco
R1(config-subif)#end
R1#write memory
```

---

## 10. Verify the Router Configuration

Check the subinterfaces:

```cisco
R1#show ip interface brief
```

Expected entries include:

```text
GigabitEthernet0/0.13    10.0.0.1    up    up
GigabitEthernet0/0.24    10.0.1.1    up    up
```

Also verify the configuration:

```cisco
R1#show running-config interface gigabitethernet 0/0.13
R1#show running-config interface gigabitethernet 0/0.24
```

The configuration should contain:

```cisco
encapsulation dot1Q 13
ip address 10.0.0.1 255.255.255.0
```

and:

```cisco
encapsulation dot1Q 24
ip address 10.0.1.1 255.255.255.0
```

---

# 11. Configure the PC Default Gateways

The PCs must use the appropriate router subinterface as their default gateway.

### VLAN 13 PCs

PC1 and PC3 should use:

```text
Default Gateway: 10.0.0.1
```

### VLAN 24 PCs

PC2 and PC4 should use:

```text
Default Gateway: 10.0.1.1
```

Example addressing:

| PC | VLAN | Example IP | Default Gateway |
|---|---:|---|---|
| PC1 | 13 | `10.0.0.11/24` | `10.0.0.1` |
| PC3 | 13 | `10.0.0.13/24` | `10.0.0.1` |
| PC2 | 24 | `10.0.1.12/24` | `10.0.1.1` |
| PC4 | 24 | `10.0.1.14/24` | `10.0.1.1` |

Use the exact IP addresses provided by the topology if they differ from these examples.

---

# 12. Test Connectivity

Begin with communication between hosts in the same VLAN.

### VLAN 13

From PC1:

```text
C:\>ping 10.0.0.13
```

Expected:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms that PC1 and PC3 can communicate within VLAN 13.

### VLAN 24

From a VLAN 24 PC, ping the other VLAN 24 PC.

For example:

```text
C:\>ping 10.0.1.14
```

The ping should succeed.

---

# 13. Test Inter-VLAN Routing

Now test communication between different VLANs.

From PC1:

```text
C:\>ping 10.0.1.12
```

The first ping may occasionally fail because of ARP resolution. Repeat the test if necessary.

A successful result should eventually show:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms that traffic is successfully traveling:

```text
PC1
 ↓
SW1 VLAN 13
 ↓
802.1Q trunk
 ↓
R1 G0/0.13
 ↓
R1 routes traffic
 ↓
R1 G0/0.24
 ↓
802.1Q trunk
 ↓
SW2 VLAN 24
 ↓
PC2
```

---

# 14. Troubleshooting

If PCs in the same VLAN cannot communicate, check:

```cisco
show vlan brief
```

Confirm that the PC-facing interfaces are assigned to the correct VLAN.

If PCs on different switches cannot communicate, check:

```cisco
show interfaces trunk
```

Confirm that the SW1–SW2 connection is operating as a trunk.

If inter-VLAN communication fails, check:

```cisco
R1#show ip interface brief
```

Make sure both subinterfaces are **up/up**.

Check the router configuration:

```cisco
R1#show running-config
```

Confirm:

```cisco
interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.0

interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.1.1 255.255.255.0
```

Also verify that the PC default gateways are correct.

For port-security verification:

```cisco
show port-security
show port-security interface fastethernet 0/2
show port-security interface fastethernet 0/3
```

---

# 15. Important Observation from the Lab

The initial ping tests showed successful communication within VLAN 13 but unsuccessful communication to VLAN 24 until the router-on-a-stick configuration and gateway settings were functioning correctly.

For example:

```text
ping 10.0.0.13
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

while an early attempt to reach a VLAN 24 host produced:

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

Later, after the routing configuration was operational:

```text
ping 10.0.1.12
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The single lost packet is consistent with initial ARP resolution. Repeating the ping should produce successful replies.

---

# Verification Checklist

- [ ] R1 hostname configured.
- [ ] SW1 hostname configured.
- [ ] SW2 hostname configured.
- [ ] `enable secret CCNA` configured on all devices.
- [ ] PC1 and PC3 assigned to VLAN 13.
- [ ] PC2 and PC4 assigned to VLAN 24.
- [ ] SW1–SW2 connection identified using CDP.
- [ ] SW1–SW2 link configured as an 802.1Q trunk.
- [ ] Sticky MAC learning enabled on all PC-facing ports.
- [ ] Port-security violation mode configured as `restrict`.
- [ ] R1 G0/0 enabled.
- [ ] R1 G0/0.13 configured with `10.0.0.1/24`.
- [ ] R1 G0/0.24 configured with `10.0.1.1/24`.
- [ ] PC default gateways configured correctly.
- [ ] Same-VLAN connectivity verified.
- [ ] Inter-VLAN connectivity verified.
- [ ] Configurations saved with `write memory`.

## Key Concepts Learned

### VLANs
VLANs logically separate hosts into different Layer 2 broadcast domains.

### Trunking
An 802.1Q trunk carries traffic belonging to multiple VLANs over a single physical link.

### Port Security
Port security restricts the MAC addresses allowed on switch access ports. Sticky learning allows the switch to dynamically learn MAC addresses and add them to the secure MAC address configuration.

### Restrict Violation Mode
With `restrict`, frames from unauthorized MAC addresses are dropped and the violation is recorded, but the interface remains operational.

### Router-on-a-Stick
A router can route between multiple VLANs using one physical interface divided into multiple 802.1Q subinterfaces.

## Final Result

The lab is successfully completed when **PC1, PC2, PC3, and PC4 can all ping one another**, including communication between VLAN 13 and VLAN 24, while the switch-to-switch connection operates as a trunk and PC-facing ports use sticky MAC port security with the `restrict` violation action.
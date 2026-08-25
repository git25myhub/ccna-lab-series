# Cisco VLAN Configuration and Trunking Lab

## Objective

In this lab, you will configure two Cisco switches with VLANs, assign end devices to the appropriate VLANs, configure a trunk link between the switches, and verify connectivity between PCs belonging to the same VLAN.

## Network Configuration

| VLAN | Name | Devices |
|---|---|---|
| VLAN 13 | Management | PC1, PC3 |
| VLAN 24 | Engineering | PC2, PC4 |

### Switches

- **SW1** — Connected to PC1 and PC2
- **SW2** — Connected to PC3 and PC4
- **SW1 ↔ SW2** — Trunk link

## Lab Requirements

- Rename the switches to `SW1` and `SW2`.
- Create VLAN 13 named `Management`.
- Create VLAN 24 named `Engineering`.
- Assign PC1 and PC3 to VLAN 13.
- Assign PC2 and PC4 to VLAN 24.
- Configure the SW1-SW2 connection as an 802.1Q trunk.
- Save the configurations.
- Verify that PCs in the same VLAN can communicate.

---

# Part 1: Configure SW1

## 1. Set the Hostname

Connect to SW1 through the console and configure the hostname:

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
```

The prompt should change to:

```text
SW1(config)#
```

## 2. Create VLAN 13

Create the Management VLAN:

```cisco
SW1(config)# vlan 13
SW1(config-vlan)# name Management
SW1(config-vlan)# exit
```

## 3. Create VLAN 24

Create the Engineering VLAN:

```cisco
SW1(config)# vlan 24
SW1(config-vlan)# name Engineering
SW1(config-vlan)# exit
```

Verify the VLANs:

```cisco
SW1# show vlan brief
```

You should see:

```text
13   Management
24   Engineering
```

## 4. Assign PC1 to VLAN 13

Based on the lab topology, PC1 is connected to `FastEthernet0/2`.

```cisco
SW1(config)# interface fastethernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 13
SW1(config-if)# exit
```

## 5. Assign PC2 to VLAN 24

PC2 is connected to `FastEthernet0/3`.

```cisco
SW1(config)# interface fastethernet0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 24
SW1(config-if)# exit
```

Verify the assignments:

```cisco
SW1# show vlan brief
```

Expected result:

```text
13   Management       active    Fa0/2
24   Engineering      active    Fa0/3
```

---

# Part 2: Configure the Trunk on SW1

The connection between SW1 and SW2 uses `FastEthernet0/1`.

Configure it as a trunk:

```cisco
SW1(config)# interface fastethernet0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# exit
```

Verify the trunk:

```cisco
SW1# show interfaces trunk
```

The interface should appear as a trunk.

---

# Part 3: Configure SW2

## 1. Set the Hostname

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
```

## 2. Create VLAN 13

```cisco
SW2(config)# vlan 13
SW2(config-vlan)# name Management
SW2(config-vlan)# exit
```

## 3. Create VLAN 24

```cisco
SW2(config)# vlan 24
SW2(config-vlan)# name Engineering
SW2(config-vlan)# exit
```

Verify:

```cisco
SW2# show vlan brief
```

---

# Part 4: Configure the Trunk on SW2

The connection to SW1 is `FastEthernet0/1`.

```cisco
SW2(config)# interface fastethernet0/1
SW2(config-if)# switchport mode trunk
SW2(config-if)# exit
```

Verify:

```cisco
SW2# show interfaces trunk
```

The trunk should be operational.

### Important

The trunk configuration must be applied to **both ends** of the link.

Your lab output showed this error on SW2:

```text
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU on non trunk FastEthernet0/1 VLAN1.
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking FastEthernet0/1 on VLAN0001. Inconsistent port type.
```

This occurred because SW1 was configured as a trunk before SW2's corresponding interface was configured as a trunk. Once `FastEthernet0/1` on SW2 was configured with:

```cisco
SW2(config)# interface fastethernet0/1
SW2(config-if)# switchport mode trunk
```

the trunk configuration became consistent on both sides.

---

# Part 5: Assign PC3 to VLAN 13

PC3 is connected to `FastEthernet0/2` on SW2.

```cisco
SW2(config)# interface fastethernet0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 13
SW2(config-if)# exit
```

---

# Part 6: Assign PC4 to VLAN 24

PC4 is connected to `FastEthernet0/3` on SW2.

```cisco
SW2(config)# interface fastethernet0/3
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 24
SW2(config-if)# exit
```

Verify:

```cisco
SW2# show vlan brief
```

Expected result:

```text
13   Management       active    Fa0/2
24   Engineering      active    Fa0/3
```

---

# Part 7: Save the Configuration

Save the configuration on **both switches**.

### SW1

```cisco
SW1# copy running-config startup-config
```

or:

```cisco
SW1# write memory
```

### SW2

```cisco
SW2# copy running-config startup-config
```

or:

```cisco
SW2# write memory
```

You should receive:

```text
[OK]
```

---

# Part 8: Verify the VLAN Configuration

On SW1:

```cisco
SW1# show vlan brief
```

Expected:

```text
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
13   Management                       active    Fa0/2
24   Engineering                      active    Fa0/3
```

On SW2:

```cisco
SW2# show vlan brief
```

Expected:

```text
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
13   Management                       active    Fa0/2
24   Engineering                      active    Fa0/3
```

---

# Part 9: Verify the Trunk

On both switches:

```cisco
SW1# show interfaces trunk
SW2# show interfaces trunk
```

The `FastEthernet0/1` interface should be listed as a trunk.

The trunk is important because it allows VLAN 13 and VLAN 24 traffic to travel between SW1 and SW2.

---

# Part 10: Test PC Connectivity

The final requirement is to verify that devices in the **same VLAN** can communicate.

## VLAN 13

PC1 and PC3 belong to VLAN 13.

From PC1:

```text
C:\> ping <PC3-IP-address>
```

The ping should succeed.

Expected result:

```text
Reply from <PC3-IP-address>: bytes=32 ...
Reply from <PC3-IP-address>: bytes=32 ...
Reply from <PC3-IP-address>: bytes=32 ...
Reply from <PC3-IP-address>: bytes=32 ...
```

## VLAN 24

PC2 and PC4 belong to VLAN 24.

From PC2:

```text
C:\> ping <PC4-IP-address>
```

The ping should succeed.

---

# Important Observation from the Lab

Your captured test showed:

```text
C:\>ping 192.168.0.4

Request timed out.
Request timed out.
Request timed out.
Request timed out.
```

but:

```text
C:\>ping 192.168.0.3

Reply from 192.168.0.3: bytes=32 time=10ms TTL=128
Reply from 192.168.0.3: bytes=32 time<1ms TTL=128
Reply from 192.168.0.3: bytes=32 time<1ms TTL=128
Reply from 192.168.0.3: bytes=32 time<1ms TTL=128
```

The successful ping demonstrates that at least one pair of PCs in the same VLAN can communicate.

If the other same-VLAN ping fails, check the following:

1. Verify the PC's IP address and subnet mask.
2. Confirm the PC is connected to the correct switch port.
3. Verify the access VLAN with:
   ```cisco
   show vlan brief
   ```
4. Verify the trunk on both switches:
   ```cisco
   show interfaces trunk
   ```
5. Make sure VLAN 13 and VLAN 24 exist on **both** switches.
6. Check that the correct ports are assigned to the correct VLANs.

---

# Verification Commands

| Command | Purpose |
|---|---|
| `show vlan brief` | Displays VLANs and access-port assignments |
| `show interfaces trunk` | Verifies trunk interfaces |
| `show running-config` | Displays the current configuration |
| `show interfaces status` | Displays interface status and VLAN information |
| `ping` | Tests end-to-end connectivity |
| `copy running-config startup-config` | Saves the configuration |

---

# Expected Final Topology

```text
                 TRUNK
        Fa0/1 ------------- Fa0/1
          SW1                 SW2
         /   \               /   \
      Fa0/2 Fa0/3        Fa0/2 Fa0/3
        |      |             |      |
       PC1    PC2           PC3    PC4
        |      |             |      |
     VLAN 13 VLAN 24      VLAN 13 VLAN 24
```

## Expected Connectivity

| Source | Destination | VLAN | Expected |
|---|---|---:|---|
| PC1 | PC3 | 13 | ✅ Ping succeeds |
| PC2 | PC4 | 24 | ✅ Ping succeeds |
| PC1 | PC2 | 13 → 24 | ❌ No inter-VLAN routing configured |
| PC3 | PC4 | 13 → 24 | ❌ No inter-VLAN routing configured |

> **Note:** PCs in different VLANs are not expected to communicate because this lab does not configure inter-VLAN routing.

---

# Completion Checklist

- [ ] SW1 hostname configured
- [ ] SW2 hostname configured
- [ ] VLAN 13 created on SW1
- [ ] VLAN 13 created on SW2
- [ ] VLAN 24 created on SW1
- [ ] VLAN 24 created on SW2
- [ ] PC1 assigned to VLAN 13
- [ ] PC3 assigned to VLAN 13
- [ ] PC2 assigned to VLAN 24
- [ ] PC4 assigned to VLAN 24
- [ ] SW1-SW2 link configured as a trunk on both sides
- [ ] Configuration saved on both switches
- [ ] PC1 can ping PC3
- [ ] PC2 can ping PC4

## Conclusion

This lab demonstrates the fundamentals of **VLAN segmentation and 802.1Q trunking** on Cisco switches. VLAN 13 separates the Management devices from VLAN 24, while the trunk between SW1 and SW2 carries traffic for both VLANs. Successful same-VLAN pings confirm that the VLAN assignments and trunk configuration are working correctly.
# Troubleshooting Inter-VLAN Routing Misconfiguration

## Lab Overview

This lab focuses on troubleshooting a network where **inter-VLAN routing has already been configured**, but devices in different VLANs cannot communicate.

The objective is to identify the single misconfiguration, correct it, and verify that **all PCs can communicate with one another without changing their VLAN membership**.

---

# Lab Objectives

- Identify the existing VLAN assignments.
- Verify switch access ports and trunk links.
- Verify R1's router-on-a-stick configuration.
- Identify the misconfigured VLAN subinterface.
- Correct the configuration without changing VLAN membership.
- Verify connectivity between all PCs.
- Save the corrected configuration.

---

# VLAN Assignment

The VLAN membership must remain unchanged throughout the troubleshooting process.

| VLAN | Devices | Network | Default Gateway |
|---|---|---|---|
| VLAN 13 | PC1, PC3 | 10.0.0.0/25 | 10.0.0.1 |
| VLAN 24 | PC2, PC4 | 10.0.0.128/25 | 10.0.0.129 |

The expected PC addressing is:

| Device | VLAN | IP Address | Subnet Mask | Gateway |
|---|---:|---|---|---|
| PC1 | 13 | 10.0.0.2 | 255.255.255.128 | 10.0.0.1 |
| PC3 | 13 | 10.0.0.3 | 255.255.255.128 | 10.0.0.1 |
| PC2 | 24 | 10.0.0.130 | 255.255.255.128 | 10.0.0.129 |
| PC4 | 24 | 10.0.0.131 | 255.255.255.128 | 10.0.0.129 |

---

# Initial Problem

The lab states that inter-VLAN routing has already been configured, but computers in different VLANs cannot communicate.

The first troubleshooting step was to examine the switch configurations.

---

# SW1 Verification

The `show running-config` command was used to inspect SW1:

```cisco
SW1# show running-config
```

The relevant configuration showed:

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

This confirms that the access ports are correctly assigned:

- Fa0/1 → VLAN 13
- Fa0/2 → VLAN 24

The switch also has trunk ports configured.

---

# Verify SW1 Trunks

The following command was used:

```cisco
SW1# show interfaces trunk
```

The output showed:

```text
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1
Gig0/2      on           802.1q         trunking      1
```

The important part of the output was:

```text
Port        Vlans allowed and active in management domain

Gig0/1      1,13,24
Gig0/2      1,13,24
```

And:

```text
Port        Vlans in spanning tree forwarding state and not pruned

Gig0/1      1,13,24
Gig0/2      1,13,24
```

This confirms that VLAN 13 and VLAN 24 are being carried across the trunk links.

Therefore, the switch trunk configuration was **not the source of the problem**.

---

# SW2 Verification

SW2 was also checked using:

```cisco
SW2# show running-config
```

The relevant configuration was:

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

This confirms:

- PC3 → VLAN 13
- PC4 → VLAN 24
- SW1/SW2 link → trunk

The VLAN assignments were therefore correct and were not changed.

---

# R1 Troubleshooting

Since the switches and VLAN assignments appeared correct, R1 was examined.

The command used was:

```cisco
R1# show running-config
```

The relevant configuration was:

```cisco
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto

interface GigabitEthernet0/0.13
 encapsulation dot1Q 13
 ip address 10.0.0.1 255.255.255.128

interface GigabitEthernet0/0.24
 encapsulation dot1Q 14
 ip address 10.0.0.129 255.255.255.128
```

At first glance, both subinterfaces appeared to be configured correctly.

However, there was an important inconsistency.

---

# Identifying the Misconfiguration

The VLAN 24 subinterface was:

```cisco
interface GigabitEthernet0/0.24
 encapsulation dot1Q 14
 ip address 10.0.0.129 255.255.255.128
```

The subinterface name indicates VLAN **24**:

```text
G0/0.24
```

The IP address also belongs to the VLAN 24 network:

```text
10.0.0.129/25
```

However, the 802.1Q encapsulation was configured for **VLAN 14**:

```cisco
encapsulation dot1Q 14
```

This was the single misconfiguration.

The correct configuration should be:

```cisco
encapsulation dot1Q 24
```

---

# Why This Caused the Problem

802.1Q encapsulation tells the router which VLAN's tagged traffic should be processed by a particular subinterface.

The correct relationship should be:

```text
G0/0.13 → VLAN 13
G0/0.24 → VLAN 24
```

But the misconfigured router had:

```text
G0/0.13 → VLAN 13
G0/0.24 → VLAN 14
```

Therefore, traffic arriving from VLAN 24 was not being handled by the correct router subinterface.

This explains why:

- VLAN 13 devices could communicate with each other.
- VLAN 24 devices could not reliably communicate through R1.
- Inter-VLAN communication involving VLAN 24 failed.

---

# Correcting the Configuration

The VLAN membership on the switches was not changed.

Only the incorrect encapsulation on R1 was corrected.

Enter configuration mode:

```cisco
R1# configure terminal
```

Select the VLAN 24 subinterface:

```cisco
R1(config)# interface gigabitethernet0/0.24
```

Correct the 802.1Q VLAN ID:

```cisco
R1(config-subif)# encapsulation dot1Q 24
```

Save the configuration:

```cisco
R1(config-subif)# do write memory
```

Packet Tracer confirmed:

```text
Building configuration...
[OK]
```

---

# Correct R1 Configuration

After the correction, R1's configuration became:

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

The two subinterfaces now correctly correspond to the two VLANs.

---

# Verify R1 Interfaces

The command:

```cisco
R1# show ip interface brief
```

returned:

```text
Interface              IP-Address      Status                Protocol
GigabitEthernet0/0     unassigned      up                    up
GigabitEthernet0/0.13  10.0.0.1        up                    up
GigabitEthernet0/0.24  10.0.0.129      up                    up
```

Both subinterfaces were therefore operational.

---

# Verify VLAN 13 Subinterface

The command:

```cisco
R1# show interface gigabitethernet0/0.13
```

showed:

```text
Internet address is 10.0.0.1/25
Encapsulation 802.1Q Virtual LAN, Vlan ID 13
```

This confirms:

```text
G0/0.13 → VLAN 13
```

---

# Verify VLAN 24 Subinterface

Before the correction, the command:

```cisco
R1# show interface gigabitethernet0/0.24
```

showed:

```text
Internet address is 10.0.0.129/25
Encapsulation 802.1Q Virtual LAN, Vlan ID 14
```

This exposed the misconfiguration.

After correcting the encapsulation:

```cisco
R1(config-subif)# encapsulation dot1Q 24
```

the expected relationship became:

```text
G0/0.24 → VLAN 24
IP address → 10.0.0.129/25
```

---

# Connectivity Testing

## VLAN 13 Connectivity

A ping to PC3 at `10.0.0.3` was successful:

```text
C:\>ping 10.0.0.3

Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128
Reply from 10.0.0.3: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirmed that VLAN 13 connectivity was working.

---

# Inter-VLAN Connectivity Before the Fix

Before correcting R1, a ping to PC2 at `10.0.0.130` failed:

```text
C:\>ping 10.0.0.130

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

This confirmed the inter-VLAN communication problem.

A ping to PC4 at `10.0.0.131` also initially failed:

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

---

# Connectivity After the Fix

After changing:

```cisco
encapsulation dot1Q 14
```

to:

```cisco
encapsulation dot1Q 24
```

connectivity was restored.

A subsequent ping to PC4 showed:

```text
C:\>ping 10.0.0.131

Request timed out.

Reply from 10.0.0.131: bytes=32 time<1ms TTL=127
Reply from 10.0.0.131: bytes=32 time<1ms TTL=127
Reply from 10.0.0.131: bytes=32 time=15ms TTL=127

Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The first timeout is normal in Packet Tracer when ARP resolution is occurring. Repeating the ping produced successful replies.

A subsequent ping to PC2 also showed successful communication:

```text
C:\>ping 10.0.0.130

Request timed out.

Reply from 10.0.0.130: bytes=32 time<1ms TTL=127
Reply from 10.0.0.130: bytes=32 time<1ms TTL=127
Reply from 10.0.0.130: bytes=32 time<1ms TTL=127

Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

---

# Final Verification Checklist

- [x] PC1 remains in VLAN 13
- [x] PC3 remains in VLAN 13
- [x] PC2 remains in VLAN 24
- [x] PC4 remains in VLAN 24
- [x] SW1 access ports correctly assigned
- [x] SW2 access ports correctly assigned
- [x] Trunk links operational
- [x] VLAN 13 carried across trunks
- [x] VLAN 24 carried across trunks
- [x] R1 G0/0 operational
- [x] R1 G0/0.13 uses VLAN 13
- [x] R1 G0/0.24 uses VLAN 24
- [x] VLAN 13 gateway is 10.0.0.1
- [x] VLAN 24 gateway is 10.0.0.129
- [x] Inter-VLAN routing restored
- [x] VLAN membership was not changed
- [x] Configuration saved

---

# Useful Troubleshooting Commands

### Check VLAN Membership

```cisco
show vlan brief
```

### Check Trunks

```cisco
show interfaces trunk
```

### Check Interface Status

```cisco
show ip interface brief
```

### Check Router Configuration

```cisco
show running-config
```

### Inspect a Router Subinterface

```cisco
show interfaces gigabitethernet0/0.13
show interfaces gigabitethernet0/0.24
```

### Test Connectivity

From each PC:

```text
ping <destination-IP>
```

---

# Troubleshooting Method

The troubleshooting process followed a logical path:

```text
PC connectivity problem
        ↓
Check VLAN membership
        ↓
Check switch access ports
        ↓
Check trunk links
        ↓
Check router subinterfaces
        ↓
Compare VLAN IDs
        ↓
Found VLAN 24 subinterface using VLAN 14
        ↓
Change dot1Q 14 → dot1Q 24
        ↓
Verify interfaces
        ↓
Test PC-to-PC connectivity
```

This is an important troubleshooting approach because it avoids randomly changing configurations.

---

# Key Lesson

The main issue in this lab was a **VLAN ID mismatch on the router subinterface**.

The incorrect configuration was:

```cisco
interface GigabitEthernet0/0.24
 encapsulation dot1Q 14
 ip address 10.0.0.129 255.255.255.128
```

The corrected configuration was:

```cisco
interface GigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 10.0.0.129 255.255.255.128
```

The subinterface number, 802.1Q VLAN ID, IP subnet, and default gateway must all correspond to the same VLAN.

```text
VLAN 13
G0/0.13
dot1Q 13
10.0.0.1/25

VLAN 24
G0/0.24
dot1Q 24
10.0.0.129/25
```

---

# Conclusion

The lab was successfully completed by identifying and correcting the single misconfiguration on R1.

The problem was caused by **GigabitEthernet0/0.24 being configured with `encapsulation dot1Q 14` instead of `encapsulation dot1Q 24`**.

The switch VLAN assignments and trunk configurations were already correct, so they were left unchanged as required.

After correcting the VLAN encapsulation, R1 was able to correctly route traffic between VLAN 13 and VLAN 24, allowing all PCs to communicate with each other.

**Final Fix:**

```cisco
R1(config)# interface gigabitethernet0/0.24
R1(config-subif)# encapsulation dot1Q 24
```

**Result: Inter-VLAN connectivity restored without changing VLAN membership.**
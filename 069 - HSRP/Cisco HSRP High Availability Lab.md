# Cisco HSRP High Availability Lab

## 📌 Lab Overview

This lab focuses on configuring and verifying **Hot Standby Router Protocol (HSRP)** to provide first-hop redundancy for two VLANs.

Two routers, **R1** and **R2**, provide redundant default gateways for VLAN 10 and VLAN 20. HSRP allows hosts to use a **virtual IP address** as their default gateway while automatically switching between routers if the active router becomes unavailable.

The lab also demonstrates **load distribution** by making R1 the active router for VLAN 10 and R2 the active router for VLAN 20.

---

## 🎯 Objectives

Configure HSRP according to the following requirements:

- Use **HSRP version 2**
- Configure `.1` as the virtual IP address for each VLAN:
  - VLAN 10 → `10.10.10.1`
  - VLAN 20 → `10.20.20.1`
- Configure **R1 as the active router for VLAN 10**
- Configure **R1 as the standby router for VLAN 20**
- Configure **R2 as the active router for VLAN 20**
- Configure **R2 as the standby router for VLAN 10**
- Enable **HSRP preemption**
- Verify HSRP operation and failover
- Verify connectivity to the external network

---

## 🗺️ HSRP Design

| VLAN | Network | Virtual IP | R1 | R2 |
|---|---|---|---|---|
| VLAN 10 | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.2` **Active** | `10.10.10.3` Standby |
| VLAN 20 | `10.20.20.0/24` | `10.20.20.1` | `10.20.20.3` Standby | `10.20.20.2` **Active** |

### HSRP Groups

- VLAN 10 → HSRP Group **10**
- VLAN 20 → HSRP Group **20**

### HSRP Timers

- Hello timer: **3 seconds**
- Hold timer: **10 seconds**

These are the default HSRP timers and were observed during verification.

---

# 🔧 Configuration

## R1 Configuration

### VLAN 10 — R1 Active

R1 uses the higher HSRP priority of **110** to become the active router for VLAN 10.

```cisco
R1(config)#interface GigabitEthernet0/1
R1(config-if)#standby version 2
R1(config-if)#standby 10 ip 10.10.10.1
R1(config-if)#standby 10 priority 110
R1(config-if)#standby 10 preempt
```

Configuration:

- Physical IP: `10.10.10.2/24`
- Virtual IP: `10.10.10.1`
- HSRP group: `10`
- Priority: `110`
- Role: **Active**
- Preemption: **Enabled**

---

### VLAN 20 — R1 Standby

R1 is configured with a lower priority for VLAN 20.

```cisco
R1(config)#interface GigabitEthernet0/2
R1(config-if)#standby version 2
R1(config-if)#standby 20 ip 10.20.20.1
R1(config-if)#standby 20 priority 90
```

Configuration:

- Physical IP: `10.20.20.3/24`
- Virtual IP: `10.20.20.1`
- HSRP group: `20`
- Priority: `90`
- Role: **Standby**

> Because R1 has a priority of 90 and R2 retains the default priority of 100, R2 becomes the active router for VLAN 20.

---

# R2 Configuration

## VLAN 10 — R2 Standby

```cisco
R2(config)#interface GigabitEthernet0/2
R2(config-if)#standby version 2
R2(config-if)#standby 10 ip 10.10.10.1
```

Configuration:

- Physical IP: `10.10.10.3/24`
- Virtual IP: `10.10.10.1`
- HSRP group: `10`
- Priority: `100` (default)
- Role: **Standby**

R1 has priority 110, so it wins the HSRP election for VLAN 10.

---

## VLAN 20 — R2 Active

```cisco
R2(config)#interface GigabitEthernet0/1
R2(config-if)#standby version 2
R2(config-if)#standby 20 ip 10.20.20.1
R2(config-if)#standby 20 preempt
```

Configuration:

- Physical IP: `10.20.20.2/24`
- Virtual IP: `10.20.20.1`
- HSRP group: `20`
- Priority: `100` (default)
- Role: **Active**
- Preemption: **Enabled**

---

# 🔍 Understanding the HSRP Election

HSRP elects an active router based primarily on **priority**.

The default priority is:

```text
100
```

A higher priority wins the election.

### VLAN 10

```text
R1 = 110
R2 = 100
```

Therefore:

```text
R1 → Active
R2 → Standby
```

### VLAN 20

```text
R1 = 90
R2 = 100
```

Therefore:

```text
R2 → Active
R1 → Standby
```

This creates a simple form of **load sharing** across the two VLANs.

---

# 🔄 HSRP Preemption

Preemption allows a router with a higher priority to automatically regain the active role after recovering from a failure.

For VLAN 10:

```text
R1 = Priority 110
R2 = Priority 100
```

If R1 fails:

```text
R2 → Active
```

When R1 returns:

```text
R1 → Active
R2 → Standby
```

This happens because R1 has a higher priority and preemption is enabled.

The same principle applies to VLAN 20, where R2 is intended to be the preferred active router.

---

# 🧪 Verification

Use the following command on both routers:

```cisco
show standby
```

You should see output similar to:

```text
GigabitEthernet0/1 - Group 10 (version 2)
  State is Active
  Virtual IP address is 10.10.10.1
  Hello time 3 sec, hold time 10 sec
  Preemption enabled
  Priority 110
```

For VLAN 20, R2 should show:

```text
GigabitEthernet0/1 - Group 20 (version 2)
  State is Active
  Virtual IP address is 10.20.20.1
  Hello time 3 sec, hold time 10 sec
  Preemption enabled
  Priority 100
```

---

## Useful Verification Commands

### Check HSRP status

```cisco
show standby
```

### Check a specific interface

```cisco
show standby GigabitEthernet0/1
```

### Check the running configuration

```cisco
show running-config
```

### Check interface addressing

```cisco
show ip interface brief
```

### Check routing

```cisco
show ip route
```

---

# 🧪 Connectivity Testing

The virtual HSRP addresses should be used as the default gateways for hosts.

### VLAN 10

Test the virtual gateway:

```text
C:\>ping 10.10.10.1
```

Expected result:

```text
Reply from 10.10.10.1
```

### VLAN 20

Test the virtual gateway:

```text
C:\>ping 10.20.20.1
```

Expected result:

```text
Reply from 10.20.20.1
```

---

# 🌐 External Connectivity

Test connectivity from VLAN 10 and VLAN 20 hosts to the external destination:

```text
C:\>ping 15.0.0.1
```

The successful tests confirm that traffic is being forwarded through the HSRP gateway and toward the external network.

You can also use:

```text
C:\>tracert 15.0.0.1
```

For VLAN 10, the first hop should normally be the currently active HSRP router:

```text
10.10.10.2
```

For VLAN 20:

```text
10.20.20.2
```

If the active router fails, the first hop should change to the standby router.

---

# 🔥 HSRP Failover Test

A critical part of this lab is verifying redundancy rather than simply checking that the configuration exists.

## VLAN 10 Failover

Initially:

```text
R1 → Active
R2 → Standby
Virtual IP → 10.10.10.1
```

Shut down R1's VLAN 10 interface:

```cisco
R1(config)#interface GigabitEthernet0/1
R1(config-if)#shutdown
```

Then check R2:

```cisco
R2#show standby
```

R2 should transition to:

```text
Active
```

Test:

```text
C:\>ping 10.10.10.1
C:\>ping 15.0.0.1
```

Connectivity should continue through R2.

Restore the interface:

```cisco
R1(config)#interface GigabitEthernet0/1
R1(config-if)#no shutdown
```

Because preemption is enabled, R1 should eventually become active again.

---

## VLAN 20 Failover

Initially:

```text
R1 → Standby
R2 → Active
Virtual IP → 10.20.20.1
```

Shut down R2's VLAN 20 interface:

```cisco
R2(config)#interface GigabitEthernet0/1
R2(config-if)#shutdown
```

R1 should take over:

```text
R1 → Active
```

Test:

```text
C:\>ping 10.20.20.1
C:\>ping 15.0.0.1
```

Restore R2:

```cisco
R2(config)#interface GigabitEthernet0/1
R2(config-if)#no shutdown
```

R2 should regain the active role because it has the higher priority for VLAN 20.

---

# ⚠️ Troubleshooting Notes

If the routers do not form an HSRP relationship, check the following:

### 1. HSRP version

Both routers must use:

```cisco
standby version 2
```

### 2. Virtual IP

Both routers in the same HSRP group must use the same virtual IP.

VLAN 10:

```text
10.10.10.1
```

VLAN 20:

```text
10.20.20.1
```

### 3. HSRP group number

The group must match between routers.

```text
VLAN 10 → Group 10
VLAN 20 → Group 20
```

### 4. Interfaces must be in the same subnet

VLAN 10:

```text
R1 = 10.10.10.2/24
R2 = 10.10.10.3/24
VIP = 10.10.10.1
```

VLAN 20:

```text
R1 = 10.20.20.3/24
R2 = 10.20.20.2/24
VIP = 10.20.20.1
```

### 5. Check priority

```cisco
show standby
```

Expected priorities:

```text
VLAN 10:
R1 = 110
R2 = 100

VLAN 20:
R1 = 90
R2 = 100
```

### 6. Check preemption

R1 VLAN 10:

```cisco
standby 10 preempt
```

R2 VLAN 20:

```cisco
standby 20 preempt
```

---

# 📋 Final HSRP State

The completed topology should operate as follows:

| VLAN | Virtual Gateway | Active Router | Standby Router | Active Priority |
|---|---|---|---|---:|
| VLAN 10 | `10.10.10.1` | **R1** | R2 | 110 |
| VLAN 20 | `10.20.20.1` | **R2** | R1 | 100 |

### Final Requirements Checklist

- [x] HSRP version 2 configured
- [x] VLAN 10 virtual IP = `10.10.10.1`
- [x] VLAN 20 virtual IP = `10.20.20.1`
- [x] R1 active for VLAN 10
- [x] R1 standby for VLAN 20
- [x] R2 active for VLAN 20
- [x] R2 standby for VLAN 10
- [x] Preemption enabled
- [x] Hello timer = 3 seconds
- [x] Hold timer = 10 seconds
- [x] HSRP virtual gateways reachable
- [x] External connectivity verified
- [x] HSRP failover tested

---

# 🧠 Key Takeaways

This lab demonstrates how HSRP provides **first-hop gateway redundancy** without requiring hosts to know which physical router is currently forwarding traffic.

Instead of configuring a physical router address as the default gateway, hosts use the HSRP virtual address:

```text
VLAN 10 → 10.10.10.1
VLAN 20 → 10.20.20.1
```

The routers automatically negotiate which device should be active.

By assigning different priorities to each VLAN, the network can also distribute the active gateway role:

```text
VLAN 10 → R1 Active
VLAN 20 → R2 Active
```

This provides both **redundancy and better utilization of the two routers**.
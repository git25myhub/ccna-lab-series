# Cisco HSRP Troubleshooting Lab

## 📌 Lab Overview

This lab focuses on troubleshooting **Hot Standby Router Protocol (HSRP)** in a redundant gateway environment.

HSRP has already been configured according to the required design, but the network contains **one misconfiguration on each router**.

The symptoms include:

- HSRP error messages being displayed.
- R1 and R2 not consistently operating with HSRP version 2.
- R2 failing to reclaim the **Active** role for VLAN 20 after recovering from a failure.

The objective is to identify and correct the misconfiguration on each router and verify that HSRP operates as intended.

---

# 🎯 Lab Objectives

Troubleshoot and fix the HSRP configuration so that:

- HSRP **version 2** is used.
- VLAN 10 uses virtual IP `10.10.10.1`.
- VLAN 20 uses virtual IP `10.20.20.1`.
- R1 is the **Active** router for VLAN 10.
- R1 is the **Standby** router for VLAN 20.
- R2 is the **Active** router for VLAN 20.
- R2 is the **Standby** router for VLAN 10.
- HSRP preemption is enabled where required.
- R2 retakes the Active role for VLAN 20 after recovering from a failure.
- HSRP error messages are eliminated.
- Connectivity to the external network remains available.

---

# 🗺️ HSRP Design

| VLAN | Network | Virtual IP | R1 | R2 |
|---|---|---|---|---|
| VLAN 10 | `10.10.10.0/24` | `10.10.10.1` | **Active** | Standby |
| VLAN 20 | `10.20.20.0/24` | `10.20.20.1` | Standby | **Active** |

### HSRP Groups

| VLAN | HSRP Group |
|---|---:|
| VLAN 10 | 10 |
| VLAN 20 | 20 |

### Expected Priorities

| Router | VLAN | Priority | Expected Role |
|---|---|---:|---|
| R1 | VLAN 10 | 110 | **Active** |
| R2 | VLAN 10 | 100 | Standby |
| R1 | VLAN 20 | 90 | Standby |
| R2 | VLAN 20 | 100 | **Active** |

---

# 🔎 Initial Troubleshooting

The first step is to inspect HSRP on both routers:

```cisco
show standby
```

---

# 🛠️ Issue Found on R1

The initial R1 output showed:

```text
GigabitEthernet0/1 - Group 10

  State is Active

  Virtual IP address is 10.10.10.1

  Local virtual MAC address is 0000.0C07.AC0A (v1 default)

  Priority 110
```

The important clue is:

```text
Local virtual MAC address is 0000.0C07.AC0A (v1 default)
```

While the VLAN 20 interface was correctly operating with HSRP version 2:

```text
GigabitEthernet0/2 - Group 20 (version 2)
```

The VLAN 10 interface was missing the HSRP version 2 configuration.

## ❌ Misconfiguration

R1's GigabitEthernet0/1 was using **HSRP version 1** instead of version 2.

This explains the inconsistent HSRP behavior and error messages.

---

# ✅ Fix R1

Enter interface configuration mode:

```cisco
R1#configure terminal
R1(config)#interface GigabitEthernet0/1
```

Configure HSRP version 2:

```cisco
R1(config-if)#standby version 2
```

Save the configuration:

```cisco
R1(config-if)#do write
```

After making the correction, verify:

```cisco
R1#show standby GigabitEthernet0/1
```

The output should now identify the group as:

```text
GigabitEthernet0/1 - Group 10 (version 2)
```

The local virtual MAC should also change to the HSRP version 2 format:

```text
0000.0C9F.F00A
```

---

# 🛠️ Issue Found on R2

The initial R2 output showed:

```text
GigabitEthernet0/1 - Group 20 (version 2)

  State is Active

  Virtual IP address is 10.20.20.1

  Preemption disabled

  Active router is local

  Standby router is 10.20.20.3

  Priority 100
```

The important problem is:

```text
Preemption disabled
```

R2 is supposed to be the preferred Active router for VLAN 20.

However, without preemption, R2 will not automatically reclaim the Active role after another router temporarily becomes Active.

---

# ❌ Misconfiguration

R2 was missing:

```cisco
standby 20 preempt
```

Without this command, R2 could recover from a failure but remain in the Standby state if R1 had taken over the Active role.

---

# ✅ Fix R2

Enter the VLAN 20 interface:

```cisco
R2#configure terminal
R2(config)#interface GigabitEthernet0/1
```

Enable HSRP preemption:

```cisco
R2(config-if)#standby 20 preempt
```

Save the configuration:

```cisco
R2(config-if)#do write
```

Verify:

```cisco
R2#show standby GigabitEthernet0/1
```

The output should now show:

```text
Preemption enabled
```

---

# 🔬 Why Preemption Matters

Consider the VLAN 20 configuration:

```text
R1 priority = 90
R2 priority = 100
```

Therefore, R2 should be the preferred Active router.

Initially:

```text
R2 → Active
R1 → Standby
```

If R2 fails:

```text
R2 → Failed
R1 → Active
```

When R2 recovers, it has the higher priority:

```text
R2 = 100
R1 = 90
```

With preemption enabled, R2 will reclaim the Active role:

```text
R2 → Active
R1 → Standby
```

Without preemption, R1 could remain Active even though R2 has the higher priority.

---

# 🧪 Verification

## Verify R1 VLAN 10

Run:

```cisco
R1#show standby GigabitEthernet0/1
```

Expected:

```text
GigabitEthernet0/1 - Group 10 (version 2)
  State is Active
  Virtual IP address is 10.10.10.1
  Preemption enabled
  Priority 110
```

The important values are:

```text
Version:       2
State:         Active
Virtual IP:    10.10.10.1
Priority:      110
```

---

## Verify R1 VLAN 20

```cisco
R1#show standby GigabitEthernet0/2
```

Expected:

```text
GigabitEthernet0/2 - Group 20 (version 2)
  State is Standby
  Virtual IP address is 10.20.20.1
  Priority 90
```

R1 should identify R2 as the Active router:

```text
Active router is 10.20.20.2
```

---

## Verify R2 VLAN 10

```cisco
R2#show standby GigabitEthernet0/2
```

Expected:

```text
GigabitEthernet0/2 - Group 10 (version 2)
  State is Standby
  Virtual IP address is 10.10.10.1
  Priority 100
```

R1 should be identified as the Active router.

---

## Verify R2 VLAN 20

```cisco
R2#show standby GigabitEthernet0/1
```

Expected:

```text
GigabitEthernet0/1 - Group 20 (version 2)
  State is Active
  Virtual IP address is 10.20.20.1
  Preemption enabled
  Priority 100
```

---

# 📋 Final Expected HSRP State

| Interface | HSRP Group | Virtual IP | Router | Priority | Expected State |
|---|---:|---|---|---:|---|
| R1 G0/1 | 10 | `10.10.10.1` | R1 | 110 | **Active** |
| R2 G0/2 | 10 | `10.10.10.1` | R2 | 100 | Standby |
| R1 G0/2 | 20 | `10.20.20.1` | R1 | 90 | Standby |
| R2 G0/1 | 20 | `10.20.20.1` | R2 | 100 | **Active** |

All HSRP interfaces should report:

```text
version 2
```

---

# 🔥 HSRP Failover Test

After correcting the configurations, test the redundancy.

## VLAN 20 — Normal Operation

Verify R2 is Active:

```cisco
R2#show standby GigabitEthernet0/1
```

Expected:

```text
State is Active
```

---

## Simulate R2 Failure

On R2:

```cisco
R2#configure terminal
R2(config)#interface GigabitEthernet0/1
R2(config-if)#shutdown
```

R1 should take over:

```text
R1 → Active
```

Verify:

```cisco
R1#show standby GigabitEthernet0/2
```

---

## Restore R2

Bring the interface back up:

```cisco
R2(config-if)#no shutdown
```

Wait for HSRP to reconverge.

Because R2 has:

```text
Priority = 100
Preemption = Enabled
```

while R1 has:

```text
Priority = 90
```

R2 should reclaim the Active role.

Verify:

```cisco
R2#show standby GigabitEthernet0/1
```

Expected:

```text
State is Active
```

R1 should return to:

```text
State is Standby
```

---

# 🌐 Connectivity Testing

Test the VLAN 10 virtual gateway:

```text
C:\>ping 10.10.10.1
```

Test the VLAN 20 virtual gateway:

```text
C:\>ping 10.20.20.1
```

Both should return successful replies.

---

# 🌍 External Network Test

The external destination used in the lab is:

```text
15.0.0.1
```

Test from a VLAN 10 host:

```text
C:\>ping 15.0.0.1
```

Expected:

```text
Reply from 15.0.0.1
```

You can also verify the forwarding path:

```text
C:\>tracert 15.0.0.1
```

Before a failure, VLAN 10 traffic should normally use:

```text
10.10.10.2
```

as its first-hop router.

VLAN 20 traffic should normally use:

```text
10.20.20.2
```

as its first-hop router.

---

# 🧰 Useful Troubleshooting Commands

### Display all HSRP information

```cisco
show standby
```

### Display HSRP for a specific interface

```cisco
show standby GigabitEthernet0/1
```

### Display interface status

```cisco
show ip interface brief
```

### Display the running configuration

```cisco
show running-config
```

### Display HSRP-related configuration

```cisco
show running-config | section standby
```

### Test the virtual gateway

```text
ping 10.10.10.1
ping 10.20.20.1
```

---

# 🧠 Troubleshooting Lessons

## 1. Always check the HSRP version

If one router is using HSRP version 1 and the other is using version 2, the routers may fail to form the expected HSRP relationship.

Look for:

```text
Group 10
```

versus:

```text
Group 10 (version 2)
```

The correct configuration is:

```cisco
standby version 2
```

---

## 2. Priority determines the preferred Active router

For VLAN 10:

```text
R1 = 110
R2 = 100
```

R1 wins.

For VLAN 20:

```text
R1 = 90
R2 = 100
```

R2 wins.

---

## 3. Preemption controls recovery behavior

A higher-priority router does not necessarily reclaim the Active role after recovering unless preemption is enabled.

For R2 VLAN 20:

```cisco
standby 20 preempt
```

is therefore essential.

---

# 📝 Misconfigurations Found

| Router | Interface | Problem | Correction |
|---|---|---|---|
| R1 | G0/1 | HSRP version 2 missing | `standby version 2` |
| R2 | G0/1 | HSRP preemption disabled | `standby 20 preempt` |

---

# ✅ Completion Checklist

- [x] HSRP version 2 configured on all HSRP interfaces
- [x] VLAN 10 virtual IP = `10.10.10.1`
- [x] VLAN 20 virtual IP = `10.20.20.1`
- [x] R1 Active for VLAN 10
- [x] R1 Standby for VLAN 20
- [x] R2 Active for VLAN 20
- [x] R2 Standby for VLAN 10
- [x] R1 VLAN 10 priority = 110
- [x] R1 VLAN 20 priority = 90
- [x] R2 VLAN 20 priority = 100
- [x] Preemption enabled for the preferred Active routers
- [x] R2 successfully retakes Active role after recovery
- [x] Virtual gateway connectivity verified
- [x] External connectivity to `15.0.0.1` verified

---

# 🏁 Conclusion

This lab demonstrates the importance of troubleshooting **HSRP version mismatches and preemption**.

The two misconfigurations were:

1. **R1 VLAN 10 was running HSRP version 1 instead of version 2.**
2. **R2 VLAN 20 had preemption disabled.**

After correcting these issues, the intended redundancy design is restored:

```text
                 VLAN 10
            10.10.10.1 (VIP)
                  |
          +-------+-------+
          |               |
       R1 Active       R2 Standby
       Priority 110    Priority 100
          |               |
          +---------------+

                 VLAN 20
            10.20.20.1 (VIP)
                  |
          +-------+-------+
          |               |
       R1 Standby       R2 Active
       Priority 90      Priority 100
                         Preempt
```

The key takeaway is that **HSRP troubleshooting should involve checking the HSRP version, virtual IP, group number, priority, state, and preemption status** rather than looking only at whether the interface is up.
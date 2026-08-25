# Cisco Port Security – Sticky Secure MAC Addresses Lab

## 📌 Lab Overview

This lab builds on basic **Cisco switch port security** by introducing **sticky secure MAC addresses**.

You will first configure standard port security and observe what happens to dynamically learned secure MAC addresses after a switch reload. You will then enable **sticky MAC address learning**, save the configuration, reload the switch, and verify that the learned secure MAC address is retained.

This demonstrates an important difference between **dynamic secure MAC addresses** and **sticky secure MAC addresses**.

---

# 🎯 Objectives

By the end of this lab, you should be able to:

- Enable port security on end-host interfaces.
- Generate traffic to dynamically learn MAC addresses.
- View secure MAC addresses on a switch.
- Examine port-security configuration in the running configuration.
- Understand what happens to dynamically learned secure MAC addresses after a reload.
- Configure sticky secure MAC address learning.
- Verify sticky MAC addresses.
- Understand how sticky MAC addresses are stored in the running configuration.
- Save sticky MAC configurations so they survive a switch reload.

---

# 🗺️ Suggested Topology

```text
PC1 -------- SW1 -------- SW2 -------- PC2
             F0/2        F0/2
```

### Device Connections

| Device | Interface | Connected Device |
|---|---|---|
| PC1 | NIC | SW1 F0/2 |
| SW1 | F0/1 | SW2 F0/1 |
| SW1 | F0/2 | PC1 |
| SW2 | F0/1 | SW1 F0/1 |
| SW2 | F0/2 | PC2 |

The main focus of this lab is **SW1 F0/2**, the interface connected to PC1.

---

# 🧠 Key Concept Before Starting

There are several types of secure MAC addresses that you should understand:

| MAC Type | Description |
|---|---|
| Static Secure | Manually configured by the administrator |
| Dynamic Secure | Learned dynamically after port security is enabled |
| Sticky Secure | Dynamically learned and added to the running configuration |

The important question in this lab is:

> **What happens to a dynamically learned secure MAC address after the switch reloads?**

You will test this before and after enabling sticky MAC addresses.

---

# 🧪 Task 1 — Enable Port Security

Enable port security on the switchports connected to the end hosts.

For this topology:

- **SW1 F0/2 → PC1**
- **SW2 F0/2 → PC2**

## SW1

```cisco
enable
configure terminal

interface fastethernet 0/2
switchport mode access
switchport port-security
```

## SW2

```cisco
enable
configure terminal

interface fastethernet 0/2
switchport mode access
switchport port-security
```

### Why configure access mode first?

Port security is normally applied to an access interface. If the interface is dynamically configured, Cisco IOS may reject the port-security command.

Use:

```cisco
switchport mode access
```

before:

```cisco
switchport port-security
```

---

# 🧪 Task 2 — Generate Traffic

From PC1, ping PC2.

Example:

```text
C:\> ping 192.168.1.12
```

A successful ping should produce replies similar to:

```text
Reply from 192.168.1.12: bytes=32 time<1ms TTL=128
Reply from 192.168.1.12: bytes=32 time<1ms TTL=128
Reply from 192.168.1.12: bytes=32 time<1ms TTL=128
Reply from 192.168.1.12: bytes=32 time<1ms TTL=128
```

The purpose of the ping is to generate Ethernet frames so that SW1 can learn PC1's MAC address.

---

# 🔍 Task 3 — View Secure MAC Addresses

On SW1, use:

```cisco
show port-security address
```

You should see PC1's MAC address listed as a secure MAC address.

Example:

```text
Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports
----    -----------       ----                          -----
1       0002.16E2.2193    SecureDynamic                 Fa0/2
```

The actual MAC address may be different in your topology.

### Expected Result

PC1's MAC address should appear on:

```text
SW1 F0/2
```

At this point, the MAC address is **dynamically learned as a secure MAC address**.

---

# 🧪 Task 4 — Examine the Running Configuration

Check SW1's F0/2 configuration:

```cisco
show running-config interface fastethernet 0/2
```

You should see something similar to:

```cisco
interface FastEthernet0/2
 switchport mode access
 switchport port-security
```

Notice that the dynamically learned MAC address is **not manually entered as a MAC-address command**.

This is important.

The switch has learned the MAC address as a secure address, but it has not yet been written into the configuration as a sticky MAC address.

---

# 💾 Task 5 — Save and Reload SW1

Save the running configuration:

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

You should receive:

```text
Building configuration...
[OK]
```

Then reload the switch:

```cisco
reload
```

Confirm the reload when prompted.

The switch will reboot and load the startup configuration.

---

# 🔍 Task 6 — Check the Secure MAC Address After Reload

After SW1 finishes rebooting, check the secure MAC address table:

```cisco
show port-security address
```

### Question

> **Is PC1's MAC address still present?**

### Expected Result

**No.**

The dynamically learned secure MAC address is not automatically preserved across a reload.

The port-security configuration itself was saved, but the dynamically learned secure MAC address was not stored as a persistent configuration entry.

This demonstrates an important limitation of ordinary dynamic secure MAC learning.

---

# 🧠 Why Did the MAC Address Disappear?

With normal dynamic port security:

```cisco
switchport port-security
```

the switch can dynamically learn a secure MAC address.

However, the learned address exists as operational state rather than as a permanent configuration entry.

When the switch reloads, that dynamically learned information is lost.

Therefore:

```text
Dynamic Secure MAC
        ↓
Switch Reload
        ↓
MAC Address Lost
```

To make dynamically learned secure MAC addresses persistent, we can use **sticky MAC learning**.

---

# 🔐 Task 7 — Enable Sticky Secure MAC Addresses

Enter interface configuration mode on SW1 F0/2:

```cisco
configure terminal

interface fastethernet 0/2
switchport port-security mac-address sticky
```

The complete configuration should be:

```cisco
interface fastethernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
```

Exit configuration mode if necessary:

```cisco
end
```

---

# 🧪 Generate Traffic Again

From PC1, ping PC2:

```text
C:\> ping 192.168.1.12
```

The traffic allows SW1 to learn PC1's MAC address as a **sticky secure MAC address**.

---

# 🔍 Task 8 — Verify the Sticky MAC Address

On SW1, run:

```cisco
show port-security address
```

Your output should resemble:

```text
Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports
----    -----------       ----                          -----
1       0002.16E2.2193    SecureSticky                  Fa0/2
```

The important difference is the **Type**:

```text
SecureSticky
```

This indicates that the MAC address was learned using sticky MAC learning.

---

# 🔎 Examine the Running Configuration

Now run:

```cisco
show running-config interface fastethernet 0/2
```

You should see something similar to:

```cisco
interface FastEthernet0/2
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0002.16E2.2193
```

Your MAC address will depend on the PC in your topology.

---

# ⭐ What Changed?

This is the key observation of the lab.

### Before Sticky MAC

The configuration contained:

```cisco
interface FastEthernet0/2
 switchport mode access
 switchport port-security
```

The MAC address was dynamically learned but was **not written into the running configuration**.

### After Sticky MAC

The configuration contains:

```cisco
switchport port-security mac-address sticky
```

and after PC1 generates traffic, Cisco IOS adds the learned MAC address:

```cisco
switchport port-security mac-address sticky 0002.16E2.2193
```

Therefore, the MAC address now appears directly in the running configuration.

---

# 💾 Task 9 — Save and Reload the Switch

Save the configuration:

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

Then reload:

```cisco
reload
```

Confirm the reload when prompted.

Wait for SW1 to completely boot.

---

# 🔍 Task 10 — Verify the Sticky MAC After Reload

Once SW1 has finished booting, run:

```cisco
show port-security address
```

You should see PC1's MAC address again:

```text
Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports
----    -----------       ----                          -----
1       0002.16E2.2193    SecureSticky                  Fa0/2
```

### Question

> **Is PC1's MAC address still present?**

### Expected Answer

**Yes.**

Because the sticky secure MAC address was saved in the configuration, it survives a switch reload.

---

# 📊 Dynamic vs Sticky MAC Addresses

| Feature | Dynamic Secure MAC | Sticky Secure MAC |
|---|---|---|
| Learned automatically | ✅ | ✅ |
| Requires manual MAC entry | ❌ | ❌ |
| Appears in secure MAC table | ✅ | ✅ |
| Appears in running configuration | ❌ | ✅ |
| Survives reload without saving learned address | ❌ | ❌ |
| Survives reload after saving configuration | ❌ | ✅ |
| Configuration command | `switchport port-security` | `switchport port-security mac-address sticky` |

---

# 🔄 Lab Demonstration

The lab demonstrates two different behaviors.

## Phase 1 — Dynamic Secure MAC

```text
Enable Port Security
        ↓
PC1 Sends Traffic
        ↓
SW1 Learns PC1 MAC
        ↓
Secure MAC Table
        ↓
Save + Reload
        ↓
MAC Address Disappears
```

## Phase 2 — Sticky Secure MAC

```text
Enable Sticky MAC
        ↓
PC1 Sends Traffic
        ↓
SW1 Learns PC1 MAC
        ↓
MAC Added to Running Configuration
        ↓
Save Configuration
        ↓
Reload SW1
        ↓
MAC Address Remains
```

---

# 🛠️ Useful Verification Commands

### View Secure MAC Addresses

```cisco
show port-security address
```

### View Port-Security Status

```cisco
show port-security
```

### View Port-Security on a Specific Interface

```cisco
show port-security interface fastethernet 0/2
```

### View Interface Configuration

```cisco
show running-config interface fastethernet 0/2
```

### View the Entire Running Configuration

```cisco
show running-config
```

### Save Configuration

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

### Reload the Switch

```cisco
reload
```

---

# 📝 Lab Questions

Answer the following questions after completing the lab:

1. What command displays secure MAC addresses?
2. What type of secure MAC address was initially learned on SW1?
3. Was the dynamically learned MAC address present after the first reload?
4. Why did the MAC address disappear?
5. What command enables sticky MAC learning?
6. What type does the MAC address show as after enabling sticky learning?
7. What additional command appears in the running configuration after PC1 generates traffic?
8. Why does the sticky MAC address survive the second reload?
9. What would happen if you enabled sticky MAC but did not save the running configuration before reloading?
10. What is the main difference between a dynamic secure MAC address and a sticky secure MAC address?

---

# 📋 Expected Configuration

After completing the sticky MAC portion of the lab, SW1 F0/2 should contain configuration similar to:

```cisco
interface FastEthernet0/2
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0002.16E2.2193
```

The MAC address shown above is the MAC address observed in the supplied lab output. In another Packet Tracer topology, use the actual MAC address learned from PC1.

---

# ✅ Completion Checklist

- [ ] Enable port security on SW1 F0/2.
- [ ] Enable port security on the PC-facing port of SW2.
- [ ] Ping PC1 to PC2.
- [ ] Verify PC1's secure MAC address on SW1.
- [ ] Examine SW1 F0/2 running configuration.
- [ ] Save SW1's configuration.
- [ ] Reload SW1.
- [ ] Verify that the dynamic secure MAC address is no longer present.
- [ ] Enable sticky MAC learning on SW1 F0/2.
- [ ] Ping PC1 to PC2 again.
- [ ] Verify `SecureSticky` in the secure MAC address table.
- [ ] Examine the running configuration.
- [ ] Confirm the sticky MAC address appears in the configuration.
- [ ] Save the running configuration.
- [ ] Reload SW1.
- [ ] Verify that PC1's sticky MAC address remains present.
- [ ] Explain why sticky MAC addresses survive a reload when the configuration is saved.

---

# 🏁 Lab Completion Criteria

The lab is successfully completed when:

- Port security is enabled on the end-host interfaces.
- PC1's MAC address is dynamically learned initially.
- The dynamic secure MAC address disappears after the first reload.
- Sticky MAC learning is enabled on SW1 F0/2.
- PC1's MAC address appears as `SecureSticky`.
- The sticky MAC address is added to the running configuration.
- The configuration is saved to startup-config.
- PC1's secure MAC address remains present after the second reload.
- You can clearly explain the difference between **dynamic secure MAC addresses** and **sticky secure MAC addresses**.

---

## 📚 Key Takeaway

**Sticky MAC addresses combine automatic MAC learning with persistent configuration.**

Instead of manually typing the PC's MAC address, the switch learns it automatically:

```cisco
switchport port-security mac-address sticky
```

The learned address is then added to the running configuration. Once the configuration is saved, the sticky secure MAC address survives a switch reload.

**Dynamic MAC learning:** learned automatically, but not persistent.

**Sticky MAC learning:** learned automatically and can become persistent when the configuration is saved.
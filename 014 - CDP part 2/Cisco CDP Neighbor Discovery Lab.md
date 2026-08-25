# Cisco CDP Neighbor Discovery Lab

## Lab Objective

In this lab, you will use **Cisco Discovery Protocol (CDP)** to discover directly connected Cisco devices and gather information about neighboring network devices without relying on device model labels in Packet Tracer.

You will identify:

- The interfaces connecting routers and switches
- The hardware model/platform of neighboring devices
- The IOS software version running on neighboring devices

---

## Important Packet Tracer Setting

Before starting the lab, make sure:

**Options → Preferences → Show Device Model Labels**

is **disabled**.

This is important because the purpose of the lab is to use **CDP commands** to discover the device information rather than simply reading the model displayed next to the device icon.

---

# Lab Tasks

## 1. Identify Router/Switch Interfaces Using CDP

On each router and switch, use:

```cisco
show cdp neighbors
```

The command displays directly connected Cisco devices and provides information such as:

- Device ID
- Local interface
- Neighbor platform
- Device capability
- Neighbor's outgoing interface

### Example

The SW1 output was:

```text
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
R1           Gig 0/1          149            R       C1900       Gig 0/1
```

This tells us:

| Device | Local Interface | Neighbor | Neighbor Interface |
|---|---|---|---|
| SW1 | GigabitEthernet0/1 | R1 | GigabitEthernet0/1 |

Therefore:

**SW1 Gi0/1 ↔ R1 Gi0/1**

---

# 2. Identify Neighboring Device Models

The basic command:

```cisco
show cdp neighbors
```

already provides the **Platform** column.

From the SW1 output:

```text
Platform: C1900
```

Therefore, SW1 identifies R1 as a:

**Cisco C1900 router**

For more detailed information, use:

```cisco
show cdp neighbors detail
```

The lab output showed:

```text
Device ID: R1
Platform: cisco C1900, Capabilities: Router
Interface: GigabitEthernet0/1
Port ID (outgoing port): GigabitEthernet0/1
```

This provides more complete information about the neighboring device.

---

# 3. Identify the IOS Version of Neighboring Devices

Use:

```cisco
show cdp neighbors detail
```

The detailed CDP output includes a **Version** section containing the neighboring device's IOS information.

For R1, SW1 received:

```text
Cisco IOS Software, C1900 Software (C1900-UNIVERSALK9-M),
Version 15.1(4)M4, RELEASE SOFTWARE (fc2)
```

Therefore:

**R1 IOS Version: 15.1(4)M4**

---

# Useful CDP Commands

## View CDP Neighbors

```cisco
show cdp neighbors
```

This is the quickest way to identify:

- Neighbor device name
- Local interface
- Neighbor platform
- Neighbor interface

---

## View Detailed CDP Information

```cisco
show cdp neighbors detail
```

This provides additional information including:

- Device ID
- Platform/model
- Device capabilities
- Local interface
- Neighbor interface
- Hold time
- IOS version
- CDP advertisement version
- Duplex information

---

## Display Information for a Specific Neighbor

Instead of displaying every neighbor, use:

```cisco
show cdp entry R1
```

For example:

```text
SW1#show cdp entry R1
```

This displays detailed CDP information specifically for R1.

---

## Verify CDP Status

```cisco
show cdp
```

This can be used to verify whether CDP is running and view its global configuration information.

---

## View CDP Information on Interfaces

```cisco
show cdp interface
```

This displays CDP information for individual interfaces.

---

# Observed Lab Results

Based on the provided output, SW1 discovered R1 using CDP.

### Connection

```text
SW1 GigabitEthernet0/1
        |
        |
R1 GigabitEthernet0/1
```

### Neighbor Device

**R1**

### Neighbor Platform

**Cisco C1900**

### Neighbor Capability

**Router**

### R1 IOS Version

**15.1(4)M4**

### R1 IOS Image

```text
C1900-UNIVERSALK9-M
```

### CDP Advertisement Version

**Version 2**

### Duplex

**Full duplex**

---

# Difference Between `show cdp neighbors` and `show cdp neighbors detail`

| Command | Information Provided |
|---|---|
| `show cdp neighbors` | Basic neighbor information |
| `show cdp neighbors detail` | Detailed neighbor information, including IOS version |
| `show cdp entry R1` | Detailed information for one specific neighbor |

### Quick Reference

```text
show cdp neighbors
        ↓
Find connected devices and interfaces

show cdp neighbors detail
        ↓
Find device models and IOS versions

show cdp entry R1
        ↓
Display detailed information about R1
```

---

# Lab Questions and Answers

## Question 1: Which interfaces connect the router and switch?

From SW1:

```text
Local Interface: GigabitEthernet0/1
Neighbor Port: GigabitEthernet0/1
```

Therefore:

**SW1 Gi0/1 is connected to R1 Gi0/1.**

---

## Question 2: What is the model of the neighboring router?

CDP identifies R1 as:

**Cisco C1900**

This information can be seen using:

```cisco
show cdp neighbors
```

or:

```cisco
show cdp neighbors detail
```

---

## Question 3: What IOS version is running on R1?

The CDP detailed output identifies the IOS version as:

**15.1(4)M4**

The relevant output is:

```text
Cisco IOS Software, C1900 Software (C1900-UNIVERSALK9-M),
Version 15.1(4)M4
```

---

# Why CDP Is Useful

CDP is particularly useful when troubleshooting or documenting an unfamiliar Cisco network.

Without physically inspecting a device or relying on labels, an administrator can use CDP to determine:

- What device is connected
- Which local interface connects to it
- Which remote interface is being used
- What platform/model the neighbor uses
- What IOS version the neighbor is running
- Device capabilities

For example, from SW1:

```cisco
show cdp neighbors detail
```

was enough to discover that the neighboring device was an **R1 Cisco C1900 router running IOS 15.1(4)M4**.

---

# Verification Commands

Run these commands after completing the lab:

```cisco
show cdp
```

```cisco
show cdp neighbors
```

```cisco
show cdp neighbors detail
```

```cisco
show cdp entry R1
```

```cisco
show cdp interface
```

---

# Completion Checklist

- [ ] Disabled **Show Device Model Labels** in Packet Tracer
- [ ] Used `show cdp neighbors`
- [ ] Identified the router/switch interfaces using CDP
- [ ] Identified neighboring device models
- [ ] Used `show cdp neighbors detail`
- [ ] Identified the IOS version of neighboring devices
- [ ] Used `show cdp entry <device>` to inspect a specific neighbor
- [ ] Verified CDP information
- [ ] Documented the discovered topology

---

# Expected Final Results

For the demonstrated SW1-to-R1 connection:

| Information | Result |
|---|---|
| Local Device | SW1 |
| Local Interface | Gi0/1 |
| Neighbor | R1 |
| Neighbor Interface | Gi0/1 |
| Neighbor Platform | Cisco C1900 |
| Neighbor Capability | Router |
| Neighbor IOS | 15.1(4)M4 |
| CDP Version | 2 |
| Duplex | Full |

**Lab completed successfully when CDP is used to identify the connected interfaces, neighboring device models, and IOS versions without relying on Packet Tracer's device model labels.**
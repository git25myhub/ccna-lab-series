# Cisco LLDP Neighbor Discovery and Troubleshooting Lab

## Lab Overview

In this lab, you will configure **Link Layer Discovery Protocol (LLDP)** on the network devices and use it to discover neighboring devices, identify connected interfaces, and retrieve information about neighboring devices.

You will also disable **Cisco Discovery Protocol (CDP)** and configure a specific interface on SW2 so that it does not send or receive LLDP advertisements.

### Topology

The lab consists of:

- **R1** — Cisco 2911 Router
- **R2** — Cisco 2901 Router
- **SW1** — Cisco Catalyst 3650
- **SW2** — Cisco Catalyst 3560

The discovered connections are:

| Local Device | Local Interface | Neighbor | Neighbor Interface |
|---|---|---|---|
| SW1 | Gi1/0/1 | R1 | Gi0/0 |
| R1 | Gi0/0 | SW1 | Gi1/0/1 |
| R1 | Gi0/1 | R2 | Gi0/0 |
| R2 | Gi0/0 | R1 | Gi0/1 |
| R2 | Gi0/1 | SW2 | Gi0/1 |
| SW2 | Gi0/1 | R2 | Gi0/1 |

> **Important:** Before beginning, disable **Show Device Model Labels** and **Always Show Port Labels in Logical Workspace** under **Options → Preferences** in Packet Tracer. This ensures that device and port information is obtained through LLDP rather than being displayed directly on the topology.

---

## Objectives

By completing this lab, you will:

1. Disable CDP on all networking devices.
2. Enable LLDP on all networking devices.
3. Identify the default LLDP timer values.
4. Use LLDP to discover neighboring devices and connected interfaces.
5. Use LLDP to identify the IOS version of neighboring devices.
6. Disable LLDP transmission and reception on SW2 F0/1.
7. Verify the final LLDP configuration.

---

## Task 1 — Disable CDP and Enable LLDP

Perform the following configuration on **R1, R2, SW1, and SW2**:

```cisco
enable
configure terminal
no cdp run
lldp run
end
write memory
```

### Explanation

- `no cdp run` disables Cisco Discovery Protocol globally.
- `lldp run` enables LLDP globally.
- `write memory` saves the configuration.

Verify the configuration with:

```cisco
show cdp
show lldp
```

CDP should be disabled, while LLDP should show an **ACTIVE** status.

---

## Task 2 — Find the Default LLDP Timer Values

Use:

```cisco
show lldp
```

The default LLDP values observed in this lab are:

| LLDP Parameter | Default Value |
|---|---:|
| Advertisement interval | 30 seconds |
| Hold time | 120 seconds |
| Interface reinitialization delay | 2 seconds |

Example output:

```text
Global LLDP Information:
    Status: ACTIVE
    LLDP advertisements are sent every 30 seconds
    LLDP hold time advertised is 120 seconds
    LLDP interface reinitialisation delay is 2 seconds
```

### Questions

1. How often are LLDP advertisements sent?
2. What is the default LLDP hold time?
3. What is the default interface reinitialization delay?

---

## Task 3 — Identify Neighboring Devices and Interfaces

Use:

```cisco
show lldp neighbors
```

or the abbreviated command:

```cisco
show lldp nei
```

This command displays:

- Device ID
- Local interface
- Hold time
- Device capabilities
- Neighbor port ID

### SW1 Example

```text
Device ID           Local Intf     Hold-time  Capability      Port ID
R1                  Gig1/0/1       120        R               Gig0/0
```

This tells us that:

**SW1 GigabitEthernet1/0/1 → R1 GigabitEthernet0/0**

### R1 Example

```text
Device ID           Local Intf     Hold-time  Capability      Port ID
SW1                 Gig0/0         120        R               Gig1/0/1
R2                  Gig0/1         120        R               Gig0/0
```

Therefore:

- R1 Gi0/0 connects to SW1 Gi1/0/1.
- R1 Gi0/1 connects to R2 Gi0/0.

### R2 Example

```text
Device ID           Local Intf     Hold-time  Capability      Port ID
R1                  Gig0/0         120        R               Gig0/1
SW2                 Gig0/1         120        R               Gig0/1
```

Therefore:

- R2 Gi0/0 connects to R1 Gi0/1.
- R2 Gi0/1 connects to SW2 Gi0/1.

### SW2 Example

```text
Device ID           Local Intf     Hold-time  Capability      Port ID
R2                  Gig0/1         120        R               Gig0/1
```

Therefore:

- SW2 Gi0/1 connects to R2 Gi0/1.

---

## Task 4 — Identify Neighbor IOS Versions

Use the detailed LLDP command:

```cisco
show lldp neighbors detail
```

or:

```cisco
show lldp nei detail
```

The **System Description** section contains information about the neighboring device's operating system and IOS version.

### Example — SW1

R1 discovers SW1 with:

```text
System Name: SW1
System Description:
Cisco IOS Software [Denali], Catalyst L3 Switch Software
(CAT3K_CAA-UNIVERSALK9-M), Version 16.3.2
```

Therefore:

**SW1 IOS Version: 16.3.2**

### Example — R2

R1 discovers R2 with:

```text
System Name: R2
System Description:
Cisco IOS Software, C2900 Software
(C2900-UNIVERSALK9-M), Version 15.1(4)M4
```

Therefore:

**R2 IOS Version: 15.1(4)M4**

### Example — SW2

R2 discovers SW2 with:

```text
System Name: SW2
System Description:
Cisco IOS Software, C3560 Software
(C3560-ADVIPSERVICESK9-M), Version 12.2(37)SE1
```

Therefore:

**SW2 IOS Version: 12.2(37)SE1**

### Neighbor Information

| Device | Neighbor | IOS Version |
|---|---|---|
| R1 | SW1 | 16.3.2 |
| R1 | R2 | 15.1(4)M4 |
| R2 | R1 | 15.1(4)M4 |
| R2 | SW2 | 12.2(37)SE1 |
| SW1 | R1 | 15.1(4)M4 |
| SW2 | R2 | 15.1(4)M4 |

---

## Task 5 — Prevent SW2 F0/1 from Sending or Receiving LLDP

The final requirement is to prevent **SW2 FastEthernet0/1** from both sending and receiving LLDP updates.

Enter:

```cisco
SW2# configure terminal
SW2(config)# interface fastethernet 0/1
SW2(config-if)# no lldp transmit
SW2(config-if)# no lldp receive
SW2(config-if)# end
SW2# write memory
```

### Explanation

```cisco
no lldp transmit
```

Prevents the interface from sending LLDP advertisements.

```cisco
no lldp receive
```

Prevents the interface from accepting LLDP advertisements.

This configuration is applied **only to F0/1**. LLDP remains enabled globally on SW2 and its other interfaces.

---

## Verification

Verify the interface configuration with:

```cisco
show lldp interface fastethernet 0/1
```

You can also check the running configuration:

```cisco
show running-config interface fastethernet 0/1
```

The configuration should contain:

```text
interface FastEthernet0/1
 no lldp transmit
 no lldp receive
```

Verify LLDP globally:

```cisco
show lldp
```

Verify neighbors:

```cisco
show lldp neighbors
```

For detailed neighbor information:

```cisco
show lldp neighbors detail
```

---

## Useful LLDP Commands

| Command | Purpose |
|---|---|
| `show lldp` | Displays global LLDP information and timers |
| `show lldp neighbors` | Displays directly connected LLDP neighbors |
| `show lldp neighbors detail` | Displays detailed information about neighbors |
| `show lldp interface` | Displays LLDP information for interfaces |
| `show lldp interface F0/1` | Displays LLDP status for a specific interface |
| `lldp run` | Enables LLDP globally |
| `no lldp run` | Disables LLDP globally |
| `lldp transmit` | Enables LLDP transmission on an interface |
| `no lldp transmit` | Disables LLDP transmission on an interface |
| `lldp receive` | Enables LLDP reception on an interface |
| `no lldp receive` | Disables LLDP reception on an interface |
| `no cdp run` | Disables CDP globally |

---

## Verification Checklist

- [ ] CDP is disabled on R1.
- [ ] CDP is disabled on R2.
- [ ] CDP is disabled on SW1.
- [ ] CDP is disabled on SW2.
- [ ] LLDP is enabled on R1.
- [ ] LLDP is enabled on R2.
- [ ] LLDP is enabled on SW1.
- [ ] LLDP is enabled on SW2.
- [ ] Default LLDP timers have been identified.
- [ ] LLDP is used to identify connected interfaces.
- [ ] LLDP detailed information is used to identify neighboring IOS versions.
- [ ] SW2 F0/1 does not transmit LLDP.
- [ ] SW2 F0/1 does not receive LLDP.
- [ ] Configurations are saved.

---

## Key Skills Practiced

This lab provides hands-on practice with:

- Cisco Discovery Protocol (CDP)
- Link Layer Discovery Protocol (LLDP)
- Neighbor discovery
- Interface identification
- IOS version discovery
- LLDP timers
- Interface-level LLDP control
- Cisco IOS verification commands
- Network troubleshooting and documentation

## Completion Criteria

The lab is complete when **CDP is disabled, LLDP is active on all networking devices, neighboring devices and their interfaces can be identified through LLDP, IOS versions can be discovered using detailed LLDP information, and SW2 F0/1 is configured to neither transmit nor receive LLDP advertisements.**
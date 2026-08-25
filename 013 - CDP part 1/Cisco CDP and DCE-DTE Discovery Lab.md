# Cisco CDP and DCE/DTE Discovery Lab

## Lab Objective

In this lab, you will use **Cisco Discovery Protocol (CDP)** to identify neighboring network devices and interfaces, determine the DCE/DTE side of a serial connection, configure the required clock rate, and control CDP operation globally and per interface.

You will also observe how CDP behaves when it is disabled and re-enabled.

---

## Lab Tasks

1. Use CDP to identify which interfaces are used to connect the routers and switches.

2. Determine which side of the serial connection between **R1 and R2** is **DCE** and which is **DTE**. Configure a clock rate of **64 Kb/s** on the DCE side.

3. Determine the default CDP **send timer** and **hold timer**, and confirm the values using a `show` command.

4. Disable CDP globally on **R1**, then attempt to view CDP neighbors.

5. Re-enable CDP globally on **R1**, then immediately check the CDP neighbor table. Determine whether **SW1 and R2** appear instantly.

6. Disable CDP on the switch interfaces connected to the PCs.

7. Save the configurations on all affected devices.

---

## Topology / Device Connections

Based on the CDP output from the lab:

| Device | Local Interface | Neighbor | Neighbor Interface |
|---|---|---|---|
| R1 | Fa0/0 | SW1 | Fa0/1 |
| R1 | Serial2/0 | R2 | Serial2/0 |
| R2 | Fa0/0 | SW2 | Fa0/1 |
| R2 | Serial2/0 | R1 | Serial2/0 |

### Switch Interfaces Connected to PCs

- **SW1:** Fa0/3 and Fa0/4
- **SW2:** Fa0/3 and Fa0/4

CDP should be disabled on these PC-facing interfaces.

---

## Key Commands

### 1. View CDP Neighbors

```cisco
show cdp neighbors
```

For more detailed information:

```cisco
show cdp neighbors detail
```

The output identifies:

- Neighbor device ID
- Local interface
- Neighbor interface
- Hold time
- Device capability
- Platform

---

### 2. Determine DCE/DTE

On R1, the following command was used:

```cisco
show controllers serial 2/0
```

The lab output showed:

```text
Interface Serial2/0
Hardware is PowerQUICC MPC860
DCE V.35, clock rate 2000000
```

Therefore:

- **R1 Serial2/0 = DCE**
- **R2 Serial2/0 = DTE**

Configure the clock rate on R1:

```cisco
configure terminal
interface serial 2/0
clock rate 64000
end
write memory
```

The configured rate is:

**64,000 bits/sec = 64 Kb/s**

> Note: Cisco IOS uses `64000` for a 64 Kb/s clock rate.

---

## 3. Verify CDP Timers

Use:

```cisco
show cdp interface
```

The lab output showed:

```text
Sending CDP packets every 60 seconds
Holdtime is 180 seconds
```

Therefore, the default CDP timers are:

| Timer | Default |
|---|---:|
| CDP Send Timer | 60 seconds |
| CDP Hold Timer | 180 seconds |

### Answer

**CDP sends advertisements every 60 seconds and keeps received information for 180 seconds.**

---

## 4. Disable CDP Globally on R1

Enter global configuration mode:

```cisco
configure terminal
no cdp run
end
```

Attempt to view the neighbors:

```cisco
show cdp neighbors
```

Expected result:

```text
% CDP is not enabled
```

### Observation

When CDP is disabled globally, R1 cannot use CDP to discover or display neighboring devices.

---

## 5. Re-enable CDP on R1

Enable CDP again:

```cisco
configure terminal
cdp run
end
```

Immediately check the neighbors:

```cisco
show cdp neighbors
```

Initially, the neighbor table may be empty:

```text
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
```

After waiting for CDP advertisements, check again:

```cisco
show cdp neighbors
```

R1 should eventually rediscover:

```text
SW1          Fas 0/0          ...    S    2960     Fas 0/1
R2           Ser 2/0          ...    R    PT1000   Ser 2/0
```

### Answer

**No. SW1 and R2 do not necessarily appear instantly.**

CDP advertisements are sent periodically. Because the default send interval is **60 seconds**, R1 may need to wait for the neighboring devices to send their next CDP advertisements before they appear in the neighbor table.

This explains why the neighbor table was initially empty and populated later in the lab.

---

## 6. Disable CDP on PC-Facing Switch Ports

On SW1:

```cisco
configure terminal
interface range fastethernet 0/3 - 4
no cdp enable
end
write memory
```

On SW2:

```cisco
configure terminal
interface range fastethernet 0/3 - 4
no cdp enable
end
write memory
```

### Why Disable CDP on PC Ports?

PCs do not normally need to participate in CDP discovery. Disabling CDP on unused or end-device-facing interfaces reduces unnecessary CDP advertisements and limits the amount of network-device information exposed on those ports.

---

## Verification Commands

Use the following commands to verify the completed configuration:

### CDP Status

```cisco
show cdp
```

### CDP Neighbors

```cisco
show cdp neighbors
```

### Detailed CDP Information

```cisco
show cdp neighbors detail
```

### CDP Interface Information

```cisco
show cdp interface
```

### Verify Serial DCE/DTE

```cisco
show controllers serial 2/0
```

### Verify Interface CDP Configuration

```cisco
show running-config
```

---

## Lab Answers

### Question 1
**Which interfaces connect the routers and switches?**

- R1 Fa0/0 ↔ SW1 Fa0/1
- R1 Serial2/0 ↔ R2 Serial2/0
- R2 Fa0/0 ↔ SW2 Fa0/1

### Question 2
**Which side is DCE?**

R1 Serial2/0 is the **DCE** side.

R2 Serial2/0 is the **DTE** side.

The DCE side was configured with:

```cisco
clock rate 64000
```

### Question 3
**What are the default CDP timers?**

- Send timer: **60 seconds**
- Hold timer: **180 seconds**

Verified with:

```cisco
show cdp interface
```

### Question 4
**What happens when CDP is disabled globally on R1?**

`show cdp neighbors` reports:

```text
% CDP is not enabled
```

### Question 5
**Do SW1 and R2 appear immediately after CDP is re-enabled?**

**No.** CDP neighbors may not appear immediately because CDP advertisements are sent periodically. After the neighboring devices send their CDP advertisements, they appear in the neighbor table.

### Question 6
**Which switch interfaces should have CDP disabled?**

- SW1 Fa0/3
- SW1 Fa0/4
- SW2 Fa0/3
- SW2 Fa0/4

Command:

```cisco
no cdp enable
```

---

## Completion Checklist

- [ ] Identified router/switch connections using CDP
- [ ] Identified R1 as the DCE side
- [ ] Identified R2 as the DTE side
- [ ] Configured `clock rate 64000` on R1 Serial2/0
- [ ] Verified CDP send timer of 60 seconds
- [ ] Verified CDP hold timer of 180 seconds
- [ ] Disabled CDP globally on R1
- [ ] Verified that CDP neighbors could not be displayed
- [ ] Re-enabled CDP globally on R1
- [ ] Observed the delay before neighbors reappeared
- [ ] Disabled CDP on SW1 Fa0/3-4
- [ ] Disabled CDP on SW2 Fa0/3-4
- [ ] Saved the configurations

---

## Expected Final State

At the end of the lab:

- **R1 Serial2/0** is the DCE interface with a **64 Kb/s clock rate**.
- **R2 Serial2/0** is the DTE interface.
- CDP is enabled globally on R1.
- R1 can discover SW1 and R2 after CDP advertisements are received.
- SW1 and SW2 have CDP disabled on their PC-facing interfaces.
- All configurations are saved.

**Lab completed successfully.**
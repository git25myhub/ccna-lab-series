# Cisco CDP and Serial Interface Configuration Lab

## 📌 Lab Overview

This lab demonstrates how to use **Cisco Discovery Protocol (CDP)** to identify neighboring network devices and interfaces, determine the **DCE/DTE** ends of a serial connection, configure the serial clock rate, assign IP addresses, and verify connectivity between two routers.

---

## 🎯 Objectives

By completing this lab, you will:

- Use CDP to discover directly connected Cisco devices.
- Identify which interfaces connect the routers and switches.
- Determine the DCE and DTE ends of a serial cable.
- Configure a `64 Kb/s` clock rate on the DCE interface.
- Configure IP addresses on the serial interfaces.
- Verify serial interface status.
- Test connectivity between R1 and R2 using ping.
- Save the configurations.

---

# 🖥️ Topology

```text
                 Serial Connection
             192.168.0.0/24 Network
        DCE                    DTE
      +-----+                +-----+
      | R1  |================| R2  |
      +-----+                +-----+
        |                       |
     Fa0/0                    Fa0/0
        |                       |
     Fa0/1                    Fa0/1
        |                       |
      +-----+                +-----+
      | SW1 |                | SW2 |
      +-----+                +-----+
```

### Devices

| Device | Interface | Connected To |
|---|---|---|
| R1 | FastEthernet0 | SW1 |
| SW1 | FastEthernet0/1 | R1 FastEthernet0 |
| R1 | Serial0 | R2 Serial0 |
| R2 | Serial0 | R1 Serial0 |
| R2 | FastEthernet0 | SW2 |
| SW2 | FastEthernet0/1 | R2 FastEthernet0 |

> The lab output confirms that SW1 sees R1 through `FastEthernet0/1`, while SW2 sees R2 through `FastEthernet0/1`.

---

# 1. Use CDP to Discover Connected Interfaces

Cisco Discovery Protocol allows Cisco devices to discover directly connected Cisco neighbors.

## On SW1

Enter privileged EXEC mode:

```cisco
SW1> enable
```

Display CDP neighbors:

```cisco
SW1# show cdp neighbors
```

The lab output shows:

```text
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
R1           Fas 0/1          135            R       C810        Fas 0
```

### SW1 → R1 Connection

```text
SW1 FastEthernet0/1
        |
        |
R1 FastEthernet0
```

Therefore:

```text
SW1 Fa0/1 → R1 Fa0
```

---

## On SW2

Display CDP neighbors:

```cisco
SW2# show cdp neighbors
```

The lab output shows:

```text
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
R2           Fas 0/1          165            R       C810        Fas 0
```

### SW2 → R2 Connection

```text
SW2 FastEthernet0/1
        |
        |
R2 FastEthernet0
```

Therefore:

```text
SW2 Fa0/1 → R2 Fa0
```

---

# 2. Understanding CDP Output

The `show cdp neighbors` command displays information about directly connected Cisco devices.

Example:

```text
Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
R1           Fas 0/1         135       R            C810       Fas 0
```

### Important Fields

| Field | Meaning |
|---|---|
| Device ID | Hostname of the neighboring device |
| Local Intrfce | Interface on the local device |
| Holdtme | Time remaining before CDP information expires |
| Capability | Capabilities of the neighbor |
| Platform | Cisco hardware platform |
| Port ID | Interface used by the neighboring device |

For SW1:

```text
Local Interface = Fa0/1
Remote Device   = R1
Remote Interface = Fa0
```

For SW2:

```text
Local Interface = Fa0/1
Remote Device   = R2
Remote Interface = Fa0
```

---

# 3. Identify the DCE and DTE Ends

The serial connection between R1 and R2 uses a **DCE/DTE serial cable**.

One end must provide clocking.

- **DCE** → Provides clocking.
- **DTE** → Receives clocking.

The clock rate must be configured on the **DCE end**.

---

## Determine the DCE Interface

On R1, use:

```cisco
R1# show controllers serial 0
```

Look for an indication such as:

```text
DCE
```

If R1 reports that Serial0 is DCE, then:

```text
R1 Serial0 = DCE
R2 Serial0 = DTE
```

If R2 reports DCE instead, configure the clock rate on R2 instead.

### In this lab

The configuration confirms that R1 Serial0 is the DCE side because the following command was successfully applied:

```cisco
R1(config-if)# clock rate 64000
```

Therefore:

```text
R1 Serial0 → DCE
R2 Serial0 → DTE
```

---

# 4. Configure the DCE Clock Rate

On R1:

```cisco
R1# configure terminal
```

Enter the serial interface:

```cisco
R1(config)# interface serial 0
```

Configure the clock rate:

```cisco
R1(config-if)# clock rate 64000
```

The configured clock rate is:

```text
64,000 bits/second
```

or:

```text
64 Kb/s
```

Verify:

```cisco
R1# show running-config
```

You should see:

```text
interface Serial0
 ip address 192.168.0.1 255.255.255.0
 clock rate 64000
```

---

# 5. Configure R1 Serial Interface

Configure the IP address:

```cisco
R1# configure terminal
R1(config)# interface serial 0
R1(config-if)# ip address 192.168.0.1 255.255.255.0
```

The `/24` subnet mask is:

```text
255.255.255.0
```

Bring the interface up if required:

```cisco
R1(config-if)# no shutdown
```

Exit configuration mode:

```cisco
R1(config-if)# end
```

Save the configuration:

```cisco
R1# write memory
```

Expected:

```text
Building configuration...
[OK]
```

---

# 6. Configure R2 Serial Interface

On R2:

```cisco
R2# configure terminal
```

Enter Serial0:

```cisco
R2(config)# interface serial 0
```

Configure the IP address:

```cisco
R2(config-if)# ip address 192.168.0.2 255.255.255.0
```

Bring the interface up:

```cisco
R2(config-if)# no shutdown
```

Exit:

```cisco
R2(config-if)# end
```

Save:

```cisco
R2# write memory
```

---

# 7. Verify R1 Configuration

Run:

```cisco
R1# show running-config
```

The Serial0 section should contain:

```text
interface Serial0
 ip address 192.168.0.1 255.255.255.0
 clock rate 64000
```

You can also use:

```cisco
R1# show ip interface brief
```

Expected result should show Serial0 with:

```text
Interface    IP-Address       Status    Protocol
Serial0      192.168.0.1     up        up
```

---

# 8. Verify R2 Configuration

Run:

```cisco
R2# show running-config
```

The Serial0 section should contain:

```text
interface Serial0
 ip address 192.168.0.2 255.255.255.0
```

Verify the interface:

```cisco
R2# show ip interface brief
```

Expected:

```text
Interface    IP-Address       Status    Protocol
Serial0      192.168.0.2     up        up
```

---

# 9. Test Connectivity from R2 to R1

From R2:

```cisco
R2# ping 192.168.0.1
```

The lab produced:

```text
Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 192.168.0.1, timeout is 2 seconds:

!!!!!

Success rate is 100 percent (5/5), round-trip min/avg/max = 11/14/17 ms
```

The five exclamation marks:

```text
!!!!!
```

indicate successful ICMP replies.

### Result

```text
R2 192.168.0.2
       |
       | Serial
       |
R1 192.168.0.1

Connectivity: SUCCESS
```

---

# 10. Test Connectivity from R1 to R2

From R1:

```cisco
R1# ping 192.168.0.2
```

A successful result should show:

```text
!!!!!
```

and:

```text
Success rate is 100 percent (5/5)
```

This confirms bidirectional connectivity.

---

# 11. Useful Verification Commands

### Display CDP neighbors

```cisco
show cdp neighbors
```

### Display detailed CDP information

```cisco
show cdp neighbors detail
```

### Identify DCE/DTE

```cisco
show controllers serial 0
```

### Display interface IP addresses and status

```cisco
show ip interface brief
```

### Display the serial interface configuration

```cisco
show running-config interface serial 0
```

### Display interface details

```cisco
show interfaces serial 0
```

### Test connectivity

```cisco
ping 192.168.0.1
ping 192.168.0.2
```

---

# 🧠 Key Concepts

| Concept | Explanation |
|---|---|
| CDP | Cisco protocol used to discover directly connected Cisco devices |
| DCE | Serial side responsible for providing clocking |
| DTE | Serial side that receives clocking |
| Clock Rate | Controls the timing of serial communication |
| `clock rate 64000` | Sets the DCE clock to 64 Kb/s |
| `/24` | Equivalent to `255.255.255.0` |
| `no shutdown` | Activates an administratively down interface |
| `show cdp neighbors` | Displays directly connected Cisco neighbors |
| `show controllers serial 0` | Helps identify DCE/DTE |
| `show ip interface brief` | Quickly verifies interface status and IP addresses |
| `ping` | Tests Layer 3 connectivity |

---

# ❓ Lab Questions

### 1. Which interface on SW1 connects to R1?

```text
SW1 FastEthernet0/1 → R1 FastEthernet0
```

### 2. Which interface on SW2 connects to R2?

```text
SW2 FastEthernet0/1 → R2 FastEthernet0
```

### 3. Which interface is connected between R1 and R2?

```text
R1 Serial0 ↔ R2 Serial0
```

### 4. Which router is the DCE in this lab?

```text
R1
```

Therefore:

```text
R1 Serial0 = DCE
R2 Serial0 = DTE
```

### 5. What clock rate was configured?

```text
64,000 bits/second
```

or:

```text
64 Kb/s
```

### 6. What IP address is assigned to R1 Serial0?

```text
192.168.0.1/24
```

### 7. What IP address is assigned to R2 Serial0?

```text
192.168.0.2/24
```

### 8. What command identifies the DCE/DTE side?

```cisco
show controllers serial 0
```

### 9. What command discovers directly connected Cisco devices?

```cisco
show cdp neighbors
```

### 10. Was connectivity successful?

**Yes.** R2 successfully pinged R1 with:

```text
Success rate is 100 percent (5/5)
```

---

# 📋 Final Configuration Summary

## R1

```text
Hostname:       R1
Interface:      Serial0
Role:           DCE
IP Address:     192.168.0.1/24
Clock Rate:     64000
```

## R2

```text
Hostname:       R2
Interface:      Serial0
Role:           DTE
IP Address:     192.168.0.2/24
Clock Rate:     Not configured
```

## Serial Network

```text
Network:        192.168.0.0/24
R1:             192.168.0.1
R2:             192.168.0.2
Clock Rate:     64 Kb/s
Connectivity:   100% successful
```

---

# 🔍 Troubleshooting

If the ping fails, check the following:

### Check interface status

```cisco
show ip interface brief
```

The Serial0 interface should show:

```text
up    up
```

### If the interface is administratively down

Configure:

```cisco
interface serial 0
no shutdown
```

### Verify the IP addresses

R1:

```text
192.168.0.1 255.255.255.0
```

R2:

```text
192.168.0.2 255.255.255.0
```

### Verify the DCE clock rate

```cisco
show controllers serial 0
```

If R1 is DCE, ensure:

```cisco
clock rate 64000
```

is configured under R1 Serial0.

### Check the serial configuration

```cisco
show running-config interface serial 0
```

---

# 💾 Save the Configuration

After completing the lab, save both routers:

```cisco
R1# copy running-config startup-config
```

and:

```cisco
R2# copy running-config startup-config
```

Or use:

```cisco
R1# write memory
R2# write memory
```

---

# ✅ Lab Completion Checklist

- [x] Connected R1 to SW1.
- [x] Connected R2 to SW2.
- [x] Used CDP on SW1 to discover R1.
- [x] Used CDP on SW2 to discover R2.
- [x] Identified R1 Serial0 as the DCE side.
- [x] Identified R2 Serial0 as the DTE side.
- [x] Configured a `64 Kb/s` clock rate on R1.
- [x] Configured R1 Serial0 as `192.168.0.1/24`.
- [x] Configured R2 Serial0 as `192.168.0.2/24`.
- [x] Verified the serial interfaces.
- [x] Tested R2 → R1 connectivity.
- [x] Achieved 100% ping success.
- [x] Saved the configurations.

---

## 🏁 Final Result

The lab successfully established a serial connection between R1 and R2 using the `192.168.0.0/24` network.

```text
                 192.168.0.0/24
          DCE                    DTE
     R1 Serial0  ==============  R2 Serial0
     192.168.0.1                 192.168.0.2
        |                             |
     SW1 Fa0/1                    SW2 Fa0/1
        |                             |
      R1 Fa0                        R2 Fa0
```

The DCE side on R1 provides the **64 Kb/s clocking**, and both routers can successfully communicate using ICMP.
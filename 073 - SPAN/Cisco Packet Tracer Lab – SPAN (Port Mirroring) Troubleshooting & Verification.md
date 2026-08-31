# Cisco Packet Tracer Lab – SPAN (Port Mirroring) Troubleshooting & Verification

## 📌 Lab Overview

This lab demonstrates how to configure and verify **SPAN (Switched Port Analyzer)**, also known as **port mirroring**, on a Cisco switch.

SPAN allows a switch to copy traffic from a specified source interface or VLAN to a destination interface connected to a monitoring device such as a server, IDS/IPS, or packet analyzer.

In this lab:

- **PC2** communicates with **PC1** using ICMP.
- **SW1** is configured to monitor traffic on `GigabitEthernet0/1`.
- **SRV1** is connected to the SPAN destination interface.
- Both **incoming and outgoing traffic** on `G0/1` are mirrored to `G0/2`.
- Packet Tracer's **Simulation Mode** is used to observe the ICMP packets and their mirrored copies.

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

1. Generate traffic between two hosts.
2. Use ARP to resolve the destination MAC address before analyzing ICMP traffic.
3. Use Packet Tracer's **Simulation Mode** to inspect packet paths.
4. Configure SPAN on a Cisco switch.
5. Mirror both **input and output traffic** from an interface.
6. Send mirrored traffic to a monitoring server.
7. Verify that both ICMP Echo and Echo Reply packets are copied to the monitoring device.

---

## 🗺️ Topology

The topology contains:

```text
             ┌──────────┐
             │   PC2    │
             └────┬─────┘
                  │
                  │
             ┌────┴─────┐
             │   SW1    │
             │          │
             │ G0/1     │ G0/2
             └────┬─────┘    └────────┐
                  │                   │
                  │                   │
             ┌────▼─────┐       ┌────▼─────┐
             │   PC1    │       │   SRV1    │
             └──────────┘       │ Monitoring│
                                └───────────┘
```

> **Note:** The exact physical connections depend on the Packet Tracer topology. The important SPAN relationship is that `G0/1` is the **source** interface and `G0/2` is the **destination** interface connected to `SRV1`.

---

# 1. Generate Initial Traffic

Before analyzing the ICMP packets in Simulation Mode, the ARP process must be completed.

From **PC2**, ping PC1:

```text
C:\>ping 10.0.1.10
```

The first ping produced:

```text
Ping statistics for 10.0.1.10:

    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The first packet timing out is expected in this situation because PC2 may initially need to perform **ARP resolution** before it can successfully send the ICMP Echo Request.

The following packets were successful:

```text
Reply from 10.0.1.10: bytes=32 time<1ms TTL=126
Reply from 10.0.1.10: bytes=32 time<1ms TTL=126
Reply from 10.0.1.10: bytes=32 time<1ms TTL=126
```

---

# 2. Verify Connectivity Again

Run the ping a second time:

```text
C:\>ping 10.0.1.10
```

The second ping completed successfully:

```text
Ping statistics for 10.0.1.10:

    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms that the ARP resolution has already taken place and PC2 can now communicate with PC1 without needing the initial ARP exchange.

---

# 3. Enter Packet Tracer Simulation Mode

Switch Packet Tracer from **Realtime** mode to **Simulation** mode.

The Simulation Mode controls are located in the bottom-right corner of Packet Tracer.

In Simulation Mode:

1. Clear any existing events if necessary.
2. Filter the displayed protocols so that **ARP** and **ICMP** can be observed.
3. Start a ping from PC2 to PC1.
4. Use **Capture/Forward** to move through the packet events one step at a time.

---

# 4. Analyze the ICMP Path

After ARP has already been resolved, initiate another ping from PC2 to PC1.

Observe the **ICMP Echo Request**.

The normal forwarding path should be:

```text
PC2
  │
  ▼
SW1
  │
  ▼
PC1
```

PC1 then generates an **ICMP Echo Reply**:

```text
PC1
  │
  ▼
SW1
  │
  ▼
PC2
```

Therefore, two types of ICMP messages should be observed:

### ICMP Echo Request

```text
PC2 ───────► SW1 ───────► PC1
```

### ICMP Echo Reply

```text
PC1 ───────► SW1 ───────► PC2
```

The purpose of this part of the lab is to understand the normal traffic path before SPAN is verified.

---

# 5. Configure SPAN on SW1

SPAN is configured using the `monitor session` command.

The required configuration is:

```cisco
SW1> enable
SW1# configure terminal

SW1(config)# monitor session 1 source interface gigabitEthernet0/1 both
SW1(config)# monitor session 1 destination interface gigabitEthernet0/2
```

The configuration used in the lab was:

```cisco
SW1(config)# monitor session 1 source int g0/1 both
SW1(config)# monitor session 1 dest int g0/2
```

Save the configuration:

```cisco
SW1(config)# do write
```

Expected result:

```text
Building configuration...

[OK]
```

---

# 6. Understand the SPAN Configuration

The configuration consists of three important components.

### SPAN Session

```cisco
monitor session 1
```

Creates SPAN session **1**.

### Source Interface

```cisco
source interface gigabitEthernet0/1 both
```

Specifies `G0/1` as the interface being monitored.

The keyword:

```text
both
```

means that both directions of traffic are mirrored:

- **RX** – traffic entering `G0/1`
- **TX** – traffic leaving `G0/1`

### Destination Interface

```cisco
destination interface gigabitEthernet0/2
```

Specifies `G0/2` as the SPAN destination.

The device connected to this interface, **SRV1**, receives copies of the monitored traffic.

---

# 7. Verify the SPAN Configuration

After configuring SPAN, verify the configuration on SW1:

```cisco
SW1# show monitor
```

You should see a SPAN session similar to:

```text
Session 1
---------
Type                   : Local Session
Source Ports           :
    Both               : Gi0/1
Destination Ports      : Gi0/2
```

You can also inspect the running configuration:

```cisco
SW1# show running-config
```

Look for:

```cisco
monitor session 1 source interface GigabitEthernet0/1 both
monitor session 1 destination interface GigabitEthernet0/2
```

---

# 8. Test SPAN in Simulation Mode

Return to Packet Tracer's **Simulation Mode**.

From PC2, ping PC1 again:

```text
C:\>ping 10.0.1.10
```

Because the SPAN session is configured, SW1 should now perform two actions when traffic crosses the monitored interface:

1. Forward the original packet toward its destination.
2. Create a copy of the packet and send the copy through the SPAN destination interface.

The traffic can therefore be visualized conceptually as:

### ICMP Echo Request

```text
                    ┌──────────────► PC1
                    │
PC2 ─────────────► SW1
                    │
                    └──────────────► SRV1
                         (copy)
```

### ICMP Echo Reply

```text
                    ┌──────────────► PC2
                    │
PC1 ─────────────► SW1
                    │
                    └──────────────► SRV1
                         (copy)
```

---

# 9. Expected Result

In Simulation Mode, confirm that **SRV1 receives copies of both directions of ICMP traffic**.

You should be able to identify:

```text
ICMP Echo Request
PC2 → SW1 → PC1
          └────→ SRV1 (mirrored copy)
```

and:

```text
ICMP Echo Reply
PC1 → SW1 → PC2
          └────→ SRV1 (mirrored copy)
```

The original packets continue toward their intended destination, while SPAN creates copies for the monitoring device.

---

# 10. Verification Checklist

The lab is successfully completed when all of the following are true:

- [x] PC2 can ping PC1.
- [x] ARP resolution has been completed before packet analysis.
- [x] Packet Tracer Simulation Mode is used.
- [x] The normal ICMP Echo Request path is observed.
- [x] The normal ICMP Echo Reply path is observed.
- [x] SW1 has a SPAN session configured.
- [x] `G0/1` is configured as the SPAN source.
- [x] Both input and output traffic are monitored.
- [x] `G0/2` is configured as the SPAN destination.
- [x] SRV1 is connected to the destination interface.
- [x] Copies of ICMP Echo Requests are sent to SRV1.
- [x] Copies of ICMP Echo Replies are sent to SRV1.

---

# 11. Important Cisco Commands

### Configure SPAN

```cisco
configure terminal
monitor session 1 source interface GigabitEthernet0/1 both
monitor session 1 destination interface GigabitEthernet0/2
end
```

### Verify SPAN

```cisco
show monitor
```

### View the running configuration

```cisco
show running-config
```

### Save the configuration

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

---

# 🧠 Key Concepts Learned

## SPAN

**SPAN (Switched Port Analyzer)** copies traffic from a source interface or VLAN to a destination interface.

It is commonly used for:

- Network troubleshooting
- Packet analysis
- IDS/IPS monitoring
- Network performance analysis
- Security monitoring
- Traffic inspection

## Source Interface

The source is the interface whose traffic is being copied.

In this lab:

```text
G0/1
```

## Destination Interface

The destination is the interface where the mirrored traffic is sent.

In this lab:

```text
G0/2
```

## `both`

The `both` keyword means that traffic in **both directions** is mirrored.

```text
RX + TX
```

Without `both`, you can specify only one direction:

```text
rx
tx
```

---

# 🔍 Troubleshooting Notes

If SRV1 does not appear to receive mirrored traffic, check the following:

### 1. Verify the SPAN session

```cisco
show monitor
```

Confirm:

```text
Source = Gi0/1
Destination = Gi0/2
```

### 2. Confirm both directions are monitored

The source should contain:

```cisco
both
```

### 3. Check interface status

```cisco
show ip interface brief
```

Confirm that the relevant interfaces are operational.

### 4. Verify the physical topology

Make sure:

```text
SW1 G0/2 ─── SRV1
```

is the actual connection used for the SPAN destination.

### 5. Use Simulation Mode

Packet Tracer's Simulation Mode is particularly useful for confirming that the mirrored packets are being generated and forwarded.

---

# 🏁 Final Result

The lab demonstrates that SPAN allows SW1 to transparently copy traffic from `GigabitEthernet0/1` to `GigabitEthernet0/2`.

The final configuration is:

```cisco
monitor session 1 source interface GigabitEthernet0/1 both
monitor session 1 destination interface GigabitEthernet0/2
```

With this configuration, **SRV1 receives copies of both ICMP Echo Requests and ICMP Echo Replies**, while the original traffic continues normally between PC2 and PC1.

This provides a practical demonstration of how SPAN can be used to send network traffic to a monitoring device without interfering with the original communication.
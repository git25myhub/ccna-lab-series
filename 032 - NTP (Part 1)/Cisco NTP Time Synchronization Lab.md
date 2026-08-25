# Cisco NTP Time Synchronization Lab

## Overview

In this lab, you will configure three Cisco routers to maintain synchronized system clocks using **Network Time Protocol (NTP)**.

R1 will act as the central NTP server. R2 will synchronize its clock with R1, and R3 will synchronize its clock with R2, creating a simple hierarchical time-synchronization structure.

## Lab Objectives

By completing this lab, you will:

- Configure the correct timezone on Cisco routers.
- Manually configure the date and time on R1.
- Configure R1 as an NTP master/server.
- Configure R2 to synchronize with R1.
- Configure R3 to synchronize with R2.
- Verify NTP synchronization using Cisco IOS commands.
- Understand NTP stratum hierarchy.

## Topology

```text
        NTP Server
           R1
      192.168.12.1
           |
           |
      192.168.12.0
           |
           R2
      192.168.23.2
           |
           |
      192.168.23.0
           |
           R3
```

### NTP Hierarchy

```text
R1
NTP Master
Stratum 8
   |
   v
R2
NTP Client
Stratum 9
   |
   v
R3
NTP Client
Stratum 10
```

> **Note:** In this Packet Tracer lab, configuring `ntp master` without specifying a stratum causes R1 to use the default NTP master stratum. The verification output shows R1's reference clock as `127.127.1.1` and R2/R3 receiving time through the NTP hierarchy.

---

# Lab Tasks

## 1. Configure R1's Timezone

Enter configuration mode on R1:

```cisco
R1> enable
R1# configure terminal
```

Configure the timezone. For the local timezone used in this lab, GMT+3 is configured:

```cisco
R1(config)# clock timezone GMT 3
```

Verify:

```cisco
R1(config)# do show clock
```

The clock should now display the `GMT` timezone with the configured offset.

---

## 2. Configure the Date and Time on R1

Set R1's clock to the required local date and time.

Example:

```cisco
R1# clock set 07:00:00 Aug 25 2026
```

Verify the configuration:

```cisco
R1# show clock detail
```

Expected output will resemble:

```text
10:00:19.67 GMT Tue Aug 25 2026
Time source is user configuration
```

The exact displayed time depends on the configured timezone.

---

## 3. Configure R1 as an NTP Server

Enter configuration mode:

```cisco
R1# configure terminal
```

Configure R1 as an NTP master:

```cisco
R1(config)# ntp master
```

Save the configuration:

```cisco
R1(config)# end
R1# write
```

Verify the NTP configuration:

```cisco
R1# show running-config | include ntp
```

You should see:

```text
ntp master
```

You can also check the clock source:

```cisco
R1# show clock detail
```

---

# 4. Configure R2 as an NTP Client

First configure R2's timezone:

```cisco
R2> enable
R2# configure terminal
R2(config)# clock timezone GMT 3
```

Configure R1 as R2's NTP server:

```cisco
R2(config)# ntp server 192.168.12.1
```

Exit configuration mode and save:

```cisco
R2(config)# end
R2# write
```

Verify the clock:

```cisco
R2# show clock detail
```

Once synchronization occurs, the output should indicate:

```text
Time source is NTP
```

Verify the NTP association:

```cisco
R2# show ntp associations
```

Example:

```text
address         ref clock       st   when     poll    reach
~192.168.12.1   127.127.1.1     8    7        16      7
```

The `~` indicates that the NTP server has been configured.

The `*` symbol indicates the current system peer when synchronization has been established.

---

# 5. Configure R3 as an NTP Client

Configure R3's timezone:

```cisco
R3> enable
R3# configure terminal
R3(config)# clock timezone GMT 3
```

Configure R2 as R3's NTP server:

```cisco
R3(config)# ntp server 192.168.23.2
```

Save the configuration:

```cisco
R3(config)# end
R3# write
```

Verify NTP synchronization:

```cisco
R3# show clock detail
```

Then verify the NTP association:

```cisco
R3# show ntp associations
```

Example:

```text
address         ref clock       st   when     poll    reach
~192.168.23.2   192.168.12.1    9    5        16      1
```

This shows that R3 is receiving its time from R2, while R2 receives its time from R1.

---

# Verification

Use the following commands on each router.

### R1

```cisco
show clock detail
show running-config | include ntp
```

R1 should be configured as the NTP master.

### R2

```cisco
show clock detail
show ntp associations
```

R2 should show:

```text
Time source is NTP
```

and should have R1 (`192.168.12.1`) as its NTP peer.

### R3

```cisco
show clock detail
show ntp associations
```

R3 should show:

```text
Time source is NTP
```

and should have R2 (`192.168.23.2`) as its NTP peer.

---

# Important Verification Commands

| Command | Purpose |
|---|---|
| `show clock` | Displays the current system time |
| `show clock detail` | Displays the time and its source |
| `show ntp associations` | Displays configured NTP peers and synchronization status |
| `show ntp status` | Displays detailed NTP synchronization information |
| `show running-config \| include ntp` | Displays NTP-related configuration |
| `write` | Saves the running configuration |

---

# Understanding the NTP Hierarchy

NTP uses **stratum levels** to indicate how far a device is from an authoritative time source.

In this lab:

```text
R1
NTP Master
   |
   | NTP
   v
R2
   |
   | NTP
   v
R3
```

R1 provides time to R2.

R2 receives time from R1 and then provides time to R3.

Therefore, R3 does **not** synchronize directly with R1. It learns the time through R2.

The reference clock shown by R1 in Packet Tracer may appear as:

```text
127.127.1.1
```

This represents the router's local NTP master/reference clock.

---

# Troubleshooting

If R2 or R3 does not synchronize, check the following:

### 1. Verify IP connectivity

From R2:

```cisco
R2# ping 192.168.12.1
```

From R3:

```cisco
R3# ping 192.168.23.2
```

The pings should succeed.

### 2. Check the NTP configuration

On R2:

```cisco
R2# show running-config | include ntp
```

Expected:

```text
ntp server 192.168.12.1
```

On R3:

```cisco
R3# show running-config | include ntp
```

Expected:

```text
ntp server 192.168.23.2
```

### 3. Check NTP associations

```cisco
show ntp associations
```

Look for the `*` symbol indicating the selected system peer.

### 4. Check the clock source

```cisco
show clock detail
```

A synchronized client should report:

```text
Time source is NTP
```

If it reports:

```text
Time source is user configuration
```

the router is not currently using NTP as its time source.

---

# Useful Configuration Summary

### R1

```cisco
enable
configure terminal
clock timezone GMT 3
ntp master
end
clock set 07:00:00 Aug 25 2026
write
```

### R2

```cisco
enable
configure terminal
clock timezone GMT 3
ntp server 192.168.12.1
end
write
```

### R3

```cisco
enable
configure terminal
clock timezone GMT 3
ntp server 192.168.23.2
end
write
```

---

# Completion Criteria

The lab is successfully completed when:

- [ ] R1 has the correct timezone configured.
- [ ] R1 has the correct date and time.
- [ ] R1 is configured as an NTP master.
- [ ] R2 has the correct timezone configured.
- [ ] R2 synchronizes with R1.
- [ ] R3 has the correct timezone configured.
- [ ] R3 synchronizes with R2.
- [ ] `show clock detail` on R2 reports `Time source is NTP`.
- [ ] `show clock detail` on R3 reports `Time source is NTP`.
- [ ] `show ntp associations` confirms the expected NTP hierarchy.
- [ ] Configurations are saved with `write`.

## Key Takeaway

This lab demonstrates how NTP can distribute accurate time across a network using a hierarchical design:

**R1 → R2 → R3**

R1 acts as the authoritative NTP master, R2 synchronizes with R1 and becomes the time source for R3, and R3 synchronizes with R2. This approach allows network devices to maintain consistent timestamps for logs, troubleshooting, security events, and other time-dependent operations.
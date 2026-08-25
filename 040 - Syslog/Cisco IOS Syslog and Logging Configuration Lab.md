# Cisco IOS Syslog and Logging Configuration Lab

## 📌 Lab Overview

In this lab, you will configure and troubleshoot **Cisco IOS logging and Syslog** on router **R1**.

The lab covers:

- Understanding Cisco Syslog severity levels
- Configuring timestamps with milliseconds
- Configuring an enable secret
- Configuring Telnet access on VTY lines
- Displaying logging messages on VTY sessions
- Configuring synchronous logging
- Enabling buffered logging
- Increasing the logging buffer size
- Sending Syslog messages to a remote Syslog server

The lab uses **R1**, **PC1**, **PC2**, and **SRV1**.

---

## 🎯 Learning Objectives

By completing this lab, you should be able to:

1. Identify the severity level of Cisco IOS Syslog messages.
2. Configure timestamps on logging messages.
3. Secure privileged EXEC mode using an enable secret.
4. Configure Telnet access through the VTY lines.
5. Enable logging messages on VTY sessions.
6. Configure synchronous logging on console and VTY lines.
7. Configure an 8192-byte logging buffer.
8. Configure R1 to send Syslog messages to SRV1.
9. Verify Syslog configuration using `show logging`.

---

# 🧪 Lab Tasks

## 1. Configure and Observe Console Syslog Messages

Connect to **R1's console port using PC2**.

Enter privileged EXEC mode and access the G0/0 interface:

```cisco
R1> enable
R1# configure terminal
R1(config)# interface g0/0
```

Shut down the interface:

```cisco
R1(config-if)# shutdown
```

You should receive messages similar to:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to down
```

Re-enable the interface:

```cisco
R1(config-if)# no shutdown
```

You should see:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
```

### Question

**What is the severity level of these messages?**

The messages contain:

```text
%LINK-5-CHANGED
%LINEPROTO-5-UPDOWN
```

The number **5** represents the Syslog severity level.

### Severity Level 5 — Notice

Severity level **5** is:

> **Notice — normal but significant condition**

Therefore, the interface state-change messages observed in this lab are **severity level 5 (Notice)**.

---

# 2. Enable Timestamps with Milliseconds

By default, logging messages may not include useful timestamps.

Configure R1 to include the date, time, and milliseconds:

```cisco
R1(config)# service timestamps log datetime msec
```

After configuration, messages should appear similar to:

```text
*Feb 28, 18:02:37.022:
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down
```

The `.022` represents milliseconds.

Verify the configuration with:

```cisco
R1# show running-config | include timestamps
```

Expected configuration:

```text
service timestamps log datetime msec
```

---

# 3. Configure the Enable Secret and Telnet

Configure the privileged EXEC password:

```cisco
R1(config)# enable secret ccna
```

Next, configure the VTY lines:

```cisco
R1(config)# line vty 0 15
R1(config-line)# password ccent
R1(config-line)# login
R1(config-line)# transport input telnet
```

This configuration:

- Sets the VTY password to `ccent`
- Requires users to authenticate when connecting
- Allows Telnet connections
- Applies to VTY lines 0 through 15

Save the configuration:

```cisco
R1(config-line)# end
R1# copy running-config startup-config
```

or:

```cisco
R1# write memory
```

---

# 4. Test Telnet from PC1

From **PC1**, connect to R1's G0/0 interface:

```text
C:\> telnet 192.168.1.1
```

You should receive:

```text
User Access Verification

Password:
```

Enter:

```text
ccent
```

Then enter privileged EXEC mode:

```text
R1> enable
Password:
```

Enter:

```text
ccna
```

You should now be at:

```text
R1#
```

---

# 5. Test Syslog Messages from the VTY Session

From the Telnet session, enter:

```cisco
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# no shutdown
```

### Question

**Why does no Syslog message appear on the VTY session?**

The reason is that Cisco IOS does not automatically display logging messages on VTY lines.

By default:

```text
Console logging: enabled
Monitor logging: disabled
```

The console receives logging messages, but interactive VTY sessions do not display them unless **monitor logging** is enabled.

Configure logging messages to be displayed on VTY lines:

```cisco
R1(config)# logging monitor
```

Alternatively, from the privileged EXEC mode:

```cisco
R1# terminal monitor
```

`terminal monitor` enables logging messages for the current VTY session.

Now interface state changes and other logging/debug messages can appear in the Telnet session.

---

# 6. Configure Synchronous Logging

Configure synchronous logging on the console:

```cisco
R1(config)# line console 0
R1(config-line)# logging synchronous
```

Configure synchronous logging on the VTY lines:

```cisco
R1(config)# line vty 0 15
R1(config-line)# logging synchronous
```

### Why use synchronous logging?

Syslog messages can appear while you are typing commands.

Without synchronous logging, a message may interrupt your command:

```text
R1(config)# interface g0/0
*Feb 28, 18:02:37.022: %LINK-5-CHANGED...
```

This can make the command difficult to read.

With:

```cisco
logging synchronous
```

Cisco IOS redisplays the command prompt and helps keep the command input organized after the logging message appears.

---

# 7. Enable Buffered Logging

Enable logging to the router's memory buffer:

```cisco
R1(config)# logging buffered
```

The default buffer size may be smaller than required for this lab.

Configure the buffer size to **8192 bytes**:

```cisco
R1(config)# logging buffered 8192
```

Verify the configuration:

```cisco
R1# show logging
```

You should see:

```text
Buffer logging: level debugging
```

and:

```text
Log Buffer (8192 bytes):
```

Your lab output showed:

```text
Logging Exception size (8192 bytes)
```

and:

```text
Log Buffer (8192 bytes):
```

This confirms that the logging buffer has been increased to **8192 bytes**.

---

# 8. Configure R1 to Send Logs to SRV1

The Syslog server **SRV1** has the IP address:

```text
192.168.1.100
```

Configure R1 to send Syslog messages to SRV1:

```cisco
R1(config)# logging 192.168.1.100
```

Cisco IOS will send Syslog messages to the server using **UDP port 514**.

Verify the configuration:

```cisco
R1# show logging
```

You should see something similar to:

```text
Logging to 192.168.1.100
(udp port 514)
```

Your output confirmed:

```text
Logging to 192.168.1.100 (udp port 514, audit disabled,
authentication disabled, encryption disabled, link up)
```

This confirms that R1 is configured to send Syslog messages to SRV1.

---

# 🔍 Verification Commands

Use the following commands to verify the completed configuration.

### View Syslog configuration

```cisco
R1# show logging
```

### View the running configuration

```cisco
R1# show running-config
```

### Verify timestamps

```cisco
R1# show running-config | include timestamps
```

Expected:

```text
service timestamps log datetime msec
```

### Verify logging buffer

```cisco
R1# show logging
```

Look for:

```text
Buffer logging: level debugging
Log Buffer (8192 bytes):
```

### Verify the Syslog server

```cisco
R1# show logging
```

Look for:

```text
Logging to 192.168.1.100
```

### Verify VTY configuration

```cisco
R1# show running-config | section line vty
```

You should see:

```text
line vty 0 15
 password ccent
 login
 transport input telnet
 logging synchronous
```

---

# 📋 Important Configuration Summary

| Feature | Configuration |
|---|---|
| Enable secret | `ccna` |
| VTY password | `ccent` |
| Remote access | Telnet |
| VTY lines | 0–15 |
| Timestamp | Date/time + milliseconds |
| Console synchronous logging | Enabled |
| VTY synchronous logging | Enabled |
| Logging monitor | Enabled |
| Logging buffer | 8192 bytes |
| Syslog server | `192.168.1.100` |
| Syslog port | UDP 514 |
| Interface Syslog severity | Level 5 — Notice |

---

# 🧠 Syslog Severity Levels

Cisco Syslog uses severity levels from **0 to 7**:

| Level | Name | Description |
|---:|---|---|
| **0** | Emergencies | System unusable |
| **1** | Alerts | Immediate action required |
| **2** | Critical | Critical condition |
| **3** | Errors | Error condition |
| **4** | Warnings | Warning condition |
| **5** | **Notice** | **Normal but significant condition** |
| **6** | Informational | Informational messages |
| **7** | Debugging | Debugging messages |

The interface messages in this lab contain:

```text
%LINK-5-CHANGED
%LINEPROTO-5-UPDOWN
```

Therefore, they are **severity level 5 — Notice**.

---

# ⚠️ Troubleshooting Notes

### Telnet fails with "Connection timed out"

If PC1 displays:

```text
% Connection timed out; remote host not responding
```

check that G0/0 is enabled:

```cisco
R1# show ip interface brief
```

G0/0 should show:

```text
GigabitEthernet0/0    ...    up    up
```

If it is administratively down:

```cisco
R1(config)# interface g0/0
R1(config-if)# no shutdown
```

---

### Syslog messages do not appear in the Telnet session

Enable terminal monitoring:

```cisco
R1# terminal monitor
```

or configure monitor logging:

```cisco
R1(config)# logging monitor
```

Remember that **console logging and monitor logging are separate functions**.

---

### Syslog messages interrupt commands

Configure:

```cisco
R1(config)# line console 0
R1(config-line)# logging synchronous
```

and:

```cisco
R1(config)# line vty 0 15
R1(config-line)# logging synchronous
```

---

### Syslog server does not receive messages

Verify:

```cisco
R1# show logging
```

Make sure R1 shows:

```text
Logging to 192.168.1.100
```

Also verify connectivity between R1 and SRV1:

```cisco
R1# ping 192.168.1.100
```

---

# ✅ Completion Checklist

The lab is complete when:

- [x] G0/0 can be shut down and re-enabled.
- [x] Interface Syslog messages are observed.
- [x] Severity level 5 is identified as **Notice**.
- [x] Logging timestamps include milliseconds.
- [x] Enable secret `ccna` is configured.
- [x] VTY password `ccent` is configured.
- [x] Telnet is enabled on VTY lines.
- [x] Logging is displayed on VTY sessions.
- [x] Synchronous logging is configured on console lines.
- [x] Synchronous logging is configured on VTY lines.
- [x] Logging buffer is enabled.
- [x] Buffer size is configured to **8192 bytes**.
- [x] SRV1 (`192.168.1.100`) is configured as the Syslog server.
- [x] `show logging` confirms the configuration.

---

## 🏁 Final Configuration Commands

A condensed version of the major configuration commands is:

```cisco
enable
configure terminal

service timestamps log datetime msec

enable secret ccna

line vty 0 15
 password ccent
 login
 transport input telnet
 logging synchronous
 exit

line console 0
 logging synchronous
 exit

logging monitor
logging buffered 8192
logging 192.168.1.100

end
write memory
```

Verify everything with:

```cisco
show logging
show running-config
```

---

## 💡 Key Takeaways

This lab demonstrates that Cisco IOS can send logging information to multiple destinations:

- **Console** — useful when connected directly to the router.
- **VTY/Monitor** — useful for remote Telnet sessions.
- **Buffer** — stores messages locally in router memory.
- **Syslog server** — provides centralized logging for network devices.

The command:

```cisco
show logging
```

is particularly important because it allows you to verify the router's logging destinations, severity levels, timestamps, buffer size, and Syslog server configuration.
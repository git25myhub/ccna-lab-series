# Cisco IOS System Configuration Dialog Lab

## 📌 Lab Overview

In this lab, you will use the **Cisco IOS System Configuration Dialog (`setup`)** to configure a router and switch. You will configure basic device identification, passwords, management access, and IP addressing without using the Basic Management Setup option.

The final configuration will allow **SW1 to communicate with R1 using the 192.168.1.0/24 network**.

---

## 🎯 Objectives

By completing this lab, you will:

- Use the Cisco IOS `setup` command.
- Configure a router using the System Configuration Dialog.
- Configure a switch using the System Configuration Dialog.
- Configure hostnames and privileged EXEC passwords.
- Configure VTY line authentication.
- Configure an IP address on R1's GigabitEthernet interface.
- Configure a management IP address on SW1's VLAN 1 interface.
- Save configurations to NVRAM.
- Verify connectivity between R1 and SW1.

---

## 🗺️ Lab Topology

```text
             192.168.1.0/24

       G0/0                    VLAN 1
+-------------+              +-------------+
|     R1      |--------------|     SW1     |
|192.168.1.1  |              |192.168.1.2  |
+-------------+              +-------------+
```

---

## 📋 Configuration Requirements

### R1

| Setting | Value |
|---|---|
| Hostname | `R1` |
| Enable secret | `Cisco` |
| Enable password | `CCNA` |
| VTY password | `CCENT` |
| SNMP | No |
| VLAN 1 | No |
| G0/0 IP address | `192.168.1.1` |
| Subnet mask | `255.255.255.0` |

### SW1

| Setting | Value |
|---|---|
| Hostname | `SW1` |
| Enable secret | `Cisco` |
| Enable password | `CCNA` |
| VTY password | `CCENT` |
| SNMP | No |
| VLAN 1 IP address | `192.168.1.2` |
| Subnet mask | `255.255.255.0` |
| Cluster command switch | No |

> **Important:** Do **not** use Basic Management Setup. When prompted, select `no` so that the extended configuration dialog is used.

---

# 🛠️ Part 1 — Configure R1

Connect **PC1 to R1's console port** and enter privileged EXEC mode.

```text
Router> enable
Router# setup
```

When prompted:

```text
Continue with configuration dialog? [yes/no]: yes
```

Choose:

```text
Would you like to enter basic management setup? [yes/no]: no
```

Configure the following:

```text
Hostname: R1
Enable secret: Cisco
Enable password: CCNA
Virtual terminal password: CCENT
SNMP network management: No
VLAN1 interface: No
GigabitEthernet0/0: 192.168.1.1
Subnet mask: 255.255.255.0
```

Leave the other interfaces unconfigured.

When the configuration summary appears, select:

```text
Enter your selection [2]: 2
```

This saves the configuration to **NVRAM** and exits the setup dialog.

---

# 🛠️ Part 2 — Configure SW1

Connect to **SW1 through its console port** and enter:

```text
Switch> enable
Switch# setup
```

Select:

```text
Continue with configuration dialog? [yes/no]: yes
```

Do **not** use Basic Management Setup:

```text
Would you like to enter basic management setup? [yes/no]: no
```

Configure:

```text
Hostname: SW1
Enable secret: Cisco
Enable password: CCNA
Virtual terminal password: CCENT
SNMP network management: No
VLAN1 interface: Yes
IP address: 192.168.1.2
Subnet mask: 255.255.255.0
Cluster command switch: No
```

When prompted to save the configuration, select:

```text
Enter your selection [2]: 2
```

The configuration should be written to NVRAM.

---

# 🔍 Part 3 — Verify the Configurations

## Verify R1

On R1, run:

```text
show startup-config
```

Confirm that the configuration contains:

```text
hostname R1
enable secret ...
enable password CCNA

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0

line vty 0 4
 password CCENT
 login
```

You can also verify the interface status:

```text
show ip interface brief
```

Expected result:

```text
GigabitEthernet0/0    192.168.1.1    YES    manual    up    up
```

---

## Verify SW1

On SW1:

```text
show startup-config
```

Confirm:

```text
hostname SW1
enable secret ...
enable password CCNA

interface Vlan1
 ip address 192.168.1.2 255.255.255.0

line vty 0 4
 password CCENT
 login
```

Then verify the VLAN interface:

```text
show ip interface brief
```

Expected:

```text
Vlan1    192.168.1.2    YES    manual    up    up
```

---

# 🧪 Part 4 — Test Connectivity

From **SW1**, ping R1:

```text
SW1# ping 192.168.1.1
```

Expected result:

```text
Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:

!!!!
Success rate is 100 percent (5/5)
```

A first ping may occasionally show one failed packet while ARP is being resolved. For example:

```text
.!!!!
Success rate is 80 percent (4/5)
```

This can be normal in Packet Tracer. Run the ping again and confirm that connectivity is working.

---

# 🔐 Password Verification

The lab uses three different password types:

| Password | Purpose |
|---|---|
| `Cisco` | Enable secret |
| `CCNA` | Enable password |
| `CCENT` | VTY/remote-access password |

The **enable secret** takes precedence over the enable password when both are configured.

The passwords are also **case-sensitive**. Therefore:

```text
Cisco
```

is different from:

```text
cisco
```

Likewise:

```text
CCENT
```

is different from:

```text
CCent
```

---

# 🧠 Key Concepts Learned

### System Configuration Dialog

The `setup` command provides an interactive method for configuring Cisco IOS devices.

```text
Router# setup
```

Selecting:

```text
no
```

for Basic Management Setup allows the extended configuration process to configure individual interfaces.

### Enable Secret vs Enable Password

The enable secret:

```text
enable secret Cisco
```

is stored in an encrypted/hashed form and takes precedence over:

```text
enable password CCNA
```

### VTY Authentication

The VTY configuration:

```text
line vty 0 4
 password CCENT
 login
```

requires the configured password for VTY access.

### Switch Management IP

A Layer 2 switch does not normally receive its management IP address on a physical switch port. Instead, the address is configured on a switched virtual interface:

```text
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
```

---

# ✅ Completion Checklist

- [ ] R1 configured using the System Configuration Dialog.
- [ ] Basic Management Setup was **not** used.
- [ ] R1 hostname is `R1`.
- [ ] R1 enable secret is configured.
- [ ] R1 enable password is configured.
- [ ] R1 VTY password is configured.
- [ ] R1 G0/0 is `192.168.1.1/24`.
- [ ] R1 configuration saved to NVRAM.
- [ ] SW1 configured using the System Configuration Dialog.
- [ ] Basic Management Setup was **not** used.
- [ ] SW1 hostname is `SW1`.
- [ ] SW1 enable secret is configured.
- [ ] SW1 enable password is configured.
- [ ] SW1 VTY password is configured.
- [ ] SW1 VLAN 1 is `192.168.1.2/24`.
- [ ] SW1 configuration saved to NVRAM.
- [ ] `show startup-config` verifies both configurations.
- [ ] SW1 can ping R1 successfully.

---

## 🏁 Lab Completion

The lab is successfully completed when **R1 and SW1 have the required configurations saved to NVRAM and SW1 can successfully ping R1 at `192.168.1.1`.**

Expected connectivity:

```text
SW1# ping 192.168.1.1

Success rate is 100 percent (5/5)
```

**Lab Status: COMPLETE ✅**
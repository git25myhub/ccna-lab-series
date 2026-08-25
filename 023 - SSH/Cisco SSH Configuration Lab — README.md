# Cisco SSH Configuration Lab

## Lab Overview

In this lab, you will configure **SSH remote access** on a Cisco router and switch. You will configure device hostnames, management IP addresses, local authentication, RSA keys, VTY lines, and SSH version 2.

The final objective is to verify that **PC1 can remotely access both R1 and SW1 using SSH**.

---

## Topology

The lab consists of:

- **R1** — Cisco 1941 Router
- **SW1** — Cisco 2960 Switch
- **PC1** — Used to test SSH connectivity

### IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1` | `255.255.255.0` |
| SW1 | VLAN 1 | `192.168.1.2` | `255.255.255.0` |

Both devices are on the same `192.168.1.0/24` management network.

---

## Objectives

By completing this lab, you will learn how to:

- Configure Cisco device hostnames.
- Configure router and switch management IP addresses.
- Create a local username and password.
- Configure a DNS domain name.
- Generate 1024-bit RSA keys.
- Configure VTY lines for local authentication.
- Restrict remote access to SSH only.
- Configure a 5-minute VTY inactivity timeout.
- Enable SSH version 2.
- Test SSH connectivity from a PC.

---

## Requirements

Configure **both R1 and SW1** with the following:

### Local User

```text
Username: cisco
Password: CCNA
```

### Domain Name

```text
cisco.com
```

### RSA Key

```text
Modulus: 1024 bits
```

### VTY Configuration

VTY lines:

```text
0 through 15
```

Requirements:

- Use the local user database for authentication.
- Permit SSH connections only.
- Disconnect inactive sessions after 5 minutes.

### SSH

```text
SSH Version 2
```

---

## Configuration Summary

### SW1

```cisco
hostname SW1

interface vlan 1
 ip address 192.168.1.2 255.255.255.0

username cisco secret CCNA

ip domain-name cisco.com

crypto key generate rsa
```

When prompted for the modulus size:

```text
1024
```

Configure the VTY lines:

```cisco
line vty 0 15
 login local
 transport input ssh
 exec-timeout 5
```

Enable SSH version 2:

```cisco
ip ssh version 2
```

---

### R1

```cisco
hostname R1

interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

username cisco secret CCNA

ip domain-name cisco.com

crypto key generate rsa
```

When prompted for the modulus size:

```text
1024
```

Configure the VTY lines:

```cisco
line vty 0 15
 login local
 transport input ssh
 exec-timeout 5
```

Enable SSH version 2:

```cisco
ip ssh version 2
```

---

## Verification

Before testing SSH, verify the configuration on both devices.

### Verify Interfaces

On R1:

```cisco
show ip interface brief
```

Expected:

```text
GigabitEthernet0/0    192.168.1.1    up    up
```

On SW1:

```cisco
show ip interface brief
```

Expected:

```text
Vlan1    192.168.1.2    up    up
```

> **Important:** On a switch, VLAN 1 must be active and have an active port for the SVI to reach an `up/up` state.

---

### Verify SSH

Run:

```cisco
show ip ssh
```

You should see SSH version 2 enabled.

---

### Verify RSA Keys

Run:

```cisco
show crypto key mypubkey rsa
```

The output should show that a 1024-bit RSA key has been generated.

---

### Verify VTY Configuration

Run:

```cisco
show running-config | section line vty
```

The VTY configuration should include:

```cisco
line vty 0 15
 login local
 transport input ssh
 exec-timeout 5
```

---

### Verify the Local User

Run:

```cisco
show running-config | include username
```

Expected:

```text
username cisco secret ...
```

---

## SSH Testing from PC1

From PC1's command prompt, test SW1:

```text
ssh -l cisco 192.168.1.2
```

Then enter:

```text
Password: CCNA
```

Test R1:

```text
ssh -l cisco 192.168.1.1
```

Then enter:

```text
Password: CCNA
```

A successful connection should place you at the respective device's CLI.

Example:

```text
R1>
```

or:

```text
SW1>
```

Exit the SSH session with:

```text
exit
```

---

## Troubleshooting

If SSH works to R1 but **times out when connecting to SW1**, check the following:

### 1. Check SW1's VLAN 1 status

```cisco
show ip interface brief
```

Make sure VLAN 1 is:

```text
up    up
```

If VLAN 1 is administratively down:

```cisco
interface vlan 1
 no shutdown
```

Also ensure that at least one switch port belonging to VLAN 1 is active.

### 2. Test connectivity from PC1

From PC1:

```text
ping 192.168.1.1
ping 192.168.1.2
```

Both addresses should respond.

### 3. Verify SSH configuration

```cisco
show ip ssh
```

### 4. Verify VTY lines

```cisco
show running-config | section line vty
```

Make sure SSH is permitted:

```cisco
transport input ssh
```

And local authentication is configured:

```cisco
login local
```

### 5. Verify the RSA keys

```cisco
show crypto key mypubkey rsa
```

---

## Expected Result

The lab is successfully completed when:

- [x] R1 is named `R1`.
- [x] SW1 is named `SW1`.
- [x] R1 G0/0 is configured as `192.168.1.1/24`.
- [x] SW1 VLAN 1 is configured as `192.168.1.2/24`.
- [x] The `cisco` local user exists on both devices.
- [x] The password/secret is `CCNA`.
- [x] The domain name is `cisco.com`.
- [x] 1024-bit RSA keys are generated on both devices.
- [x] VTY lines 0–15 use the local user database.
- [x] VTY lines accept SSH only.
- [x] VTY sessions have a 5-minute inactivity timeout.
- [x] SSH version 2 is enabled.
- [x] PC1 can successfully SSH into R1.
- [x] PC1 can successfully SSH into SW1.

---

## Skills Practiced

This lab reinforces the following CCNA skills:

- Basic Cisco IOS configuration
- Switch SVI configuration
- Router interface configuration
- Local user authentication
- SSH configuration
- RSA key generation
- VTY line security
- Remote device management
- Basic network troubleshooting

---

## Lab Completion

**Success Criteria:** PC1 must be able to establish authenticated **SSH sessions to both R1 (`192.168.1.1`) and SW1 (`192.168.1.2`)** using the local `cisco` account.
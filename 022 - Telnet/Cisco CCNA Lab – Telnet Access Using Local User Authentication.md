# Cisco CCNA Lab – Telnet Access Using Local User Authentication

## Lab Objective

The objective of this lab is to configure and verify **remote Telnet access** to both a Cisco router and switch using the local user database for authentication.

You will configure:

- Management IP addresses
- A local username and password
- VTY lines 0–15
- Local database authentication
- Telnet-only VTY access
- Remote access testing from PC1

The lab is complete when you can successfully Telnet from **PC1 to both R1 and SW1** and authenticate using the configured local account.

---

## Network Addressing

Configure the management interfaces with the IP addresses indicated in the network diagram.

For the configuration shown in this lab:

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1` | `255.255.255.0` |
| SW1 | VLAN 1 | `192.168.1.2` | `255.255.255.0` |

Both devices are therefore reachable on the same management network:

```text
192.168.1.0/24
```

---

## 1. Configure R1's G0/0 Interface

Configure `GigabitEthernet0/0` on R1 with:

```text
IP address: 192.168.1.1
Subnet mask: 255.255.255.0
```

The interface must also be enabled.

Example configuration:

```text
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
```

Verify the interface with:

```text
show ip interface brief
```

Expected result:

```text
GigabitEthernet0/0    192.168.1.1    YES manual    up    up
```

---

## 2. Configure SW1's VLAN 1 Interface

Configure the switch management interface with:

```text
IP address: 192.168.1.2
Subnet mask: 255.255.255.0
```

Example configuration:

```text
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
```

Verify the interface:

```text
show ip interface brief
```

Expected result:

```text
Vlan1    192.168.1.2    YES manual    up    up
```

> **Important:** A switch virtual interface (SVI) becomes operational only when the VLAN exists and there is an active port in that VLAN.

---

## 3. Configure the Local User Account

Create the following local user account on **both R1 and SW1**:

```text
Username: cisco
Password: CCNA
```

Use a secret rather than a plain-text password:

```text
username cisco secret CCNA
```

Verify the configuration with:

```text
show running-config
```

The password will appear encrypted in the running configuration, for example:

```text
username cisco secret 5 <encrypted-value>
```

---

## 4. Configure VTY Lines 0–15

The VTY lines provide remote access to the Cisco devices.

Configure **VTY lines 0 through 15** on both R1 and SW1.

The requirements are:

1. Use the local user database for authentication.
2. Allow only Telnet connections.

Configuration:

```text
line vty 0 15
 login local
 transport input telnet
```

### Explanation

#### `login local`

```text
login local
```

tells the device to authenticate remote users against the locally configured username database.

Therefore, the user must enter:

```text
Username: cisco
Password: CCNA
```

#### `transport input telnet`

```text
transport input telnet
```

allows Telnet connections to the VTY lines.

It prevents other remote-access protocols from being accepted on those lines.

---

## 5. Verify the VTY Configuration

On both devices, use:

```text
show running-config
```

Look for:

```text
line vty 0 4
 login local
 transport input telnet
line vty 5 15
 login local
 transport input telnet
```

On some Cisco IOS versions, `line vty 0 15` may be entered as one configuration block, while the resulting configuration is displayed as separate VTY ranges.

The important requirement is that **VTY lines 0–15** use:

```text
login local
transport input telnet
```

---

## 6. Test Connectivity Before Telnet

Before testing Telnet, verify that PC1 can reach both management IP addresses.

From PC1:

```text
ping 192.168.1.1
ping 192.168.1.2
```

Both devices should respond.

If the ping fails, troubleshoot IP addressing and Layer 2 connectivity before attempting Telnet.

Useful commands include:

```text
show ip interface brief
show vlan brief
show interfaces
```

---

## 7. Telnet from PC1 to SW1

From PC1's command prompt:

```text
telnet 192.168.1.2
```

You should receive:

```text
User Access Verification

Username:
```

Enter:

```text
Username: cisco
Password: CCNA
```

A successful login should provide access to the SW1 CLI:

```text
SW1>
```

The supplied lab output confirms successful authentication:

```text
Trying 192.168.1.2 ...Open

User Access Verification

Username: cisco
Password:

SW1>
```

---

## 8. Telnet from PC1 to R1

From PC1:

```text
telnet 192.168.1.1
```

Enter the configured credentials:

```text
Username: cisco
Password: CCNA
```

A successful connection should provide:

```text
R1>
```

The supplied lab output confirms that R1 was successfully accessed through Telnet:

```text
Trying 192.168.1.1 ...Open

User Access Verification

Username: cisco
Password:

R1>
```

---

## Troubleshooting

### Problem: SW1 cannot be reached

Check:

```text
show ip interface brief
```

Make sure VLAN 1 has:

```text
192.168.1.2/24
```

and is operational.

Also verify that the physical switchport connecting SW1 to the network is active.

---

### Problem: R1 cannot be reached

Check:

```text
show ip interface brief
```

Verify:

```text
GigabitEthernet0/0    192.168.1.1    up    up
```

If the interface is administratively down:

```text
interface gigabitEthernet0/0
 no shutdown
```

---

### Problem: Telnet asks for a password but username authentication does not work

Check the local database:

```text
show running-config | include username
```

You should see an entry similar to:

```text
username cisco secret 5 <encrypted-value>
```

Then verify the VTY lines:

```text
show running-config | section line vty
```

Make sure they contain:

```text
login local
transport input telnet
```

---

### Problem: Telnet connection is refused

Verify that Telnet is permitted on the VTY lines:

```text
show running-config | section line vty
```

The required configuration is:

```text
transport input telnet
```

If the configuration contains:

```text
transport input ssh
```

Telnet connections will not be accepted.

---

## Useful Verification Commands

### Check interface addressing

```text
show ip interface brief
```

### Check VLAN status on SW1

```text
show vlan brief
```

### Check local users

```text
show running-config | include username
```

### Check VTY configuration

```text
show running-config | section line vty
```

### Check the complete configuration

```text
show running-config
```

### Test IP connectivity

```text
ping 192.168.1.1
ping 192.168.1.2
```

### Test Telnet

```text
telnet 192.168.1.1
telnet 192.168.1.2
```

---

## Expected Final Configuration

### R1

```text
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

username cisco secret CCNA

line vty 0 15
 login local
 transport input telnet
```

### SW1

```text
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
 no shutdown

username cisco secret CCNA

line vty 0 15
 login local
 transport input telnet
```

---

## Verification Checklist

- [ ] R1 G0/0 is configured with `192.168.1.1/24`.
- [ ] R1 G0/0 is enabled.
- [ ] SW1 VLAN 1 is configured with `192.168.1.2/24`.
- [ ] SW1 VLAN 1 is operational.
- [ ] User `cisco` exists on R1.
- [ ] User `cisco` exists on SW1.
- [ ] The password/secret is `CCNA`.
- [ ] VTY lines 0–15 use `login local`.
- [ ] VTY lines 0–15 allow Telnet.
- [ ] PC1 can ping R1.
- [ ] PC1 can ping SW1.
- [ ] PC1 can Telnet to R1.
- [ ] PC1 can Telnet to SW1.
- [ ] The `cisco` credentials successfully authenticate on both devices.
- [ ] The configurations are saved with `write memory` or `copy running-config startup-config`.

---

## Completion Criteria

The lab is successfully completed when PC1 can establish authenticated Telnet sessions to **both R1 and SW1**:

```text
PC1 → Telnet → R1 (192.168.1.1)
PC1 → Telnet → SW1 (192.168.1.2)
```

using:

```text
Username: cisco
Password: CCNA
```

and both devices provide access to their respective CLI prompts.
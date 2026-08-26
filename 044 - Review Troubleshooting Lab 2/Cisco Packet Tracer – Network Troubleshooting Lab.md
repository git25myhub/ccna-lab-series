# Cisco Packet Tracer – Network Troubleshooting Lab

## Lab Overview

This lab focuses on troubleshooting and resolving multiple network configuration issues across a Cisco router-based network.

The objective is to identify the root cause of each problem, apply the appropriate configuration fix, verify the correction, and confirm end-to-end network connectivity.

The problems should be troubleshot **in the specified order**, because some issues depend on services or connectivity being restored earlier in the troubleshooting process.

---

## Troubleshooting Objectives

Troubleshoot and fix the following problems:

1. **R2 and R3 are not receiving the RIP route to `192.168.1.0/24` from R1.**
2. **Hosts in the `192.168.2.0/24` network are not receiving IP addresses through DHCP.**
3. **PAT is not functioning on R1.**
4. **Hosts in the `192.168.1.0/24` network are not receiving DNS server information through DHCP.**
5. **R1 cannot be accessed successfully using SSH.**

---

## Network Information

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1/24` | LAN / NAT Inside |
| R1 | G0/1 | `1.2.3.1/24` | WAN / NAT Outside |
| R2 | G0/0 | `192.168.2.1/24` | LAN / DHCP Relay |
| R2 | G0/1 | `1.2.3.2/24` | Transit |
| R3 | G0/0 | `30.0.0.1/24` | Server Network |
| R3 | G0/1 | `1.2.3.3/24` | Transit |
| Server | NIC | `30.0.0.100/24` | DNS / Syslog Server |

### DHCP Networks

**R1 DHCP Pool – 192.168.1.0/24**

- Network: `192.168.1.0/24`
- Default Gateway: `192.168.1.1`
- Excluded Addresses: `192.168.1.1 – 192.168.1.10`
- DNS Server: `30.0.0.100`

**R1 DHCP Pool – 192.168.2.0/24**

- Network: `192.168.2.0/24`
- Default Gateway: `192.168.2.1`
- Excluded Addresses: `192.168.2.1 – 192.168.2.10`
- DNS Server: `30.0.0.100`

---

# 1. Troubleshoot RIP Routing

## Problem

R2 and R3 are not initially receiving the `192.168.1.0/24` route from R1.

### Initial Verification

On R2:

```cisco
R2#show ip rip database
```

The expected route was initially missing:

```text
192.168.1.0/24
```

After allowing RIP to advertise the correct network from R1, R2 received:

```text
192.168.1.0/24
    [1] via 1.2.3.1
```

R3 subsequently learned the same route:

```text
192.168.1.0/24
    [1] via 1.2.3.1
```

### R1 RIP Configuration

R1 uses RIP version 2:

```cisco
router rip
 version 2
 network 1.0.0.0
 network 192.168.1.0
 no auto-summary
```

### Verification Commands

```cisco
R1#show ip protocols
R2#show ip rip database
R3#show ip rip database
R2#show ip route rip
R3#show ip route rip
```

### Expected Result

R2 and R3 should have a RIP-learned route to:

```text
192.168.1.0/24
```

---

# 2. Troubleshoot DHCP on the 192.168.2.0/24 Network

## Problem

Hosts connected to the `192.168.2.0/24` network were unable to obtain an IP address from DHCP.

The affected PC initially displayed:

```text
Autoconfiguration IPv4 Address: 169.254.x.x
```

and:

```text
DHCP request failed.
```

### Root Cause

The DHCP server is located on R1, while the clients are connected to R2.

DHCP broadcasts do not normally cross a router interface. Therefore, R2 must relay DHCP requests to R1.

### Fix

Configure a DHCP relay on R2's LAN interface:

```cisco
R2(config)#interface GigabitEthernet0/0
R2(config-if)#ip helper-address 1.2.3.1
```

Save the configuration:

```cisco
R2(config)#do write memory
```

### Verification

Check the interface:

```cisco
R2#show ip interface GigabitEthernet0/0
```

Verify that the helper address is present.

Then renew the PC's DHCP lease:

```text
C:\>ipconfig /release
C:\>ipconfig /renew
```

### Expected Result

The client should receive an address such as:

```text
IPv4 Address:       192.168.2.11
Subnet Mask:        255.255.255.0
Default Gateway:    192.168.2.1
DNS Server:         30.0.0.100
```

---

# 3. Troubleshoot PAT on R1

## Problem

PAT was not creating translations for hosts in the `192.168.1.0/24` network.

### Initial Verification

Check NAT translations:

```cisco
R1#show ip nat translations
```

Initially:

```text
Total translations: 0
```

Check NAT statistics:

```cisco
R1#show ip nat statistics
```

The output showed:

```text
Total translations: 0
Hits: 0
Misses: 72
```

### Root Cause

The PAT configuration referenced access list **2**:

```cisco
ip nat inside source list 2 interface GigabitEthernet0/1 overload
```

However, the configured access list was **ACL 1**:

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
```

Therefore, the NAT rule was referencing the wrong ACL.

### Fix

Remove the incorrect NAT statement:

```cisco
R1(config)#no ip nat inside source list 2 interface GigabitEthernet0/1 overload
```

Configure PAT using ACL 1:

```cisco
R1(config)#ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

### NAT Interface Configuration

R1's inside interface:

```cisco
interface GigabitEthernet0/0
 ip nat inside
```

R1's outside interface:

```cisco
interface GigabitEthernet0/1
 ip nat outside
```

### Verification

Generate traffic from the inside network, for example:

```text
C:\>ping 30.0.0.100
```

Then check:

```cisco
R1#show ip nat translations
```

A successful translation should resemble:

```text
Pro  Inside global     Inside local
icmp 1.2.3.1:5         192.168.1.13:5
icmp 1.2.3.1:6         192.168.1.13:6
```

This confirms that PAT is translating the private inside address to R1's outside interface address.

---

# 4. Troubleshoot DNS Information in DHCP

## Problem

Hosts in the `192.168.1.0/24` network were receiving an IP address and default gateway but were not receiving DNS server information.

### Initial Client Verification

The client showed:

```text
IPv4 Address:       192.168.1.13
Default Gateway:    192.168.1.1
DNS Servers:        0.0.0.0
```

This indicated that DHCP was working, but the DHCP pool was missing the DNS server option.

### Root Cause

The `1pool` DHCP pool contained:

```cisco
ip dhcp pool 1pool
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
```

but did not contain a `dns-server` statement.

### Fix

Enter the DHCP pool:

```cisco
R1(config)#ip dhcp pool 1pool
```

Configure the DNS server:

```cisco
R1(dhcp-config)#dns-server 30.0.0.100
```

Exit and save:

```cisco
R1(dhcp-config)#exit
R1(config)#do write memory
```

### Verify DHCP Configuration

```cisco
R1#show running-config | section dhcp
```

The corrected pool should contain:

```cisco
ip dhcp pool 1pool
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 30.0.0.100
```

### Renew the Client Address

On the PC:

```text
C:\>ipconfig /release
C:\>ipconfig /renew
```

Verify:

```text
C:\>ipconfig /all
```

Expected:

```text
IPv4 Address:       192.168.1.13
Subnet Mask:        255.255.255.0
Default Gateway:    192.168.1.1
DNS Server:         30.0.0.100
```

### DNS Verification

Test the server by hostname:

```text
C:\>ping srv1
```

Expected result:

```text
Pinging 30.0.0.100 with 32 bytes of data:

Reply from 30.0.0.100
Reply from 30.0.0.100
Reply from 30.0.0.100
Reply from 30.0.0.100
```

---

# 5. Troubleshoot SSH Access to R1

## Problem

R1 could not initially be accessed successfully using SSH.

### Initial Configuration

R1 already had:

```cisco
username cisco password 0 ccna
ip ssh version 2
ip domain-name cisco.com
```

However, the VTY lines were configured for Telnet:

```cisco
line vty 0 4
 login local
 transport input telnet

line vty 5 15
 login local
 transport input telnet
```

### Root Cause

SSH was enabled globally, but the VTY lines were only accepting Telnet connections.

Therefore, an SSH connection was not properly permitted.

### Fix

Configure all VTY lines to accept SSH:

```cisco
R1(config)#line vty 0 15
R1(config-line)#transport input ssh
```

Save the configuration:

```cisco
R1(config-line)#do write memory
```

### SSH Configuration Requirements

R1 should have:

```cisco
username cisco password 0 ccna
ip domain-name cisco.com
ip ssh version 2
```

The VTY lines should contain:

```cisco
line vty 0 15
 login local
 transport input ssh
```

### Test SSH

From a PC:

```text
C:\>ssh -l cisco 192.168.1.1
```

Enter the password:

```text
ccna
```

Successful authentication should provide access to the R1 CLI:

```text
R1>
```

---

# Final R1 Configuration

After completing the troubleshooting, the relevant portions of R1 should resemble:

```cisco
!
hostname R1
!
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.2.1 192.168.2.10
!
ip dhcp pool 1pool
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 30.0.0.100
!
ip dhcp pool 2pool
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 30.0.0.100
!
username cisco password 0 ccna
!
ip ssh version 2
ip domain-name cisco.com
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
!
interface GigabitEthernet0/1
 ip address 1.2.3.1 255.255.255.0
 ip nat outside
!
router rip
 version 2
 network 1.0.0.0
 network 192.168.1.0
 no auto-summary
!
ip nat inside source list 1 interface GigabitEthernet0/1 overload
!
access-list 1 permit 192.168.1.0 0.0.0.255
!
line vty 0 15
 login local
 transport input ssh
!
```

R2 should contain the DHCP relay:

```cisco
interface GigabitEthernet0/0
 ip address 192.168.2.1 255.255.255.0
 ip helper-address 1.2.3.1
```

---

# Verification Checklist

Use the following commands to verify each repaired service.

### RIP

```cisco
R2#show ip route rip
R3#show ip route rip
```

Confirm:

```text
R 192.168.1.0/24
```

is present.

### DHCP

On R1:

```cisco
R1#show ip dhcp binding
R1#show ip dhcp pool
```

On R2:

```cisco
R2#show ip interface GigabitEthernet0/0
```

On the client:

```text
C:\>ipconfig /all
```

### PAT

On R1:

```cisco
R1#show ip nat translations
R1#show ip nat statistics
```

Generate traffic before checking translations.

### DNS

On the client:

```text
C:\>ipconfig /all
C:\>ping srv1
```

The DNS server should be:

```text
30.0.0.100
```

### SSH

From the client:

```text
C:\>ssh -l cisco 192.168.1.1
```

Verify that the session successfully reaches:

```text
R1>
```

---

# Key Troubleshooting Lessons

## 1. Always verify the actual configuration

Do not assume a service is correctly configured simply because a related command exists.

Useful commands include:

```cisco
show running-config
show ip protocols
show ip route
show ip interface
```

## 2. Check dependencies

DHCP on a remote LAN requires a DHCP relay.

In this lab:

```text
PC → R2 → R1 DHCP Server
```

Therefore R2 requires:

```cisco
ip helper-address 1.2.3.1
```

## 3. Match ACLs with NAT statements

The NAT rule must reference an ACL that actually exists and permits the intended inside addresses.

Correct combination:

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
```

and:

```cisco
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

## 4. DHCP can provide more than IP addresses

A DHCP pool can provide:

- IP address
- Subnet mask
- Default gateway
- DNS server
- Domain information
- Other DHCP options

## 5. SSH requires more than `ip ssh version 2`

SSH access also requires appropriate VTY configuration.

The important combination is:

```cisco
username cisco password 0 ccna
ip domain-name cisco.com
ip ssh version 2
```

and:

```cisco
line vty 0 15
 login local
 transport input ssh
```

---

# Final Validation

The lab is considered successfully completed when all of the following are true:

- [x] R2 learns `192.168.1.0/24` through RIP.
- [x] R3 learns `192.168.1.0/24` through RIP.
- [x] Hosts in `192.168.2.0/24` successfully obtain DHCP addresses.
- [x] PAT creates translations for `192.168.1.0/24` hosts.
- [x] Hosts in `192.168.1.0/24` receive DNS server `30.0.0.100`.
- [x] Clients can resolve/test `srv1`.
- [x] R1 accepts SSH connections.
- [x] The configuration is saved using `write memory` / `copy running-config startup-config`.

## Useful General Troubleshooting Commands

```cisco
show running-config
show ip interface brief
show ip route
show ip protocols
show ip rip database
show ip dhcp binding
show ip dhcp pool
show ip nat translations
show ip nat statistics
show access-lists
show ip ssh
show users
```

This lab demonstrates a practical troubleshooting workflow: **identify the symptom → inspect the configuration → determine the root cause → apply the smallest appropriate fix → verify the result → save the configuration.**
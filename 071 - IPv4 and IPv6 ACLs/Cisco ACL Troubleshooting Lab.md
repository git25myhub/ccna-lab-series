# Cisco ACL Troubleshooting Lab

## 📌 Lab Overview

This lab focuses on configuring and applying **IPv4 and IPv6 Access Control Lists (ACLs)** to control traffic between specific hosts and networks.

The objective is to troubleshoot the network, identify where ACLs need to be applied, configure the required access-control policies, and verify that permitted traffic continues to function normally.

The lab includes:

- Standard numbered IPv4 ACL
- Extended named IPv4 ACL
- IPv6 ACL
- IPv6 VTY access control for Telnet
- ACL placement and direction
- ACL verification and connectivity testing

---

## 🎯 Lab Objectives

Configure and apply the following access-control requirements:

| Requirement | ACL Type | Expected Result |
|---|---|---|
| PC4 cannot access `10.4.4.0/24` | Standard numbered IPv4 ACL | PC4 is denied access to the network |
| PC5 cannot access `2001:DB8:22:22::/64` | IPv6 ACL | PC5 is denied access to the IPv6 network |
| PC3 cannot communicate with PC1 | Extended named IPv4 ACL | PC3 cannot communicate with PC1 |
| Only PC6 can Telnet to R2 using IPv6 | IPv6 VTY ACL | PC6 is permitted; other IPv6 devices are denied |

The Telnet password for R2 is:

```text
ccna
```

---

# 🧠 ACL Concepts Used

## Standard Numbered IPv4 ACL

A standard ACL filters traffic based primarily on the **source IPv4 address**.

Example:

```cisco
access-list 1 deny host 10.2.2.12
access-list 1 permit any
```

The ACL must then be applied to an interface:

```cisco
ip access-group 1 out
```

Because standard ACLs only examine the source address, placement is important.

---

## Extended Named IPv4 ACL

Extended ACLs provide much more granular control because they can filter based on:

- Source IP
- Destination IP
- Protocol
- TCP/UDP ports

Example:

```cisco
ip access-list extended G0/1_IN
 deny ip host 10.2.2.11 host 10.1.1.11
 permit ip any any
```

Applied with:

```cisco
interface g0/1
 ip access-group G0/1_IN in
```

---

## IPv6 ACL

IPv6 ACLs are configured using:

```cisco
ipv6 access-list <NAME>
```

For example:

```cisco
ipv6 access-list G0/2_IN
 deny ipv6 host 2001:db8:3:3::11 2001:db8:22:22::/64
 permit ipv6 any any
```

The ACL is applied using:

```cisco
ipv6 traffic-filter G0/2_IN in
```

---

# 🔧 Configuration and Troubleshooting

## 1. Standard Numbered ACL — Block PC4

### Requirement

PC4 must not be able to access:

```text
10.4.4.0/24
```

The standard ACL was configured on R2.

### Configuration

```cisco
R2(config)# access-list 1 deny host 10.2.2.12
R2(config)# access-list 1 permit any
```

The ACL was applied outbound on the appropriate interface:

```cisco
R2(config)# interface g0/0
R2(config-if)# ip access-group 1 out
```

### Verification

From PC4:

```text
C:\> ping 10.4.4.100
```

Expected result:

```text
Reply from 10.12.12.2: Destination host unreachable.
```

This confirms that PC4 is unable to reach the `10.4.4.0/24` network.

Traffic from other permitted hosts should continue to work.

---

# 2. IPv6 ACL — Block PC5

### Requirement

PC5 must not access:

```text
2001:DB8:22:22::/64
```

An IPv6 ACL was created on R1.

### Configuration

```cisco
R1(config)# ipv6 access-list G0/2_IN
R1(config-ipv6-acl)# deny ipv6 host 2001:db8:3:3::11 2001:db8:22:22::/64
R1(config-ipv6-acl)# permit ipv6 any any
```

The ACL was applied inbound on the appropriate interface:

```cisco
R1(config)# interface g0/2
R1(config-if)# ipv6 traffic-filter G0/2_IN in
```

### Verification

From PC5:

```text
C:\> ping 2001:db8:22:22::100
```

Expected result:

```text
Reply from 2001:DB8:3:3::1: Destination host unreachable.
```

This confirms that PC5 is being denied access to the `2001:DB8:22:22::/64` network.

Other IPv6 traffic should remain permitted.

---

# 3. Extended Named ACL — Block PC3 From PC1

### Requirement

PC3 must not communicate with PC1.

The important addresses are:

```text
PC3: 10.2.2.11
PC1: 10.1.1.11
```

Because the requirement specifies both a **source and destination**, an extended ACL is appropriate.

### Configuration

On R1:

```cisco
R1(config)# ip access-list extended G0/1_IN
R1(config-ext-nacl)# deny ip host 10.2.2.11 host 10.1.1.11
R1(config-ext-nacl)# permit ip any any
```

Apply the ACL inbound:

```cisco
R1(config)# interface g0/1
R1(config-if)# ip access-group G0/1_IN in
```

### Verification

From PC3:

```text
C:\> ping 10.1.1.11
```

Expected result:

```text
Reply from 10.2.2.1: Destination host unreachable.
```

PC3 should therefore be unable to communicate with PC1.

The `permit ip any any` statement is important because it allows all other IPv4 traffic that is not specifically denied.

---

# 4. IPv6 ACL — Allow Only PC6 to Telnet to R2

### Requirement

PC6 must be able to Telnet to R2 using IPv6.

All other IPv6 devices must be prevented from accessing R2 through Telnet.

PC6's IPv6 address:

```text
2001:DB8:3:3::12
```

R2's IPv6 address used for Telnet:

```text
2001:DB8:12:12::2
```

Telnet uses:

```text
TCP port 23
```

---

## Configure the IPv6 ACL

On R2:

```cisco
R2(config)# ipv6 access-list TELNET
R2(config-ipv6-acl)# permit tcp host 2001:db8:3:3::12 any eq telnet
```

This permits TCP/Telnet traffic originating specifically from PC6.

---

## Apply the ACL to the VTY Lines

```cisco
R2(config)# line vty 0 15
R2(config-line)# ipv6 access-class TELNET in
```

The IPv6 ACL is therefore applied directly to incoming IPv6 VTY connections.

---

## Configure the Telnet Password

The required password is:

```text
ccna
```

The VTY configuration should include authentication using the configured password:

```cisco
R2(config)# line vty 0 15
R2(config-line)# password ccna
R2(config-line)# login
```

---

# 🧪 Verification

## Test Telnet From an Unauthorized IPv6 Device

Attempt to connect to R2:

```text
C:\> telnet 2001:db8:12:12::2
```

The connection should be refused or otherwise fail because the source IPv6 address is not permitted by the `TELNET` ACL.

Example:

```text
% Connection refused by remote host
```

---

## Test Telnet From PC6

From PC6:

```text
C:\> telnet 2001:db8:12:12::2
```

Expected result:

```text
Trying 2001:DB8:12:12::2 ...Open

User Access Verification

Password:
```

Enter:

```text
ccna
```

Successful authentication should provide access to R2:

```text
R2#
```

This confirms that PC6 is permitted to Telnet to R2 while unauthorized IPv6 devices are blocked.

---

# 🔍 Useful Verification Commands

After configuring the ACLs, use the following commands to verify the configuration.

### IPv4 ACLs

```cisco
show access-lists
```

or:

```cisco
show ip access-lists
```

Check where an ACL is applied:

```cisco
show ip interface
```

Look for:

```text
Inbound access list
Outbound access list
```

---

### IPv6 ACLs

```cisco
show ipv6 access-list
```

Check IPv6 interfaces:

```cisco
show ipv6 interface
```

Look for the configured IPv6 traffic filter.

---

### VTY Configuration

```cisco
show running-config | section line vty
```

Verify that the IPv6 access-class is applied:

```text
ipv6 access-class TELNET in
```

---

### Routing Verification

Because ACL troubleshooting can sometimes be confused with routing problems, verify the routing table:

```cisco
show ip route
```

and:

```cisco
show ipv6 route
```

Also verify EIGRP neighbors if required:

```cisco
show ip eigrp neighbors
show ipv6 eigrp neighbors
```

---

# ⚠️ Important ACL Rules

## 1. ACLs Have an Implicit Deny

Every ACL has an implicit deny at the end.

For example:

```cisco
access-list 1 deny host 10.2.2.12
```

effectively means:

```text
Deny PC4
Deny everything else
```

Therefore, when the intention is to block only one source and permit all other traffic, add:

```cisco
permit any
```

Similarly, IPv6 ACLs commonly require:

```cisco
permit ipv6 any any
```

after a specific deny statement.

---

## 2. ACL Order Matters

ACL statements are processed from **top to bottom**.

The first matching statement is used.

For example:

```cisco
deny ip host 10.2.2.11 host 10.1.1.11
permit ip any any
```

works correctly because traffic from PC3 to PC1 matches the deny statement first.

---

## 3. ACL Direction Matters

An ACL must be applied in the correct direction.

IPv4:

```cisco
ip access-group <ACL> in
ip access-group <ACL> out
```

IPv6:

```cisco
ipv6 traffic-filter <ACL> in
ipv6 traffic-filter <ACL> out
```

A correctly written ACL applied in the wrong direction may appear not to work.

---

# 📋 Final ACL Summary

| Device | ACL | Type | Applied To | Direction |
|---|---|---|---|---|
| R2 | `1` | Standard IPv4 | `G0/0` | Out |
| R1 | `G0/2_IN` | IPv6 | `G0/2` | In |
| R1 | `G0/1_IN` | Extended Named IPv4 | `G0/1` | In |
| R2 | `TELNET` | IPv6 VTY ACL | `line vty 0 15` | In |

---

# ✅ Completion Criteria

The lab is successfully completed when all of the following are true:

- [x] PC4 cannot access `10.4.4.0/24`.
- [x] Other permitted IPv4 traffic continues to work.
- [x] PC5 cannot access `2001:DB8:22:22::/64`.
- [x] Other permitted IPv6 traffic continues to work.
- [x] PC3 cannot communicate with PC1.
- [x] Other IPv4 communication remains functional.
- [x] PC6 can Telnet to R2 using IPv6.
- [x] The Telnet password is `ccna`.
- [x] Other IPv6 devices cannot Telnet to R2.
- [x] ACLs are applied to the correct interfaces or VTY lines.
- [x] ACL configurations survive a reload.

---

# 💾 Save the Configuration

After completing the lab, save the running configuration:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

Expected output:

```text
Building configuration...
[OK]
```

---

# 🏁 Conclusion

This lab demonstrates how Cisco ACLs can be used to enforce specific traffic policies without disrupting the rest of the network.

The key troubleshooting skills practiced include:

- Identifying the correct source and destination addresses
- Selecting between standard and extended ACLs
- Configuring IPv6 ACLs
- Applying ACLs in the correct direction
- Controlling VTY access with IPv6 ACLs
- Understanding implicit denies
- Verifying ACL behavior using connectivity tests and show commands

The main principle is to **match the ACL type and placement to the traffic that needs to be controlled**, while explicitly permitting legitimate traffic that should remain unaffected.
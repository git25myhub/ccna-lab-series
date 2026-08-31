# Cisco Packet Tracer Lab – ACL Troubleshooting

## 📌 Lab Overview

This lab focuses on troubleshooting and correcting **Access Control List (ACL)** misconfigurations in a dual-stack IPv4/IPv6 network.

Several ACLs have already been configured to enforce specific security policies. However, the network is not behaving as intended because **three ACLs contain configuration errors**.

The objective is to identify the misconfigured ACLs, determine what is preventing the intended traffic from being permitted or denied, and correct the configurations without unnecessarily modifying working ACLs.

---

# 🎯 Lab Objectives

By completing this lab, you will learn how to:

- Troubleshoot **standard numbered IPv4 ACLs**.
- Troubleshoot **extended named IPv4 ACLs**.
- Troubleshoot **IPv6 ACLs**.
- Understand how ACL placement affects traffic.
- Identify incorrect source/destination addresses and wildcard masks.
- Verify ACL behavior using `show` commands.
- Test ACLs using `ping` and `telnet`.
- Correct ACL configuration errors while preserving the intended security policy.

---

# 🧩 ACL Requirements

The network has four ACL requirements.

| ACL Type | Required Behavior |
|---|---|
| Standard Numbered ACL | PC4 must not access `10.4.4.0/24` |
| IPv6 ACL | PC5 must not access `2001:DB8:22:22::/64` |
| Extended Named ACL | PC3 must not communicate with PC1 |
| IPv6 ACL | PC6 can Telnet to R2; other IPv6 devices cannot |

The Telnet password configured for R2 is:

```text
ccna
```

However, **three of the ACLs are misconfigured** and must be corrected.

---

# 🗺️ Lab Requirements

The final network must enforce the following security policies.

## 1. PC4 → 10.4.4.0/24

PC4 must be prevented from accessing:

```text
10.4.4.0/24
```

Other traffic should continue to operate normally.

### Expected behavior

```text
PC4 ─────X────► 10.4.4.0/24
```

The connection should be denied by the standard numbered ACL.

---

# 2. PC5 → 2001:DB8:22:22::/64

PC5 must not be able to access the IPv6 network:

```text
2001:DB8:22:22::/64
```

### Expected behavior

```text
PC5 ─────X────► 2001:DB8:22:22::/64
```

The IPv6 ACL should block traffic from PC5 toward this destination network.

---

# 3. PC3 → PC1

PC3 must not be able to communicate with PC1.

The network uses an **Extended Named ACL** to enforce this restriction.

### Expected behavior

```text
PC3 ─────X────► PC1
```

The extended ACL should specifically identify the traffic between PC3 and PC1.

Because this is an extended ACL, it can match traffic based on parameters such as:

- Source address
- Destination address
- Protocol
- Source port
- Destination port

---

# 4. IPv6 Telnet Access to R2

The final IPv6 ACL controls Telnet access to R2.

The intended policy is:

```text
PC6 ─────────► R2
       Telnet
```

PC6 **must be permitted** to Telnet to R2.

All other IPv6 devices must be prevented from Telnetting to R2.

The Telnet password is:

```text
ccna
```

### Expected behavior

```text
PC6 ─────────► R2
       Telnet       ✓ ALLOWED
```

Other IPv6 devices:

```text
PC5 ─────X────► R2
PC7 ─────X────► R2
       Telnet       ✗ DENIED
```

---

# 🔎 Troubleshooting Strategy

The key to this lab is to avoid immediately changing every ACL.

Instead, troubleshoot systematically.

Start by testing each requirement.

---

# Step 1 – Test the Standard ACL

From **PC4**, attempt to reach a device in:

```text
10.4.4.0/24
```

For example:

```text
C:\> ping <destination-IP>
```

If the ping succeeds when it should fail, investigate the standard ACL.

---

# Step 2 – Inspect IPv4 ACLs

On the relevant router, display the configured ACLs:

```cisco
show access-lists
```

You can also use:

```cisco
show ip access-lists
```

Look for the standard numbered ACL.

A standard ACL generally looks like:

```cisco
access-list <number> deny <source>
access-list <number> permit any
```

Pay particular attention to:

- Incorrect source IP address
- Incorrect wildcard mask
- Incorrect ACL number
- Incorrect permit/deny statements
- Missing `permit any`

---

# Step 3 – Check Where the ACL Is Applied

An ACL can be configured correctly but still fail to provide the intended behavior if it is applied to the wrong interface or direction.

Check the interfaces:

```cisco
show running-config
```

Look for:

```cisco
ip access-group <ACL-number> in
```

or:

```cisco
ip access-group <ACL-number> out
```

Determine whether the ACL is applied in the correct direction.

---

# Step 4 – Test the IPv6 ACL

From **PC5**, attempt to ping or otherwise communicate with a host inside:

```text
2001:DB8:22:22::/64
```

The traffic should be blocked.

If PC5 can successfully communicate with the destination network, inspect the IPv6 ACL.

Use:

```cisco
show ipv6 access-list
```

Check for:

- Incorrect IPv6 source address
- Incorrect destination prefix
- Incorrect prefix length
- Incorrect permit/deny action
- Incorrect protocol
- Incorrect ACL application

---

# Step 5 – Verify IPv6 ACL Application

IPv6 ACLs are applied directly to interfaces.

Inspect the router configuration:

```cisco
show running-config
```

Look for:

```cisco
ipv6 traffic-filter <ACL-NAME> in
```

or:

```cisco
ipv6 traffic-filter <ACL-NAME> out
```

Confirm that the ACL is applied to the correct interface and in the correct direction.

---

# Step 6 – Troubleshoot the Extended Named ACL

Test communication between PC3 and PC1.

From PC3:

```text
C:\> ping <PC1-IP>
```

The expected result is that PC3 cannot communicate with PC1.

If communication succeeds, inspect the named ACL:

```cisco
show ip access-lists
```

or:

```cisco
show access-lists
```

Find the extended named ACL.

An extended ACL may contain entries similar to:

```cisco
ip access-list extended <ACL-NAME>
 deny ip <PC3-network> <wildcard> host <PC1-IP>
 permit ip any any
```

Check carefully that the ACL identifies the correct:

```text
Source = PC3
Destination = PC1
```

A common troubleshooting mistake is reversing the source and destination addresses.

---

# Step 7 – Verify the IPv6 Telnet Restriction

The final requirement is slightly different.

**PC6 must be allowed to Telnet to R2**, while all other IPv6 devices must be denied.

From PC6, test:

```text
telnet <R2-IPv6-address>
```

When prompted for the password, enter:

```text
ccna
```

PC6 should successfully establish the Telnet session.

Then test Telnet from another IPv6 device.

The connection should fail.

---

# 🔐 Understanding the Telnet ACL

The ACL controlling Telnet access should effectively implement:

```text
Permit PC6 → R2 TCP/23
Deny other IPv6 devices → R2 TCP/23
```

The ACL may use the IPv6 equivalent of TCP port 23:

```text
tcp
```

and:

```text
eq telnet
```

or:

```text
eq 23
```

The exact syntax depends on the configuration already provided in the lab.

---

# ⚠️ Pay Attention to ACL Order

ACL entries are processed from **top to bottom**.

The first matching statement determines the result.

For example:

```cisco
permit ipv6 host <PC6> host <R2>
deny ipv6 any host <R2>
```

is different from:

```cisco
deny ipv6 any host <R2>
permit ipv6 host <PC6> host <R2>
```

In the second example, the traffic from PC6 may be denied before the permit statement is ever reached.

Therefore, when troubleshooting ACLs, always inspect the **order of ACEs (Access Control Entries)**.

---

# 🧠 Important ACL Concepts

## Standard IPv4 ACL

A standard ACL primarily examines the:

```text
Source IPv4 address
```

Example:

```cisco
access-list 10 deny host 192.168.4.10
access-list 10 permit any
```

Standard ACLs are generally less specific than extended ACLs.

---

## Extended IPv4 ACL

Extended ACLs can examine:

- Source IP
- Destination IP
- Protocol
- Source port
- Destination port

Example:

```cisco
ip access-list extended BLOCK-PC3
 deny ip host <PC3-IP> host <PC1-IP>
 permit ip any any
```

---

## IPv6 ACL

IPv6 ACLs are configured using named access lists:

```cisco
ipv6 access-list <NAME>
```

They are applied to interfaces using:

```cisco
ipv6 traffic-filter <NAME> in
```

or:

```cisco
ipv6 traffic-filter <NAME> out
```

---

# 🔍 Useful Verification Commands

### Display IPv4 ACLs

```cisco
show access-lists
```

```cisco
show ip access-lists
```

### Display IPv6 ACLs

```cisco
show ipv6 access-list
```

### Check the interface configuration

```cisco
show running-config
```

### Check interface status

```cisco
show ip interface brief
```

For IPv6:

```cisco
show ipv6 interface brief
```

### Check a specific interface

```cisco
show ip interface <interface>
```

This can show whether an IPv4 ACL is applied.

For IPv6:

```cisco
show ipv6 interface <interface>
```

---

# 📊 Verification Table

| Test | Expected Result |
|---|---|
| PC4 → `10.4.4.0/24` | ❌ Denied |
| PC5 → `2001:DB8:22:22::/64` | ❌ Denied |
| PC3 → PC1 | ❌ Denied |
| PC6 → R2 Telnet | ✅ Allowed |
| Other IPv6 device → R2 Telnet | ❌ Denied |
| Unrelated permitted traffic | ✅ Allowed |

---

# 🛠️ Troubleshooting Checklist

When an ACL does not behave correctly, check the following in order:

### 1. Identify the traffic

Determine:

```text
Source
Destination
Protocol
Port
Address family
```

### 2. Inspect the ACL

```cisco
show access-lists
```

or:

```cisco
show ipv6 access-list
```

### 3. Check the ACEs

Look for:

- Wrong IP address
- Wrong wildcard mask
- Wrong IPv6 prefix
- Wrong protocol
- Wrong port
- Wrong permit/deny action
- Incorrect ACE order

### 4. Check ACL placement

Determine:

```text
Which interface?
Inbound or outbound?
```

### 5. Test again

Use:

```text
ping
telnet
```

to verify the result.

### 6. Check ACL hit counters

The ACL output may show match counters.

These counters are useful for determining whether traffic is actually reaching a particular ACL entry.

---

# 🏁 Completion Criteria

The lab is successfully completed when:

- [x] PC4 cannot access `10.4.4.0/24`.
- [x] PC5 cannot access `2001:DB8:22:22::/64`.
- [x] PC3 cannot communicate with PC1.
- [x] PC6 can Telnet to R2.
- [x] The Telnet password is `ccna`.
- [x] Other IPv6 devices cannot Telnet to R2.
- [x] The three ACL misconfigurations have been identified and corrected.
- [x] Legitimate traffic continues to work normally.

---

# 📝 Key Takeaways

This lab reinforces several important ACL troubleshooting principles:

1. **An ACL can be syntactically correct but logically wrong.**
2. **ACL direction matters** — inbound and outbound filtering produce different results.
3. **ACL placement matters** — an ACL must be applied where it can actually inspect the desired traffic.
4. **ACLs are processed top-to-bottom.**
5. **The first matching ACE wins.**
6. **Always consider the implicit deny at the end of an ACL.**
7. **Standard ACLs primarily identify traffic by source address.**
8. **Extended ACLs provide much more granular control.**
9. **IPv6 ACLs use `ipv6 access-list` and `ipv6 traffic-filter`.**
10. **Testing with both `ping` and `telnet` helps confirm whether the intended security policy is actually working.**

---

# 🏆 Final Objective

The goal is not simply to make the network reachable.

The goal is to make the network behave **exactly according to the defined security policy**:

```text
PC4 ───X───► 10.4.4.0/24

PC5 ───X───► 2001:DB8:22:22::/64

PC3 ───X───► PC1

PC6 ─────────► R2
               Telnet ✓

Other IPv6 ──X──► R2
               Telnet ✗
```

After correcting the three ACL misconfigurations, verify every requirement with connectivity tests before considering the lab complete.
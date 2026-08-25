# Named Standard ACLs – Network Access Restriction Lab

## Objective

Configure **named standard Access Control Lists (ACLs)** on R1 and R2 to restrict communication between specific networks.

The completed configuration must satisfy these requirements:

- Hosts in the **192.168.1.0/24** network cannot communicate with hosts in the **192.168.2.0/24** network.
- Hosts in the **192.168.2.0/24** network cannot access the **192.168.3.0/24** network.
- Other traffic should remain permitted.

## Network Requirements

| Network | Purpose |
|---|---|
| `192.168.1.0/24` | Network 1 |
| `192.168.2.0/24` | Network 2 |
| `192.168.3.0/24` | Network 3 |
| `12.0.0.0` | R1–R2 WAN/serial connection |

Example addresses observed during testing include:

- `192.168.1.11`
- `192.168.1.12`
- `192.168.2.13`
- `192.168.2.14`
- `192.168.3.100`
- `192.168.3.101`

## Lab Requirements

### Requirement 1 — Block 192.168.1.0/24 ↔ 192.168.2.0/24

Hosts on:

```text
192.168.1.0/24
```

must not be able to communicate with hosts on:

```text
192.168.2.0/24
```

This restriction must work in **both directions**.

Therefore:

- `192.168.1.0/24 → 192.168.2.0/24` = **DENY**
- `192.168.2.0/24 → 192.168.1.0/24` = **DENY**

### Requirement 2 — Block 192.168.2.0/24 → 192.168.3.0/24

Hosts on:

```text
192.168.2.0/24
```

must not be able to access:

```text
192.168.3.0/24
```

Therefore:

```text
192.168.2.0/24 → 192.168.3.0/24 = DENY
```

## Important Concept: Standard ACL Limitation

A **standard ACL** filters traffic based only on the **source IP address**.

It cannot directly match a destination network.

For example:

```text
deny 192.168.2.0 0.0.0.255
```

means:

> Deny traffic whose source address belongs to `192.168.2.0/24`.

Because of this limitation, standard ACLs should normally be placed **close to the destination**.

This is why the ACLs in this lab are applied outbound on the interfaces leading toward the protected networks.

---

# R1 Configuration

## 1. Create the `2to1` Named Standard ACL

This ACL prevents hosts from the `192.168.2.0/24` network from reaching the `192.168.1.0/24` network.

```text
R1# configure terminal

R1(config)# ip access-list standard 2to1
R1(config-std-nacl)# deny 192.168.2.0 0.0.0.255
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit
```

### Explanation

```text
deny 192.168.2.0 0.0.0.255
```

blocks traffic originating from the entire `192.168.2.0/24` network.

```text
permit any
```

allows all other source addresses.

Without the `permit any`, the ACL would have an implicit deny at the end and could block additional traffic.

## 2. Create the `1to2` Named Standard ACL

This ACL prevents hosts from the `192.168.1.0/24` network from reaching the `192.168.2.0/24` network.

```text
R1(config)# ip access-list standard 1to2
R1(config-std-nacl)# deny 192.168.1.0 0.0.0.255
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit
```

## 3. Apply `2to1` to R1

The ACL should be applied **outbound** on the interface connected toward the `192.168.1.0/24` network.

```text
R1(config)# interface fastethernet0/0
R1(config-if)# ip access-group 2to1 out
```

## 4. Apply `1to2` to R1

The ACL should be applied **outbound** on the interface connected toward the `192.168.2.0/24` network.

```text
R1(config)# interface fastethernet1/0
R1(config-if)# ip access-group 1to2 out
```

Save the configuration:

```text
R1(config-if)# end
R1# write memory
```

---

# R2 Configuration

## 5. Create the `2to3` Named Standard ACL

This ACL prevents the `192.168.2.0/24` network from accessing the `192.168.3.0/24` network.

```text
R2# configure terminal

R2(config)# ip access-list standard 2to3
R2(config-std-nacl)# deny 192.168.2.0 0.0.0.255
R2(config-std-nacl)# permit any
R2(config-std-nacl)# exit
```

## 6. Apply `2to3` to R2

Apply the ACL **outbound** on the interface connected toward the `192.168.3.0/24` network.

```text
R2(config)# interface fastethernet0/0
R2(config-if)# ip access-group 2to3 out
```

Save the configuration:

```text
R2(config-if)# end
R2# write memory
```

---

# Complete Configuration Summary

### R1

```text
R1(config)# ip access-list standard 2to1
R1(config-std-nacl)# deny 192.168.2.0 0.0.0.255
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit

R1(config)# ip access-list standard 1to2
R1(config-std-nacl)# deny 192.168.1.0 0.0.0.255
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit

R1(config)# interface fastethernet0/0
R1(config-if)# ip access-group 2to1 out
R1(config-if)# exit

R1(config)# interface fastethernet1/0
R1(config-if)# ip access-group 1to2 out
R1(config-if)# end

R1# write memory
```

### R2

```text
R2(config)# ip access-list standard 2to3
R2(config-std-nacl)# deny 192.168.2.0 0.0.0.255
R2(config-std-nacl)# permit any
R2(config-std-nacl)# exit

R2(config)# interface fastethernet0/0
R2(config-if)# ip access-group 2to3 out
R2(config-if)# end

R2# write memory
```

---

# Verification

## Check ACLs on R1

```text
R1# show access-lists
```

or:

```text
R1# show ip access-lists
```

You should see:

```text
Standard IP access list 2to1
    deny 192.168.2.0 0.0.0.255
    permit any

Standard IP access list 1to2
    deny 192.168.1.0 0.0.0.255
    permit any
```

## Check ACL on R2

```text
R2# show ip access-lists
```

Expected:

```text
Standard IP access list 2to3
    deny 192.168.2.0 0.0.0.255
    permit any
```

## Check Interface ACL Assignments

On R1:

```text
R1# show ip interface fastethernet0/0
R1# show ip interface fastethernet1/0
```

On R2:

```text
R2# show ip interface fastethernet0/0
```

Look for the lines indicating that an IP access list is applied in the **outbound** direction.

You can also use:

```text
R1# show running-config
R2# show running-config
```

---

# Connectivity Testing

## Test 1 — 192.168.1.0/24 → 192.168.2.0/24

From a host on `192.168.1.0/24`, test a host such as:

```text
C:\> ping 192.168.2.13
```

Expected:

```text
Request timed out.
```

The communication should be **blocked**.

Test another host:

```text
C:\> ping 192.168.2.14
```

Expected:

```text
Request timed out.
```

## Test 2 — 192.168.2.0/24 → 192.168.1.0/24

From a host on `192.168.2.0/24`:

```text
C:\> ping 192.168.1.11
```

Expected:

```text
Destination host unreachable
```

or another failed-ping response.

Test another host:

```text
C:\> ping 192.168.1.12
```

Expected:

```text
Destination host unreachable
```

The communication should be **blocked**.

## Test 3 — 192.168.2.0/24 → 192.168.3.0/24

From a host on `192.168.2.0/24`:

```text
C:\> ping 192.168.3.100
```

Expected:

```text
Request timed out.
```

Test another host:

```text
C:\> ping 192.168.3.101
```

Expected:

```text
Request timed out.
```

The communication should be **blocked**.

---

# ACL Logic

The final traffic policy can be represented as:

```text
192.168.1.0/24
       |
       | X  DENY
       v
192.168.2.0/24
       |
       | X  DENY
       v
192.168.3.0/24
```

And in the reverse direction:

```text
192.168.2.0/24
       |
       | X  DENY
       v
192.168.1.0/24
```

The `permit any` statement ensures that traffic from other source networks is not unnecessarily blocked.

---

# Understanding Named Standard ACLs

Instead of using a numbered ACL such as:

```text
access-list 1 deny 192.168.1.0 0.0.0.255
```

a named ACL is created with:

```text
ip access-list standard 1to2
```

This provides a meaningful name that makes the configuration easier to understand and troubleshoot.

The ACL can then be modified from ACL configuration mode:

```text
R1(config)# ip access-list standard 1to2
R1(config-std-nacl)#
```

This is especially useful in larger networks where many ACLs are configured.

---

# Important Troubleshooting Note

The ACL must be applied in the **correct direction**.

For this lab:

```text
R1 Fa0/0 → 2to1 → OUT
R1 Fa1/0 → 1to2 → OUT
R2 Fa0/0 → 2to3 → OUT
```

The reason is that standard ACLs match the **source** of the traffic. By placing the ACL close to the destination, the source-based filtering affects the intended destination network without unnecessarily blocking the same source from reaching other networks.

Also remember that ACLs are processed **top-to-bottom** and stop at the first matching statement.

---

# Completion Checklist

- [ ] Created named standard ACL `2to1` on R1.
- [ ] Created named standard ACL `1to2` on R1.
- [ ] Created named standard ACL `2to3` on R2.
- [ ] Blocked `192.168.2.0/24 → 192.168.1.0/24`.
- [ ] Blocked `192.168.1.0/24 → 192.168.2.0/24`.
- [ ] Blocked `192.168.2.0/24 → 192.168.3.0/24`.
- [ ] Added `permit any` to preserve other traffic.
- [ ] Applied the ACLs in the correct outbound direction.
- [ ] Verified the ACLs using `show ip access-lists`.
- [ ] Tested connectivity from all relevant networks.
- [ ] Saved the configurations using `write memory`.

## Key Takeaway

This lab demonstrates how **named standard ACLs** can control network access using the **source IP address**. Because standard ACLs cannot identify the destination, their placement is particularly important. Applying them close to the destination allows source-based restrictions to be implemented without unnecessarily affecting other network traffic.
# Cisco Standard ACL Configuration Lab

## Objective

The purpose of this lab is to configure **standard IPv4 Access Control Lists (ACLs)** to control traffic between different networks.

The required security policies are:

1. **Only computers in the `192.168.1.0/24` network can access SRV1.**
2. **PC4 must not be able to communicate with the `192.168.1.0/24` network.**
3. Other traffic should remain permitted unless explicitly denied.

This lab demonstrates how standard ACLs filter traffic based on the **source IP address**.

---

# Standard ACL Overview

A **standard ACL** filters traffic based only on the **source IPv4 address**.

Standard numbered ACLs use the range:

```text
1–99
```

The lab uses ACL number:

```text
1
```

A standard ACL can contain statements such as:

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 deny host 192.168.2.14
access-list 1 permit any
```

The ACL statements are processed **from top to bottom**.

Once a packet matches an entry, processing stops.

---

# Important ACL Concept: Implicit Deny

Every ACL has an implicit:

```text
deny any
```

at the end.

For example:

```cisco
access-list 1
 permit 192.168.1.0 0.0.0.255
```

effectively means:

```text
permit 192.168.1.0/24
deny everything else
```

This behavior is important when creating an ACL that should allow only a particular source network.

---

# Requirement 1

## Only `192.168.1.0/24` Can Access SRV1

The first requirement is:

> Only computers in the `192.168.1.0/24` network can access SRV1.

The ACL should therefore:

- Permit traffic sourced from `192.168.1.0/24`.
- Implicitly deny traffic from all other networks.

---

# Step 1: Create the Standard ACL on R2

Enter global configuration mode:

```cisco
R2#configure terminal
```

Create standard ACL 1:

```cisco
R2(config)#access-list 1 permit 192.168.1.0 0.0.0.255
```

Save the configuration:

```cisco
R2(config)#do write
```

Verify the ACL:

```cisco
R2(config)#do show access-lists
```

Expected output:

```text
Standard IP access list 1
    10 permit 192.168.1.0 0.0.0.255
```

---

# Step 2: Apply ACL 1 to the Correct Interface

The ACL must be applied to the interface through which traffic travels **toward SRV1**.

In the provided configuration, ACL 1 was applied outbound on `FastEthernet0/0`:

```cisco
R2(config)#interface FastEthernet0/0
R2(config-if)#ip access-group 1 out
```

Save the configuration:

```cisco
R2(config-if)#do write
```

### Why outbound?

The ACL is positioned where traffic is leaving R2 toward SRV1.

Because the ACL is standard, it examines the **source address** of each packet.

A packet sourced from:

```text
192.168.1.0/24
```

is permitted.

Packets sourced from other networks do not match the permit statement and are therefore rejected by the implicit deny.

---

# Requirement 2

## PC4 Cannot Communicate With `192.168.1.0/24`

The second requirement is:

> PC4 must not be able to communicate with the `192.168.1.0/24` network.

From the provided configuration, PC4 has the IP address:

```text
192.168.2.14
```

Therefore, the ACL must deny traffic sourced from:

```text
192.168.2.14
```

while allowing other traffic.

---

# Step 3: Create the ACL on R1

Enter global configuration mode:

```cisco
R1#configure terminal
```

Create the deny statement:

```cisco
R1(config)#access-list 1 deny host 192.168.2.14
```

Then permit all other traffic:

```cisco
R1(config)#access-list 1 permit any
```

Save:

```cisco
R1(config)#do write
```

Verify:

```cisco
R1(config)#do show access-lists
```

Expected output:

```text
Standard IP access list 1
    10 deny host 192.168.2.14
    20 permit any
```

---

# Step 4: Apply the ACL to R1

Apply ACL 1 outbound on the interface toward the `192.168.1.0/24` network:

```cisco
R1(config)#interface FastEthernet0/0
R1(config-if)#ip access-group 1 out
```

Save the configuration:

```cisco
R1(config-if)#do write
```

The resulting ACL behavior is:

```text
Source: 192.168.2.14
        ↓
       R1
        ↓
192.168.1.0/24
        ↓
      DENIED
```

Traffic from other sources is permitted by:

```cisco
access-list 1 permit any
```

---

# Understanding the Wildcard Mask

The first ACL uses:

```text
192.168.1.0 0.0.0.255
```

The wildcard mask:

```text
0.0.0.255
```

matches every host address in the `192.168.1.0/24` network.

Therefore:

```text
192.168.1.1
192.168.1.10
192.168.1.11
192.168.1.100
192.168.1.254
```

can all match the ACL entry.

The second ACL uses:

```cisco
deny host 192.168.2.14
```

which matches exactly one source address.

---

# Verification

## Verify ACL Configuration

On R1:

```cisco
show access-lists
```

Expected:

```text
Standard IP access list 1
    10 deny host 192.168.2.14
    20 permit any
```

On R2:

```cisco
show access-lists
```

Expected:

```text
Standard IP access list 1
    10 permit 192.168.1.0 0.0.0.255
```

---

# Verify ACL Placement

Use:

```cisco
show ip interface FastEthernet0/0
```

Look for an entry similar to:

```text
Outgoing access list is 1
```

This confirms that ACL 1 is applied in the outbound direction.

---

# Verify Connectivity

## Test 1: PC in `192.168.1.0/24` → SRV1

A computer in the `192.168.1.0/24` network should be able to access SRV1.

Example:

```text
C:\>ping 192.168.3.100
```

Expected behavior:

```text
Reply from 192.168.3.100
```

The first packet may time out due to ARP resolution, but subsequent packets should succeed.

The provided test showed:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

This indicates that connectivity was ultimately successful.

---

# Test 2: Unauthorized Network → SRV1

A computer outside the `192.168.1.0/24` network should not be able to access SRV1.

Because ACL 1 contains only:

```cisco
permit 192.168.1.0 0.0.0.255
```

all other source addresses are rejected by the implicit deny.

Expected result:

```text
Request timed out.
```

---

# Test 3: PC4 → `192.168.1.0/24`

PC4 has:

```text
192.168.2.14
```

Attempt:

```text
C:\>ping 192.168.1.11
```

Expected result:

```text
Destination host unreachable.
```

The provided test showed:

```text
Reply from 192.168.2.1: Destination host unreachable.
```

with:

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

This confirms that PC4 is unable to reach the `192.168.1.0/24` network.

---

# ACL Processing Order

The R1 ACL is:

```cisco
access-list 1 deny host 192.168.2.14
access-list 1 permit any
```

Traffic is processed as follows:

| Source | Match | Result |
|---|---|---|
| `192.168.2.14` | `deny host 192.168.2.14` | Denied |
| `192.168.2.15` | `permit any` | Permitted |
| `10.0.0.10` | `permit any` | Permitted |
| Any other source | `permit any` | Permitted |

The order is important.

If `permit any` were placed before the deny statement:

```cisco
access-list 1 permit any
access-list 1 deny host 192.168.2.14
```

PC4 would be permitted because it would match the first entry and the deny statement would never be reached.

---

# Useful Verification Commands

### Display all ACLs

```cisco
show access-lists
```

### Display a specific ACL

```cisco
show access-lists 1
```

### Display interface ACL information

```cisco
show ip interface FastEthernet0/0
```

### Display the running configuration

```cisco
show running-config
```

### Test connectivity

```cisco
ping <destination-ip>
```

### Check the routing table

```cisco
show ip route
```

---

# ACL Counters

You can also use:

```cisco
show access-lists
```

to see how many packets have matched each ACL statement.

For example:

```text
Standard IP access list 1
    10 deny host 192.168.2.14 (4 matches)
    20 permit any (12 matches)
```

The match counters are useful for confirming that the ACL is actually processing traffic.

---

# Common Mistakes

## Mistake 1: Applying the ACL in the wrong direction

Remember:

```cisco
ip access-group 1 out
```

filters traffic leaving the interface.

```cisco
ip access-group 1 in
```

filters traffic entering the interface.

The direction must be selected based on where the traffic should be filtered.

---

## Mistake 2: Forgetting the implicit deny

This ACL:

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
```

does **not** permit everyone else.

It effectively means:

```text
Permit 192.168.1.0/24
Deny everything else
```

---

## Mistake 3: Using the wrong wildcard mask

For a `/24` network:

```text
Network mask:    255.255.255.0
Wildcard mask:   0.0.0.255
```

Therefore:

```cisco
permit 192.168.1.0 0.0.0.255
```

is correct.

---

## Mistake 4: Putting `permit any` before the deny

Incorrect:

```cisco
access-list 1 permit any
access-list 1 deny host 192.168.2.14
```

Correct:

```cisco
access-list 1 deny host 192.168.2.14
access-list 1 permit any
```

ACL entries are evaluated from top to bottom.

---

# Final Configuration

## R2 — Restrict Access to SRV1

```cisco
R2#configure terminal
R2(config)#access-list 1 permit 192.168.1.0 0.0.0.255
R2(config)#interface FastEthernet0/0
R2(config-if)#ip access-group 1 out
R2(config-if)#end
R2#write
```

This permits only traffic sourced from `192.168.1.0/24` toward SRV1.

---

## R1 — Block PC4 From `192.168.1.0/24`

```cisco
R1#configure terminal
R1(config)#access-list 1 deny host 192.168.2.14
R1(config)#access-list 1 permit any
R1(config)#interface FastEthernet0/0
R1(config-if)#ip access-group 1 out
R1(config-if)#end
R1#write
```

This blocks PC4 while allowing other sources to communicate with the `192.168.1.0/24` network.

---

# Final Verification Checklist

- [x] Standard ACL created on R2.
- [x] `192.168.1.0/24` permitted to access SRV1.
- [x] ACL applied to the SRV1-facing interface.
- [x] Standard ACL created on R1.
- [x] PC4 (`192.168.2.14`) explicitly denied.
- [x] Other traffic permitted with `permit any`.
- [x] ACL applied in the correct direction.
- [x] ACL configuration saved.
- [x] PC in `192.168.1.0/24` can reach SRV1.
- [x] PC4 cannot reach `192.168.1.0/24`.

---

# Key Takeaways

1. **Standard ACLs filter traffic based on the source IP address.**
2. Standard numbered ACLs use numbers from **1–99**.
3. ACL entries are processed from **top to bottom**.
4. Every ACL has an implicit **`deny any`** at the end.
5. The wildcard mask `0.0.0.255` represents a `/24` network.
6. `host 192.168.2.14` matches only PC4.
7. `permit any` allows all traffic that has not already been denied.
8. The ACL must be applied to the correct interface and in the correct direction.
9. `show access-lists` is useful for verifying ACL entries and match counters.
10. `show ip interface` can confirm which ACL is applied to an interface.

## Lab Completion

The lab is successfully completed when **only the `192.168.1.0/24` network can access SRV1**, while **PC4 (192.168.2.14) is unable to communicate with the `192.168.1.0/24` network**.
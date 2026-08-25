# Floating Static Route with RIP

## Lab Overview

In this Cisco Packet Tracer lab, **RIP is already configured on the router interfaces**, except for the direct connection between **R1 and R3**.

R1 currently learns the `10.0.0.0/24` network dynamically through RIP via **R2**. The objective is to configure a **floating static route** on R1 that provides a backup path to the same network through **R3**.

The floating static route must have a higher administrative distance than RIP so that RIP remains the preferred route during normal operation.

The lab is successfully completed when:

- R1 uses RIP through R2 when the R1–R2 connection is operational.
- R1 automatically switches to the floating static route through R3 when the R1–R2 connection fails.
- R1 returns to using RIP when the R1–R2 connection is restored.

---

## Objectives

By completing this lab, you will learn how to:

- Interpret a RIP-learned route.
- Understand administrative distance.
- Configure a floating static route.
- Use a static route as a backup to a dynamic routing protocol.
- Verify route selection using `show ip route`.
- Use `traceroute` to identify the active forwarding path.
- Test routing failover by shutting down an interface.
- Restore the primary path and verify that RIP becomes preferred again.

---

# Network Scenario

The relevant topology contains three routers:

```text
             RIP
        R1 -------- R2
         \          \
          \          \
           \          R3
            \        /
             --------
```

The important networks are:

| Network | Purpose |
|---|---|
| `10.0.0.0/24` | Remote destination network |
| `192.168.12.0/24` | R1–R2 connection |
| `192.168.13.0/24` | R1–R3 connection |
| `192.168.23.0/24` | R2–R3 connection |

The exact interface/IP assignments are already configured.

---

# Understanding the Initial Routing Table

Before configuring the backup route, R1 has the following relevant route:

```text
R 10.0.0.0 [120/2] via 192.168.12.2, Serial3/0
```

The important information is:

```text
R
```

The `R` indicates that the route was learned through **RIP**.

The administrative distance is:

```text
120
```

The RIP metric shown is:

```text
2
```

Therefore, R1 currently reaches `10.0.0.0/24` through:

```text
R1 → R2 → R3 → 10.0.0.0/24
```

---

# Why a Floating Static Route Is Needed

The R1–R2 path is the primary path to `10.0.0.0/24`.

However, R1 also has a direct connection to R3 through:

```text
192.168.13.0/24
```

R3 can therefore provide an alternative path.

We want R1 to use:

```text
R1 → R2
```

under normal conditions, but switch to:

```text
R1 → R3
```

if the R1–R2 connection fails.

A normal static route has an administrative distance of **1**, while RIP has an administrative distance of **120**.

If we configured:

```text
ip route 10.0.0.0 255.255.255.0 192.168.13.3
```

the static route would be preferred over RIP because:

```text
1 < 120
```

That would make the static route the **primary route**, which is not what this lab requires.

Instead, we configure the static route with an administrative distance greater than RIP:

```text
121
```

Therefore:

```text
RIP = 120
Floating static route = 121
```

Since `120 < 121`, RIP is preferred while it is available.

---

# Configure the Floating Static Route

On R1, enter global configuration mode:

```text
R1> enable
R1# configure terminal
```

Configure the floating static route:

```text
R1(config)# ip route 10.0.0.0 255.255.255.0 192.168.13.3 121
```

The command consists of:

| Component | Meaning |
|---|---|
| `ip route` | Configure a static route |
| `10.0.0.0` | Destination network |
| `255.255.255.0` | Destination subnet mask |
| `192.168.13.3` | R3's next-hop address |
| `121` | Administrative distance |

Save the configuration:

```text
R1(config)# do write
```

---

# Verify the Primary Route

Use:

```text
R1# show ip route
```

Under normal conditions, R1 should continue to show the RIP route:

```text
R 10.0.0.0 [120/2] via 192.168.12.2, Serial3/0
```

The floating static route is configured, but it does **not** appear as the active route because RIP has a better administrative distance.

This demonstrates the key concept:

```text
RIP AD = 120
Static AD = 121

120 < 121
```

Therefore, RIP wins.

---

# Verify the Current Path

Use `traceroute` from R1:

```text
R1# traceroute 10.0.0.11
```

The expected path is:

```text
1   192.168.12.2
2   192.168.23.3
3   10.0.0.11
```

This confirms that R1 is currently forwarding traffic through R2.

The normal path is:

```text
R1 → R2 → R3 → 10.0.0.0/24
```

---

# Test the Backup Route

To simulate failure of the primary R1–R2 connection, shut down R1's `Serial3/0` interface.

Enter:

```text
R1# configure terminal
R1(config)# interface serial3/0
R1(config-if)# shutdown
```

The interface should transition to an administratively down state.

You should see messages similar to:

```text
%LINK-5-CHANGED: Interface Serial3/0, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial3/0, changed state to down
```

---

# Verify Route Failover

Now check the routing table:

```text
R1(config-if)# do show ip route
```

The RIP route through R2 should no longer be available.

Instead, R1 should install the floating static route:

```text
S 10.0.0.0 [121/0] via 192.168.13.3
```

The route is now active because:

```text
RIP route → unavailable
Floating static route → available
```

R1 therefore forwards traffic directly to R3.

---

# Verify the Backup Path

Run:

```text
R1(config-if)# do traceroute 10.0.0.11
```

The expected path should be:

```text
1   192.168.13.3
2   10.0.0.11
```

This confirms that the floating static route is working.

The backup path is:

```text
R1 → R3 → 10.0.0.0/24
```

---

# Restore the Primary Path

After testing failover, bring R1's `Serial3/0` interface back up:

```text
R1(config-if)# no shutdown
```

The interface should return to the up state:

```text
%LINK-5-CHANGED: Interface Serial3/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial3/0, changed state to up
```

Save the configuration:

```text
R1(config-if)# do write
```

---

# Verify RIP Has Returned

Check the routing table:

```text
R1(config-if)# do show ip route
```

R1 should once again prefer the RIP route:

```text
R 10.0.0.0 [120/2] via 192.168.12.2, Serial3/0
```

The floating static route is no longer the active route because RIP has returned.

Run:

```text
R1(config-if)# do traceroute 10.0.0.11
```

The path should again be:

```text
1   192.168.12.2
2   192.168.23.3
3   10.0.0.11
```

---

# Route Selection Summary

| Condition | Active Route | AD | Path |
|---|---|---:|---|
| R1–R2 link is up | RIP | 120 | R1 → R2 → R3 |
| R1–R2 link fails | Floating static | 121 | R1 → R3 |
| R1–R2 link restored | RIP | 120 | R1 → R2 → R3 |

The key principle is:

```text
Lower Administrative Distance = Preferred Route
```

Therefore:

```text
RIP:             120
Floating Static: 121
```

RIP is preferred whenever it is available.

---

# Verification Commands

### Display the routing table

```text
show ip route
```

### Display only RIP routes

```text
show ip route rip
```

### Display static routes

```text
show ip route static
```

### Check interface status

```text
show ip interface brief
```

### Test connectivity

```text
ping 10.0.0.11
```

### Identify the forwarding path

```text
traceroute 10.0.0.11
```

### Display the configured static route

```text
show running-config
```

---

# Troubleshooting

## Floating static route is not installed

Check that the command includes the correct administrative distance:

```text
ip route 10.0.0.0 255.255.255.0 192.168.13.3 121
```

Without `121`, the route has the default static administrative distance of `1` and will become the preferred route over RIP.

---

## RIP is still being used after shutting down Serial3/0

Check:

```text
show ip route
```

Make sure the floating static route points to:

```text
192.168.13.3
```

Also verify the R1–R3 interface:

```text
show ip interface brief
```

`Serial2/0` must be operational.

---

## R1 cannot reach 10.0.0.0/24 through R3

Verify connectivity to R3:

```text
ping 192.168.13.3
```

Then check R3's routing table:

```text
R3# show ip route
```

R3 must have a route to the `10.0.0.0/24` network.

---

## Incorrect command syntax

A command such as:

```text
do sh pi route
```

will generate an error because `pi` is not a valid abbreviation in this context.

Use:

```text
do show ip route
```

or:

```text
show ip route
```

---

# Important Cisco Commands

| Command | Purpose |
|---|---|
| `ip route` | Configure a static route |
| `ip route ... 121` | Configure a floating static route |
| `show ip route` | Display the routing table |
| `show ip route rip` | Display RIP-learned routes |
| `show ip route static` | Display static routes |
| `show ip interface brief` | Check interface status |
| `shutdown` | Administratively disable an interface |
| `no shutdown` | Re-enable an interface |
| `ping` | Test IP connectivity |
| `traceroute` | Display the path to a destination |
| `do write` | Save the configuration |

---

# Completion Checklist

- [ ] Confirm RIP is already configured.
- [ ] Confirm R1 learns `10.0.0.0/24` through R2 using RIP.
- [ ] Configure the floating static route on R1.
- [ ] Use administrative distance `121`.
- [ ] Verify RIP remains the preferred route.
- [ ] Verify the normal path through R2 using `traceroute`.
- [ ] Shut down R1 `Serial3/0`.
- [ ] Verify the floating static route becomes active.
- [ ] Verify the backup path through R3 using `traceroute`.
- [ ] Restore R1 `Serial3/0`.
- [ ] Verify RIP becomes the preferred route again.
- [ ] Save the final configuration.

---

# Final Configuration

The essential configuration on R1 is:

```text
R1(config)# ip route 10.0.0.0 255.255.255.0 192.168.13.3 121
```

This creates a backup route to `10.0.0.0/24` through R3.

The result is a simple form of routing redundancy:

```text
                 Primary
        R1 ---------------- R2
         \                   \
          \                   \
           \                   R3
            \                /
             ----------------
                 Backup
```

**Primary path:** R1 → R2 → R3

**Backup path:** R1 → R3

The floating static route remains inactive during normal operation and automatically becomes active when the RIP-learned route is no longer available.
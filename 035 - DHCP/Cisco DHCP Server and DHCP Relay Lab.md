# Cisco DHCP Server and DHCP Relay Lab

## Lab Overview

In this lab, you will configure **R1 as a DHCP server** and create three DHCP pools for different IPv4 networks.

You will then configure **R2 as a DHCP client** on its G0/0 interface so that it dynamically obtains an address from R1. Finally, R2 will be configured as a **DHCP relay agent** on G0/1, allowing hosts on the `20.0.0.0/24` network to obtain DHCP addresses from the DHCP server on R1.

This lab demonstrates an important real-world DHCP design: **DHCP clients and the DHCP server do not need to be on the same Layer 2 network when a router is configured as a DHCP relay agent.**

---

## Objectives

By completing this lab, you will:

1. Configure three DHCP pools on R1.
2. Configure DHCP exclusions for the required networks.
3. Configure R2 G0/0 as a DHCP client.
4. Verify that R2 dynamically receives an IP address from R1.
5. Configure R2 G0/1 as a DHCP relay agent.
6. Configure DHCP clients on the `20.0.0.0/24` network.
7. Verify successful DHCP address assignment.
8. Verify DHCP operation using Cisco IOS commands.

---

## DHCP Addressing Requirements

### DHCP Pool Summary

| Pool | Network | Default Gateway | DNS Server | Excluded Addresses |
|---|---|---|---|---|
| `10pool` | `10.0.0.0/24` | `10.0.0.1` | `10.0.0.1` | `10.0.0.1 – 10.0.0.10` |
| `20pool` | `20.0.0.0/24` | `20.0.0.1` | `20.0.0.1` | `20.0.0.1 – 20.0.0.10` |
| `12pool` | `192.168.12.0/24` | Not specified | Not specified | Not specified |

### Important Addressing Concept

The `12pool` is used to provide addresses from:

```text
192.168.12.0/24
```

R2's G0/0 interface will use DHCP to obtain an address from this pool.

In the completed lab, R2 received:

```text
192.168.12.2/24
```

This makes R1 reachable by R2 at:

```text
192.168.12.1
```

---

# Task 1 — Configure DHCP Pools on R1

## Configure the 10pool

On R1:

```cisco
enable
configure terminal

ip dhcp excluded-address 10.0.0.1 10.0.0.10

ip dhcp pool 10pool
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1
 dns-server 10.0.0.1
 exit
```

The DHCP server will assign addresses from the `10.0.0.0/24` network while reserving:

```text
10.0.0.1 – 10.0.0.10
```

The first dynamically available address is therefore:

```text
10.0.0.11
```

### Verification

A client successfully received:

```text
IP Address:       10.0.0.11
Subnet Mask:      255.255.255.0
Default Gateway:  10.0.0.1
DNS Server:       10.0.0.1
```

This confirms that the DHCP pool is working correctly.

---

# Task 2 — Configure the 20pool

Configure the second DHCP pool:

```cisco
ip dhcp excluded-address 20.0.0.1 20.0.0.10

ip dhcp pool 20pool
 network 20.0.0.0 255.255.255.0
 default-router 20.0.0.1
 dns-server 20.0.0.1
 exit
```

The reserved addresses are:

```text
20.0.0.1 – 20.0.0.10
```

Therefore, DHCP clients can begin receiving addresses from:

```text
20.0.0.11
```

In the completed lab, the client eventually received:

```text
IP Address:       20.0.0.12
Subnet Mask:      255.255.255.0
Default Gateway:  20.0.0.1
DNS Server:       20.0.0.1
```

---

# Task 3 — Configure the 12pool

Create the third DHCP pool:

```cisco
ip dhcp pool 12pool
 network 192.168.12.0 255.255.255.0
 exit
```

This pool provides addresses from:

```text
192.168.12.0/24
```

The purpose of this pool is to provide an address to **R2 G0/0**, which will act as the DHCP client.

---

# Task 4 — Configure R2 G0/0 as a DHCP Client

On R2, configure G0/0 to obtain its IPv4 address dynamically:

```cisco
enable
configure terminal

interface gigabitEthernet 0/0
 ip address dhcp
 no shutdown
 exit
```

R2 should send a DHCP request through G0/0.

Verify the assigned address:

```cisco
show ip interface brief
```

The expected result is similar to:

```text
Interface              IP-Address      OK? Method Status
GigabitEthernet0/0     192.168.12.2    YES DHCP   up
```

The lab output confirms that R2 received:

```text
192.168.12.2/24
```

from R1.

### Verify the DHCP Client

You can also use:

```cisco
show dhcp lease
```

This provides information about the DHCP lease obtained by R2.

---

# Task 5 — Configure R2 G0/1 as a DHCP Relay Agent

The `20.0.0.0/24` network is connected to R2's G0/1 interface.

DHCP clients on this network initially send DHCP Discover messages as broadcasts. Routers normally do not forward Layer 2 broadcasts.

To allow these DHCP requests to reach R1, configure R2 as a **DHCP relay agent**.

On R2:

```cisco
interface gigabitEthernet 0/1
 ip helper-address 192.168.12.1
 exit
```

The complete configuration is:

```cisco
interface gigabitEthernet 0/1
 ip helper-address 192.168.12.1
```

### What Does `ip helper-address` Do?

The command:

```cisco
ip helper-address 192.168.12.1
```

tells R2 to forward DHCP requests received on G0/1 to the DHCP server at:

```text
192.168.12.1
```

The process is:

```text
DHCP Client
     |
     | DHCP Discover
     v
R2 G0/1
     |
     | DHCP Relay
     v
R1 192.168.12.1
     |
     | DHCP Offer
     v
R2
     |
     v
DHCP Client
```

This allows the DHCP server on R1 to provide addresses to clients located on a different subnet.

---

# Task 6 — Test DHCP from a Client

On a PC connected to the `20.0.0.0/24` network, open the command prompt.

First check the current configuration:

```text
ipconfig
```

If the PC has an old or automatically generated address, release it:

```text
ipconfig /release
```

Then request a new DHCP lease:

```text
ipconfig /renew
```

A successful result should resemble:

```text
IP Address......................: 20.0.0.12
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 20.0.0.1
DNS Server......................: 20.0.0.1
```

The important point is that the client is receiving an address from the **20pool**, even though the DHCP server itself is located on another network.

---

# DHCP Troubleshooting

If the PC initially displays:

```text
DHCP request failed.
```

do not immediately assume that the configuration is incorrect.

In Packet Tracer, DHCP requests may occasionally require another attempt while the DHCP relay and server state converge.

Try:

```text
ipconfig /release
ipconfig /renew
```

again.

In the completed lab, the first two renewal attempts failed, but a subsequent renewal successfully obtained:

```text
20.0.0.12
```

with:

```text
Default Gateway: 20.0.0.1
DNS Server:       20.0.0.1
```

---

# DHCP Verification Commands

## On R1

Display configured DHCP pools:

```cisco
show running-config | section dhcp
```

Display DHCP bindings:

```cisco
show ip dhcp binding
```

Display DHCP pool statistics:

```cisco
show ip dhcp pool
```

Display DHCP conflicts:

```cisco
show ip dhcp conflict
```

The configuration should show the three pools:

```text
10pool
20pool
12pool
```

---

## On R2

Verify interface addressing:

```cisco
show ip interface brief
```

Verify the DHCP lease:

```cisco
show dhcp lease
```

Verify the DHCP relay configuration:

```cisco
show running-config interface gigabitEthernet 0/1
```

You should see:

```text
interface GigabitEthernet0/1
 ip helper-address 192.168.12.1
```

---

# Important Concepts

## DHCP Server

R1 acts as the DHCP server and contains three pools:

```text
10pool
20pool
12pool
```

Each pool defines the network from which addresses can be assigned.

## DHCP Client

R2 G0/0 acts as a DHCP client:

```cisco
ip address dhcp
```

It dynamically receives its address from R1.

## DHCP Relay Agent

R2 G0/1 acts as a DHCP relay:

```cisco
ip helper-address 192.168.12.1
```

This allows DHCP broadcasts from the `20.0.0.0/24` network to reach the DHCP server.

---

# Verification Checklist

- [ ] `10pool` is configured on R1.
- [ ] `10.0.0.1 – 10.0.0.10` are excluded from the 10pool.
- [ ] `20pool` is configured on R1.
- [ ] `20.0.0.1 – 20.0.0.10` are excluded from the 20pool.
- [ ] `12pool` is configured for `192.168.12.0/24`.
- [ ] R2 G0/0 is configured as a DHCP client.
- [ ] R2 G0/0 is enabled with `no shutdown`.
- [ ] R2 G0/0 receives an address from the `192.168.12.0/24` pool.
- [ ] R2 G0/1 has `ip helper-address 192.168.12.1`.
- [ ] The client on the `20.0.0.0/24` network is configured for DHCP.
- [ ] The client receives a `20.0.0.x` address.
- [ ] The client receives `20.0.0.1` as its default gateway.
- [ ] The client receives `20.0.0.1` as its DNS server.
- [ ] R1's DHCP bindings can be verified.
- [ ] Configurations are saved.

---

# Key Commands

```cisco
! DHCP exclusions
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 20.0.0.1 20.0.0.10

! DHCP pools
ip dhcp pool 10pool
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1
 dns-server 10.0.0.1

ip dhcp pool 20pool
 network 20.0.0.0 255.255.255.0
 default-router 20.0.0.1
 dns-server 20.0.0.1

ip dhcp pool 12pool
 network 192.168.12.0 255.255.255.0

! R2 DHCP client
interface gigabitEthernet 0/0
 ip address dhcp
 no shutdown

! R2 DHCP relay
interface gigabitEthernet 0/1
 ip helper-address 192.168.12.1
```

---

## Completion Criteria

The lab is successfully completed when **R1 is operating as a DHCP server with all three required pools, R2 dynamically obtains an address on G0/0 from the `192.168.12.0/24` pool, R2 relays DHCP requests from G0/1 to R1, and a client on the `20.0.0.0/24` network successfully receives an IP address, subnet mask, default gateway, and DNS server through DHCP.**
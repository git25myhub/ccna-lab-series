# IPv6 SLAAC and Static Routing Lab

## Lab Objective

The goal of this lab is to configure IPv6 addressing, enable Stateless Address Autoconfiguration (SLAAC), and establish full IPv6 connectivity between the networks using static IPv6 routes.

By completing this lab, you will practice:

- Enabling IPv6 routing on Cisco routers
- Configuring IPv6 addresses on router interfaces
- Enabling SLAAC on router interfaces
- Understanding IPv6 link-local addresses
- Configuring static IPv6 routes
- Verifying IPv6 connectivity using Cisco IOS commands

---

## Lab Tasks

### 1. Configure IPv6 on the Routers

Enable IPv6 routing on the routers and configure the required IPv6 addresses.

#### R1

Configure:

- G0/0: `2001:DB8:123:123::1/64`
- G0/1: `2001:DB8:1:1::1/64`

Enable IPv6 unicast routing:

```cisco
R1(config)# ipv6 unicast-routing
```

Configure G0/0:

```cisco
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:DB8:123:123::1/64
R1(config-if)# no shutdown
```

Configure G0/1:

```cisco
R1(config)# interface g0/1
R1(config-if)# ipv6 address 2001:DB8:1:1::1/64
R1(config-if)# no shutdown
```

---

#### R2

Configure G0/1 with:

`2001:DB8:2:2::1/64`

```cisco
R2(config)# ipv6 unicast-routing
R2(config)# interface g0/1
R2(config-if)# ipv6 address 2001:DB8:2:2::1/64
R2(config-if)# no shutdown
```

G0/0 will use SLAAC rather than a manually configured global IPv6 address.

---

#### R3

Configure G0/1 with:

`2001:DB8:3:3::1/64`

```cisco
R3(config)# ipv6 unicast-routing
R3(config)# interface g0/1
R3(config-if)# ipv6 address 2001:DB8:3:3::1/64
R3(config-if)# no shutdown
```

G0/0 will use SLAAC.

---

## 2. Configure SLAAC

SLAAC allows an IPv6 interface to automatically obtain its IPv6 addressing information from Router Advertisement messages.

Configure R2's G0/0:

```cisco
R2(config)# interface g0/0
R2(config-if)# ipv6 address autoconfig
R2(config-if)# no shutdown
```

Configure R3's G0/0:

```cisco
R3(config)# interface g0/0
R3(config-if)# ipv6 address autoconfig
R3(config-if)# no shutdown
```

The routers should automatically generate a link-local address and learn the IPv6 network prefix advertised on the connected network.

---

## 3. Configure Static IPv6 Routes

Static IPv6 routes must be configured so that all routers know how to reach remote IPv6 networks.

Use the following command format:

```cisco
ipv6 route <destination-network>/<prefix-length> <next-hop>
```

For example:

```cisco
R1(config)# ipv6 route 2001:DB8:2:2::/64 <R2-next-hop>
```

and:

```cisco
R1(config)# ipv6 route 2001:DB8:3:3::/64 <R2-next-hop>
```

Configure the appropriate routes on R1, R2, and R3 according to the topology.

The objective is to ensure that every router has routes to all remote IPv6 networks.

---

## IPv6 Addressing Summary

| Router | Interface | IPv6 Configuration |
|---|---|---|
| R1 | G0/0 | `2001:DB8:123:123::1/64` |
| R1 | G0/1 | `2001:DB8:1:1::1/64` |
| R2 | G0/0 | SLAAC |
| R2 | G0/1 | `2001:DB8:2:2::1/64` |
| R3 | G0/0 | SLAAC |
| R3 | G0/1 | `2001:DB8:3:3::1/64` |

---

## Verification

### Check IPv6 Interface Status

Use:

```cisco
show ipv6 interface brief
```

Example:

```text
R2# show ipv6 interface brief

GigabitEthernet0/0         [up/up]
    FE80::2D0:FFFF:FE69:3801
GigabitEthernet0/1         [up/up]
    FE80::2D0:FFFF:FE69:3802
    2001:DB8:2:2::1
```

R3 should similarly show a link-local address on G0/0 and the configured global address on G0/1.

---

### Check the IPv6 Routing Table

Use:

```cisco
show ipv6 route
```

Look for:

- Connected (`C`) routes
- Local (`L`) routes
- Static (`S`) routes

Example:

```text
R1# show ipv6 route
```

Static routes should appear with an `S` designation.

---

### Test IPv6 Connectivity

Use IPv6 ping to test communication between the routers:

```cisco
ping ipv6 2001:DB8:2:2::1
```

and:

```cisco
ping ipv6 2001:DB8:3:3::1
```

You should also test connectivity between the networks connected to the SLAAC interfaces.

---

### Check IPv6 Neighbors

Use:

```cisco
show ipv6 neighbors
```

This displays discovered IPv6 neighbors and their link-local addresses.

---

## Useful Verification Commands

```cisco
show ipv6 interface brief
show ipv6 interface
show ipv6 route
show ipv6 neighbors
ping ipv6 <IPv6-address>
traceroute ipv6 <IPv6-address>
```

---

## Important Concepts

### IPv6 Link-Local Addresses

IPv6 routers automatically generate link-local addresses, normally from the `FE80::/10` range.

For example:

```text
FE80::2D0:FFFF:FE69:3801
```

Link-local addresses are used for communication on the local IPv6 segment and are also commonly used as next-hop addresses for IPv6 routing.

### SLAAC

SLAAC allows IPv6 devices to automatically configure their addresses using Router Advertisement messages.

The command:

```cisco
ipv6 address autoconfig
```

allows the interface to automatically obtain its IPv6 addressing information.

### IPv6 Unicast Routing

Cisco routers require IPv6 unicast routing to be enabled before they can forward IPv6 packets between interfaces:

```cisco
ipv6 unicast-routing
```

Without this command, the router may have IPv6 addresses configured but will not perform normal IPv6 routing between interfaces.

### Static IPv6 Routing

Static IPv6 routes manually tell a router where to forward packets destined for remote IPv6 networks.

Static routing is useful in small networks and labs because it provides direct control over the routing table.

---

## Saving the Configuration

After completing the configuration, save it:

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

## Completion Criteria

The lab is successfully completed when:

- [ ] IPv6 unicast routing is enabled on the required routers.
- [ ] R1 G0/0 is configured with `2001:DB8:123:123::1/64`.
- [ ] R1 G0/1 is configured with `2001:DB8:1:1::1/64`.
- [ ] R2 G0/1 is configured with `2001:DB8:2:2::1/64`.
- [ ] R3 G0/1 is configured with `2001:DB8:3:3::1/64`.
- [ ] R2 G0/0 is configured for SLAAC.
- [ ] R3 G0/0 is configured for SLAAC.
- [ ] Static IPv6 routes provide reachability to all remote networks.
- [ ] IPv6 interfaces are up/up.
- [ ] The IPv6 routing tables contain the expected routes.
- [ ] End-to-end IPv6 pings succeed.
- [ ] Configurations are saved.

---

## Key Commands Learned

```cisco
ipv6 unicast-routing
ipv6 address <address>/<prefix>
ipv6 address autoconfig
no shutdown
show ipv6 interface brief
show ipv6 interface
show ipv6 route
show ipv6 neighbors
ipv6 route <network>/<prefix> <next-hop>
ping ipv6 <address>
traceroute ipv6 <address>
write memory
```

## Final Result

After completing the lab, R1, R2, and R3 should have functioning IPv6 interfaces, R2 and R3 should obtain IPv6 addressing through SLAAC on their G0/0 interfaces, and static IPv6 routing should provide full connectivity across the entire topology.
# Cisco IOS DHCP and DNS Lab

## Lab Overview

This lab demonstrates how to configure a Cisco router as a **DHCP server**, provide DNS information to DHCP clients, and configure a Layer 2 switch with a **default gateway** and **DNS server**.

The lab also demonstrates the difference between:

- Connectivity using an **IP address**
- Name resolution using **DNS**
- A switch's ability to reach remote networks using a configured default gateway
- A switch's ability to resolve hostnames using a configured DNS server

---

## Objectives

By completing this lab, you will learn how to:

- Configure a DHCP pool on a Cisco router.
- Exclude addresses from a DHCP address range.
- Configure the default gateway provided to DHCP clients.
- Configure a DNS server in a DHCP pool.
- Verify DHCP operation from a PC.
- Test connectivity using IP addresses and hostnames.
- Configure a Cisco switch with a default gateway.
- Configure a Cisco switch with a DNS server.
- Understand why DNS resolution fails when no DNS server is configured.

---

## Network Information

### DHCP Pool: `1pool`

| Setting | Value |
|---|---|
| Network | `192.168.1.0/24` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.1.1` |
| Excluded Range | `192.168.1.1 - 192.168.1.10` |
| DHCP Client Example | `192.168.1.11` |
| DNS Server | `20.0.0.100` |

### Servers

| Device | IP Address |
|---|---|
| SRV1 | `10.0.0.101` |
| SRV2 | `10.0.0.102` |
| DNS1 | `20.0.0.100` |

### Switch

| Setting | Value |
|---|---|
| SW1 VLAN 1 | `192.168.1.2` |
| Default Gateway | `192.168.1.1` |
| DNS Server | `20.0.0.100` |

---

## Task 1 – Configure the DHCP Pool on R1

Configure R1 as a DHCP server.

First, exclude the addresses from `192.168.1.1` through `192.168.1.10`:

```cisco
R1# configure terminal
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
```

Create the DHCP pool:

```cisco
R1(config)# ip dhcp pool 1pool
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# exit
```

At this stage, the DHCP pool provides clients with an IP address and default gateway, but no DNS server has been configured.

Save the configuration:

```cisco
R1# write memory
```

---

## Task 2 – Test PC1 Before DNS Configuration

On PC1, renew the DHCP lease:

```text
C:\> ipconfig /release
C:\> ipconfig /renew
```

Verify the addressing information:

```text
C:\> ipconfig
```

PC1 should receive an address similar to:

```text
IP Address......................: 192.168.1.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.1.1
DNS Server......................: 0.0.0.0
```

Test SRV1 by IP address:

```text
C:\> ping 10.0.0.101
```

The ping should succeed after the initial Packet Tracer ARP/DNS-related delay.

Now test SRV1 by hostname:

```text
C:\> ping srv1
```

The hostname lookup should fail:

```text
Ping request could not find host srv1.
Please check the name and try again.
```

### Observation

PC1 can reach SRV1 using its IP address because routing and IP connectivity are working.

However, PC1 cannot resolve `srv1` because it has **no DNS server configured**.

---

## Task 3 – Add DNS to the DHCP Pool

Configure DNS1 as the DNS server supplied to DHCP clients.

Enter the DHCP pool:

```cisco
R1# configure terminal
R1(config)# ip dhcp pool 1pool
R1(dhcp-config)# dns-server 20.0.0.100
R1(dhcp-config)# exit
```

Save the configuration:

```cisco
R1# write memory
```

---

## Task 4 – Renew PC1's DHCP Lease

PC1 must renew its DHCP lease to receive the newly configured DNS server.

```text
C:\> ipconfig /release
C:\> ipconfig /renew
```

Verify the configuration:

```text
C:\> ipconfig
```

PC1 should now show:

```text
IP Address......................: 192.168.1.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.1.1
DNS Server......................: 20.0.0.100
```

Test SRV1 by hostname:

```text
C:\> ping srv1
```

Expected result:

```text
Pinging 10.0.0.101 with 32 bytes of data:
Reply from 10.0.0.101
Reply from 10.0.0.101
Reply from 10.0.0.101
Reply from 10.0.0.101
```

Now test SRV2:

```text
C:\> ping srv2
```

The hostname should resolve to:

```text
10.0.0.102
```

and the ping should succeed.

### Observation

After DHCP supplies `20.0.0.100` as the DNS server, PC1 can resolve both:

```text
srv1 → 10.0.0.101
srv2 → 10.0.0.102
```

This demonstrates that **DNS resolution and IP connectivity are separate functions**.

---

## Task 5 – Test Connectivity from SW1

SW1 has the management address:

```text
192.168.1.2/24
```

From SW1, test SRV1 by IP address:

```cisco
SW1# ping 10.0.0.101
```

The ping should fail because SW1 does not yet have a default gateway configured.

Now test the hostname:

```cisco
SW1# ping srv1
```

This also fails because SW1 does not have a DNS server configured.

---

## Task 6 – Configure SW1's Default Gateway and DNS Server

Configure R1 as SW1's default gateway:

```cisco
SW1# configure terminal
SW1(config)# ip default-gateway 192.168.1.1
```

Configure DNS1 as SW1's DNS server:

```cisco
SW1(config)# ip name-server 20.0.0.100
```

Exit configuration mode:

```cisco
SW1(config)# exit
```

Save the configuration:

```cisco
SW1# write memory
```

---

## Task 7 – Test SW1 Again

Test SRV1 by IP address:

```cisco
SW1# ping 10.0.0.101
```

The ping should now succeed.

Test SRV1 by hostname:

```cisco
SW1# ping srv1
```

SW1 should first perform DNS resolution:

```text
Translating "srv1"...domain server (20.0.0.100)
```

The hostname should resolve to:

```text
10.0.0.101
```

The ping should then succeed.

---

## Verification Commands

### R1 – Verify DHCP Configuration

```cisco
R1# show ip dhcp pool
```

Check DHCP bindings:

```cisco
R1# show ip dhcp binding
```

Check DHCP configuration:

```cisco
R1# show running-config | section dhcp
```

You should see:

```text
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool 1pool
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 20.0.0.100
```

### SW1 – Verify Gateway and DNS

```cisco
SW1# show running-config
```

Look for:

```text
ip default-gateway 192.168.1.1
ip name-server 20.0.0.100
```

You can also verify the management interface:

```cisco
SW1# show ip interface brief
```

Expected VLAN 1 address:

```text
Vlan1    192.168.1.2    YES manual    up    up
```

---

## Lab Results

| Test | Before DNS/Gateway | After Configuration |
|---|---|---|
| PC1 → SRV1 by IP | Succeeds | Succeeds |
| PC1 → SRV1 by name | Fails | Succeeds |
| PC1 → SRV2 by name | Fails | Succeeds |
| SW1 → SRV1 by IP | Fails | Succeeds |
| SW1 → SRV1 by name | Fails | Succeeds |

---

## Key Concepts

### DHCP

DHCP automatically provides clients with network configuration such as:

- IP address
- Subnet mask
- Default gateway
- DNS server

In this lab, R1 provides all of these parameters to PC1.

### DHCP Excluded Addresses

The command:

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.10
```

prevents R1 from assigning those addresses to DHCP clients.

This leaves:

```text
192.168.1.11 – 192.168.1.254
```

available for DHCP allocation.

### Default Gateway

PC1 uses:

```text
192.168.1.1
```

as its default gateway to reach networks outside its local `192.168.1.0/24` subnet.

SW1 also needs a default gateway because its management interface is on the `192.168.1.0/24` network while SRV1 and DNS1 are on different networks.

### DNS

DNS translates hostnames into IP addresses.

For example:

```text
srv1 → 10.0.0.101
srv2 → 10.0.0.102
```

Without a DNS server, a device can still communicate with a destination by IP address, but it cannot resolve the hostname.

---

## Final Configuration

### R1

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.10

ip dhcp pool 1pool
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 20.0.0.100
```

### SW1

```cisco
interface vlan 1
 ip address 192.168.1.2 255.255.255.0

ip default-gateway 192.168.1.1
ip name-server 20.0.0.100
```

---

## Conclusion

This lab demonstrates how DHCP can be used to automatically provide clients with IP addressing, a default gateway, and DNS information.

Initially, PC1 could reach SRV1 using its IP address but could not resolve `srv1` because no DNS server was provided. After adding `20.0.0.100` to the DHCP pool and renewing PC1's lease, hostname resolution worked correctly.

SW1 demonstrated the same concepts manually. Without a default gateway, it could not reach SRV1's remote network. Without a DNS server, it could not resolve `srv1`. Configuring both allowed SW1 to successfully ping SRV1 by both IP address and hostname.

**Lab successfully completed when PC1 and SW1 can reach SRV1 by both IP address and hostname.**
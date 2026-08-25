# Cisco IOS Loopback Interface Configuration Lab

## 📌 Lab Overview

In this lab, you will configure a point-to-point connection between **R1 and R2**, create loopback interfaces on both routers, and test connectivity between the physical interfaces and loopback addresses.

You will also remove the loopback interfaces at the end of the lab to practice interface creation and deletion in Cisco IOS.

---

## 🎯 Objectives

By completing this lab, you will:

- Configure IP addresses on physical router interfaces.
- Enable router interfaces using `no shutdown`.
- Create Loopback0 interfaces.
- Assign `/32` addresses to loopback interfaces.
- Test connectivity to local and remote interfaces.
- Verify interface status using `show ip interface brief`.
- Understand how loopback interfaces appear in the routing table.
- Remove loopback interfaces using `no interface`.

---

## 🗺️ Lab Topology

```text
                 192.168.1.0/24

        G0/0                     G0/0
+----------------+          +----------------+
|      R1        |----------|       R2       |
| 192.168.1.1    |          | 192.168.1.2    |
|                |          |                |
| Lo0            |          | Lo0            |
| 1.1.1.1/32     |          | 2.2.2.2/32     |
+----------------+          +----------------+
```

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1` | `255.255.255.0` |
| R2 | G0/0 | `192.168.1.2` | `255.255.255.0` |
| R1 | Loopback0 | `1.1.1.1` | `255.255.255.255` |
| R2 | Loopback0 | `2.2.2.2` | `255.255.255.255` |

---

# 🛠️ Part 1 — Configure R1's Physical Interface

Enter privileged EXEC mode:

```text
R1> enable
```

Enter global configuration mode:

```text
R1# configure terminal
```

Configure GigabitEthernet0/0:

```text
R1(config)# interface gigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
```

The interface should transition to an **up/up** state.

Save the configuration:

```text
R1(config-if)# do write
```

or:

```text
R1(config-if)# do copy running-config startup-config
```

---

# 🛠️ Part 2 — Configure R2's Physical Interface

On R2:

```text
R2> enable
R2# configure terminal
```

Select the physical interface:

```text
R2(config)# interface gigabitEthernet0/0
```

Configure the IP address:

```text
R2(config-if)# ip address 192.168.1.2 255.255.255.0
```

Enable the interface:

```text
R2(config-if)# no shutdown
```

Save the configuration:

```text
R2(config-if)# do write
```

---

# 🔍 Part 3 — Test the Physical Connection

From R2, test connectivity to R1:

```text
R2# ping 192.168.1.1
```

A successful result should look similar to:

```text
Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:

.!!!!

Success rate is 80 percent (4/5)
```

The first packet may fail because the routers are resolving the destination MAC address through ARP.

Run the ping again:

```text
R2# ping 192.168.1.1
```

You should normally receive:

```text
!!!!!
Success rate is 100 percent (5/5)
```

---

# 🔄 Part 4 — Create R1 Loopback0

On R1, enter:

```text
R1(config)# interface loopback0
```

Assign the `/32` address:

```text
R1(config-if)# ip address 1.1.1.1 255.255.255.255
```

Save the configuration:

```text
R1(config-if)# do write
```

Unlike physical interfaces, loopback interfaces are **virtual interfaces** and do not require `no shutdown`.

---

# 🔄 Part 5 — Create R2 Loopback0

On R2:

```text
R2(config)# interface loopback0
```

Assign the `/32` address:

```text
R2(config-if)# ip address 2.2.2.2 255.255.255.255
```

Save:

```text
R2(config-if)# do write
```

---

# 🧪 Part 6 — Test R1's Loopback

From R1, ping its own loopback:

```text
R1# ping 1.1.1.1
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

Now test R1's connectivity to R2's loopback:

```text
R1# ping 2.2.2.2
```

This should succeed **only if R1 has a route to `2.2.2.2`**.

---

# 🧪 Part 7 — Test R2's Loopback

From R2, ping its own loopback:

```text
R2# ping 2.2.2.2
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

Then test R1's loopback:

```text
R2# ping 1.1.1.1
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

---

## ⚠️ Important Routing Note

The two physical interfaces are directly connected:

```text
192.168.1.0/24
```

However, the loopback networks:

```text
1.1.1.1/32
2.2.2.2/32
```

are **not directly connected** to the opposite router.

Therefore, in a normal Cisco IOS configuration, remote loopback pings require a route to the remote loopback.

For example, a static route could be configured on R1:

```text
R1(config)# ip route 2.2.2.2 255.255.255.255 192.168.1.2
```

And on R2:

```text
R2(config)# ip route 1.1.1.1 255.255.255.255 192.168.1.1
```

Then verify:

```text
R1# ping 2.2.2.2
R2# ping 1.1.1.1
```

> **Lab note:** If your Packet Tracer activity already contains static routes or another routing configuration, the remote loopback pings may work without adding these routes. Do not add routing configuration unless the lab requires it.

---

# 🔎 Part 8 — Examine the Routing Table

On R1:

```text
R1# show ip route
```

You should see the local loopback as a connected route:

```text
C    1.1.1.1/32 is directly connected, Loopback0
```

You should also see the physical network:

```text
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
```

On R2:

```text
R2# show ip route
```

You should see:

```text
C    2.2.2.2/32 is directly connected, Loopback0
```

This demonstrates that Cisco IOS automatically adds connected routes for configured interfaces.

---

# 🔍 Part 9 — Verify Interface Status

Run:

```text
R1# show ip interface brief
```

Expected R1 output should include:

```text
Interface              IP-Address      Status      Protocol
GigabitEthernet0/0     192.168.1.1     up          up
Loopback0              1.1.1.1         up          up
```

On R2:

```text
R2# show ip interface brief
```

Expected:

```text
Interface              IP-Address      Status      Protocol
GigabitEthernet0/0     192.168.1.2     up          up
Loopback0              2.2.2.2         up          up
```

---

# 🗑️ Part 10 — Remove the Loopback Interfaces

The final requirement is to remove the loopback interface from each router.

### R1

Enter global configuration mode:

```text
R1# configure terminal
```

Remove Loopback0:

```text
R1(config)# no interface loopback0
```

Verify:

```text
R1# show ip interface brief
```

Loopback0 should no longer appear.

---

### R2

On R2:

```text
R2# configure terminal
```

Remove Loopback0:

```text
R2(config)# no interface loopback0
```

Verify:

```text
R2# show ip interface brief
```

Loopback0 should no longer appear.

Save the final configuration:

```text
R2(config)# do write
```

---

# 🧠 Key Concepts Learned

### Physical Interface

A physical interface such as:

```text
interface GigabitEthernet0/0
```

requires an IP address and normally needs to be enabled with:

```text
no shutdown
```

### Loopback Interface

A loopback is a virtual interface:

```text
interface Loopback0
```

It can be assigned a `/32` address:

```text
ip address 1.1.1.1 255.255.255.255
```

Loopbacks are commonly used for:

- Router identification
- OSPF router IDs
- BGP router IDs
- Management
- Testing
- Stable source addresses

### Removing an Interface

To completely remove a loopback interface and its configuration:

```text
no interface loopback0
```

This is different from simply shutting down the interface.

---

# 📝 Useful Verification Commands

| Command | Purpose |
|---|---|
| `show ip interface brief` | View interface IP addresses and status |
| `show ip route` | View the routing table |
| `show running-config` | View current configuration |
| `show startup-config` | View saved configuration |
| `ping <IP>` | Test IP connectivity |
| `no interface loopback0` | Remove Loopback0 |
| `write` | Save running configuration |

---

# ✅ Completion Checklist

- [ ] R1 G0/0 configured with `192.168.1.1/24`.
- [ ] R2 G0/0 configured with `192.168.1.2/24`.
- [ ] R1 G0/0 enabled.
- [ ] R2 G0/0 enabled.
- [ ] R1 Loopback0 configured with `1.1.1.1/32`.
- [ ] R2 Loopback0 configured with `2.2.2.2/32`.
- [ ] R1 successfully pinged `1.1.1.1`.
- [ ] R2 successfully pinged `2.2.2.2`.
- [ ] Remote loopback connectivity was tested.
- [ ] Routing was verified where required.
- [ ] R1 Loopback0 was removed.
- [ ] R2 Loopback0 was removed.
- [ ] Final interface status was verified.
- [ ] Final configuration was saved.

---

## 🏁 Lab Completion

The lab is complete when the physical interfaces are configured and operational, the loopback interfaces have been created and tested, and **Loopback0 has been removed from both R1 and R2**.

Final physical connectivity:

```text
R1 G0/0  192.168.1.1/24
       |
       |
R2 G0/0  192.168.1.2/24
```

**Lab Status: COMPLETE ✅**
# Cisco Static NAT – Internet Simulation Lab

## Lab Overview

This lab demonstrates how **Network Address Translation (NAT)** allows private inside hosts to communicate with an external network.

RIP has already been configured between R1 and R2, allowing the routers to reach their respective inside networks. However, the serial connection between R1 and R2 represents the **Internet**, where ACLs prevent the private IP addresses of PC1, PC2, and PC3 from being accepted.

The solution is to configure **static NAT on R1**, translating each PC's private address into a public address before traffic crosses the simulated Internet.

---

## Objectives

By completing this lab, you will learn how to:

- Identify why private hosts cannot reach an external network.
- Understand the role of NAT when communicating across an Internet-like connection.
- Configure inside and outside NAT interfaces.
- Configure static NAT mappings.
- Verify NAT translations.
- Test connectivity after implementing NAT.
- Understand how NAT changes the source IP address of packets.

---

## Network Information

### Inside Network

| Device | Private IP | NAT Public IP |
|---|---:|---:|
| PC1 | `192.168.1.11` | `1.2.3.11` |
| PC2 | `192.168.1.12` | `1.2.3.12` |
| PC3 | `192.168.1.13` | `1.2.3.13` |

### External Network

| Device | IP Address |
|---|---|
| SRV1 | `1.1.1.100` |

### NAT Interfaces on R1

| Interface | Role |
|---|---|
| GigabitEthernet0/0 | NAT Inside |
| Serial0/3/0 | NAT Outside |

---

## Task 1 – Determine Why the Pings Fail

RIP has already been configured so that R1 and R2 can reach their inside networks.

From each PC, attempt to ping SRV1:

```text
C:\> ping 1.1.1.100
```

Initially, the ping should fail.

Example:

```text
Pinging 1.1.1.100 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 1.1.1.100:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

### Why Does the Ping Fail?

The PCs are using private IP addresses:

```text
192.168.1.11
192.168.1.12
192.168.1.13
```

These addresses are being sent toward the simulated Internet.

The serial connection between R1 and R2 has ACLs that simulate the behavior of an Internet connection and prevent these private source addresses from passing through.

Therefore, even though **RIP provides routing**, routing alone is not enough.

The source addresses need to be translated into public addresses before the traffic crosses the simulated Internet.

---

# Task 2 – Configure Static NAT on R1

## Step 1 – Identify the Inside Interface

The interface connecting R1 to the PCs is:

```text
GigabitEthernet0/0
```

Configure it as the NAT inside interface:

```cisco
R1# configure terminal
R1(config)# interface gigabitEthernet0/0
R1(config-if)# ip nat inside
```

---

## Step 2 – Identify the Outside Interface

The serial interface connects R1 to the simulated Internet:

```text
Serial0/3/0
```

Configure it as the NAT outside interface:

```cisco
R1(config-if)# interface serial0/3/0
R1(config-if)# ip nat outside
R1(config-if)# exit
```

At this point, R1 knows which interface connects to the **inside** network and which connects to the **outside** network.

---

## Step 3 – Configure the Static NAT Mappings

Configure PC1:

```cisco
R1(config)# ip nat inside source static 192.168.1.11 1.2.3.11
```

Configure PC2:

```cisco
R1(config)# ip nat inside source static 192.168.1.12 1.2.3.12
```

Configure PC3:

```cisco
R1(config)# ip nat inside source static 192.168.1.13 1.2.3.13
```

Save the configuration:

```cisco
R1(config)# end
R1# write memory
```

---

## NAT Translation Table

The final static NAT configuration should create the following mappings:

| Inside Local | Inside Global |
|---|---|
| `192.168.1.11` | `1.2.3.11` |
| `192.168.1.12` | `1.2.3.12` |
| `192.168.1.13` | `1.2.3.13` |

In simple terms:

```text
PC1: 192.168.1.11 → 1.2.3.11
PC2: 192.168.1.12 → 1.2.3.12
PC3: 192.168.1.13 → 1.2.3.13
```

---

# Task 3 – Test Connectivity Again

From PC1:

```text
C:\> ping 1.1.1.100
```

From PC2:

```text
C:\> ping 1.1.1.100
```

From PC3:

```text
C:\> ping 1.1.1.100
```

The pings should now succeed.

Example successful result:

```text
Pinging 1.1.1.100 with 32 bytes of data:

Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126
Reply from 1.1.1.100: bytes=32 time=2ms TTL=126
Reply from 1.1.1.100: bytes=32 time=1ms TTL=126

Ping statistics for 1.1.1.100:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

The first ping may occasionally time out because of ARP or Packet Tracer's initial processing. Retry the ping if necessary.

---

# Verification

## Verify NAT Configuration

On R1:

```cisco
R1# show running-config | include ip nat
```

You should see:

```text
ip nat inside
ip nat outside
ip nat inside source static 192.168.1.11 1.2.3.11
ip nat inside source static 192.168.1.12 1.2.3.12
ip nat inside source static 192.168.1.13 1.2.3.13
```

---

## Verify NAT Interfaces

Use:

```cisco
R1# show ip nat statistics
```

This allows you to verify that NAT is configured and that the inside and outside interfaces are correctly identified.

---

## Verify NAT Translations

Use:

```cisco
R1# show ip nat translations
```

You should see entries similar to:

```text
Pro  Inside global      Inside local       Outside local      Outside global
---  1.2.3.11          192.168.1.11       ---                ---
---  1.2.3.12          192.168.1.12       ---                ---
---  1.2.3.13          192.168.1.13       ---                ---
```

The static entries remain present even before traffic is generated.

---

# Understanding the NAT Process

Before NAT:

```text
PC1
192.168.1.11
      |
      v
     R1
      |
      v
 Simulated Internet
      |
      v
    SRV1
  1.1.1.100
```

The source address is:

```text
192.168.1.11
```

The simulated Internet ACL does not allow this private address to pass.

After NAT:

```text
PC1
192.168.1.11
      |
      v
     R1
      |
      | NAT
      v
1.2.3.11
      |
      v
 Simulated Internet
      |
      v
    SRV1
  1.1.1.100
```

R1 translates:

```text
192.168.1.11 → 1.2.3.11
```

The external network therefore sees the packet as coming from:

```text
1.2.3.11
```

The same process occurs for PC2 and PC3.

---

# Important NAT Terminology

### Inside Local

The actual private address assigned to the internal host.

Examples:

```text
192.168.1.11
192.168.1.12
192.168.1.13
```

### Inside Global

The address representing the internal host to the outside network.

Examples:

```text
1.2.3.11
1.2.3.12
1.2.3.13
```

### NAT Inside

The interface connected toward the private network.

```text
GigabitEthernet0/0
```

### NAT Outside

The interface connected toward the external network.

```text
Serial0/3/0
```

---

# Final R1 Configuration

The relevant NAT configuration should look like:

```cisco
interface GigabitEthernet0/0
 ip nat inside

interface Serial0/3/0
 ip nat outside

ip nat inside source static 192.168.1.11 1.2.3.11
ip nat inside source static 192.168.1.12 1.2.3.12
ip nat inside source static 192.168.1.13 1.2.3.13
```

---

# Troubleshooting

If the pings still fail, check the following:

### 1. Verify NAT inside/outside interfaces

```cisco
R1# show running-config
```

Make sure:

```text
GigabitEthernet0/0 → ip nat inside
Serial0/3/0 → ip nat outside
```

### 2. Verify the PCs' IP addresses

PC1:

```text
192.168.1.11
```

PC2:

```text
192.168.1.12
```

PC3:

```text
192.168.1.13
```

### 3. Verify the static translations

```cisco
R1# show ip nat translations
```

Make sure each private address has the correct public translation.

### 4. Test the external server

```text
C:\> ping 1.1.1.100
```

### 5. Check the NAT statistics

```cisco
R1# show ip nat statistics
```

---

# Lab Results

| PC | Private Address | NAT Address | SRV1 Ping Before NAT | SRV1 Ping After NAT |
|---|---|---|---|---|
| PC1 | `192.168.1.11` | `1.2.3.11` | ❌ Fails | ✅ Succeeds |
| PC2 | `192.168.1.12` | `1.2.3.12` | ❌ Fails | ✅ Succeeds |
| PC3 | `192.168.1.13` | `1.2.3.13` | ❌ Fails | ✅ Succeeds |

---

# Key Takeaways

- **RIP provides routing**, but routing alone does not solve the problem of private IP addresses crossing the simulated Internet.
- The serial connection between R1 and R2 is configured to simulate an Internet connection with ACL restrictions.
- **NAT translates private IP addresses into public addresses**.
- `ip nat inside` identifies the interface facing the private network.
- `ip nat outside` identifies the interface facing the external network.
- Static NAT creates a permanent one-to-one mapping between private and public addresses.
- After NAT is configured, PC1, PC2, and PC3 can successfully reach SRV1.

## Completion Criteria

The lab is successfully completed when:

- [x] R1's inside interface is configured for NAT.
- [x] R1's outside interface is configured for NAT.
- [x] PC1 is mapped to `1.2.3.11`.
- [x] PC2 is mapped to `1.2.3.12`.
- [x] PC3 is mapped to `1.2.3.13`.
- [x] PC1 can ping SRV1.
- [x] PC2 can ping SRV1.
- [x] PC3 can ping SRV1.

**Lab completed: Static NAT successfully allows the inside PCs to communicate with SRV1 across the simulated Internet.**
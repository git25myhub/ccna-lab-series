# RIPv2, Syslog, PAT, DHCP, DHCP Relay and SSH Lab

## Lab Objective

The goal of this lab is to configure a small routed network using several essential Cisco networking services and security features.

By completing this lab, you will practice:

- Configuring RIPv2 dynamic routing
- Disabling RIP auto-summary
- Configuring centralized Syslog
- Configuring Port Address Translation (PAT)
- Configuring a Cisco router as a DHCP server
- Configuring DHCP relay using `ip helper-address`
- Configuring SSH version 2 for secure remote management
- Verifying network connectivity and services

---

## Lab Tasks

### 1. Configure RIPv2 Between R1, R2, and R3

Configure RIP version 2 on all three routers.

RIPv2 should advertise all directly connected networks and auto-summary should be disabled.

### R1

```cisco
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# no auto-summary
R1(config-router)# network 192.168.1.0
R1(config-router)# network 1.2.3.0
```

### R2

```cisco
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# no auto-summary
R2(config-router)# network 192.168.2.0
R2(config-router)# network 1.2.3.0
```

### R3

```cisco
R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# no auto-summary
R3(config-router)# network 30.0.0.0
R3(config-router)# network 1.2.3.0
```

### Verify RIP

Use:

```cisco
show ip protocols
show ip route
show ip rip database
```

You should see RIP-learned routes marked with `R` in the routing table.

For example, R3 learned the `192.168.1.0/24` network through R1:

```text
192.168.1.0/24
    [1] via 1.2.3.1
```

---

# 2. Configure Syslog

Configure R1, R2, and R3 to send system log messages to the Syslog server, SRV1.

SRV1's address is:

```text
30.0.0.100
```

### R1

```cisco
R1(config)# logging 30.0.0.100
```

### R2

```cisco
R2(config)# logging 30.0.0.100
```

### R3

```cisco
R3(config)# logging 30.0.0.100
```

Save the configuration:

```cisco
write memory
```

### Verify Syslog

Use:

```cisco
show logging
```

The router should show the configured Syslog server.

On SRV1, verify that the Syslog service is enabled and that messages from the routers are being received.

---

# 3. Configure PAT on R1 and R2

Port Address Translation allows multiple inside hosts to share the router's outside interface address when accessing external networks.

For this lab, PAT will translate inside hosts to the router's G0/1 interface.

---

## R1 PAT Configuration

### Configure G0/0 as the inside interface

```cisco
R1(config)# interface g0/0
R1(config-if)# ip nat inside
```

### Configure G0/1 as the outside interface

```cisco
R1(config)# interface g0/1
R1(config-if)# ip nat outside
```

### Create the NAT access list

```cisco
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

### Configure PAT

```cisco
R1(config)# ip nat inside source list 1 interface g0/1 overload
```

---

## R2 PAT Configuration

### Configure G0/0 as the inside interface

```cisco
R2(config)# interface g0/0
R2(config-if)# ip nat inside
```

### Configure G0/1 as the outside interface

```cisco
R2(config)# interface g0/1
R2(config-if)# ip nat outside
```

### Create the NAT access list

```cisco
R2(config)# access-list 1 permit 192.168.2.0 0.0.0.255
```

### Configure PAT

```cisco
R2(config)# ip nat inside source list 1 interface g0/1 overload
```

### Verify PAT

Use:

```cisco
show ip nat translations
show ip nat statistics
```

Generate traffic from an inside PC and check whether a translation appears.

---

# 4. Configure R1 as a DHCP Server

R1 will provide DHCP addressing for two different networks.

## DHCP Pool 1

| Setting | Value |
|---|---|
| Pool Name | `1pool` |
| Network | `192.168.1.0/24` |
| Default Gateway | `192.168.1.1` |
| DNS Server | `30.0.0.100` |
| Excluded Range | `192.168.1.1 - 192.168.1.10` |

### Exclude Addresses

```cisco
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
```

### Create Pool 1

```cisco
R1(config)# ip dhcp pool 1pool
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 30.0.0.100
```

---

## DHCP Pool 2

| Setting | Value |
|---|---|
| Pool Name | `2pool` |
| Network | `192.168.2.0/24` |
| Default Gateway | `192.168.2.1` |
| DNS Server | `30.0.0.100` |
| Excluded Range | `192.168.2.1 - 192.168.2.10` |

### Exclude Addresses

```cisco
R1(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.10
```

### Create Pool 2

```cisco
R1(config)# ip dhcp pool 2pool
R1(dhcp-config)# network 192.168.2.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.2.1
R1(dhcp-config)# dns-server 30.0.0.100
```

---

## Verify DHCP

Use:

```cisco
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
```

On a Packet Tracer PC, release and renew the DHCP lease:

```text
C:\> ipconfig /release
C:\> ipconfig /renew
```

A PC on the `192.168.1.0/24` network should receive an address beginning at `192.168.1.11`.

Example:

```text
IP Address......................: 192.168.1.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.1.1
DNS Server......................: 30.0.0.100
```

---

# 5. Configure R2 as a DHCP Relay

R2's clients are on a different subnet from the DHCP server on R1.

Because DHCP Discover messages are broadcast messages, they do not normally cross a router.

R2 must therefore relay DHCP requests to R1.

Configure the interface facing the DHCP clients:

```cisco
R2(config)# interface g0/0
R2(config-if)# ip helper-address 1.2.3.1
```

The address `1.2.3.1` represents R1's address on the R1-R2 link.

### Verify DHCP Relay

On a client connected to R2:

```text
C:\> ipconfig /release
C:\> ipconfig /renew
```

The client should receive an address from the `2pool` DHCP pool.

Expected addressing:

```text
Network:       192.168.2.0/24
Gateway:       192.168.2.1
DNS:           30.0.0.100
```

---

# 6. Configure SSH Version 2 on R1

SSH provides secure remote access to the router.

R1 must be configured with:

- Username: `cisco`
- Password: `ccna`
- Domain name: `cisco.com`
- RSA key modulus: `1024 bits`
- SSH version: `2`

---

## Configure the Domain Name

```cisco
R1(config)# ip domain-name cisco.com
```

---

## Create the Local User

```cisco
R1(config)# username cisco password ccna
```

---

## Generate RSA Keys

```cisco
R1(config)# crypto key generate rsa
```

When prompted for the modulus size, enter:

```text
1024
```

The resulting key name should be based on the router hostname and domain:

```text
R1.cisco.com
```

---

## Configure the VTY Lines

```cisco
R1(config)# line vty 0 15
R1(config-line)# login local
R1(config-line)# transport input ssh
R1(config-line)# exit
```

---

## Force SSH Version 2

```cisco
R1(config)# ip ssh version 2
```

Save the configuration:

```cisco
R1(config)# end
R1# write memory
```

---

# SSH Verification

Use:

```cisco
show ip ssh
```

The output should indicate that SSH version 2 is enabled.

You can also verify the VTY configuration:

```cisco
show running-config | section line vty
```

---

## Test SSH from a PC

From a Packet Tracer PC:

```text
C:\> ssh -l cisco 192.168.1.1
```

Enter the password:

```text
ccna
```

A successful connection should provide access to R1's CLI.

Example:

```text
C:\>ssh -l cisco 192.168.1.1
Password:

R1>
```

The successful SSH connection confirms that the local username, RSA keys, VTY configuration, and SSH service are working.

---

# Addressing and Services Summary

| Device | Interface/Service | Address/Configuration |
|---|---|---|
| R1 | LAN | `192.168.1.0/24` |
| R1 | DHCP Pool | `1pool` |
| R1 | DHCP Gateway | `192.168.1.1` |
| R1 | DNS | `30.0.0.100` |
| R1 | PAT | G0/1 |
| R1 | Syslog | `30.0.0.100` |
| R1 | SSH User | `cisco` |
| R1 | SSH Password | `ccna` |
| R1 | Domain | `cisco.com` |
| R2 | LAN | `192.168.2.0/24` |
| R2 | DHCP Relay | `1.2.3.1` |
| R2 | PAT | G0/1 |
| R2 | Syslog | `30.0.0.100` |
| R3 | Server Network | `30.0.0.0/24` |
| R3 | Syslog | `30.0.0.100` |
| SRV1 | Syslog/DNS | `30.0.0.100` |

---

# Useful Verification Commands

### RIP

```cisco
show ip protocols
show ip route
show ip rip database
```

### Syslog

```cisco
show logging
```

### NAT/PAT

```cisco
show ip nat translations
show ip nat statistics
```

### DHCP

```cisco
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
```

### SSH

```cisco
show ip ssh
show running-config | section line vty
```

### Connectivity

```cisco
ping <destination-ip>
traceroute <destination-ip>
```

---

# Troubleshooting Checklist

If DHCP is not working:

- Check that the DHCP pool network is correct.
- Confirm the default gateway matches the router interface.
- Confirm excluded addresses were configured.
- On R2, verify `ip helper-address 1.2.3.1`.
- Confirm R1 and R2 have a working route between their networks.

If PAT is not working:

- Verify the inside interface is configured with `ip nat inside`.
- Verify the outside interface is configured with `ip nat outside`.
- Verify the NAT ACL matches the inside network.
- Check `show ip nat translations`.
- Check `show ip nat statistics`.

If RIP is not working:

- Verify `version 2`.
- Verify `no auto-summary`.
- Verify all required networks are advertised.
- Check `show ip route`.
- Check `show ip protocols`.

If SSH is not working:

- Verify the username and password.
- Verify the domain name.
- Verify RSA keys exist.
- Verify `ip ssh version 2`.
- Verify `login local` on the VTY lines.
- Verify `transport input ssh`.

---

# Completion Criteria

The lab is successfully completed when:

- [ ] R1, R2, and R3 are running RIPv2.
- [ ] RIP auto-summary is disabled.
- [ ] All required connected networks are advertised.
- [ ] R1, R2, and R3 send Syslog messages to SRV1 (`30.0.0.100`).
- [ ] PAT is configured on R1 using G0/1.
- [ ] PAT is configured on R2 using G0/1.
- [ ] R1 provides DHCP for the `192.168.1.0/24` network.
- [ ] R1 provides DHCP for the `192.168.2.0/24` network.
- [ ] DHCP addresses `.1` through `.10` are excluded from both pools.
- [ ] R2 forwards DHCP requests to R1 using `ip helper-address`.
- [ ] Clients successfully obtain DHCP addresses.
- [ ] Clients receive the correct default gateway and DNS server.
- [ ] SSH version 2 is enabled on R1.
- [ ] R1 has the `cisco` local user with password `ccna`.
- [ ] R1 uses the `cisco.com` domain.
- [ ] R1 has a 1024-bit RSA key.
- [ ] VTY lines allow SSH and use the local user database.
- [ ] The PC can successfully SSH to R1.
- [ ] The configuration is saved with `write memory`.

---

## Final Result

After completing the lab, R1, R2, and R3 should dynamically exchange routes using RIPv2, while SRV1 provides centralized Syslog services. R1 should provide DHCP services for both LANs, with R2 acting as a DHCP relay. R1 and R2 should provide PAT for their respective inside networks, and R1 should be securely accessible through SSH version 2.

This lab combines **dynamic routing, network services, NAT/PAT, DHCP, DHCP relay, Syslog, and secure remote management** into one practical Cisco networking configuration.
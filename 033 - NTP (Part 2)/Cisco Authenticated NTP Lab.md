# Cisco Authenticated NTP Lab

## Overview

In this lab, you will configure **authenticated Network Time Protocol (NTP)** across three Cisco routers.

R1 will act as the NTP master and will authenticate NTP requests using two MD5 authentication keys. R2 will synchronize with R1 using **Key 1**, while R3 will synchronize with R1 using **Key 2**.

This lab builds on basic NTP configuration by adding **NTP authentication**, ensuring that only devices using the correct authentication keys can establish trusted NTP relationships.

## Lab Objectives

By completing this lab, you will:

- Configure the correct timezone on R1.
- Configure the date and time on R1.
- Configure R1 as an NTP master using the default stratum.
- Enable NTP authentication on R1.
- Configure NTP authentication keys on R1.
- Configure R1 to trust both authentication keys.
- Configure R2 to authenticate with R1 using Key 1.
- Configure R3 to authenticate with R1 using Key 2.
- Verify successful NTP synchronization.
- Understand the role of trusted NTP keys.

---

# NTP Authentication Design

The lab uses the following authentication scheme:

| Router | NTP Role | NTP Server | Authentication Key |
|---|---|---|---|
| R1 | NTP Master | Local clock | Key 1 + Key 2 |
| R2 | NTP Client | 192.168.12.1 | Key 1 |
| R3 | NTP Client | 192.168.13.1 | Key 2 |

### Authentication Keys

| Key Number | Authentication Type | Password |
|---|---|---|
| 1 | MD5 | `cisco1` |
| 2 | MD5 | `cisco2` |

The topology is:

```text
                    R1
              NTP Master
             /          \
            /            \
           R2             R3
       Key 1            Key 2
      cisco1            cisco2
```

R2 and R3 both synchronize **directly with R1**, rather than synchronizing with each other.

---

# 1. Configure R1's Timezone

Enter privileged EXEC mode and configuration mode:

```cisco
R1> enable
R1# configure terminal
```

Configure the timezone:

```cisco
R1(config)# clock timezone GMT 3
```

This configures R1 for **GMT+3**, which matches the timezone used in this lab.

---

# 2. Configure R1's Date and Time

Set the date and time on R1:

```cisco
R1# clock set 07:00:00 Aug 25 2026
```

Verify the clock:

```cisco
R1# show clock detail
```

The exact displayed time will depend on the configured timezone.

The clock should initially show:

```text
Time source is user configuration
```

because the time was manually configured.

---

# 3. Configure R1 as the NTP Master

Enter configuration mode:

```cisco
R1# configure terminal
```

Configure R1 as the NTP master:

```cisco
R1(config)# ntp master
```

No stratum number is specified, so the router uses the **default NTP master stratum**.

---

# 4. Enable NTP Authentication on R1

Enable NTP authentication:

```cisco
R1(config)# ntp authenticate
```

This tells R1 to require authentication for authenticated NTP relationships.

---

# 5. Configure NTP Authentication Keys on R1

Configure Key 1:

```cisco
R1(config)# ntp authentication-key 1 md5 cisco1
```

Configure Key 2:

```cisco
R1(config)# ntp authentication-key 2 md5 cisco2
```

The resulting authentication configuration is:

```text
Key 1 → MD5 → cisco1
Key 2 → MD5 → cisco2
```

---

# 6. Configure R1's Trusted Keys

R1 must be configured to trust both keys.

Configure Key 1:

```cisco
R1(config)# ntp trusted-key 1
```

Configure Key 2:

```cisco
R1(config)# ntp trusted-key 2
```

R1 now trusts both authentication keys.

Save the configuration:

```cisco
R1(config)# end
R1# write
```

---

# 7. Configure R2 for Authenticated NTP

R2 must use **Key 1** when communicating with R1.

Enter configuration mode:

```cisco
R2> enable
R2# configure terminal
```

Configure the timezone:

```cisco
R2(config)# clock timezone GMT 3
```

Enable NTP authentication:

```cisco
R2(config)# ntp authenticate
```

Configure Key 1 using the same password as R1:

```cisco
R2(config)# ntp authentication-key 1 md5 cisco1
```

Tell R2 to trust Key 1:

```cisco
R2(config)# ntp trusted-key 1
```

Configure R1 as the NTP server and specify Key 1:

```cisco
R2(config)# ntp server 192.168.12.1 key 1
```

Save the configuration:

```cisco
R2(config)# end
R2# write
```

---

# 8. Verify R2 Synchronization

Check R2's clock:

```cisco
R2# show clock detail
```

A successfully synchronized router should display:

```text
Time source is NTP
```

Check the NTP association:

```cisco
R2# show ntp associations
```

Example:

```text
address         ref clock       st   when     poll    reach
*~192.168.12.1  127.127.1.1     8    15       16      3
```

The important indicators are:

- `~` — the server was manually configured.
- `*` — the server is the selected system peer.
- `127.127.1.1` — R1's local NTP master reference clock.
- `st 8` — R1's NTP stratum in this Packet Tracer environment.

After a short period, the `reach` value should increase as successful NTP exchanges occur.

---

# 9. Configure R3 for Authenticated NTP

R3 must use **Key 2** when communicating with R1.

Enter configuration mode:

```cisco
R3> enable
R3# configure terminal
```

Configure the timezone:

```cisco
R3(config)# clock timezone GMT 3
```

Enable NTP authentication:

```cisco
R3(config)# ntp authenticate
```

Configure Key 2:

```cisco
R3(config)# ntp authentication-key 2 md5 cisco2
```

Tell R3 to trust Key 2:

```cisco
R3(config)# ntp trusted-key 2
```

Configure R1 as the NTP server using Key 2:

```cisco
R3(config)# ntp server 192.168.13.1 key 2
```

Save the configuration:

```cisco
R3(config)# end
R3# write
```

---

# 10. Verify R3 Synchronization

Check the NTP association:

```cisco
R3# show ntp associations
```

Example:

```text
address         ref clock       st   when     poll    reach
~192.168.13.1   127.127.1.1     8    1        16      37
```

The `~` indicates that R1 has been configured as an NTP server.

Check the clock:

```cisco
R3# show clock detail
```

Once synchronization is established, the output should indicate:

```text
Time source is NTP
```

---

# Complete Configuration

## R1 — NTP Master

```cisco
enable
configure terminal

clock timezone GMT 3

ntp master
ntp authenticate

ntp authentication-key 1 md5 cisco1
ntp authentication-key 2 md5 cisco2

ntp trusted-key 1
ntp trusted-key 2

end
write
```

Set the clock separately:

```cisco
clock set 07:00:00 Aug 25 2026
```

---

## R2 — NTP Client Using Key 1

```cisco
enable
configure terminal

clock timezone GMT 3

ntp authenticate
ntp authentication-key 1 md5 cisco1
ntp trusted-key 1

ntp server 192.168.12.1 key 1

end
write
```

---

## R3 — NTP Client Using Key 2

```cisco
enable
configure terminal

clock timezone GMT 3

ntp authenticate
ntp authentication-key 2 md5 cisco2
ntp trusted-key 2

ntp server 192.168.13.1 key 2

end
write
```

---

# Verification Commands

## Check the System Clock

```cisco
show clock
```

or:

```cisco
show clock detail
```

## Check NTP Associations

```cisco
show ntp associations
```

## Check NTP Status

```cisco
show ntp status
```

## Check NTP Configuration

```cisco
show running-config | include ntp
```

---

# Understanding NTP Authentication

NTP authentication prevents a router from blindly accepting time information from an unauthorized source.

In this lab, R1 has two authentication keys:

```text
Key 1 = cisco1
Key 2 = cisco2
```

R2 knows only Key 1:

```text
R2 → Key 1 → R1
```

R3 knows only Key 2:

```text
R3 → Key 2 → R1
```

R1 trusts both:

```text
R1 → Key 1
R1 → Key 2
```

This allows R1 to authenticate both clients while keeping the authentication configuration separate.

### Important

The key number and password must match between the NTP server and client.

For R2:

```text
R1: Key 1 = cisco1
R2: Key 1 = cisco1
```

For R3:

```text
R1: Key 2 = cisco2
R3: Key 2 = cisco2
```

If the key number or password is incorrect, authenticated NTP synchronization will fail.

---

# Troubleshooting

## R2 Is Not Synchronizing

Check connectivity:

```cisco
R2# ping 192.168.12.1
```

Check the NTP configuration:

```cisco
R2# show running-config | include ntp
```

Verify that R2 contains:

```text
ntp authenticate
ntp authentication-key 1 md5 cisco1
ntp trusted-key 1
ntp server 192.168.12.1 key 1
```

---

## R3 Is Not Synchronizing

Check connectivity:

```cisco
R3# ping 192.168.13.1
```

Check the NTP configuration:

```cisco
R3# show running-config | include ntp
```

Verify that R3 contains:

```text
ntp authenticate
ntp authentication-key 2 md5 cisco2
ntp trusted-key 2
ntp server 192.168.13.1 key 2
```

---

## Check R1's Authentication Configuration

On R1:

```cisco
R1# show running-config | include ntp
```

You should see configuration similar to:

```text
ntp master
ntp authenticate
ntp authentication-key 1 md5 cisco1
ntp authentication-key 2 md5 cisco2
ntp trusted-key 1
ntp trusted-key 2
```

---

# Common Mistakes

### Wrong Key Number

For example, configuring R2 with:

```cisco
ntp server 192.168.12.1 key 2
```

would be incorrect because R2 is supposed to use Key 1.

### Wrong Password

R2 must use:

```text
cisco1
```

not:

```text
cisco2
```

R3 must use:

```text
cisco2
```

not:

```text
cisco1
```

### Missing Trusted Key

Creating an authentication key is not enough. The key must also be trusted:

```cisco
ntp trusted-key 1
```

or:

```cisco
ntp trusted-key 2
```

### Missing NTP Authentication

Both the NTP master and authenticated clients should have:

```cisco
ntp authenticate
```

---

# Completion Criteria

The lab is successfully completed when:

- [ ] R1 has the correct timezone configured.
- [ ] R1 has the correct date and time.
- [ ] R1 is configured as an NTP master.
- [ ] R1 uses the default NTP stratum.
- [ ] NTP authentication is enabled on R1.
- [ ] R1 has Key 1 configured as `cisco1`.
- [ ] R1 has Key 2 configured as `cisco2`.
- [ ] R1 trusts Key 1 and Key 2.
- [ ] R2 uses Key 1 to authenticate with R1.
- [ ] R2 is synchronized with R1.
- [ ] R3 uses Key 2 to authenticate with R1.
- [ ] R3 is synchronized with R1.
- [ ] `show clock detail` reports `Time source is NTP` on R2 and R3.
- [ ] `show ntp associations` shows R1 as the selected NTP peer.
- [ ] All configurations are saved.

# Key Takeaway

This lab demonstrates how to secure NTP using authentication.

Instead of allowing any configured NTP relationship to provide time, the routers verify the NTP source using a shared authentication key.

The final design is:

```text
                 R1
          NTP Master / Server
          Key 1: cisco1
          Key 2: cisco2
             /       \
            /         \
       Key 1           Key 2
       cisco1          cisco2
          /               \
         R2               R3
     NTP Client       NTP Client
```

R2 authenticates using **Key 1**, while R3 authenticates using **Key 2**. R1 trusts both keys and provides the authoritative time source for the network.
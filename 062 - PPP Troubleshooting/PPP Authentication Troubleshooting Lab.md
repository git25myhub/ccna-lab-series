# PPP Authentication Troubleshooting Lab

## 📌 Lab Overview

In this lab, PPP connections between the customer routers and service provider routers have already been configured according to the network diagram.

However, **one misconfiguration has been intentionally introduced on each networking device**. These errors prevent or affect PPP connectivity.

The objective is to:

1. Troubleshoot the PPP connections.
2. Identify the misconfiguration on each router.
3. Correct the configuration.
4. Verify that all PPP serial links are operational.
5. Confirm connectivity to the corresponding service provider routers.

> **Note:** SPR1, SPR2, SPR3, and SPR4 are pre-configured and should **not** be modified.

---

# 🎯 Lab Objectives

By completing this lab, you will practice:

- Troubleshooting PPP serial connections.
- Identifying PPP authentication problems.
- Troubleshooting PAP authentication.
- Troubleshooting CHAP authentication.
- Comparing interface status and protocol status.
- Using `show running-config` to locate configuration errors.
- Correcting PPP authentication configuration.
- Verifying connectivity with `ping`.
- Saving corrected configurations.

---

# 🗺️ Network Addressing

| Router | Interface | IP Address | Service Provider |
|---|---|---|---|
| R1 | Serial0/0 | 100.0.0.2/30 | SPR1 – 100.0.0.1 |
| R2 | Serial0/0 | 200.0.0.2/30 | SPR2 – 200.0.0.1 |
| R3 | Serial0/0 | 130.0.0.2/30 | SPR3 – 130.0.0.1 |
| R4 | Serial0/0 | 140.0.0.2/30 | SPR4 – 140.0.0.1 |

---

# 🔍 Troubleshooting Methodology

The troubleshooting process followed these steps:

1. Check the affected serial interface.
2. Check the IP addressing.
3. Verify PPP encapsulation.
4. Check the PPP authentication method.
5. Check local usernames/passwords.
6. Look for unnecessary or conflicting PPP authentication commands.
7. Reset the interface after making corrections.
8. Verify that the interface reaches `up/up`.
9. Test connectivity with `ping`.
10. Save the configuration.

Useful commands include:

```cisco
show interface serial0/0
show ip interface brief
show running-config
ping <peer-ip-address>
```

---

# 🔴 Device 1 — R1

## Initial Problem

R1's Serial0/0 interface showed:

```text
Serial0/0 is up, line protocol is down (disabled)
```

The interface was already using PPP:

```text
Encapsulation PPP
LCP Closed
```

The configuration showed:

```cisco
interface Serial0/0
 ip address 100.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 clock rate 2000000
```

The problem was that R1 was configured for PAP authentication but was **not sending the credentials required by SPR1**.

The existing local account was:

```cisco
username Packet password 0 Tracer
```

However, R1 needed to send:

```text
Username: Cisco
Password: CCNA
```

---

## Fix

Enter Serial0/0 configuration mode:

```cisco
R1#configure terminal
R1(config)#interface serial0/0
```

Configure the PAP credentials that R1 sends to SPR1:

```cisco
R1(config-if)#ppp pap sent-username Cisco password CCNA
```

Reset the interface:

```cisco
R1(config-if)#shutdown
R1(config-if)#no shutdown
```

---

## Verification

Check the interface:

```cisco
R1#show ip interface brief
```

Result:

```text
Serial0/0    100.0.0.2    YES manual    up    up
```

Test the connection:

```cisco
R1#ping 100.0.0.1
```

Result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

### R1 Result

✅ PPP PAP authentication restored.

---

# 🟠 Device 2 — R2

## Initial Problem

R2's Serial0/0 interface was configured for CHAP:

```cisco
interface Serial0/0
 ip address 200.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication chap
```

However, the local username was incorrectly configured as:

```cisco
username R2 password 0 CCNA
```

For CHAP authentication, R2 needed credentials corresponding to the service provider router:

```text
Username: SPR2
Password: CCNA
```

Therefore, the incorrect username `R2` was the misconfiguration.

---

## Fix

Remove the incorrect username:

```cisco
R2(config)#no username R2 password CCNA
```

Create the correct CHAP username:

```cisco
R2(config)#username SPR2 password CCNA
```

The final relevant configuration is:

```cisco
username SPR2 password 0 CCNA

interface Serial0/0
 ip address 200.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication chap
```

Reset the interface:

```cisco
R2(config-if)#shutdown
R2(config-if)#no shutdown
```

---

## Verification

Check the interface:

```cisco
R2#show ip interface brief
```

Successful result:

```text
Serial0/0    200.0.0.2    YES manual    up    up
```

Test SPR2:

```cisco
R2#ping 200.0.0.1
```

Result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

R2 also successfully reached R1:

```cisco
R2#ping 100.0.0.2
```

Result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

### R2 Result

✅ CHAP authentication restored.

---

# 🟡 Device 3 — R3

## Initial Problem

R3's configuration contained:

```cisco
username CCNA password 0 CCENT
```

and:

```cisco
interface Serial0/0
 ip address 130.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication chap
 ppp pap sent-username Cisco password 0 CCNP
 no keepalive
```

There were two authentication-related configurations present:

- CHAP authentication
- PAP sent-username configuration

The service provider connection required **PAP**, so the incorrect CHAP configuration was causing the PPP authentication setup to be inconsistent with the expected configuration.

---

## Fix

Remove CHAP authentication:

```cisco
R3(config)#interface serial0/0
R3(config-if)#no ppp authentication chap
```

Configure PAP:

```cisco
R3(config-if)#ppp authentication pap
```

The corrected authentication configuration becomes:

```cisco
interface Serial0/0
 ip address 130.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username Cisco password CCNP
 no keepalive
```

Reset the interface:

```cisco
R3(config-if)#shutdown
R3(config-if)#no shutdown
```

---

## Verification

Check the interface:

```cisco
R3#show ip interface brief
```

Result:

```text
Serial0/0    130.0.0.2    YES manual    up    up
```

Test SPR3:

```cisco
R3#ping 130.0.0.1
```

Result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

### R3 Result

✅ PPP authentication corrected and connectivity restored.

---

# 🟢 Device 4 — R4

## Initial Problem

R4's Serial0/0 interface was initially operational:

```text
Serial0/0 is up, line protocol is up (connected)
Encapsulation PPP
LCP Open
Open: IPCP, CDPCP
```

However, R4's configuration did not contain PPP authentication:

```cisco
interface Serial0/0
 ip address 140.0.0.2 255.255.255.252
 encapsulation ppp
```

The service provider connection required CHAP authentication.

R4 already had the required local credentials:

```cisco
username SPR4 password 0 CCIE
```

The missing configuration was:

```cisco
ppp authentication chap
```

---

## Fix

Enter Serial0/0 configuration mode:

```cisco
R4#configure terminal
R4(config)#interface serial0/0
```

Enable CHAP authentication:

```cisco
R4(config-if)#ppp authentication chap
```

Reset the interface:

```cisco
R4(config-if)#shutdown
R4(config-if)#no shutdown
```

---

## Verification

Check the interface:

```cisco
R4#show ip interface brief
```

Expected:

```text
Serial0/0    140.0.0.2    YES manual    up    up
```

Test SPR4:

```cisco
R4#ping 140.0.0.1
```

Result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

### R4 Result

✅ PPP CHAP authentication restored.

---

# 🧩 Summary of Misconfigurations

| Device | Problem | Correction | Result |
|---|---|---|---|
| **R1** | Missing PAP sent credentials | Added `ppp pap sent-username Cisco password CCNA` | ✅ Fixed |
| **R2** | Incorrect CHAP username `R2` | Changed username to `SPR2` | ✅ Fixed |
| **R3** | Incorrect CHAP authentication | Changed CHAP to PAP | ✅ Fixed |
| **R4** | Missing CHAP authentication | Added `ppp authentication chap` | ✅ Fixed |

---

# 🔧 Final Relevant Configurations

## R1

```cisco
username Packet password 0 Tracer

interface Serial0/0
 ip address 100.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username Cisco password 0 CCNA
```

## R2

```cisco
username SPR2 password 0 CCNA

interface Serial0/0
 ip address 200.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication chap
```

## R3

```cisco
username CCNA password 0 CCENT

interface Serial0/0
 ip address 130.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username Cisco password 0 CCNP
 no keepalive
```

## R4

```cisco
username SPR4 password 0 CCIE

interface Serial0/0
 ip address 140.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication chap
```

---

# 🧪 Final Verification

All four customer routers should have their Serial0/0 interfaces in an operational state:

```text
R1  Serial0/0  → 100.0.0.2  → up/up
R2  Serial0/0  → 200.0.0.2  → up/up
R3  Serial0/0  → 130.0.0.2  → up/up
R4  Serial0/0  → 140.0.0.2  → up/up
```

Connectivity tests:

```cisco
R1#ping 100.0.0.1
R2#ping 200.0.0.1
R3#ping 130.0.0.1
R4#ping 140.0.0.1
```

All tests returned:

```text
!!!!!
Success rate is 100 percent (5/5)
```

---

# 💾 Save the Configurations

After confirming connectivity, save the corrected configurations.

On each router:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

---

# 📚 Key Troubleshooting Lessons

### 1. `up/down` is an important clue

When a serial interface is physically up but the protocol is down:

```text
Status: up
Protocol: down
```

the physical connection is generally working, but a Layer 2 problem such as **PPP negotiation or authentication** may be preventing the protocol from coming up.

### 2. Check PPP encapsulation

Use:

```cisco
show interface serial0/0
```

Look for:

```text
Encapsulation PPP
```

Both ends of a PPP link must use PPP encapsulation.

### 3. Check authentication configuration

Use:

```cisco
show running-config
```

Look for commands such as:

```cisco
ppp authentication pap
ppp authentication chap
ppp pap sent-username ...
```

An incorrect authentication method can prevent the PPP session from establishing.

### 4. CHAP usernames matter

With CHAP, the username configured on the router must correspond to the identity expected during authentication.

For example:

```cisco
username SPR2 password CCNA
```

was required on R2.

### 5. Reset PPP interfaces after changes

After changing authentication settings, resetting the interface forces PPP negotiation to start again:

```cisco
shutdown
no shutdown
```

---

# ✅ Lab Completion Checklist

- [x] R1 PPP connection restored.
- [x] R1 PAP authentication corrected.
- [x] R1 PAP sent username/password configured.
- [x] R2 PPP connection restored.
- [x] R2 CHAP username corrected.
- [x] R3 PPP authentication corrected.
- [x] R3 PAP authentication configured.
- [x] R4 CHAP authentication configured.
- [x] All four Serial0/0 interfaces verified as `up/up`.
- [x] R1 successfully pinged SPR1.
- [x] R2 successfully pinged SPR2.
- [x] R3 successfully pinged SPR3.
- [x] R4 successfully pinged SPR4.
- [x] Configurations saved.
- [x] SPR1–SPR4 were not modified.

---

# 🏁 Conclusion

The lab was successfully completed by identifying and correcting one PPP-related misconfiguration on each customer router.

The troubleshooting demonstrated the importance of checking **interface state, PPP encapsulation, authentication method, usernames, and authentication credentials** when diagnosing serial PPP connectivity problems.

After the corrections, all four PPP connections were operational and the routers successfully communicated with their respective service provider routers.
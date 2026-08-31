# PPP PAP and CHAP Authentication Lab

## 📌 Lab Overview

This lab focuses on configuring **PPP (Point-to-Point Protocol)** authentication between routers using two different authentication methods:

- **PAP (Password Authentication Protocol)** between **R1 and SPR1**
- **CHAP (Challenge Handshake Authentication Protocol)** between **R2 and SPR2**

The authentication configuration is performed only on **R1 and R2**, because **SPR1 and SPR2 are already pre-configured**.

The objective is to establish authenticated PPP serial links and verify connectivity between the routers.

---

## 🎯 Lab Objectives

By completing this lab, you will:

1. Configure PPP encapsulation on a serial interface.
2. Configure **two-way PAP authentication** between R1 and SPR1.
3. Configure local username/password credentials on R1.
4. Configure R1 with the credentials it must send to SPR1.
5. Configure **two-way CHAP authentication** between R2 and SPR2.
6. Configure the required CHAP username/password on R2.
7. Verify that the PPP links successfully authenticate.
8. Test connectivity using `ping`.

---

# 🗺️ Network Information

| Router | Interface | IP Address | Peer |
|---|---|---|---|
| R1 | Serial0/0 | 100.0.0.2/30 | SPR1 – 100.0.0.1 |
| R2 | Serial0/0 | 200.0.0.2/30 | SPR2 – 200.0.0.1 |

### Authentication Requirements

| Link | Authentication | Credentials |
|---|---|---|
| R1 ↔ SPR1 | PAP | R1 local user: `Packet / Tracer` |
| R1 → SPR1 | PAP Sent Credentials | `Cisco / CCNA` |
| R2 ↔ SPR2 | CHAP | Username: `SPR2`, Password: `CCNA` |

---

# 🔐 Part 1 — Configure PPP PAP on R1

## Step 1: Check the Serial Interface

Initially, R1's Serial0/0 interface was configured with HDLC encapsulation and the line protocol was down.

```cisco
R1#show interface serial0/0
```

Important initial information:

```text
Serial0/0 is up, line protocol is down (disabled)
Internet address is 100.0.0.2/30
Encapsulation HDLC
```

A ping to the peer initially failed:

```text
R1#ping 100.0.0.1

Success rate is 0 percent (0/5)
```

This is expected because the serial connection was not yet configured for the required PPP authentication.

---

## Step 2: Change Encapsulation to PPP

Enter Serial0/0 configuration mode and change the encapsulation from HDLC to PPP.

```cisco
R1#configure terminal
R1(config)#interface serial0/0
R1(config-if)#encapsulation ppp
```

After changing the encapsulation, the line protocol came up:

```text
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial0/0, changed state to up
```

Verify the interface:

```cisco
R1#show ip interface brief
```

Expected result:

```text
Serial0/0    100.0.0.2    YES manual    up    up
```

---

## Step 3: Create the Local PAP User Account

The lab requires R1 to have the following local account:

- Username: `Packet`
- Password: `Tracer`

Configure it with:

```cisco
R1(config)#username Packet password Tracer
```

This local account is used to authenticate the remote peer when PAP authentication is performed.

---

## Step 4: Enable PAP Authentication

Configure PAP authentication on R1's Serial0/0 interface:

```cisco
R1(config)#interface serial0/0
R1(config-if)#ppp authentication pap
```

At this point, the interface may temporarily go down because PPP authentication is being negotiated.

---

## Step 5: Configure R1's PAP Sent Credentials

The lab specifically requires R1 to send the following credentials to SPR1:

- Sent username: `Cisco`
- Sent password: `CCNA`

Configure:

```cisco
R1(config-if)#ppp pap sent-username Cisco password CCNA
```

The final PAP configuration on R1 is therefore:

```cisco
interface Serial0/0
 ip address 100.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username Cisco password CCNA
```

---

## Step 6: Reset the Interface

After configuring PPP authentication, reset the interface so that PPP authentication can renegotiate.

```cisco
R1(config-if)#shutdown
R1(config-if)#no shutdown
```

Expected messages:

```text
%LINK-5-CHANGED: Interface Serial0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial0/0, changed state to up
```

---

## Step 7: Verify R1 PAP Connectivity

Check the interface:

```cisco
R1#show ip interface brief
```

Expected:

```text
Serial0/0    100.0.0.2    YES manual    up    up
```

Test connectivity to SPR1:

```cisco
R1#ping 100.0.0.1
```

Successful result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

This confirms that the PPP PAP authentication between R1 and SPR1 is working.

---

# 🔑 Part 2 — Configure PPP CHAP on R2

## Step 1: Check the R2 Serial Interface

Initially, R2's Serial0/0 interface was using HDLC:

```text
Serial0/0 is up, line protocol is down (disabled)
Internet address is 200.0.0.2/30
Encapsulation HDLC
```

A ping to the peer initially failed:

```cisco
R2#ping 200.0.0.1
```

Result:

```text
Success rate is 0 percent (0/5)
```

---

## Step 2: Create the CHAP Username and Password

For CHAP authentication, R2 needs a local username matching the hostname/identity expected from SPR2.

Configure:

```cisco
R2(config)#username SPR2 password CCNA
```

The credentials are:

```text
Username: SPR2
Password: CCNA
```

---

## Step 3: Change Encapsulation to PPP

Configure PPP encapsulation on Serial0/0:

```cisco
R2(config)#interface serial0/0
R2(config-if)#encapsulation ppp
```

---

## Step 4: Enable CHAP Authentication

Enable CHAP on the serial interface:

```cisco
R2(config-if)#ppp authentication chap
```

Unlike PAP, CHAP uses a challenge-response process rather than sending the password directly across the link.

---

## Step 5: Reset the Interface

Reset Serial0/0 to initiate a fresh PPP/CHAP negotiation:

```cisco
R2(config-if)#shutdown
R2(config-if)#no shutdown
```

The interface should return to:

```text
Serial0/0    200.0.0.2    YES manual    up    up
```

---

# 🧪 Verification

## Verify R1

Run:

```cisco
R1#show ip interface brief
```

Expected:

```text
Serial0/0    100.0.0.2    YES manual    up    up
```

Test the PAP peer:

```cisco
R1#ping 100.0.0.1
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

---

## Verify R2

Run:

```cisco
R2#show ip interface brief
```

Expected:

```text
Serial0/0    200.0.0.2    YES manual    up    up
```

Test the CHAP peer:

```cisco
R2#ping 200.0.0.1
```

Expected:

```text
!!!!!
Success rate is 100 percent (5/5)
```

R2 was also able to successfully ping R1:

```cisco
R2#ping 100.0.0.2
```

Result:

```text
!!!!!
Success rate is 100 percent (5/5)
```

---

# 🔎 Final Configurations

## R1

```cisco
!
hostname R1
!
username Packet password 0 Tracer
!
interface Serial0/0
 ip address 100.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication pap
 ppp pap sent-username Cisco password 0 CCNA
!
ip route 0.0.0.0 0.0.0.0 100.0.0.1
!
```

### R1 PAP Configuration Explained

```cisco
username Packet password Tracer
```

Creates the local username/password used for PAP authentication.

```cisco
encapsulation ppp
```

Changes the serial link from HDLC to PPP.

```cisco
ppp authentication pap
```

Enables PAP authentication.

```cisco
ppp pap sent-username Cisco password CCNA
```

Specifies the username and password that R1 sends to SPR1 during PAP authentication.

---

## R2

```cisco
!
hostname R2
!
username SPR2 password 0 CCNA
!
interface Serial0/0
 ip address 200.0.0.2 255.255.255.252
 encapsulation ppp
 ppp authentication chap
!
ip route 0.0.0.0 0.0.0.0 200.0.0.1
!
```

### R2 CHAP Configuration Explained

```cisco
username SPR2 password CCNA
```

Creates the local credentials required for CHAP authentication.

```cisco
encapsulation ppp
```

Enables PPP encapsulation.

```cisco
ppp authentication chap
```

Enables CHAP authentication on the serial interface.

---

# 📊 PAP vs CHAP

| Feature | PAP | CHAP |
|---|---|---|
| Authentication type | Two-way authentication | Two-way authentication |
| Password transmission | Sent during authentication | Not sent directly |
| Security | Less secure | More secure |
| Challenge-response | No | Yes |
| Configuration | `ppp authentication pap` | `ppp authentication chap` |
| Example | R1 ↔ SPR1 | R2 ↔ SPR2 |

### PAP

PAP uses a simple username/password exchange. In this lab, R1 sends:

```text
Username: Cisco
Password: CCNA
```

R1 also contains the local account:

```text
Username: Packet
Password: Tracer
```

### CHAP

CHAP uses a challenge-response mechanism. The password is used to calculate a response rather than being transmitted directly across the link.

R2 therefore requires:

```text
Username: SPR2
Password: CCNA
```

---

# 💾 Save the Configuration

After successful verification, save the configurations:

### R1

```cisco
R1#copy running-config startup-config
```

or:

```cisco
R1#write memory
```

### R2

```cisco
R2#copy running-config startup-config
```

or:

```cisco
R2#write memory
```

---

# ✅ Lab Completion Checklist

- [x] PPP encapsulation configured on R1.
- [x] PAP authentication enabled on R1.
- [x] `Packet / Tracer` local user configured on R1.
- [x] R1 configured to send `Cisco / CCNA` to SPR1.
- [x] R1 Serial0/0 reached an `up/up` state.
- [x] R1 successfully pinged `100.0.0.1`.
- [x] PPP encapsulation configured on R2.
- [x] CHAP authentication enabled on R2.
- [x] `SPR2 / CCNA` credentials configured on R2.
- [x] R2 Serial0/0 reached an `up/up` state.
- [x] R2 successfully pinged `200.0.0.1`.
- [x] R2 successfully pinged `100.0.0.2`.
- [x] Configurations saved.

---

# 🏁 Conclusion

The lab was successfully completed by configuring **PPP authentication using both PAP and CHAP**.

**R1** was configured for **two-way PAP authentication** with SPR1. A local `Packet / Tracer` account was created, while R1 was configured to send `Cisco / CCNA` credentials to the pre-configured SPR1 router.

**R2** was configured for **two-way CHAP authentication** with SPR2 using the `SPR2 / CCNA` credentials.

Successful ping tests and `up/up` serial interfaces confirmed that PPP encapsulation and authentication were functioning correctly.
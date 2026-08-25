# Cisco IOS Console Password and Password Encryption Lab

## 📌 Lab Overview

This lab demonstrates how to configure basic security on a Cisco router using a **console connection**. You will configure the router hostname, enable secret, console password, and password encryption, then verify and save the configuration.

### 🎯 Objectives

By completing this lab, you will:

- Connect to a Cisco router using a PC console connection.
- Configure the router hostname.
- Configure an `enable secret`.
- Configure and secure the console line with a password.
- Verify the running configuration.
- Understand the difference between encrypted and unencrypted passwords.
- Enable password encryption using `service password-encryption`.
- Save the router configuration.

---

## 🖥️ Topology

```text
PC1
 |
 | RS-232 / Console Cable
 |
Console Port
    |
   R1
```

### Devices

| Device | Role |
|---|---|
| PC1 | Console terminal |
| R1 | Cisco Router |

---

# 1. Connect PC1 to R1

Connect:

```text
PC1 RS-232 Port → R1 Console Port
```

Open **PC1 → Desktop → Terminal** in Packet Tracer.

Use the default console settings:

```text
Bits Per Second: 9600
Data Bits: 8
Parity: None
Stop Bits: 1
Flow Control: None
```

Press **Enter** to access R1.

---

# 2. Configure the Router Hostname

Enter privileged EXEC mode and global configuration mode:

```cisco
Router> enable
Router# configure terminal
```

Change the hostname:

```cisco
Router(config)# hostname R1
```

The prompt should now show:

```text
R1(config)#
```

---

# 3. Configure the Enable Secret

Configure `cisco` as the enable secret:

```cisco
R1(config)# enable secret cisco
```

The enable secret is stored in the configuration in a hashed/encrypted form.

Verify:

```cisco
R1(config)# do show running-config
```

You should see something similar to:

```text
enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
```

> **Note:** The exact hash will be different on different devices.

---

# 4. Configure the Console Password

Enter console line configuration mode:

```cisco
R1(config)# line console 0
```

Configure the console password:

```cisco
R1(config-line)# password ccna
```

Make the router require the password when connecting through the console:

```cisco
R1(config-line)# login
```

Exit configuration mode:

```cisco
R1(config-line)# end
```

---

# 5. Verify the Running Configuration

Display the running configuration:

```cisco
R1# show running-config
```

Before enabling password encryption, the console section should look similar to:

```text
line con 0
 password ccna
 login
```

### ❓ Is the console password encrypted?

**No.**

At this point, the console password appears as plain text:

```text
password ccna
```

This demonstrates that configuring a console password alone does **not** hide the password in the running configuration.

---

# 6. Enable Password Encryption

Enter global configuration mode:

```cisco
R1# configure terminal
```

Enable password encryption:

```cisco
R1(config)# service password-encryption
```

Exit configuration mode:

```cisco
R1(config)# end
```

---

# 7. Verify Password Encryption

Check the running configuration again:

```cisco
R1# show running-config
```

The console configuration should now look similar to:

```text
line con 0
 password 7 0822455D0A16
 login
```

The exact encrypted value may differ.

The important change is that:

### Before

```text
password ccna
```

### After

```text
password 7 0822455D0A16
```

The console password is no longer displayed as plain text.

> **Important:** `service password-encryption` uses Cisco's Type 7 password obfuscation. It is not considered strong cryptographic protection. The `enable secret` provides stronger protection than a Type 7 line password.

---

# 8. Save the Configuration

Save the running configuration to NVRAM:

```cisco
R1# copy running-config startup-config
```

When prompted:

```text
Destination filename [startup-config]?
```

Press **Enter**.

Alternatively:

```cisco
R1# write memory
```

Expected result:

```text
Building configuration...
[OK]
```

---

# 9. Test Console Authentication

Exit the console session:

```cisco
R1# exit
```

You should see:

```text
R1 con0 is now available

Press RETURN to get started.
```

Press **Enter**.

The router should request the console password:

```text
User Access Verification

Password:
```

Enter:

```text
ccna
```

You should reach:

```text
R1>
```

Test privileged EXEC mode:

```cisco
R1> enable
Password:
```

Enter:

```text
cisco
```

You should reach:

```text
R1#
```

---

# 10. Verification Commands

Use the following commands to verify the configuration:

### Display running configuration

```cisco
R1# show running-config
```

### Display startup configuration

```cisco
R1# show startup-config
```

### Verify the hostname

```cisco
R1# show running-config | include hostname
```

Expected:

```text
hostname R1
```

### Verify the enable secret

```cisco
R1# show running-config | include enable secret
```

Expected:

```text
enable secret 5 ...
```

### Verify password encryption

```cisco
R1# show running-config | include service password-encryption
```

Expected:

```text
service password-encryption
```

### Verify the console configuration

```cisco
R1# show running-config | section line con
```

Expected:

```text
line con 0
 password 7 ...
 login
```

---

# 11. Final Configuration

The important portions of the final configuration should resemble:

```cisco
!
version 15.1
service password-encryption
!
hostname R1
!
enable secret 5 <hashed-secret>
!
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
!
no cdp run
!
line con 0
 password 7 <encrypted-password>
 login
!
line aux 0
!
line vty 0 4
 login
!
end
```

---

# 🧠 Key Concepts

| Configuration | Purpose |
|---|---|
| `hostname R1` | Changes the router's hostname |
| `enable secret cisco` | Secures privileged EXEC mode |
| `line console 0` | Enters console-line configuration |
| `password ccna` | Sets the console password |
| `login` | Requires the configured console password |
| `service password-encryption` | Obfuscates plaintext line passwords |
| `show running-config` | Displays the active configuration |
| `copy running-config startup-config` | Saves configuration to NVRAM |

---

# ❓ Lab Questions

### 1. What password is used to access the console?

```text
ccna
```

### 2. What password is used to enter privileged EXEC mode?

```text
cisco
```

### 3. Was the console password encrypted before enabling `service password-encryption`?

**No.** It appeared as:

```text
password ccna
```

### 4. What command encrypts plaintext passwords in the configuration?

```cisco
service password-encryption
```

### 5. What type of encryption is used for the console password?

Cisco **Type 7** password obfuscation.

### 6. Is the `enable secret` displayed in plain text?

**No.** It is stored as a hashed secret.

### 7. Why is `login` required under `line console 0`?

The `login` command tells IOS to require the configured console password when someone connects through the console.

### 8. Why should the configuration be saved?

Saving the configuration copies it from **RAM (running configuration)** to **NVRAM (startup configuration)** so it can be restored after a router reload.

---

# ✅ Lab Completion Checklist

- [x] Connected PC1 RS-232 to R1 console.
- [x] Accessed R1 through the console.
- [x] Changed hostname to `R1`.
- [x] Configured `enable secret cisco`.
- [x] Configured console password `ccna`.
- [x] Enabled console authentication with `login`.
- [x] Verified the running configuration.
- [x] Confirmed the console password was initially visible in plaintext.
- [x] Enabled `service password-encryption`.
- [x] Verified the console password was obfuscated.
- [x] Tested console authentication.
- [x] Tested privileged EXEC authentication.
- [x] Saved the configuration to NVRAM.

---

## 🏁 Final Result

R1 is now configured with basic console and privileged-mode security:

```text
Hostname:              R1
Enable Secret:         cisco
Console Password:      ccna
Console Login:         Enabled
Password Encryption:   Enabled
Configuration:         Saved
```
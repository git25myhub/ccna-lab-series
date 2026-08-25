# Cisco IOS Enable Password Encryption Lab

## 📘 Lab Overview

This lab demonstrates how Cisco IOS handles the **privileged EXEC (`enable`) password** and how the `service password-encryption` command affects passwords stored in the running configuration.

You will:

- Connect two routers using their GigabitEthernet interfaces.
- Configure router hostnames.
- Configure an enable password.
- Examine how the password appears in the running configuration.
- Enable password encryption.
- Verify the encrypted password.
- Disable password encryption and observe what happens to the existing password.

---

## 🖥️ Topology

```text
        GigabitEthernet0/0
   R1 ---------------------- R2
        GigabitEthernet0/0
```

### Devices

| Device | Interface | Connection |
|---|---|---|
| R1 | G0/0 | R2 G0/0 |
| R2 | G0/0 | R1 G0/0 |

### Credentials

| Setting | Value |
|---|---|
| Enable password | `cisco` |
| Router 1 hostname | `R1` |
| Router 2 hostname | `R2` |

---

# 🎯 Objectives

By completing this lab, you should understand:

1. How to configure a Cisco router hostname.
2. How to configure an enable password.
3. How an unencrypted enable password appears in `show running-config`.
4. What `service password-encryption` does.
5. How Cisco IOS Type 7 password encryption appears in the configuration.
6. What happens when password encryption is disabled.

---

# 📝 Lab Tasks

## 1. Connect R1 and R2

Connect:

```text
R1 GigabitEthernet0/0
        |
        |
R2 GigabitEthernet0/0
```

For this lab, the physical connection is the focus; no IP addressing is required.

---

## 2. Configure the Hostnames

Configure the routers according to the topology.

### R1

```cisco
enable
configure terminal
hostname R1
```

### R2

```cisco
enable
configure terminal
hostname R2
```

Verify:

```cisco
show running-config
```

The configuration should contain:

```text
hostname R1
```

or:

```text
hostname R2
```

---

# 🔐 3. Configure the Enable Password

Configure the enable password as:

```text
cisco
```

### R1

```cisco
R1# configure terminal
R1(config)# enable password cisco
```

### R2

```cisco
R2# configure terminal
R2(config)# enable password cisco
```

---

# 🔎 4. View the Password in the Running Configuration

Use:

```cisco
do show running-config
```

or exit configuration mode and use:

```cisco
show running-config
```

You should see something similar to:

```text
!
no service password-encryption
!
hostname R2
!
enable password cisco
!
```

### ❓ Question

**Is the password encrypted?**

**Answer: No.**

The password is displayed in plain text:

```text
enable password cisco
```

This demonstrates that the traditional `enable password` command does **not** automatically encrypt the password.

---

# 🔒 5. Enable Password Encryption

Enter global configuration mode and enable password encryption:

```cisco
R2(config)# service password-encryption
```

Do the same on R1:

```cisco
R1(config)# service password-encryption
```

The command enables Cisco IOS password encryption for applicable plaintext passwords stored in the configuration.

---

# 🔎 6. View the Running Configuration Again

Run:

```cisco
do show running-config
```

You should now see something similar to:

```text
!
service password-encryption
!
hostname R2
!
enable password 7 0822455D0A16
!
```

The exact encrypted value may differ depending on the password and IOS implementation.

### ❓ Question

**Is the password encrypted?**

**Answer: Yes.**

The password is now displayed as:

```text
enable password 7 0822455D0A16
```

The `7` indicates **Cisco Type 7 password obfuscation**.

---

# 🔓 7. Disable Password Encryption

Disable the service:

```cisco
R2(config)# no service password-encryption
```

Do the same on R1:

```cisco
R1(config)# no service password-encryption
```

---

# 🔎 8. View the Running Configuration

Run:

```cisco
do show running-config
```

You may see:

```text
!
no service password-encryption
!
hostname R2
!
enable password 7 0822455D0A16
!
```

### ❓ Question

**Is the password still encrypted?**

**Answer: Yes.**

This is an important observation from the lab.

Disabling:

```cisco
no service password-encryption
```

does **not decrypt passwords that have already been encrypted**.

Instead, it prevents future applicable plaintext passwords from automatically being encrypted.

Therefore, the existing password remains:

```text
enable password 7 0822455D0A16
```

---

# 💻 Complete Configuration Example

## R1

```cisco
enable
configure terminal
hostname R1
enable password cisco
service password-encryption
end
show running-config
```

To disable password encryption:

```cisco
configure terminal
no service password-encryption
end
show running-config
```

---

## R2

```cisco
enable
configure terminal
hostname R2
enable password cisco
service password-encryption
end
show running-config
```

To disable password encryption:

```cisco
configure terminal
no service password-encryption
end
show running-config
```

---

# 📊 Expected Results

| Stage | Configuration | Password Appearance |
|---|---|---|
| Before encryption | `no service password-encryption` | `enable password cisco` |
| After encryption enabled | `service password-encryption` | `enable password 7 ...` |
| After encryption disabled | `no service password-encryption` | `enable password 7 ...` |

---

# 🧠 Key Concepts

## `enable password`

The command:

```cisco
enable password cisco
```

sets the password used to enter privileged EXEC mode.

However, by itself, it stores the password in plaintext in the configuration.

---

## `service password-encryption`

The command:

```cisco
service password-encryption
```

causes applicable plaintext passwords in the configuration to be stored using Cisco's **Type 7** password obfuscation.

Example:

```text
enable password 7 0822455D0A16
```

---

## `no service password-encryption`

The command:

```cisco
no service password-encryption
```

disables the automatic encryption service.

It **does not reverse existing Type 7 encrypted passwords**.

This explains the important result observed in the lab:

```text
no service password-encryption
!
enable password 7 0822455D0A16
```

The password remains encrypted/obfuscated because it was already converted to Type 7.

---

# ⚠️ Security Note

Cisco Type 7 is **not considered strong password protection**. It is reversible obfuscation rather than modern cryptographic password hashing.

For real Cisco configurations, prefer:

```cisco
enable secret cisco
```

instead of:

```cisco
enable password cisco
```

`enable secret` provides substantially stronger protection and is the recommended method for securing privileged EXEC access.

---

# 🧪 Verification Commands

Use the following commands throughout the lab:

```cisco
show running-config
```

From configuration mode:

```cisco
do show running-config
```

Check the password-encryption status with:

```cisco
show running-config | include password
```

You can also check for the service configuration:

```cisco
show running-config | include service
```

---

# ❓ Lab Questions and Answers

### 1. Is the enable password encrypted immediately after using `enable password cisco`?

**No.**

It appears as:

```text
enable password cisco
```

---

### 2. What happens after entering `service password-encryption`?

The applicable plaintext password is converted to Cisco Type 7 format:

```text
enable password 7 0822455D0A16
```

---

### 3. What does the number `7` represent?

It identifies the password as using **Cisco Type 7 obfuscation**.

---

### 4. What happens when `no service password-encryption` is entered?

Password encryption is disabled for future applicable plaintext passwords, but passwords already converted to Type 7 remain in that form.

---

### 5. Does `no service password-encryption` decrypt the existing password?

**No.**

The existing Type 7 password remains:

```text
enable password 7 0822455D0A16
```

---

# ✅ Lab Completion Checklist

- [ ] R1 connected to R2 using G0/0
- [ ] R1 hostname configured
- [ ] R2 hostname configured
- [ ] Enable password `cisco` configured on R1
- [ ] Enable password `cisco` configured on R2
- [ ] Running configuration checked before encryption
- [ ] Confirmed password was initially visible in plaintext
- [ ] `service password-encryption` enabled
- [ ] Running configuration checked after encryption
- [ ] Confirmed Type 7 password representation
- [ ] `no service password-encryption` configured
- [ ] Running configuration checked again
- [ ] Confirmed existing Type 7 password remains encrypted/obfuscated
- [ ] Lab questions answered

---

# 🏁 Conclusion

This lab demonstrates the difference between a plaintext Cisco `enable password` and a password affected by the `service password-encryption` command.

The key lesson is that:

```cisco
enable password cisco
```

stores the password in plaintext unless password encryption is enabled.

After:

```cisco
service password-encryption
```

the password appears in Type 7 format.

Finally:

```cisco
no service password-encryption
```

does not convert an already encrypted password back to plaintext. The existing Type 7 value remains in the configuration.

For production environments, use **`enable secret` rather than `enable password`** for privileged EXEC authentication.
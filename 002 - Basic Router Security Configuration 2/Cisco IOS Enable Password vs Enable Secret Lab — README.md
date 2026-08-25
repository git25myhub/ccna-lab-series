# Cisco IOS Enable Password vs Enable Secret Lab

## 📘 Lab Overview

This lab demonstrates the difference between the Cisco IOS **`enable password`** and **`enable secret`** commands.

You will configure both passwords on R1 and R2, test privileged EXEC access, inspect how each password is stored in the running configuration, enable password encryption, save the configuration, and reload the routers to verify the configuration persists.

---

# 🖥️ Topology

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
| Enable secret | `ccna` |
| R1 hostname | `R1` |
| R2 hostname | `R2` |

---

# 🎯 Objectives

By completing this lab, you should understand:

- The difference between `enable password` and `enable secret`.
- Which credential takes precedence when both are configured.
- How `enable secret` is stored in the running configuration.
- How `service password-encryption` affects the enable password.
- How to save a Cisco configuration.
- How to reload a router and verify that the configuration persists.

---

# 📝 Lab Tasks

## 1. Connect the Two Routers

Connect the routers using their GigabitEthernet0/0 interfaces:

```text
R1 G0/0  ----------------  R2 G0/0
```

No IP addressing is required for this particular lab.

---

# 2. Configure the Hostnames

## R1

```cisco
R1# configure terminal
R1(config)# hostname R1
```

## R2

```cisco
R2# configure terminal
R2(config)# hostname R2
```

Verify with:

```cisco
show running-config
```

You should see:

```text
hostname R1
```

on R1 and:

```text
hostname R2
```

on R2.

---

# 🔐 3. Configure the Enable Password

Configure the enable password as:

```text
cisco
```

## R1

```cisco
R1(config)# enable password cisco
```

## R2

```cisco
R2(config)# enable password cisco
```

At this point, the routers have an enable password, but we will also configure an enable secret.

---

# 🔒 4. Configure the Enable Secret

Configure the enable secret as:

```text
ccna
```

## R1

```cisco
R1(config)# enable secret ccna
```

## R2

```cisco
R2(config)# enable secret ccna
```

The routers now have **both**:

```text
enable password cisco
enable secret ccna
```

---

# 🚪 5. Exit to EXEC Mode and Test Privileged EXEC Access

Exit configuration mode:

```cisco
R1(config)# exit
R1#
```

Then exit back to user EXEC mode:

```cisco
R1# exit
```

You should return to:

```text
R1>
```

Enter privileged EXEC mode:

```cisco
R1> enable
```

The router will request a password.

Enter:

```text
ccna
```

### ❓ Which password should you use?

**Answer: `ccna`**

When both `enable password` and `enable secret` are configured, the **enable secret takes precedence**.

Therefore:

```text
enable password cisco
enable secret ccna
```

requires:

```text
ccna
```

to enter privileged EXEC mode.

The same applies to R2.

---

# 🔎 6. View the Running Configuration

Run:

```cisco
R1# show running-config
```

You should see something similar to:

```text
!
hostname R1
!
enable secret 5 $1$mERr$Bok4KDfVutXOJolNq009M/
enable password cisco
!
```

### ❓ Which password is encrypted?

The **enable secret** is encrypted/hashed.

The configuration contains:

```text
enable secret 5 $1$mERr$Bok4KDfVutXOJolNq009M/
```

while the enable password may appear as:

```text
enable password cisco
```

if password encryption has not yet been enabled.

---

# 🔐 7. Enable Password Encryption

Enter global configuration mode:

```cisco
R1# configure terminal
```

Enable password encryption:

```cisco
R1(config)# service password-encryption
```

Do the same on R2:

```cisco
R2(config)# configure terminal
R2(config)# service password-encryption
```

---

# 🔎 View the Running Configuration Again

Run:

```cisco
R1# show running-config
```

The relevant section should now look similar to:

```text
!
service password-encryption
!
hostname R1
!
enable secret 5 $1$mERr$Bok4KDfVutXOJolNq009M/
enable password 7 0822455D0A16
!
```

### ❓ What changed?

The `enable password` has changed from plaintext:

```text
enable password cisco
```

to Type 7 format:

```text
enable password 7 0822455D0A16
```

The `enable secret` remains protected separately.

### Important

`service password-encryption` does **not** change the enable secret into Type 7.

It primarily affects passwords that would otherwise appear in plaintext in the configuration.

---

# 🧠 Understanding the Difference

The lab demonstrates an important Cisco IOS security concept.

### `enable password`

```cisco
enable password cisco
```

Without password encryption:

```text
enable password cisco
```

With:

```cisco
service password-encryption
```

it becomes something similar to:

```text
enable password 7 0822455D0A16
```

---

### `enable secret`

```cisco
enable secret ccna
```

The secret is stored in a protected form, for example:

```text
enable secret 5 $1$mERr$Bok4KDfVutXOJolNq009M/
```

The exact value will differ depending on the router and password.

---

# ⭐ Which Password Takes Precedence?

If both are configured:

```text
enable password cisco
enable secret ccna
```

the router uses:

```text
ccna
```

for privileged EXEC authentication.

Therefore:

| Password | Value | Used for `enable`? |
|---|---|---|
| Enable password | `cisco` | ❌ No |
| Enable secret | `ccna` | ✅ Yes |

This is one of the most important concepts demonstrated by the lab.

---

# 💾 8. Save the Configuration

Exit configuration mode:

```cisco
R1(config)# end
```

Save the configuration using:

```cisco
R1# write memory
```

or:

```cisco
R1# wr
```

You should see:

```text
Building configuration...

[OK]
```

Your lab output demonstrates this successfully:

```text
R1#wr

Building configuration...

[OK]
```

### Alternative Command

You can also use:

```cisco
R1# copy running-config startup-config
```

Then press **Enter** when prompted for the destination filename.

The same process should be performed on R2.

---

# 🔄 9. Reload the Router

To confirm that the configuration has been saved:

```cisco
R1# reload
```

The router will ask for confirmation.

Follow the prompts to reload.

After the router finishes booting, you should see:

```text
R1>
```

Enter privileged EXEC mode:

```cisco
R1> enable
```

Enter:

```text
ccna
```

You should successfully reach:

```text
R1#
```

This confirms that the enable secret was saved to startup configuration and survived the reload.

Repeat the process on R2.

---

# 🧪 Verification Commands

## Check the running configuration

```cisco
show running-config
```

## Check the startup configuration

```cisco
show startup-config
```

## Check the password-related configuration

```cisco
show running-config | include enable
```

Expected output after encryption:

```text
enable secret 5 $1$mERr$Bok4KDfVutXOJolNq009M/
enable password 7 0822455D0A16
```

## Check whether password encryption is enabled

```cisco
show running-config | include service
```

Expected:

```text
service password-encryption
```

---

# 📊 Expected Results

| Stage | Enable Password | Enable Secret |
|---|---|---|
| Configure password | `cisco` | — |
| Configure secret | `cisco` | `ccna` |
| Enter privileged EXEC | Not used | **`ccna`** |
| Before password encryption | Plaintext | Protected |
| After `service password-encryption` | Type 7 | Protected |
| After saving/reload | Persists | Persists |

---

# ❓ Lab Questions and Answers

### 1. Which password do you use to enter privileged EXEC mode?

**Answer: `ccna`**

The `enable secret` takes precedence over the `enable password`.

---

### 2. Which password is encrypted before enabling password encryption?

**Answer: The enable secret.**

The `enable password` can appear in plaintext.

---

### 3. What changes after entering `service password-encryption`?

The enable password is converted from plaintext:

```text
enable password cisco
```

to Type 7:

```text
enable password 7 0822455D0A16
```

---

### 4. Does `service password-encryption` encrypt the enable secret?

The enable secret is already stored in a protected form. The command primarily changes applicable plaintext passwords such as the traditional `enable password`.

---

### 5. What password is required after reloading the router?

**`ccna`**, because the enable secret takes precedence.

---

### 6. Why save the configuration before reloading?

Cisco IOS maintains separate **running** and **startup** configurations.

Saving the configuration copies the current configuration to NVRAM so that it can be restored after a reboot.

---

# ⚠️ Important Security Note

For real-world Cisco configurations, use:

```cisco
enable secret <password>
```

rather than relying on:

```cisco
enable password <password>
```

The traditional enable password is weaker, even when Type 7 password encryption is enabled.

Type 7 is designed primarily to prevent casual visibility in configuration output; it should **not** be treated as strong cryptographic protection.

---

# 🧰 Complete Configuration Example

## R1

```cisco
enable
configure terminal
hostname R1
enable password cisco
enable secret ccna
service password-encryption
end
write memory
```

## R2

```cisco
enable
configure terminal
hostname R2
enable password cisco
enable secret ccna
service password-encryption
end
write memory
```

---

# 🏁 Final Configuration Example

After completing the lab, the relevant portion of R1 should look similar to:

```text
!
service password-encryption
!
hostname R1
!
enable secret 5 $1$mERr$Bok4KDfVutXOJolNq009M/
enable password 7 0822455D0A16
!
```

The exact Type 5 hash and Type 7 value may vary.

---

# 📝 Key Takeaways

1. `enable password` configures a traditional privileged EXEC password.
2. `enable secret` provides the preferred privileged EXEC password mechanism.
3. When both exist, **`enable secret` takes precedence**.
4. `service password-encryption` converts applicable plaintext passwords to Type 7.
5. The enable secret remains protected independently of Type 7 password encryption.
6. `write memory` saves the running configuration to startup configuration.
7. After a reload, the saved enable secret can still be used to enter privileged EXEC mode.

---

# ✅ Lab Completion Checklist

- [ ] R1 connected to R2 using G0/0
- [ ] R1 hostname configured
- [ ] R2 hostname configured
- [ ] Enable password `cisco` configured on both routers
- [ ] Enable secret `ccna` configured on both routers
- [ ] Tested privileged EXEC access
- [ ] Confirmed `ccna` is the required password
- [ ] Viewed the running configuration
- [ ] Identified the protected enable secret
- [ ] Enabled `service password-encryption`
- [ ] Verified the enable password changed to Type 7
- [ ] Saved the configuration
- [ ] Reloaded the router
- [ ] Confirmed the configuration persisted
- [ ] Successfully entered privileged EXEC mode after reload
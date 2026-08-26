# Cisco IOS Password Recovery, Configuration Backup & IOS Upgrade Lab

## 📌 Lab Overview

In this lab, you will perform three important Cisco router administration tasks on **R1**:

1. Recover access to R1 by entering **ROMMON mode** and changing the enable secret to `ccna`.
2. Back up R1's startup configuration to the **TFTP server TFTP1**.
3. Upgrade R1's Cisco IOS image using the image stored on **TFTP1**.

This lab demonstrates practical skills used when managing Cisco routers that have lost privileged EXEC access, protecting configurations through backups, and performing IOS software upgrades.

---

# 🎯 Learning Objectives

By completing this lab, you should be able to:

- Access a Cisco router's ROMMON mode.
- Understand the purpose of the configuration register.
- Perform password recovery without deleting the existing configuration.
- Restore the startup configuration into the running configuration.
- Change the router's enable secret.
- Restore the normal configuration-register value.
- Back up a router configuration to a TFTP server.
- Verify files stored in router flash.
- Copy an IOS image from a TFTP server to flash.
- Configure the router to boot using the upgraded IOS image.
- Verify the IOS version after an upgrade.

---

# 🧪 Lab Requirements

| Device | Purpose |
|---|---|
| **R1** | Cisco 2911 router |
| **TFTP1** | TFTP server |
| **PC/Console connection** | Used to access R1 |
| **IOS image** | `c2900-universalk9-mz.SPA.155-3.M4a.bin` |

The original IOS image observed on R1 was:

```text
c2900-universalk9-mz.SPA.151-4.M4.bin
```

The target IOS image is:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

---

# 1. 🔐 Perform Password Recovery on R1

The objective is to change the enable secret to:

```text
ccna
```

without losing the existing startup configuration.

## Enter ROMMON Mode

Packet Tracer requires a slightly different procedure from physical Cisco hardware.

### Packet Tracer Procedure

1. Open **R1**.
2. Select the **Physical** tab.
3. Locate the router's power switch.
4. Turn the router **off**.
5. Turn the router **back on**.
6. Switch to the **CLI** tab.
7. Immediately press:

```text
Ctrl + Break
```

before IOS finishes booting.

You should eventually see:

```text
rommon 1 >
```

This indicates that R1 has entered **ROMMON mode**.

---

# 2. Change the Configuration Register

At the ROMMON prompt, configure the router to ignore the startup configuration during the next boot:

```text
rommon 1 > confreg 0x2142
```

The value:

```text
0x2142
```

instructs the router to bypass the startup configuration when booting IOS.

Now restart the router:

```text
rommon 2 > reset
```

R1 should reboot.

---

# 3. Bypass the Startup Configuration

When IOS finishes booting, you should see the initial configuration dialog:

```text
Would you like to enter the initial configuration dialog? [yes/no]:
```

Enter:

```text
no
```

Press **Enter** when prompted:

```text
Press RETURN to get started!
```

You should now reach:

```text
Router>
```

Enter privileged EXEC mode:

```cisco
Router> enable
Router#
```

Because the configuration was bypassed, the router should not require the original enable secret.

---

# 4. Restore the Existing Configuration

The startup configuration is still stored in NVRAM.

Copy it into the running configuration:

```cisco
Router# copy startup-config running-config
```

When prompted:

```text
Destination filename [running-config]?
```

Press **Enter**.

Your output should be similar to:

```text
731 bytes copied in 0.416 secs (1757 bytes/sec)
```

The original configuration is now loaded into RAM.

### Important

Do **not** use:

```cisco
copy running-config startup-config
```

at this point.

That would overwrite the existing startup configuration with the temporary configuration.

The correct recovery procedure is:

```cisco
copy startup-config running-config
```

---

# 5. Change the Enable Secret

Enter global configuration mode:

```cisco
R1# configure terminal
```

Configure the new enable secret:

```cisco
R1(config)# enable secret ccna
```

The recovered router now uses:

```text
ccna
```

as the privileged EXEC password.

---

# 6. Restore the Normal Configuration Register

After password recovery, the configuration register must be restored so that R1 loads the startup configuration normally during future boots.

Configure:

```cisco
R1(config)# config-register 0x2102
```

The normal Cisco configuration-register value is:

```text
0x2102
```

Exit configuration mode:

```cisco
R1(config)# end
```

Save the configuration:

```cisco
R1# write memory
```

or:

```cisco
R1# copy running-config startup-config
```

---

# 7. Verify Password Recovery

Test privileged EXEC access.

Exit to user EXEC mode:

```cisco
R1# disable
R1> enable
Password:
```

Enter:

```text
ccna
```

You should successfully return to:

```text
R1#
```

You can also verify the configuration register with:

```cisco
R1# show version
```

Near the bottom of the output, look for something similar to:

```text
Configuration register is 0x2102
```

This confirms that normal boot behavior has been restored.

---

# 8. 🌐 Verify Connectivity to TFTP1

Before transferring files, R1 must be able to reach the TFTP server.

The TFTP server in this lab uses:

```text
192.168.1.100
```

From R1:

```cisco
R1# ping 192.168.1.100
```

A successful result should look similar to:

```text
!!!!!
Success rate is 100 percent (5/5)
```

## Troubleshooting

Your lab output showed:

```text
Success rate is 0 percent (0/5)
```

This means R1 could not reach TFTP1 at that point.

Before attempting a TFTP transfer, check:

```cisco
R1# show ip interface brief
```

Make sure the interface connected to the TFTP network is:

```text
up
up
```

If G0/0 is administratively down:

```cisco
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# no shutdown
R1(config-if)# end
```

Then test again:

```cisco
R1# ping 192.168.1.100
```

A TFTP transfer should not be attempted until basic IP connectivity works.

---

# 9. 💾 Back Up R1's Startup Configuration to TFTP1

Once connectivity to TFTP1 is working, back up the startup configuration.

Use:

```cisco
R1# copy startup-config tftp:
```

R1 will ask for the TFTP server address:

```text
Address or name of remote host []?
```

Enter:

```text
192.168.1.100
```

You will then be asked for the destination filename:

```text
Destination filename [R1-confg]?
```

You can accept the default by pressing **Enter**, or provide a descriptive filename such as:

```text
R1-startup-config
```

The transfer should finish with a message similar to:

```text
Writing startup-config....!!!!
[OK]
```

---

# 10. 🔍 Verify the Configuration Backup

On **TFTP1**, open the TFTP service and verify that the configuration file was received.

The backup should contain R1's startup configuration.

This provides a recovery copy that can be restored if the router's configuration is accidentally lost or corrupted.

---

# 11. 📦 Check the Existing IOS Image

Before upgrading IOS, inspect R1's flash memory:

```cisco
R1# show flash:
```

Your lab output showed:

```text
System flash directory:
File  Length   Name/status

3   33591768   c2900-universalk9-mz.SPA.151-4.M4.bin
2   28282      sigdef-category.xml
1   227537     sigdef-default.xml
```

The existing IOS image is:

```text
c2900-universalk9-mz.SPA.151-4.M4.bin
```

The router currently has approximately:

```text
255744000 bytes
```

of flash storage.

---

# 12. 🚀 Upgrade the IOS Image

The target image on TFTP1 is:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Copy the new image from TFTP1 to R1's flash:

```cisco
R1# copy tftp: flash:
```

When prompted for the remote host, enter:

```text
192.168.1.100
```

When prompted for the source filename, enter:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

When prompted for the destination filename, press **Enter** to accept the same filename.

The transfer should begin.

You should see progress indicators such as:

```text
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

followed by a successful transfer message.

---

# 13. 🔍 Verify the New IOS Image in Flash

After the transfer completes, check flash:

```cisco
R1# show flash:
```

You should now see both the old and new IOS images:

```text
c2900-universalk9-mz.SPA.151-4.M4.bin
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

The new image must exist in flash before configuring the router to boot from it.

---

# 14. ⚙️ Configure R1 to Boot the New IOS

Configure the new image as the boot image:

```cisco
R1# configure terminal
R1(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
R1(config)# end
```

Save the configuration:

```cisco
R1# write memory
```

Verify the boot configuration:

```cisco
R1# show running-config | include boot
```

You should see:

```text
boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
```

---

# 15. 🔄 Reload R1

After confirming that the new IOS image exists in flash and is configured as the boot image, reload the router:

```cisco
R1# reload
```

Confirm the reload when prompted.

R1 should reboot and load the new IOS image.

---

# 16. ✅ Verify the IOS Upgrade

After R1 finishes booting, run:

```cisco
R1# show version
```

Look for the IOS version.

The upgraded router should report the new image/version corresponding to:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Also verify the boot configuration:

```cisco
R1# show running-config | include boot
```

And check the flash contents:

```cisco
R1# show flash:
```

---

# 🧠 Important Commands

## Password Recovery

```cisco
rommon 1 > confreg 0x2142
rommon 2 > reset
```

After IOS boots:

```cisco
R1> enable
R1# copy startup-config running-config
R1# configure terminal
R1(config)# enable secret ccna
R1(config)# config-register 0x2102
R1(config)# end
R1# write memory
```

---

## TFTP Configuration Backup

```cisco
R1# copy startup-config tftp:
```

TFTP server:

```text
192.168.1.100
```

---

## IOS Upgrade

Check flash:

```cisco
R1# show flash:
```

Copy IOS from TFTP:

```cisco
R1# copy tftp: flash:
```

Configure boot image:

```cisco
R1(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Save:

```cisco
R1# write memory
```

Reload:

```cisco
R1# reload
```

Verify:

```cisco
R1# show version
```

---

# ⚠️ Common Mistakes

### Mistake 1 — Forgetting to restore `0x2102`

After password recovery, leaving the router at:

```text
0x2142
```

causes R1 to bypass the startup configuration on subsequent boots.

Always restore:

```cisco
config-register 0x2102
```

---

### Mistake 2 — Overwriting the startup configuration

During recovery, use:

```cisco
copy startup-config running-config
```

not:

```cisco
copy running-config startup-config
```

The purpose is to load the existing configuration into RAM so it can be modified.

---

### Mistake 3 — Attempting TFTP without connectivity

If:

```cisco
ping 192.168.1.100
```

returns:

```text
Success rate is 0 percent
```

resolve the connectivity problem first.

Check:

```cisco
show ip interface brief
```

and verify the interface connected to the TFTP network is **up/up**.

---

### Mistake 4 — Configuring a boot image that does not exist

Before configuring:

```cisco
boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
```

verify that the file exists:

```cisco
show flash:
```

The filename must match exactly.

---

### Mistake 5 — Forgetting to save the configuration

After configuring the new boot image and configuration register, save the configuration:

```cisco
write memory
```

---

# 🔍 Verification Checklist

Use these commands before considering the lab complete:

```cisco
show version
show flash:
show running-config
show running-config | include boot
show ip interface brief
ping 192.168.1.100
```

---

# 📋 Lab Completion Checklist

- [ ] R1 successfully enters ROMMON mode.
- [ ] Configuration register is temporarily set to `0x2142`.
- [ ] R1 boots while bypassing the startup configuration.
- [ ] Existing startup configuration is copied into running configuration.
- [ ] Enable secret is changed to `ccna`.
- [ ] Configuration register is restored to `0x2102`.
- [ ] R1's configuration is saved.
- [ ] R1 can reach TFTP1 at `192.168.1.100`.
- [ ] Startup configuration is backed up to TFTP1.
- [ ] Existing IOS image is verified with `show flash:`.
- [ ] `c2900-universalk9-mz.SPA.155-3.M4a.bin` is copied from TFTP1 to R1.
- [ ] New IOS image is visible in flash.
- [ ] New IOS image is configured as the boot image.
- [ ] R1 is reloaded.
- [ ] `show version` confirms the upgraded IOS.
- [ ] Configuration and connectivity remain functional after the upgrade.

---

# 🏁 Final Verification

A successful lab should leave R1 with:

| Item | Expected Result |
|---|---|
| Enable secret | `ccna` |
| Configuration register | `0x2102` |
| TFTP server | `192.168.1.100` |
| Configuration backup | Stored on TFTP1 |
| Original IOS | `15.1(4)M5` / `151-4.M4` image |
| Target IOS image | `c2900-universalk9-mz.SPA.155-3.M4a.bin` |
| New boot image | Target IOS image |
| R1 connectivity | Operational |

---

## 💡 Key Takeaways

This lab demonstrates three essential network-engineering recovery and maintenance procedures:

**Password recovery** allows an administrator to regain privileged access without deleting the existing configuration.

**TFTP configuration backup** provides a recovery copy of the router configuration and is useful before performing major changes.

**IOS upgrades** allow the router to run a newer software image. The administrator should always verify available flash space, transfer the image successfully, configure the correct boot image, save the configuration, reload the router, and verify the resulting IOS version.
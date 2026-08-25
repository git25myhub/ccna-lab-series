# Cisco Packet Tracer Lab – Port Security with Static Secure MAC Addresses

## Objective

Configure **switch port security** on SW1 and SW2 using manually assigned secure MAC addresses. Test how port security reacts when PCs are connected to the wrong switch port, observe the resulting security violation and err-disabled state, and restore connectivity.

## Topology

- PC1 → SW1 F0/2
- PC2 → SW2 F0/2
- SW1 and SW2 connected through the appropriate uplink
- PC1 IP address: `192.168.1.11`
- PC2 IP address: `192.168.1.12`

## Lab Tasks

### 1. Generate Traffic

From PC1, ping PC2:

```text
C:\>ping 192.168.1.12
```

Confirm that the ping succeeds. This generates traffic that allows the switches to learn the PCs' MAC addresses.

### 2. View the MAC Address Table

On SW1, enter:

```cisco
enable
show mac address-table
```

Identify and record the MAC address learned on **Fa0/2** for PC1.

Example:

```text
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
1       0002.16e2.2193    DYNAMIC     Fa0/2
```

On SW2, repeat the process:

```cisco
enable
show mac address-table
```

Record PC2's MAC address on **Fa0/2**.

Example:

```text
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
1       0007.ec28.0631    STATIC      Fa0/2
```

> **Note:** The MAC addresses shown above are from this lab session. Your Packet Tracer file may use different MAC addresses.

### 3. Configure Port Security on SW1

Configure Fa0/2 as an access port and enable port security:

```cisco
SW1#configure terminal
SW1(config)#interface fastethernet 0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport port-security
SW1(config-if)#switchport port-security mac-address 0002.16e2.2193
SW1(config-if)#end
SW1#write memory
```

Verify the configuration:

```cisco
SW1#show mac address-table
```

The secure MAC address should appear as **STATIC** on Fa0/2.

### 4. Configure Port Security on SW2

Configure Fa0/2 on SW2 for PC2:

```cisco
SW2#configure terminal
SW2(config)#interface fastethernet 0/2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport port-security
SW2(config-if)#switchport port-security mac-address 0007.ec28.0631
SW2(config-if)#end
SW2#write memory
```

Verify:

```cisco
SW2#show mac address-table
```

PC2's secure MAC address should be associated with Fa0/2.

## 5. Test the Original Topology

Reconnect the PCs to their original switch ports:

```text
PC1 → SW1 F0/2
PC2 → SW2 F0/2
```

From PC1:

```text
C:\>ping 192.168.1.12
```

The ping should succeed because each PC is connected to the port whose secure MAC address matches the configured address.

Example successful result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

## 6. Swap the PC Connections

Disconnect the cables and change the topology:

```text
PC1 → SW2 F0/2
PC2 → SW1 F0/2
```

The secure MAC addresses are now incorrect for the interfaces:

- SW1 F0/2 expects PC1's MAC but receives PC2's MAC.
- SW2 F0/2 expects PC2's MAC but receives PC1's MAC.

## 7. Test After Swapping the PCs

From PC1:

```text
C:\>ping 192.168.1.12
```

The ping should fail or experience significant packet loss.

Check SW2:

```cisco
SW2#show interfaces fastethernet 0/2
```

In this lab, SW2 reported:

```text
FastEthernet0/2 is down, line protocol is down (err-disabled)
```

A port-security violation was also reported:

```text
%PORT_SECURITY-2-PSECURE_VIOLATION:
Security violation occurred, caused by MAC address
0002.16E2.2193 on port FastEthernet0/2.
```

This confirms that port security detected an unauthorized MAC address.

## 8. Restore the Original Cabling

Reconnect the PCs to their original interfaces:

```text
PC1 → SW1 F0/2
PC2 → SW2 F0/2
```

Attempt the ping again:

```text
C:\>ping 192.168.1.12
```

The ping may still fail because a port placed into an **err-disabled** state does not automatically return to normal operation simply because the correct device is connected again.

## 9. Identify the Problem

Check the interface status:

```cisco
SW2#show interfaces fastethernet 0/2
```

If the interface displays:

```text
FastEthernet0/2 is down, line protocol is down (err-disabled)
```

the port has been disabled as a result of the port-security violation.

The same troubleshooting approach should be used on SW1 if it also enters an err-disabled state.

## 10. Fix the Problem

Manually recover the affected interface by shutting it down and bringing it back up:

```cisco
SW2#configure terminal
SW2(config)#interface fastethernet 0/2
SW2(config-if)#shutdown
SW2(config-if)#no shutdown
SW2(config-if)#end
SW2#write memory
```

Verify the interface:

```cisco
SW2#show interfaces fastethernet 0/2
```

The interface should now be:

```text
FastEthernet0/2 is up, line protocol is up
```

Repeat the recovery process on SW1 if necessary.

## 11. Final Connectivity Test

From PC1:

```text
C:\>ping 192.168.1.12
```

Expected result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

A successful ping confirms that the correct PCs are connected to their secured interfaces and that the err-disabled port has been recovered.

## Verification Commands

### View the MAC address table

```cisco
show mac address-table
```

### View port-security status

```cisco
show port-security
```

### View port-security details for an interface

```cisco
show port-security interface fastethernet 0/2
```

### Check interface status

```cisco
show interfaces fastethernet 0/2
```

### Check the running configuration

```cisco
show running-config interface fastethernet 0/2
```

## Key Concepts Learned

- **Port security** restricts which MAC addresses are allowed on a switch port.
- A **static secure MAC address** is manually configured by the administrator.
- Connecting a different PC to a secured port can cause a **port-security violation**.
- With the default violation behavior in this lab, the interface can enter an **err-disabled** state.
- Moving the correct PC back to the port does not necessarily restore the interface automatically.
- An err-disabled interface can be manually recovered with:

```cisco
shutdown
no shutdown
```

- `show port-security interface` is useful for troubleshooting port-security violations.

## Lab Result

The lab is successfully completed when:

- PC1 can ping PC2 under the original topology.
- Static secure MAC addresses are configured on SW1 F0/2 and SW2 F0/2.
- Connecting the PCs to the wrong secured ports causes a port-security violation.
- The affected interface enters an err-disabled state.
- The interface is successfully recovered.
- PC1 can once again ping PC2 with **0% packet loss**.
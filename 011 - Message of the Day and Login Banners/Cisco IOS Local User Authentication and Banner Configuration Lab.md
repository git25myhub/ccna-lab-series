# Cisco IOS Local User Authentication and Banner Configuration

## Objective

In this lab, you will configure local user authentication on R1 and secure console access using the local username database. You will also configure a Message of the Day (MOTD) banner and a login warning banner.

## Lab Requirements

- **Router:** R1
- **Access method:** PC1 console connection
- **Username 1:** `ccna`
- **Secret:** `Cisco`
- **Username 2:** `ccnp`
- **Secret:** `Cisco`
- **MOTD banner:** `Welcome to Packet Tracer`
- **Login banner:** `Authorized users only!`

## Tasks

### 1. Connect to R1

Use **PC1** to connect to **R1** through the console port.

### 2. Create Local User Accounts

Enter global configuration mode and create the two local users:

```cisco
R1# configure terminal

R1(config)# username ccna secret Cisco
R1(config)# username ccnp secret Cisco
```

### 3. Configure Console Authentication

Configure the console line to authenticate users against the local username database:

```cisco
R1(config)# line console 0
R1(config-line)# login local
R1(config-line)# exit
```

### 4. Configure the MOTD Banner

Configure the Message of the Day banner:

```cisco
R1(config)# banner motd #Welcome to Packet Tracer#
```

### 5. Configure the Login Banner

Configure the login warning banner:

```cisco
R1(config)# banner login #Authorized users only!#
```

### 6. Save the Configuration

Save the running configuration:

```cisco
R1(config)# end
R1# copy running-config startup-config
```

Alternatively:

```cisco
R1# write memory
```

## Verification

Display the running configuration:

```cisco
R1# show running-config
```

You should see entries similar to:

```cisco
username ccna secret 5 ...
username ccnp secret 5 ...

banner login ^CAuthorized users only!^C
banner motd ^CWelcome to Packet Tracer^C

line con 0
 login local
```

## 7. Test Authentication

Log out of the router:

```cisco
R1# exit
```

Press **Enter** when prompted to start the console session.

The banners should appear before the username prompt:

```text
Welcome to Packet Tracer
Authorized users only!

User Access Verification

Username:
Password:
```

Log in using either:

```text
Username: ccna
Password: Cisco
```

or:

```text
Username: ccnp
Password: Cisco
```

After successful authentication, you should receive:

```text
R1>
```

## Important Observation

The banners in the provided lab output displayed correctly, but there is a spelling mistake in the configured MOTD:

```text
Welcome to parcket trace
```

The required banner is:

```text
Welcome to Packet Tracer
```

Correct it with:

```cisco
R1# configure terminal
R1(config)# banner motd #Welcome to Packet Tracer#
R1(config)# end
R1# write memory
```

The login banner should also include the exclamation mark:

```text
Authorized users only!
```

## Expected Result

The lab is successfully completed when:

- Both `ccna` and `ccnp` users exist on R1.
- Both users authenticate using the local database.
- The console uses `login local`.
- The MOTD banner displays **Welcome to Packet Tracer**.
- The login banner displays **Authorized users only!**.
- Users can successfully log in after logging out.
- The configuration is saved to startup-config.

## Key Commands Learned

| Command | Purpose |
|---|---|
| `username <name> secret <password>` | Creates a local user account |
| `line console 0` | Enters console-line configuration |
| `login local` | Uses the local username database for authentication |
| `banner motd` | Configures the Message of the Day banner |
| `banner login` | Configures a login warning banner |
| `show running-config` | Displays the current configuration |
| `copy running-config startup-config` | Saves the configuration |
| `write memory` | Saves the configuration |

## Lab Questions

1. Did the MOTD banner display when reconnecting to R1?
2. Did the login banner display correctly?
3. Could you log in using both `ccna` and `ccnp`?
4. What happens if you enter an incorrect username or password?
5. Why is `login local` required on the console line?

## Conclusion

This lab demonstrates how to configure **local user authentication** and **login security banners** on a Cisco IOS router. The `login local` command forces console users to authenticate using the locally configured username database, while MOTD and login banners provide information and security warnings to users accessing the device.
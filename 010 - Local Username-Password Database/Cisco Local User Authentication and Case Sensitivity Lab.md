# Cisco Local User Authentication and Case Sensitivity Lab

## Lab Overview

This lab focuses on configuring **local user authentication on a Cisco router** and investigating whether Cisco IOS treats usernames and passwords as case-sensitive.

The lab uses **R1** and **PC1**, with PC1 connected to R1 through the console port.

The objective is to create multiple local user accounts, configure the console line to authenticate against the local user database, test the credentials, and determine how Cisco IOS handles username and password capitalization.

## Objectives

By completing this lab, you will:

- Access a Cisco router through the console port.
- Create local user accounts.
- Configure console authentication using the local user database.
- Test multiple user credentials.
- Determine whether Cisco IOS usernames are case-sensitive.
- Determine whether Cisco IOS passwords are case-sensitive.
- Verify local user configuration using the running configuration.
- Understand the difference between usernames and passwords in Cisco IOS authentication.

## Initial Configuration

The following users were required:

| Username | Password |
|---|---|
| `ccna` | `cisco` |
| `ccnp` | `CISCO` |

A third account was later created:

| Username | Password |
|---|---|
| `CCNA` | `router` |

The console line was configured to use the local user database:

```text
line console 0
 login local
```

## Configuration Steps

### 1. Create the Local Users

The first two accounts were configured on R1:

```text
username ccna secret cisco
username ccnp secret CISCO
```

Using the `secret` keyword stores the password in an encrypted/hashed form in the configuration rather than displaying it in plain text.

### 2. Configure Local Console Authentication

The console line was configured to authenticate against the locally configured usernames:

```text
line console 0
 login local
```

The `login local` command tells Cisco IOS to use the local username database when a user connects through the console.

## Password Case-Sensitivity Test

The router was logged out and accessed again using both accounts.

The following credentials were tested:

```text
Username: ccna
Password: cisco
```

and:

```text
Username: ccnp
Password: CISCO
```

The tests demonstrated that **passwords are case-sensitive**.

For example, the password:

```text
CISCO
```

is different from:

```text
cisco
```

Therefore, the exact capitalization used when creating the password must be used during authentication.

## Creating the Third User

A third account was created:

```text
username CCNA secret router
```

At this point, the router contained usernames that differed only by capitalization:

```text
ccna
CCNA
```

This was used to test whether Cisco IOS treats usernames as case-sensitive.

## Username Case-Sensitivity Test

After creating the `CCNA` account, the router was accessed using:

```text
Username: CCNA
Password: router
```

The account successfully authenticated.

The configuration was then examined using:

```text
show running-config
```

The local user entries appeared as:

```text
username ccna secret 5 ...
username ccnp secret 5 ...
```

The newly created `CCNA` account did not appear as a separate username entry.

Instead, the existing `ccna` entry was associated with the new password.

This demonstrates that, in this Cisco IOS environment, **usernames are not case-sensitive**. The usernames:

```text
ccna
CCNA
```

refer to the same local user account.

Consequently, configuring:

```text
username CCNA secret router
```

replaced the password associated with the existing `ccna` username.

## Important Finding

The most important distinction from this lab is:

### Usernames

**Not case-sensitive.**

These refer to the same username:

```text
ccna
CCNA
CcNa
```

### Passwords

**Case-sensitive.**

These are different passwords:

```text
cisco
CISCO
Cisco
```

Therefore, username capitalization does not create a separate account, while password capitalization changes the credential.

## Verification

The local user database can be reviewed with:

```text
show running-config
```

The console authentication configuration can be verified with:

```text
show running-config
```

Look for:

```text
line con 0
 login local
```

The configured usernames will appear near the beginning of the running configuration.

## Key Concepts Practiced

- Console access
- Local user authentication
- `username` command
- `secret` passwords
- `login local`
- Cisco IOS authentication
- Username case sensitivity
- Password case sensitivity
- Running configuration verification
- Secure credential storage

## Final Answers

### Are passwords case-sensitive?

**Yes.**

Cisco IOS treats passwords as case-sensitive.

For example:

```text
CISCO
```

and:

```text
cisco
```

are different passwords.

### Are usernames case-sensitive?

**No, in this Cisco IOS environment.**

For example:

```text
ccna
```

and:

```text
CCNA
```

refer to the same username.

Creating `CCNA` after `ccna` therefore changed the password associated with the existing account rather than creating a completely separate user.

## Useful Commands

```text
enable
configure terminal
username <username> secret <password>
line console 0
login local
end
show running-config
```

## Completion Criteria

The lab was successfully completed when:

- The required local users were configured.
- Console authentication used the local user database.
- Both initial accounts were tested successfully.
- Password case sensitivity was verified.
- The `CCNA` account was created and tested.
- The running configuration was inspected.
- Username and password case-sensitivity behavior was identified.

## Skills Demonstrated

This lab demonstrates practical knowledge of **Cisco IOS local authentication and credential management**. It also reinforces an important troubleshooting and security concept: **usernames and passwords do not necessarily follow the same case-sensitivity rules**, so authentication failures should be investigated carefully rather than assuming that capitalization is irrelevant.
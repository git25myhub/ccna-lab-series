# Extended ACL Troubleshooting Lab

## Objective

Configure **extended Access Control Lists (ACLs)** on R1 to control which hosts are allowed to access the servers.

The final ACL configuration must enforce the following requirements:

- **Only PC1** can access **SRV1**.
- **Only hosts on the `192.168.2.0/24` network** can access **SRV2**.
- All other traffic should continue to be permitted.

## Network Requirements

| Device / Network | Address |
|---|---|
| PC1 | `192.168.1.11` |
| R1 Fa0/0 | `192.168.1.1` |
| R1 Fa1/0 | `192.168.2.1` |
| 192.168.2.0 Network | `192.168.2.0/24` |
| R1 Serial2/0 | `12.0.0.1` |
| SRV1 | `192.168.3.100` |
| SRV2 | `192.168.3.101` |

## Task

Configure an **extended ACL 100** on R1 to implement the access-control requirements.

### Requirement 1 – SRV1

Only PC1 (`192.168.1.11`) should be able to access SRV1 (`192.168.3.100`).

Traffic from all other source addresses to SRV1 must be denied.

### Requirement 2 – SRV2

Only hosts belonging to the `192.168.2.0/24` network should be able to access SRV2 (`192.168.3.101`).

Traffic from all other source networks to SRV2 must be denied.

### Requirement 3 – Other Traffic

The ACL should not unnecessarily block other traffic.

After the specific deny/permit statements, allow other IP traffic with an appropriate final permit statement.

## Configuration

Enter configuration mode on R1:

```text
R1# configure terminal
```

Create the extended ACL:

```text
R1(config)# access-list 100 permit ip host 192.168.1.11 host 192.168.3.100
R1(config)# access-list 100 deny ip any host 192.168.3.100

R1(config)# access-list 100 permit ip 192.168.2.0 0.0.0.255 host 192.168.3.101
R1(config)# access-list 100 deny ip any host 192.168.3.101

R1(config)# access-list 100 permit ip any any
```

Apply the ACL to R1's Serial2/0 interface in the **outbound** direction:

```text
R1(config)# interface serial2/0
R1(config-if)# ip access-group 100 out
```

Save the configuration:

```text
R1(config-if)# end
R1# write memory
```

## Why the ACL Is Applied Outbound

SRV1 and SRV2 are located on the network beyond R1's Serial2/0 interface.

Applying ACL 100 **outbound** on Serial2/0 allows R1 to inspect traffic immediately before it leaves toward the server network.

The ACL is processed from **top to bottom**, and the first matching statement determines whether the packet is permitted or denied.

## ACL Logic

The ACL should effectively behave as follows:

```text
PC1 ---------------------> SRV1
192.168.1.11                192.168.3.100
        PERMIT

Other hosts --------------> SRV1
        DENY


192.168.2.0/24 -----------> SRV2
                            192.168.3.101
        PERMIT

Other hosts --------------> SRV2
        DENY


All other IP traffic
        PERMIT
```

## Verification

Display the ACL:

```text
R1# show access-lists 100
```

You should see entries similar to:

```text
Extended IP access list 100
    permit ip host 192.168.1.11 host 192.168.3.100
    deny ip any host 192.168.3.100
    permit ip 192.168.2.0 0.0.0.255 host 192.168.3.101
    deny ip any host 192.168.3.101
    permit ip any any
```

Check the interface configuration:

```text
R1# show ip interface serial2/0
```

Confirm that ACL 100 is applied **outbound** on Serial2/0.

You can also verify the interface summary:

```text
R1# show ip interface brief
```

## Testing

Test connectivity from the appropriate PCs.

### SRV1 Tests

From **PC1**:

```text
ping 192.168.3.100
```

Expected result:

```text
SUCCESS
```

From other PCs:

```text
ping 192.168.3.100
```

Expected result:

```text
FAIL
```

### SRV2 Tests

From a host on the `192.168.2.0/24` network:

```text
ping 192.168.3.101
```

Expected result:

```text
SUCCESS
```

From a host outside the `192.168.2.0/24` network:

```text
ping 192.168.3.101
```

Expected result:

```text
FAIL
```

## Important ACL Concepts

### Extended ACLs

Extended ACLs can filter traffic based on multiple criteria, including:

- Source IP address
- Destination IP address
- Protocol
- TCP/UDP
- Source and destination ports

This makes them more precise than standard ACLs.

### Wildcard Mask

The network `192.168.2.0/24` is represented in an ACL using:

```text
192.168.2.0 0.0.0.255
```

The wildcard mask `0.0.0.255` means that the last octet can vary.

Therefore, the statement:

```text
permit ip 192.168.2.0 0.0.0.255 host 192.168.3.101
```

allows any host from `192.168.2.0/24` to reach SRV2.

### ACL Processing Order

ACL entries are evaluated from **top to bottom**.

For example:

```text
permit PC1 -> SRV1
deny any -> SRV1
```

must appear in this order.

If the deny statement appeared first, PC1 would also be denied.

### Implicit Deny

Every ACL has an implicit:

```text
deny ip any any
```

at the end.

Therefore, the final:

```text
permit ip any any
```

is necessary in this lab if other traffic should remain allowed.

## Completion Criteria

The lab is successfully completed when:

- [ ] PC1 can access SRV1.
- [ ] Other hosts cannot access SRV1.
- [ ] Hosts on `192.168.2.0/24` can access SRV2.
- [ ] Hosts outside `192.168.2.0/24` cannot access SRV2.
- [ ] Other unrelated IP traffic remains permitted.
- [ ] ACL 100 is applied outbound on R1 Serial2/0.
- [ ] The configuration is saved with `write memory`.

## Useful Commands

```text
show access-lists 100
show ip interface serial2/0
show ip interface brief
show running-config
write memory
```

## Key Takeaway

This lab demonstrates how **extended ACLs** can provide precise traffic filtering by matching both the **source and destination IP addresses**. The key is to place specific permit and deny statements before the general `permit ip any any` statement and apply the ACL in the correct direction on the appropriate interface.
# FTP Connectivity Troubleshooting – Finance2 to File Server

## Project Overview

This Cisco Packet Tracer troubleshooting lab investigates an FTP connectivity issue affecting the Finance2 workstation. The user could not retrieve `Document.txt` from the File Server using FTP over TCP port 21 and suspected that an access control list (ACL) was blocking the application.

The investigation identified an OSPF network-type mismatch on the link between the Branch 1 Layer 3 switch (BR1-SW1) and the Branch 1 core switch. Both ends of the link were configured with the OSPF network type `broadcast` initially. Changing only the BR1-SW1 side to `point-to-point` left the two ends mismatched. Configuring the corresponding core-switch interface as `point-to-point` resolved the issue.

## Lab Objective

Enable Finance2 to retrieve `Document.txt` from the File Server using FTP.

**File Server:** `192.168.2.251`  
**FTP port:** TCP 21  
**Required FTP commands:**

```text
ftp 192.168.2.251
get Document.txt
```

## Topology

The network contains three OSPF areas:

- **Area 1:** Branch 1 (stub area)
- **Area 0:** OSPF backbone / Internet area
- **Area 2:** HQ

The affected path runs from the Branch 1 access layer through the distribution and core switches, then across the wider network towards the File Server.

See `images/topology.png` for the network topology.

## Problem

Finance2 was unable to retrieve `Document.txt` from the File Server using FTP. The lab description suggested that an ACL might be blocking the application, but that was only an initial theory and needed to be verified.

The Admin PC could successfully retrieve the file, while Finance2 could not. This showed that FTP was working for at least one client, but did not establish that the Finance2 traffic path was working correctly.

See `images/Before-fix.png` for the initial failure.

## Investigation and Troubleshooting

1. **Compared client behaviour.**  
   Tested FTP from the Admin PC and confirmed that it could retrieve the file. Tested from Finance2 and confirmed that the failure was specific to the Finance2 path/client.

2. **Checked basic reachability with ping.**  
   Tested connectivity between devices and across the OSPF areas. Inter-area communication was possible, but the Branch 1 departmental PCs could reach their gateways without successfully reaching destinations beyond the Branch 1 core-switch path.

3. **Narrowed the investigation to the Branch 1 switching path.**  
   Investigated the Layer 3 core switch, its distribution-switch connections, and the links between them. SSH access to the core switch from the Admin PC timed out, so the core switch could not initially be inspected directly from that workstation.

4. **Accessed the core switch through BR1-SW1.**  
   From the BR1-SW1 command line, used SSH to connect to the core switch at `10.10.10.6`:

   ```text
   ssh -l Admin 10.10.10.6
   ```

   This provided access to the core switch for further investigation.

5. **Identified the OSPF network-type mismatch.**  
   Used CDP neighbour information to identify the core-switch interface connected to BR1-SW1. The connected core-switch interface was `FastEthernet0/1`.

   Checked the OSPF interface configuration on the core switch:

   ```text
   show cdp neighbors
   show ip ospf interface FastEthernet0/1
   ```

   The core-switch interface was still using the OSPF network type `broadcast`, while the corresponding BR1-SW1 interface (`FastEthernet0/2`) had already been changed to `point-to-point`. The two ends of the link therefore had mismatched OSPF network types.

## Root Cause

The OSPF network type was inconsistent across the link connecting BR1-SW1 and the Branch 1 core switch:

| Device | Interface | OSPF network type |
|---|---|---|
| BR1-SW1 | FastEthernet0/2 | Point-to-point |
| Branch 1 core switch | FastEthernet0/1 | Broadcast (before correction) |

The mismatch prevented the link from operating as intended for OSPF adjacency and routing.

## Corrective Action

Configured the core-switch interface `FastEthernet0/1` to use the same OSPF network type as the BR1-SW1 side: `point-to-point`.

The BR1-SW1 interface `FastEthernet0/2` had already been changed to `point-to-point` during the investigation.

The exact configuration commands are documented in [`config.txt`](config.txt).

## Verification

After applying the matching OSPF network type on the core switch:

1. Tested connectivity from the core switch to the File Server; the ping was successful.
2. Returned to Finance2 and initiated an FTP session to the File Server.
3. Logged in using the File Server credentials supplied by the lab.
4. Retrieved `Document.txt` successfully using:

   ```text
   ftp 192.168.2.251
   get Document.txt
   ```

The successful file retrieval from Finance2 confirmed that the lab objective had been achieved.

See `images/Lab-complete.png` for the successful FTP transfer and retrieved file.

## Configuration Files

- [`config.txt`](config.txt) – OSPF network-type changes applied to BR1-SW1 and the Branch 1 core switch.

## Lab Constraints Respected

- No ACLs were removed.
- No static routes were added.
- No new routing protocols were implemented.
- No new technology or protocol roles were introduced.
- The existing OSPF design was retained; only the mismatched OSPF network type on the two ends of the link was corrected.

## Key Learning Outcomes

- Understood that OSPF network-type settings must be compatible across a link.
- Practised using CDP to identify a neighbouring device and the connected interface.
- Used `show ip ospf interface` to inspect OSPF interface parameters.
- Used SSH from one network device to access another device when direct management access from the Admin PC was unsuccessful.
- Distinguished an initial ACL hypothesis from the evidence gathered during troubleshooting.
- Verified the repair using both network connectivity testing and the application-level FTP file transfer.

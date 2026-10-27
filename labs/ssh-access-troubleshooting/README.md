# Troubleshooting SSH Access Using Hostnames

## Project Overview

This project involved troubleshooting a network where the Admin user located off ASW5 was unable to access network infrastructure devices through SSH using their hostnames.

Although the Admin could successfully ping the DNS server and SSH into devices using their IP addresses, hostname-based SSH connections failed.

The objective was to identify and resolve the issue so that network infrastructure devices could be accessed remotely using their hostnames.

## Problem Statement

The Admin located off ASW5 reported that they were unable to SSH into network infrastructure devices using their hostnames.

Initial testing confirmed that the DNS server was reachable. The lab was considered complete when infrastructure devices could be accessed successfully through their hostnames.

**Credentials provided by the lab:**
- Username: `Admin`
- Password: `witty`

## Network Topology

The lab contains network infrastructure devices, an Admin workstation connected off ASW5, a DNS server, and a DHCP server.

See `topology.png` for the network layout.

## Troubleshooting Process

### 1. Verified DNS Server Connectivity

I first confirmed that the Admin workstation could reach the DNS server by pinging its IP address.

**Result:** The ping was successful, confirming basic IP connectivity to the DNS server.

### 2. Tested SSH Access Using IP Addresses

I attempted to connect to the infrastructure devices using their IP addresses.

Example:

```text
ssh -l Admin <device-ip-address>
```

**Result:** SSH connections using IP addresses worked successfully.

This indicated that SSH access was operational and that the issue was more likely related to hostname resolution than basic network connectivity or SSH authentication.

### 3. Tested SSH Access Using Hostnames

I then attempted to access a device using its hostname.

Example:

```text
ssh -l Admin dsw3
```

**Result:** The connection failed when the hostname was used.

Since SSH worked with IP addresses but failed with hostnames, I focused on investigating DNS and DHCP-related configuration.

### 4. Checked DNS Records

I accessed the DNS server and verified that the DNS service was enabled and that the network infrastructure devices were listed in its DNS records.

**Result:** The DNS service was enabled, and the device records were present.

This suggested that the issue was not simply the absence of records on the DNS server.

### 5. Checked DHCP Configuration

Because the network used DHCP to assign IP addresses, I inspected the DHCP server configuration.

I verified that the DHCP service was enabled and that the configured address pools were present and accurate.

I also used the following command on the Admin workstation to inspect its network configuration:

```cmd
ipconfig /all
```

This allowed me to check the workstation's IP address, DHCP information, and DNS server settings.

### 6. Identified the Root Cause

While inspecting the DHCP server, I discovered that the DNS option under the DHCP service was disabled. The infrastructure devices and their corresponding IP addresses were also missing from the DHCP server's DNS records.

Although the DNS server itself contained the device records, the DHCP server's DNS configuration was incomplete.

**Root cause:** The DHCP server's DNS service option was disabled, and the required infrastructure device hostname-to-IP mappings were not configured in its DNS settings.

### 7. Applied the Fix

I made the following changes under the DHCP server's DNS settings:

- Enabled the DNS option.
- Added the infrastructure devices and their corresponding IP addresses to the DNS records.

I then retested SSH access using the devices' hostnames.

Example:

```text
ssh -l Admin dsw3
```

**Result:** SSH connections using hostnames succeeded, and I was able to log into the network infrastructure devices as intended.

## Root Cause Summary

The issue was caused by incomplete DNS-related configuration on the DHCP server. The DNS option was disabled, and the required infrastructure device records were missing from its DNS settings.

As a result, SSH connections using IP addresses worked, but connections using hostnames failed.

## Solution Summary

| Check | Initial Result | Final Result |
|---|---|---|
| Ping to DNS server | Successful | Successful |
| SSH using IP address | Successful | Successful |
| SSH using hostname | Failed | Successful |
| DNS service on DNS server | Enabled | Enabled |
| DNS option under DHCP | Disabled | Enabled |
| Infrastructure hostname records under DHCP | Missing | Added |

## Verification

After applying the fix, I retried SSH connections using the infrastructure devices' hostnames.

The connections succeeded, confirming that the hostname-based access issue had been resolved.

See `lab-complete.png` for the successful lab result.

## Screenshots

### Network Topology

![Network topology](topology.png)

### Before the Fix

![Failed hostname-based SSH access](before-fix.png)

### After the Fix

![Successful lab completion](lab-complete.png)

## Key Takeaways

This lab reinforced several practical networking and troubleshooting concepts:

- Differentiating IP connectivity issues from hostname resolution issues.
- Testing SSH connectivity using both IP addresses and hostnames.
- Verifying DNS records and service status.
- Inspecting DHCP configuration and DNS-related settings.
- Using `ipconfig /all` to inspect a Windows workstation's network configuration.
- Troubleshooting systematically by isolating the problem before making configuration changes.

The key lesson was that successful connectivity to a DNS server does not automatically guarantee successful hostname resolution. DNS records, DHCP-related DNS configuration, and the client's DNS settings must also be checked.

## Project Files

- `README.md` — Troubleshooting documentation, root cause, solution, and verification.
- `config.txt` — Relevant configuration notes and verification commands.
- `topology.png` — Network topology.
- `before-fix.png` — Evidence of the issue before applying the fix.
- `lab-complete.png` — Evidence of successful hostname-based SSH access after the fix.

# DHCP, DNS & Common IP Services

## 1. DHCP (Dynamic Host Configuration Protocol)

Automatically provides IP address, mask, default gateway, DNS server, and lease time. UDP: server **67**, client **68**.

### DORA process
1. **Discover** - client broadcasts (src 0.0.0.0, dst 255.255.255.255)
2. **Offer** - server proposes an address
3. **Request** - client requests the offered address (broadcast)
4. **Acknowledge** - server confirms

Lease renewal: T1 at 50 % (unicast renew), T2 at 87.5 % (broadcast rebind).

### DHCP relay
Broadcasts don't cross routers. Configure `ip helper-address <server-ip>` on the **client-facing** router interface; the router converts the broadcast to a unicast to the server (and sets the `giaddr` so the server knows which pool to use).

### Router as DHCP server
```text
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 lease 1
```
Always exclude static addresses (gateway, servers) **before** the pool hands them out.

### Verification & troubleshooting
```text
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip dhcp server statistics
```
Client side (Windows): `ipconfig /release`, `ipconfig /renew`, `ipconfig /all`.

| Symptom | Check |
|---|---|
| Host gets 169.254.x.x | No DHCP reached: VLAN, trunk, helper-address, server pool, `giaddr` |
| Wrong gateway/DNS | Pool options |
| Pool exhausted | Lease time, scope size, conflicts |
| Rogue DHCP server | DHCP snooping (see security notes) |

## 2. DNS

Resolves names to IPs. UDP/TCP **53**. Hierarchy: root -> TLD -> authoritative.

| Record | Purpose |
|---|---|
| A | Name -> IPv4 |
| AAAA | Name -> IPv6 |
| CNAME | Alias to another name |
| MX | Mail server |
| NS | Authoritative name server |
| PTR | IP -> name (reverse lookup) |

Cisco IOS:
```text
ip domain-lookup                 ! enabled by default
ip name-server 8.8.8.8
ip domain-name lab.local
ip host R2 10.0.0.2              ! local static host entry
no ip domain-lookup              ! stops IOS waiting on mistyped commands
```
Troubleshoot with `nslookup`, `ping <name>` vs `ping <IP>` (if IP works and name fails, it's DNS).

## 3. NTP (Network Time Protocol)

Keeps clocks synchronised - essential for logs and certificates. UDP 123. **Stratum** = distance from the reference clock (lower is better; stratum 1 attaches to an atomic/GPS source).
```text
ntp server 192.168.1.100
ntp master 3                 ! make this device a lab time source
show ntp status
show ntp associations
```

## 4. Syslog

Centralised logging, UDP 514. **Severity levels (0-7):** 0 Emergency · 1 Alert · 2 Critical · 3 Error · 4 Warning · 5 Notice · 6 Informational · 7 Debug.
```text
logging host 192.168.1.50
logging trap informational
service timestamps log datetime msec
logging buffered 16384
show logging
```
Remember: setting a level logs that level **and more severe** ones.

## 5. SNMP

Manages devices. UDP 161 (polling), 162 (traps/informs). Components: **manager (NMS)**, **agent**, **MIB** (OIDs).
- Messages: Get, Set, Trap, Inform.
- **SNMPv1/v2c:** community strings in clear text. **SNMPv3:** authentication + encryption (use this).
```text
snmp-server community READONLY ro
snmp-server host 192.168.1.50 version 2c READONLY
```

## 6. FHRP (First Hop Redundancy Protocols)

Provide a **virtual gateway IP** so hosts survive a gateway failure.

| Protocol | Standard | Notes |
|---|---|---|
| **HSRP** | Cisco | Active / Standby; virtual MAC `0000.0C07.ACxx`; priority default 100; `preempt` needed to reclaim role |
| **VRRP** | IEEE (open) | Master / Backup |
| GLBP | Cisco | Load balances across gateways |

```text
interface g0/0
 ip address 10.0.0.2 255.255.255.0
 standby version 2
 standby 1 ip 10.0.0.1
 standby 1 priority 110
 standby 1 preempt
show standby brief
```

## 7. Other services worth remembering

- **TFTP** (UDP 69) / **FTP** - file transfer (IOS images, configs). TFTP is unauthenticated.
- **QoS** - see `security-wireless-automation.md`.
- **SSH** - secure management (`configs/.../02-secure-management-ssh.txt`).

Configs: `08-dhcp-dns-services.txt`.

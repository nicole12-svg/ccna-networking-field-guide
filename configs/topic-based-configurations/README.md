# Topic-Based Configurations

Cisco IOS / IOS XE configuration snippets for lab use, grouped by topic. Each file pairs with a note in `../../notes/`.

| File | Topic | Companion note |
|---|---|---|
| `01-basic-device-config.txt` | Hostname, passwords, banners, console | `network-fundamentals.md` |
| `02-secure-management-ssh.txt` | SSH, VTY hardening, management SVI | `network-fundamentals.md` |
| `03-vlans-trunking.txt` | VLANs, access/trunk ports | `switching-vlans-trunking.md` |
| `04-inter-vlan-routing.txt` | ROAS and SVIs | `routing-ospf.md` |
| `05-static-routing.txt` | Static, default, floating routes | `routing-ospf.md` |
| `06-ospf.txt` | OSPFv2 single area | `routing-ospf.md` |
| `07-stp-etherchannel.txt` | Rapid PVST+, guards, LACP | `stp-etherchannel.md` |
| `08-dhcp-dns-services.txt` | DHCP, relay, DNS, NTP, syslog, SNMP, HSRP | `dhcp-dns.md` |
| `09-nat-pat.txt` | Static NAT, dynamic NAT, PAT | `nat-acls.md` |
| `10-acls.txt` | Standard, extended, named ACLs | `nat-acls.md` |
| `11-switch-security.txt` | Port security, DHCP snooping, DAI | `security-wireless-automation.md` |
| `12-ipv6.txt` | IPv6 addressing, routing, SLAAC | `ipv6.md` |
| `13-automation-examples.md` | Ansible, RESTCONF, Python | `security-wireless-automation.md` |

**Notes**
- Lines starting with `!` are comments in IOS.
- Passwords and addresses are placeholders for labs - never reuse them in production.
- Interface names and some commands (e.g. `switchport trunk encapsulation dot1q`) differ by platform and IOS version; adjust to your device.
- Packet Tracer supports most, but not all, of these commands (e.g. some DAI/snooping/RESTCONF features).

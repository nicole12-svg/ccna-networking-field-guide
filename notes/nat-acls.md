# NAT, PAT & Access Control Lists

## 1. NAT / PAT

NAT translates private (RFC 1918) addresses to public ones, conserving IPv4 space.

### Terminology
| Term | Meaning |
|---|---|
| **Inside local** | Inside host's private address as seen on the inside |
| **Inside global** | Inside host's public address as seen from the outside |
| **Outside global** | Outside host's real public address |
| **Outside local** | Outside host's address as seen from the inside (usually same as outside global) |

### Types
1. **Static NAT** - 1:1 permanent mapping (e.g. publish an internal web server).
2. **Dynamic NAT** - inside addresses mapped from a **pool** of public addresses (1:1, first come first served; pool can run out).
3. **PAT / NAT overload** - many inside hosts share **one** public IP (or interface address), distinguished by **port numbers**. Most common.

### Configuration checklist
1. Mark `ip nat inside` and `ip nat outside` interfaces.
2. Define which traffic to translate (ACL).
3. Define the translation (static / pool / overload).

### Verification & troubleshooting
```text
show ip nat translations
show ip nat statistics
clear ip nat translation *
debug ip nat           ! lab only
```
| Symptom | Check |
|---|---|
| No translations | inside/outside swapped, ACL doesn't match source, no route/default route to ISP |
| Pool exhausted | Use overload |
| Works one way | Missing return route, or ACL on edge blocks it |
| Stale translations | `clear ip nat translation *` after changes |

## 2. Access Control Lists (ACLs)

Ordered permit/deny rules evaluated **top to bottom, first match wins**, with an **implicit deny any** at the end.

### Types
| Type | Number ranges | Matches on |
|---|---|---|
| Standard | 1-99, 1300-1999 | Source IP only |
| Extended | 100-199, 2000-2699 | Protocol, source, destination, ports |
| Named | name | Either style; editable by sequence number |

### Wildcard masks
Inverse of a subnet mask: `0` = must match, `1` = don't care.
- `/24` -> `0.0.0.255` · `/26` -> `0.0.0.63` · `/30` -> `0.0.0.3`
- Single host: `host 10.1.1.1` (= `10.1.1.1 0.0.0.0`) · Any: `any`.
- Wildcard = 255.255.255.255 minus the subnet mask.

### Placement rules of thumb
- **Standard ACL:** as close to the **destination** (it only sees the source).
- **Extended ACL:** as close to the **source** (drop unwanted traffic early).
- Apply with `ip access-group <name|number> in|out` on an interface; `access-class` on VTY lines.
- One ACL **per interface, per direction, per protocol**.
- Router-originated traffic is **not** filtered by outbound ACLs.

### Editing
Use named ACLs and sequence numbers: `ip access-list extended NAME` then `15 permit ...` or `no 20`. For numbered ACLs, re-entering with sequence numbers also works on modern IOS.

### Verification
```text
show access-lists
show ip access-lists
show ip interface g0/0 | include access list
```
Hit counters (`(15 matches)`) show which rule is acting. Check for: wrong direction, wrong interface, order of statements, forgotten `permit ip any any`, wildcard typos.

Configs: `09-nat-pat.txt`, `10-acls.txt`.

# Ethernet Switching, VLANs & Trunking

## 1. How a switch forwards frames

A switch builds a **MAC address table** (CAM table) from **source MACs** and forwards by **destination MAC**.

| Situation | Action |
|---|---|
| Unknown source MAC | **Learn** it (MAC + ingress port + VLAN) |
| Known unicast destination | **Forward** out the single matching port |
| Unknown unicast destination | **Flood** out all ports in the VLAN except the ingress port |
| Broadcast / multicast (default) | **Flood** |
| Destination is on the ingress port's segment | **Filter** (drop) |

- Default MAC aging time: 300 seconds.
- `show mac address-table` · `show mac address-table dynamic` · `clear mac address-table dynamic`

## 2. VLANs

A VLAN is a logical **broadcast domain**. Benefits: smaller broadcast domains, segmentation/security, and flexibility.

- Standard range **1-1005** (stored in `vlan.dat`), extended **1006-4094**.
- Defaults: VLAN 1 (default), 1002-1005 (legacy FDDI/Token Ring, can't be deleted).
- Hosts in different VLANs need a **router / L3 switch** to communicate.
- Common design: one VLAN = one IP subnet.

### Access ports
Carry **one** VLAN, untagged. Connect to end hosts.
```text
interface g0/1
 switchport mode access
 switchport access vlan 10
```
Voice VLAN: `switchport voice vlan 20` (phone tagged, PC untagged).

## 3. Trunking (802.1Q)

A trunk carries **multiple VLANs** between switches/routers using a 4-byte **802.1Q tag** (includes 12-bit VLAN ID + 3-bit priority/CoS).

- **Native VLAN:** frames are sent **untagged** (default VLAN 1). Both ends must match or you get native VLAN mismatch (CDP/STP warnings, leaking traffic). Best practice: use an unused VLAN as native.
- **Allowed VLANs:** limit with `switchport trunk allowed vlan`.
- Modern Catalyst switches support only 802.1Q; older ones may need `switchport trunk encapsulation dot1q` first.

```text
interface g0/24
 switchport trunk encapsulation dot1q    ! only on older platforms
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,999
```

### DTP (Dynamic Trunking Protocol)
| Mode | Behaviour |
|---|---|
| `access` | Always access |
| `trunk` | Always trunk, actively negotiates |
| `dynamic desirable` | Actively tries to form trunk |
| `dynamic auto` | Forms trunk only if other side initiates |

Best practice: **hard-code** modes and disable negotiation with `switchport nonegotiate`.
Auto + Auto = access. Desirable + anything (not access) = trunk.

## 4. Verification

```text
show vlan brief
show interfaces trunk
show interfaces g0/1 switchport
show mac address-table vlan 10
show interfaces status
```

## 5. Common VLAN/trunk faults

| Symptom | Likely cause |
|---|---|
| Host can't reach same-VLAN neighbour | Port in wrong VLAN, VLAN doesn't exist on the switch, VLAN not allowed on trunk |
| Port shows in no VLAN after VLAN deleted | Access VLAN removed - recreate it |
| One side access, other side trunk | Mode mismatch |
| Native VLAN mismatch log messages | Different native VLANs on trunk ends |
| Hosts can't reach other VLANs | No inter-VLAN routing, wrong gateway, wrong subnet |

## 6. Other Layer 2 tools

- **CDP** (Cisco proprietary) and **LLDP** (IEEE 802.1AB): neighbour discovery. `show cdp neighbors detail`, `show lldp neighbors`. Disable on untrusted ports.
- **Switch ports:** speed/duplex auto-negotiate; hard-code both ends or leave both auto.

Inter-VLAN routing is covered in `routing-ospf.md`. Configs: `03-vlans-trunking.txt`.

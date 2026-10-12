# STP, EtherChannel & Layer 2 Redundancy

## 1. Why STP

Redundant L2 links create **loops**: broadcast storms, MAC table instability, multiple frame copies. STP (IEEE 802.1D) logically blocks redundant paths and unblocks them if the active path fails.

## 2. STP operation

**Bridge ID** = Priority (default 32768, in steps of 4096) + **VLAN ID (extended system ID)** + MAC address. **Lowest** BID wins.

Process:
1. **Elect a root bridge** (lowest BID) per VLAN. All its ports are designated.
2. Each non-root switch picks one **root port** (lowest total path cost to root; ties: lowest sender BID, then lowest sender port ID).
3. Each segment elects one **designated port** (lowest cost to root; ties: lowest BID).
4. Remaining ports become **blocking/alternate** (non-designated).

**Port costs (classic short method):** 10 Mbps = 100 · 100 Mbps = 19 · 1 Gbps = 4 · 10 Gbps = 2.

BPDUs are sent every **2 s** (hello). Multicast `0180.C200.0000`.

### 802.1D port states
Blocking (20 s max age) -> Listening (15 s) -> Learning (15 s) -> Forwarding. Convergence ~30-50 s.

## 3. STP variants

| Variant | Standard | Notes |
|---|---|---|
| STP | 802.1D | One tree for all VLANs (CST) |
| **PVST+** | Cisco | One 802.1D tree **per VLAN**; Cisco default |
| RSTP | 802.1w | Fast convergence (seconds); states: Discarding, Learning, Forwarding |
| **Rapid PVST+** | Cisco | RSTP per VLAN; **recommended** |
| MSTP | 802.1s | Maps groups of VLANs to a few instances |

### RSTP port roles
Root, Designated, **Alternate** (backup path to root), **Backup** (backup on shared segment), Disabled. Link types: point-to-point (full duplex), shared (half duplex), edge (PortFast).

## 4. STP protection features

| Feature | Purpose |
|---|---|
| **PortFast** | Edge ports go straight to forwarding. Only on host ports. |
| **BPDU Guard** | Err-disables a PortFast port that receives a BPDU |
| **BPDU Filter** | Suppresses BPDUs (use with care) |
| **Root Guard** | Prevents a port from becoming a root port (protects root placement) |
| **Loop Guard** | Prevents alternate/root ports from becoming designated if BPDUs stop (unidirectional links) |

## 5. Verification

```text
show spanning-tree
show spanning-tree vlan 10
show spanning-tree summary
show spanning-tree interface g0/1 detail
```
Look at: Root ID vs Bridge ID (is this the root?), port roles/states, cost.

## 6. EtherChannel (link aggregation)

Bundles up to **8** physical links into one logical **Port-channel**: more bandwidth, redundancy, and STP treats it as a single link (no blocked member ports). Load balancing is per-flow (hash of MAC/IP/port), so a single flow uses one link.

| Protocol | Standard | Modes |
|---|---|---|
| **LACP** | IEEE 802.3ad | `active` / `passive` |
| PAgP | Cisco | `desirable` / `auto` |
| Static | - | `on` (no negotiation) |

**Forms a channel when:** active+active, active+passive, desirable+desirable, desirable+auto, on+on. **Does not form:** passive+passive, auto+auto, mixing protocols, `on` with a negotiating mode.

**Member ports must match:** speed, duplex, VLAN/trunk mode, native & allowed VLANs, and STP settings.

```text
show etherchannel summary          ! flags: SU = L2 in use, P = bundled, D = down, I = standalone, s = suspended
show etherchannel port-channel
show interfaces port-channel 1
```

Common faults: mismatched member config (ports suspended `s`), mismatched modes, one end not configured, wrong `channel-group` numbers (numbers are locally significant, but both ends need matching bundles).

## 7. First-hop redundancy (Layer 3)

See HSRP/VRRP/GLBP in `dhcp-dns.md` (IP services).

Configs: `07-stp-etherchannel.txt`.

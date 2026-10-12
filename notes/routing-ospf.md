# Routing: Inter-VLAN, Static & OSPF

## 1. Routing basics

A router forwards by **longest prefix match** in its routing table. If several routes have the same prefix length, the one with the lowest **administrative distance (AD)** wins; for the same protocol, the lowest **metric** wins.

| Route source | AD |
|---|---|
| Connected | 0 |
| Static | 1 |
| eBGP | 20 |
| EIGRP | 90 |
| **OSPF** | **110** |
| RIP | 120 |
| Unknown | 255 |

Route codes: `C` connected · `L` local · `S` static · `O` OSPF · `*` candidate default.
`show ip route` · `show ip route 10.1.1.0` · `show ip route ospf`

**Forwarding decision on a packet:** drop to default gateway if not local -> router de-encapsulates, finds route, re-encapsulates with new L2 header (TTL decremented, new MACs, same IPs).

## 2. Inter-VLAN routing

### Router-on-a-stick (ROAS)
One physical router link (trunk) with **sub-interfaces**, one per VLAN. Each sub-interface: `encapsulation dot1q <vlan>` and an IP (the VLAN's default gateway). The native VLAN sub-interface uses `native`.

### Layer 3 switch (SVI)
`interface vlan 10` + IP address, with `ip routing` enabled. Faster; usual in campus networks. **Routed port:** `no switchport` + IP.

Verification: `show ip interface brief`, `show interfaces trunk`, ping across VLANs, `show ip route`.

Common faults: missing `encapsulation dot1q`, wrong VLAN number on sub-interface, trunk not allowing VLANs, SVI down because VLAN doesn't exist or no active port in it, `ip routing` missing, host gateway wrong.

## 3. Static routing

```text
ip route <network> <mask> <next-hop | exit-interface>
```
- **Next-hop route:** recursive lookup; preferred on multi-access (Ethernet).
- **Exit-interface only:** OK on point-to-point; on Ethernet it triggers ARP for every destination (avoid).
- **Fully specified:** both interface and next hop.
- **Default route:** `ip route 0.0.0.0 0.0.0.0 <next-hop>`.
- **Floating static route:** higher AD (e.g. 200) so it's used only if the preferred route disappears.
- **Host route:** /32.
- **Summary route:** one route covering multiple networks.

Static routes need to exist **in both directions** - return path is a very common lab fault.

## 4. Dynamic routing concepts

- **IGP vs EGP:** IGPs run inside an AS (OSPF, EIGRP, RIP); EGP between ASes (BGP).
- **Distance vector** (RIP, EIGRP) vs **link state** (OSPF, IS-IS).
- **Metric** compares routes of the same protocol; **AD** compares protocols.
- **Convergence:** time for all routers to agree after a change.

## 5. OSPFv2 (single area focus)

Link-state: every router in an area builds the same **LSDB** and runs **Dijkstra SPF** to build its routing table.

- Uses IP protocol 89; multicast `224.0.0.5` (all OSPF routers) and `224.0.0.6` (DR/BDR).
- **Cost** = reference bandwidth / interface bandwidth. Default reference = 100 Mbps, so FastEthernet and Gigabit both cost **1** - change it with `auto-cost reference-bandwidth 10000` (consistently on all routers). Total path cost = sum of outgoing interface costs.
- **Router ID (RID)** selection: `router-id` command -> highest loopback IP -> highest active physical interface IP. Set manually.
- **Areas:** Area 0 = backbone; other areas must connect to it. Single-area OSPF is the CCNA focus.

### Neighbour states
Down -> Init -> **2-Way** -> ExStart -> Exchange -> Loading -> **Full**.
On multi-access segments, routers only reach **Full** with the DR and BDR; others stay at 2-Way (this is normal).

### Hello/Dead timers
Broadcast & point-to-point: Hello **10 s**, Dead **40 s**. These must match to form a neighbour relationship.

### DR/BDR election (broadcast networks)
Highest **interface priority** (default 1; 0 = never) -> highest RID. Not pre-emptive. Not needed on point-to-point links: `ip ospf network point-to-point`.

### Neighbour adjacency requirements (checklist)
- Same **area**, same **subnet & mask**
- Matching **hello/dead** timers
- Matching **authentication** (if used)
- **Unique RIDs**
- Same **network type**/MTU, interface not **passive**
- ACLs not blocking OSPF

### LSAs to know
Type 1 Router LSA (each router, within area) · Type 2 Network LSA (DR, multi-access) · Type 3 Summary LSA (between areas).

### Route types in `show ip route`
`O` intra-area · `O IA` inter-area · `O E1/E2` external · default route can be injected with `default-information originate`.

## 6. Verification & troubleshooting

```text
show ip ospf neighbor
show ip ospf interface brief
show ip ospf interface g0/0
show ip protocols
show ip route ospf
show ip ospf database
debug ip ospf adj          ! lab only
```

| Symptom | Check |
|---|---|
| No neighbours | Layer 1/2, IP/mask, area, timers, passive-interface, network statement wildcard, ACL, auth |
| Stuck in 2-Way | Normal for DROther-DROther; otherwise check DR election |
| Stuck in ExStart/Exchange | MTU mismatch |
| Route missing | Network statement doesn't cover interface, neighbour down, summarisation/filtering |
| Suboptimal path | Reference bandwidth / interface cost / bandwidth statements |

Configs: `04-inter-vlan-routing.txt`, `05-static-routing.txt`, `06-ospf.txt`.

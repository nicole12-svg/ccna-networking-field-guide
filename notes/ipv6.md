# IPv6 Fundamentals

## 1. Basics

- 128-bit addresses, written as eight 16-bit hex groups: `2001:0db8:0000:0000:0000:0000:0000:0001`.
- **Shortening rules:** drop leading zeros in a group; replace **one** run of consecutive all-zero groups with `::` -> `2001:db8::1`.
- Prefix length notation like IPv4 CIDR (`/64`). Standard LAN subnet = **/64** (64-bit interface ID).
- No broadcast; uses **multicast** and **anycast**. ARP is replaced by **NDP** (Neighbor Discovery Protocol, ICMPv6).

## 2. Address types

| Type | Prefix | Notes |
|---|---|---|
| Global unicast | `2000::/3` (2000-3FFF) | Internet-routable |
| Unique local | `FC00::/7` (in practice `FD00::/8`) | Private, like RFC 1918 |
| Link-local | `FE80::/10` | Auto-assigned on every IPv6 interface; not routed; used for NDP, routing protocol next hops |
| Loopback | `::1/128` | |
| Unspecified | `::/128` | |
| Default route | `::/0` | |
| Multicast | `FF00::/8` | |
| Anycast | From unicast space | Same address on multiple devices; nearest answers |

### Important multicast addresses
- `FF02::1` all nodes · `FF02::2` all routers
- `FF02::5` OSPFv3 all routers · `FF02::6` OSPFv3 DR/BDR
- `FF02::A` EIGRP · `FF02::1:2` DHCPv6 relay agents/servers
- `FF02::1:FFxx:xxxx` **solicited-node** multicast (used by NDP for address resolution)

## 3. Interface ID generation

- **Manual** configuration.
- **EUI-64:** take the 48-bit MAC, split in half, insert `FFFE` in the middle, **flip the 7th bit** (U/L bit) of the first byte.
  - MAC `0200.1234.5678` -> `0000.0010 ...`: first byte 02 -> 00 after flip -> interface ID `0000:12FF:FE34:5678`.
  - Command: `ipv6 address 2001:db8:1:1::/64 eui-64`.
- **Random** (privacy extensions, common on hosts).

## 4. Address assignment methods

| Method | Who provides what |
|---|---|
| Static | Manual |
| **SLAAC** | Router sends **RA** with prefix; host builds its own interface ID. No server. |
| **Stateless DHCPv6** | SLAAC for address; DHCPv6 only for extras (DNS). RA "O" flag. |
| **Stateful DHCPv6** | DHCPv6 server assigns addresses. RA "M" flag. |

## 5. NDP message types (ICMPv6)

| Message | Type | Purpose |
|---|---|---|
| Router Solicitation (RS) | 133 | Host asks for routers |
| Router Advertisement (RA) | 134 | Router announces prefix/gateway info |
| Neighbor Solicitation (NS) | 135 | "Who has this IPv6 address?" (ARP-like); also DAD |
| Neighbor Advertisement (NA) | 136 | Reply |
| Redirect | 137 | Better next hop |

**DAD** (Duplicate Address Detection) uses NS to verify an address is unique.

## 6. Key commands

```text
ipv6 unicast-routing                      ! enable IPv6 routing on a router
ipv6 address 2001:db8:acad:1::1/64
ipv6 address fe80::1 link-local
ipv6 route 2001:db8:2::/64 2001:db8:12::2
ipv6 route ::/0 2001:db8:12::2

show ipv6 interface brief
show ipv6 route
show ipv6 neighbors
ping 2001:db8:acad:1::2
```

## 7. IPv4 vs IPv6 migration

- **Dual stack:** run both (most common).
- **Tunnelling:** carry IPv6 inside IPv4.
- **Translation (NAT64):** convert between families.

Config: `configs/topic-based-configurations/12-ipv6.txt`

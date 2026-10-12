# IPv4 Addressing & Subnetting

## 1. Fundamentals

- IPv4 address = 32 bits in four octets (dotted decimal). Subnet mask separates **network** and **host** portions.
- **Network address:** all host bits 0. **Broadcast address:** all host bits 1. **Usable hosts** = 2^h - 2.
- **Private ranges (RFC 1918):**
  - `10.0.0.0/8`
  - `172.16.0.0/12` (172.16.0.0 - 172.31.255.255)
  - `192.168.0.0/16`
- **Special ranges:** `127.0.0.0/8` loopback · `169.254.0.0/16` APIPA/link-local · `224.0.0.0/4` multicast · `0.0.0.0/0` default route.
- **Classful reference:** A = 1-126 (/8) · B = 128-191 (/16) · C = 192-223 (/24) · D = 224-239 multicast · E = 240-255 reserved.

## 2. Cheat table

| CIDR | Mask | Block size (in interesting octet) | Hosts |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /31 | 255.255.255.254 | 2 | 2 (point-to-point, RFC 3021) |
| /32 | 255.255.255.255 | 1 | host route |

Powers of 2: 1, 2, 4, 8, 16, 32, 64, 128, 256.

## 3. Subnetting method (the "magic number" way)

**Question:** What are the network, broadcast, and usable range of `192.168.10.77/27`?

1. /27 -> mask `255.255.255.224` -> interesting octet is the 4th; block size = 256 - 224 = **32**.
2. Multiples of 32: 0, 32, 64, 96...
3. 77 falls between 64 and 96 -> **Network = 192.168.10.64**.
4. Broadcast = next network - 1 = **192.168.10.95**.
5. Usable range = **.65 - .94**.

### Splitting a network
Need N subnets from a block? Borrow `b` bits where 2^b >= N. New prefix = old prefix + b.
Example: split `192.168.1.0/24` into 4 subnets -> borrow 2 bits -> /26 -> 192.168.1.0, .64, .128, .192.

## 4. VLSM (Variable Length Subnet Masking)

Use different prefix lengths so addresses aren't wasted.

**Method:**
1. List requirements, largest first.
2. Assign the first subnet at the start of the block.
3. Next subnet begins at the first address after the previous broadcast.
4. Point-to-point router links use /30 (or /31).

**Example:** from `172.16.0.0/24`
| Need | Hosts | Prefix | Subnet | Range |
|---|---|---|---|---|
| LAN A | 100 | /25 | 172.16.0.0 | .1 - .126 |
| LAN B | 50 | /26 | 172.16.0.128 | .129 - .190 |
| LAN C | 20 | /27 | 172.16.0.192 | .193 - .222 |
| WAN link | 2 | /30 | 172.16.0.224 | .225 - .226 |

## 5. Route summarisation

Find the common leading bits across contiguous networks.
`192.168.0.0/24` through `192.168.3.0/24` -> third octet 00000000-00000011 -> summary **192.168.0.0/22**.

## 6. Troubleshooting addressing

- Wrong mask on a host -> it thinks remote hosts are local (ARP for them) or vice versa.
- Default gateway outside the host's subnet -> no off-subnet traffic.
- Duplicate IPs -> intermittent connectivity.
- `169.254.x.x` on a host -> DHCP failed.
- Tools: `ping`, `traceroute`/`tracert`, `ipconfig /all`, `show ip interface brief`, `show arp` / `arp -a`.

## 7. ARP

Resolves IPv4 -> MAC on the local segment. Host sends a broadcast ARP request; the owner replies with unicast. Off-subnet traffic ARPs for the **default gateway**.

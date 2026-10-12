# Network Fundamentals

Summary notes from the CCNA 200-301 syllabus. Commands are Cisco IOS / IOS XE; availability varies by platform and version.

## 1. Network components

| Device | Layer | Role |
|---|---|---|
| Hub | 1 | Repeats bits out all ports (legacy, one collision domain) |
| Switch (L2) | 2 | Forwards frames by destination MAC; each port is its own collision domain |
| Router | 3 | Forwards packets between networks by destination IP; each interface is its own broadcast domain |
| Layer 3 switch | 2/3 | Switching plus routing (SVIs, routed ports) |
| Firewall / NGFW / IPS | 3-7 | Filters/inspects traffic by policy, application, or signature |
| Access point (AP) | 2 | Bridges wireless clients to the wired LAN |
| Wireless LAN controller (WLC) | - | Centrally manages lightweight APs |
| Endpoints / servers | - | Sources and destinations of traffic |

**Collision domain:** where frames can collide (only relevant with hubs or half-duplex).
**Broadcast domain:** the set of devices that receive a Layer 2 broadcast (bounded by routers or VLANs).

## 2. OSI vs TCP/IP

| # | OSI layer | PDU | TCP/IP layer | Examples |
|---|---|---|---|---|
| 7 | Application | Data | Application | HTTP, DNS, DHCP, SSH, SNMP |
| 6 | Presentation | Data | Application | TLS, encoding, compression |
| 5 | Session | Data | Application | Session setup/teardown |
| 4 | Transport | Segment (TCP) / Datagram (UDP) | Transport | TCP, UDP, ports |
| 3 | Network | Packet | Internet | IP, ICMP, routing |
| 2 | Data Link | Frame | Network Access | Ethernet, MAC, 802.1Q |
| 1 | Physical | Bits | Network Access | Cables, signalling |

Mnemonic (top to bottom): *All People Seem To Need Data Processing*.

**Encapsulation:** data -> segment (add L4 header) -> packet (add IP header) -> frame (add Ethernet header + FCS) -> bits. De-encapsulation is the reverse.

## 3. TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (3-way handshake: SYN, SYN-ACK, ACK) | Connectionless |
| Reliability | Sequence numbers, ACKs, retransmission | None |
| Flow control | Windowing | None |
| Overhead | Higher | Lower |
| Used by | HTTP/S, SSH, FTP, SMTP | DNS queries, DHCP, TFTP, VoIP, SNMP |

### Common ports
FTP 20/21 (TCP) · SSH 22 (TCP) · Telnet 23 (TCP) · SMTP 25 (TCP) · DNS 53 (UDP/TCP) · DHCP 67/68 (UDP) · TFTP 69 (UDP) · HTTP 80 (TCP) · NTP 123 (UDP) · SNMP 161/162 (UDP) · HTTPS 443 (TCP) · Syslog 514 (UDP)

## 4. Ethernet and cabling

- **Ethernet frame:** Preamble | Dest MAC | Src MAC | Type (or 802.1Q tag + Type) | Payload (46-1500 B) | FCS.
- **MAC address:** 48 bits, written in hex (e.g. `aaaa.bbbb.cccc`). First 24 bits = OUI (vendor).
- **Cable types:**
  - **UTP copper** (Cat5e/6): max 100 m. 10/100/1000BASE-T.
  - **Multimode fibre (MMF):** short distance, cheaper optics (e.g. 1000BASE-SX, 10GBASE-SR).
  - **Single-mode fibre (SMF):** long distance (e.g. 1000BASE-LX, 10GBASE-LR).
- **Straight-through:** unlike devices (PC-switch, switch-router). **Crossover:** like devices (switch-switch, router-router). Most modern interfaces use **Auto-MDIX**, so either cable works.
- **Duplex/speed mismatch** causes late collisions, CRC errors, and poor throughput. Check with `show interfaces`.
- **PoE:** 802.3af (15.4 W), 802.3at (30 W), 802.3bt (60/100 W).

## 5. Interface troubleshooting

`show interfaces status` / `show ip interface brief`

| Status / Protocol | Meaning |
|---|---|
| up / up | Working |
| up / down | Layer 2 problem (encapsulation, clocking, keepalives) |
| down / down | Layer 1 problem (cable, remote end shut down, no signal) |
| administratively down / down | Interface is `shutdown` |
| err-disabled | Port disabled by a security feature (port security, BPDU guard, etc.) |

Counters to watch: **input errors, CRC, runts, giants, collisions, late collisions**.

## 6. Cisco IOS CLI essentials

| Mode | Prompt | Enter with |
|---|---|---|
| User EXEC | `Router>` | Login |
| Privileged EXEC | `Router#` | `enable` |
| Global config | `Router(config)#` | `configure terminal` |
| Interface config | `Router(config-if)#` | `interface g0/0` |
| Line config | `Router(config-line)#` | `line vty 0 15` |

- `?` for help, **Tab** to complete, `Ctrl+C` / `end` to go back to privileged mode, `no` prefix to negate a command.
- `show running-config` (RAM) vs `show startup-config` (NVRAM). Save with `copy running-config startup-config` (or `write memory`).
- `show ip interface brief`, `show version`, `show interfaces`, `show vlan brief`, `show ip route`.
- Filter output: `show run | include ...`, `| section ...`, `| begin ...`.
- `reload` reboots; `erase startup-config` then `reload` returns a device to factory defaults.

## 7. Secure device management (summary)

- Use `enable secret` (hashed) rather than `enable password`; add `service password-encryption` for line passwords (weak Type 7 obfuscation, not real security).
- Prefer **SSH v2** over Telnet. Needs hostname, domain name, RSA key, local user, and `transport input ssh`.
- Add login banners, console/VTY timeouts, and login-attempt limits.
- Disable unused ports and put them in an unused VLAN.
- Full commands: `configs/topic-based-configurations/01-basic-device-config.txt` and `02-secure-management-ssh.txt`.

## 8. Lab-troubleshooting mindset

1. Start at Layer 1 and work up (or use divide and conquer from the middle).
2. Compare **expected** vs **actual** behaviour.
3. Change one thing at a time; verify; document.

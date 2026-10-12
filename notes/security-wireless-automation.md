# Security, Wireless, Architectures, QoS, Virtualisation, Cloud & Automation

## Part A - Network security fundamentals

### Concepts
- **CIA triad:** Confidentiality, Integrity, Availability.
- **Threat / vulnerability / exploit / mitigation.** Attacks: DoS/DDoS, spoofing, reconnaissance, malware, social engineering (phishing), man-in-the-middle.
- **AAA:** Authentication (who), Authorization (what), Accounting (what they did). Servers: **RADIUS** (UDP, open) and **TACACS+** (TCP 49, Cisco, separates AAA, encrypts whole body).
- **Passwords:** length, complexity, MFA; certificates and biometrics. Store hashed.
- **VPNs:** **site-to-site** (routers/firewalls, IPsec) and **remote access** (client, TLS/IPsec). IPsec provides confidentiality, integrity, authentication, anti-replay.
- **Firewalls:** stateful, next-gen firewalls (app-aware), IPS (inline detection/prevention).
- **Security programs:** user awareness, training, physical access control.

### Layer 2 attack mitigation (switch security)
| Threat | Mitigation |
|---|---|
| MAC flooding / unauthorised devices | **Port security** |
| Rogue DHCP server / DHCP starvation | **DHCP snooping** |
| ARP spoofing/poisoning | **Dynamic ARP Inspection (DAI)** (uses snooping table) |
| VLAN hopping (DTP spoofing, double tagging) | Disable DTP (`switchport nonegotiate`), explicit access/trunk modes, change native VLAN, unused ports -> unused VLAN + shutdown |
| STP manipulation | PortFast + BPDU Guard, Root Guard |
| Unauthenticated access | **802.1X** port-based auth |

#### Port security
- Limits and/or locks which MACs may use a port (static, **dynamic**, or **sticky** learned).
- **Violation modes:** `protect` (drops, silent) · `restrict` (drops + logs/counters) · `shutdown` (default; port goes **err-disabled**).
- Recover a port: `shutdown` / `no shutdown`, or `errdisable recovery cause psecure-violation`.
- `show port-security`, `show port-security interface f0/1`, `show errdisable recovery`.

#### DHCP snooping
Marks ports **trusted** (uplinks, DHCP server) or **untrusted** (default, hosts). Untrusted ports can't send DHCP server messages. Builds a binding table (MAC, IP, VLAN, port). Often rate-limits DHCP requests.

#### DAI
Validates ARP packets against the DHCP snooping binding table on untrusted ports. Uplinks are trusted.

## Part B - Wireless

### 802.11 and RF
| Standard | Marketing name | Band |
|---|---|---|
| 802.11b/g | - | 2.4 GHz |
| 802.11n | Wi-Fi 4 | 2.4 / 5 GHz |
| 802.11ac | Wi-Fi 5 | 5 GHz |
| 802.11ax | Wi-Fi 6 | 2.4 / 5 GHz (6E adds 6 GHz) |

- **2.4 GHz:** longer range, more interference; non-overlapping channels **1, 6, 11**. **5 GHz:** more channels, shorter range, higher speed.
- **SSID** = network name; **BSS** = one AP + clients; **ESS** = multiple APs sharing an SSID (roaming); **BSSID** = AP radio MAC.
- Wireless is **half-duplex, CSMA/CA**.

### Wireless security
| Method | Notes |
|---|---|
| WEP | Broken - never use |
| WPA | Legacy (TKIP) |
| **WPA2** | AES-CCMP; Personal (PSK) or Enterprise (802.1X/RADIUS) |
| **WPA3** | SAE replaces PSK handshake, stronger; protected management frames; Personal/Enterprise |

### AP and WLC architectures
- **Autonomous AP:** standalone, configured individually.
- **Lightweight AP:** managed by a **WLC** via **CAPWAP** tunnels (control + data). Split-MAC architecture.
- **Cloud-based** (e.g. Meraki), **embedded** WLC on a switch, **Mobility Express**.
- **AP modes:** Local (default), FlexConnect (local switching at branch), Monitor, Sniffer, Bridge/Mesh, Flex+Bridge.
- WLC management: **GUI/HTTPS**, SSH, console. Typical GUI tasks: create WLAN (SSID, security, VLAN/interface), QoS profile (Platinum/Gold/Silver/Bronze), WPA2/3 PSK.

## Part C - Network architectures

| Design | Description |
|---|---|
| **2-tier (collapsed core)** | Access + distribution/core combined; small/medium campus |
| **3-tier** | Access -> Distribution -> Core; large campus |
| **Spine-leaf** | Every leaf connects to every spine; data centre; predictable latency |
| WAN | Connects remote sites: MPLS, Metro Ethernet, Internet VPN |
| SOHO | One device often combines router, switch, AP, firewall |
| On-prem vs cloud | CapEx vs OpEx; control vs elasticity |

Also: **PoE** powers APs/phones; **LAN vs WAN**, **north-south** vs **east-west** traffic.

## Part D - QoS fundamentals

Prioritises traffic when links congest (voice/video are delay-sensitive).
- **Characteristics:** bandwidth, delay, jitter, loss. Voice: <150 ms one-way delay, <30 ms jitter, <1 % loss.
- **Classification & marking:** CoS (L2, 802.1Q), **DSCP** (L3). EF (46) = voice; AF = assured classes; CS = class selector; BE (0) = best effort.
- **Trust boundary:** where markings are trusted (typically the access switch or IP phone).
- **Queuing:** FIFO, WFQ, **CBWFQ**, **LLQ** (strict-priority queue for voice).
- **Congestion avoidance:** tail drop vs **WRED**.
- **Policing** (drops/remarks excess; no buffering) vs **shaping** (buffers/delays excess; outbound).
- Per-Hop Behaviours (PHB) apply the treatment per class.

## Part E - Virtualisation & cloud

- **Hypervisors:** **Type 1** (bare metal: ESXi, Hyper-V, KVM) and **Type 2** (hosted: VirtualBox, Workstation). VMs have virtual NICs, vSwitches.
- **Containers** share the host OS kernel (Docker, Kubernetes) - lighter than VMs.
- **Cloud service models:** **IaaS** (VMs/network), **PaaS** (runtime/platform), **SaaS** (finished application).
- **Deployment models:** public, private, hybrid, community.
- **Characteristics:** on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service.
- **VRF** = virtual routing tables on one router (separates tenants/traffic).

## Part F - Network automation & programmability

### SDN & controller-based networking
| Plane | Function |
|---|---|
| **Data** | Forwards traffic (ASIC) |
| **Control** | Builds forwarding tables (routing protocols, STP, ARP) |
| **Management** | Configuration/monitoring (SSH, SNMP, NETCONF) |

SDN centralises the control plane in a **controller** (e.g. **Cisco Catalyst Center / DNA Center**). **Southbound APIs** (NETCONF, RESTCONF, OpenFlow, CLI/SNMP) talk to devices; **northbound APIs** (REST) talk to apps/users. **Overlay** (VXLAN, fabric) vs **underlay** (physical IP). Intent-based networking: describe desired outcome, system implements and validates.

Traditional vs controller-based: device-by-device CLI (error-prone, config drift) vs centralised policy, automation, and assurance.

### REST APIs
| HTTP verb | CRUD | Purpose |
|---|---|---|
| POST | Create | |
| GET | Read | |
| PUT / PATCH | Update (replace / modify) | |
| DELETE | Delete | |

Response codes: 200 OK · 201 Created · 204 No Content · 400 Bad Request · 401 Unauthorized · 403 Forbidden · 404 Not Found · 500 Server Error. Authentication: basic, token, OAuth, API key. Anatomy: protocol, host, **resource** path, query parameters.

### Data formats
**JSON** (key-value, `{}` objects, `[]` arrays), **XML** (tags), **YAML** (indentation-based), **YANG** (data *models* describing config/state, used with NETCONF/RESTCONF).

```json
{ "interface": { "name": "GigabitEthernet0/1", "enabled": true, "ipv4": ["10.0.0.1/24"] } }
```

### Configuration management tools
| Tool | Language | Architecture | Style |
|---|---|---|---|
| **Ansible** | YAML playbooks | Agentless, SSH/API | Push, procedural/declarative modules |
| **Puppet** | Puppet DSL (manifests) | Agent-based | Pull, declarative |
| **Chef** | Ruby (recipes) | Agent-based | Pull |
| **Terraform** | HCL | Agentless, provider APIs | Infrastructure as code, declarative |

Benefits: consistency, speed, version control, reduced configuration drift.

Configs/examples: `11-switch-security.txt`, `13-automation-examples.md`.

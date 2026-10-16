# Network Troubleshooting Lab: NAT & DNS

## 1. Project Overview

This project involved troubleshooting a broken network in **Cisco Packet Tracer** after completing my CCNA study course. The lab required restoring connectivity between end hosts and an external web application, `timecard.com`.

The main challenge was identifying why the end hosts could not reach the required destination despite the network having an existing DNS configuration and NAT setup.

Rather than rebuilding the network, I investigated the existing configuration, tested possible causes, and made targeted changes to restore connectivity.

### Skills Demonstrated

- Cisco IOS configuration and troubleshooting
- Network Address Translation (NAT)
- Port Address Translation (PAT / NAT Overload)
- DNS resolution
- Access Control Lists (ACLs)
- IP addressing and subnetting
- Connectivity testing using `ping`
- Network troubleshooting and verification
- Cisco Packet Tracer

## 2. The Problem

The network contained end hosts, a router, a DNS server, and an external web application.

Initially, the end hosts could not successfully reach the required `timecard.com` web application.

The DNS server contained a record mapping the hostname to an IP address:

| Hostname | IP Address |
|---|---|
| `timecard.com` | `214.1.1.10` |

However, the required connectivity was not working. I needed to determine whether the problem was related to DNS, routing, NAT, or another part of the network configuration.

## 3. Initial Investigation

I began by examining the existing network configuration instead of immediately changing multiple settings.

The investigation focused on:

- IP addresses and subnet configurations.
- DNS server settings and the `timecard.com` record.
- Router interfaces and their inside/outside NAT roles.
- The existing NAT configuration and address pool.
- Connectivity between the end hosts and external destinations.

This helped narrow the investigation toward the router's NAT configuration.

## 4. Investigating the NAT Configuration

The router was using a pool-based NAT rule:

```cisco
ip nat inside source list NAT_LIST pool NAT_POOL
```

This configuration uses addresses defined in `NAT_POOL` to translate traffic identified by `NAT_LIST`.

During the investigation, I found that the pool contained private IP addresses. Private addresses are not globally routable across the public Internet, so they are not suitable as public translation addresses for ordinary Internet access.

This made the existing NAT configuration a key suspect.

### Initial approach

I first investigated the existing pool-based configuration and considered whether correcting it would restore connectivity. The existing setup did not provide the required connectivity, so I switched to interface-based PAT.

## 5. Implementing the Fix

I removed the pool-based NAT rule:

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
```

I also removed the NAT pool configuration because it was no longer needed for the new approach.

I then configured PAT using the router's Internet-facing interface:

```cisco
ip nat inside source list NAT_LIST interface g0/0/1 overload
```

The interface roles in the final topology were:

| Interface | Role | NAT Configuration |
|---|---|---|
| `G0/0/0` | LAN-facing | `ip nat inside` |
| `G0/0/1` | Internet-facing | `ip nat outside` |

The `overload` keyword enables Port Address Translation (PAT), allowing multiple internal hosts to share the IP address of the specified interface by using different transport-layer port numbers.

### Why this worked

Instead of translating internal traffic using addresses from the private NAT pool, the router used the IP address assigned to its Internet-facing interface.

**Key lesson:** The interface specified in this command determines which interface IP address is used for translation. The `ip nat inside` and `ip nat outside` commands separately identify the NAT boundary.

## 6. Verification and Testing

After applying the change, I retested connectivity from the end hosts.

The following results were observed:

- **Internet connectivity:** End hosts could successfully ping external destinations.
- **DNS resolution:** `timecard.com` resolved to `214.1.1.10`.
- **Hostname connectivity:** Pinging `timecard.com` returned successful ICMP replies.
- **Web application access:** The browser successfully reached `timecard.com` and displayed the **"Lab Complete"** message.

These tests demonstrated that the required network connectivity and web application access had been restored.

### Understanding the DNS test

When an end host runs:

```text
ping timecard.com
```

it first resolves the hostname through DNS. The resulting IP address is then used as the destination of the ICMP echo requests.

In this lab:

```text
timecard.com → 214.1.1.10
```

The successful ping demonstrated successful hostname resolution and connectivity to the resolved address.

DNS resolution alone does not guarantee reachability; the underlying network path must also work.

## 7. Troubleshooting Outcome

The network's end-to-end connectivity was restored after replacing the pool-based NAT rule with interface-based PAT.

However, Packet Tracer's Activity Results panel continued to report that the activity was incomplete, despite the successful browser test.

This highlighted an important distinction: **a network can pass functional connectivity tests while an automated assessment still reports unmet configuration requirements.**

The remaining assessment discrepancy was not conclusively resolved during this troubleshooting session.

## 8. Lessons Learned

1. **Troubleshoot systematically.** Investigate addressing, DNS, routing, and NAT rather than changing several configurations at once.
2. **Understand NAT and PAT.** Dynamic NAT uses a configured address pool, while PAT allows multiple internal hosts to share a translated address using port numbers.
3. **Choose appropriate translation addresses.** Private addresses in a NAT pool are not suitable as globally routable public Internet source addresses.
4. **Understand interface roles.** NAT inside/outside designations and the interface specified in a translation rule serve different purposes.
5. **Separate DNS resolution from connectivity.** DNS maps names to addresses; routing and NAT enable packets to reach their destinations and receive replies.
6. **Verify with multiple tests.** Successful pings, hostname resolution, and browser access provide complementary evidence that a network is functioning.
7. **Distinguish functional success from assessment success.** Automated lab grading may check specific configuration requirements in addition to end-to-end connectivity.

## 9. Conclusion

This lab provided practical experience troubleshooting a network with NAT and DNS issues in Cisco Packet Tracer.

By examining the existing configuration, identifying the unsuitable pool-based NAT setup, and replacing it with interface-based PAT, I restored connectivity from the end hosts to the required web application.

This project represents my continued hands-on practice following CCNA study, with a focus on diagnosing network faults, applying targeted fixes, and documenting the reasoning behind each change.

---

**Environment:** Cisco Packet Tracer  
**Project type:** Network troubleshooting  
**Focus areas:** NAT, PAT, DNS, ACLs, routing, and connectivity verification

## Project Files

- [`configs.txt`](configs.txt) — relevant NAT configuration commands and explanations.
- [`images/topology.png`](images/topology.png) — network topology screenshot.

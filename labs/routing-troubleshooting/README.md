# Network Troubleshooting: Restoring Access to `networks.com`

## Project Overview

This project documents a network troubleshooting scenario in which users could not access `networks.com` after their PCs were changed to use Google's public DNS server instead of the local DNS server.

The investigation focused on verifying network reachability, tracing the path toward the DNS network, identifying a routing loop, and restoring connectivity without violating the lab's configuration constraints.

## Problem Statement

Users reported that they were unable to reach the website `networks.com`. The PCs had recently been configured to use Google's DNS server rather than the local DNS server.

**Objective:** Investigate the issue and restore access to `networks.com` using the Admin PC.

## Lab Constraints

The troubleshooting process had to respect the following restrictions:

- Do not change the administrative state of any interfaces.
- Do not add or remove static routes.
- Do not enable any new routing protocols.
- Do not add or remove ACLs.
- Do not modify OSPF configurations.
- Do not configure NAT.

The lab also specified that SW4 was on the `192.168.100.0/28` network.

## Tools and Commands Used

- `ipconfig /all` — identify the Admin PC's default gateway and network configuration.
- `ping` — test reachability to the gateway and DNS server.
- `ssh` — remotely access network devices for troubleshooting.
- `show ip route` — inspect routing-table entries and route sources.
- `traceroute` — trace the path toward the DNS network and help locate the routing loop.
- `ip route` — adjust the administrative distance of the existing static route.
- `clear ip route *` — clear routing-table entries so the routing table could reconverge.
- `copy running-config startup-config` (or `write memory`) — save the configuration.

## Troubleshooting Process

### 1. Verified the Admin PC's Network Configuration

I started on the Admin PC and ran:

```text
ipconfig /all
```

I used the output to identify the PC's default gateway. I then tested reachability to the gateway:

```text
ping <default-gateway-ip>
```

The gateway responded successfully, confirming that the Admin PC could reach its local gateway.

### 2. Tested Reachability to the DNS Server

I connected to the gateway device over SSH and attempted to ping the DNS server's IP address. The ping failed, indicating that the issue was not limited to the website or browser: the DNS server's network was not reachable from that device.

### 3. Inspected the Routing Table

I checked the gateway's routing table:

```text
show ip route
```

The routing table contained an OSPF route toward the DNS network through Router1. I then used `traceroute` to investigate the path and found that traffic was looping between Router1 and Router3.

### 4. Investigated the Route Through Router1 and Router3

I connected to Router1 over SSH and checked its routing table. I found that a static route toward the DNS network pointed through Router3.

Next, I inspected Router3's interfaces and discovered that its interface connected to the DNS server was **administratively down**.

Because the lab explicitly prohibited changing interface administrative states, I did not enable the interface. I instead looked for an alternative path that would comply with the constraints.

### 5. Adjusted the Administrative Distance of the Existing Static Route

The existing static route had a default administrative distance of `1`, which made it preferred over the OSPF route when both routes were available for the same destination prefix.

Rather than adding or removing a route, I modified the existing static route to use an administrative distance of `112`:

```text
ip route 8.8.8.0 255.255.255.240 192.168.200.2 112
```

This made the static route less preferred than the OSPF route (administrative distance `110`). The router could therefore select the alternative route through Router2.

I then cleared the routing table to trigger route relearning/reconvergence:

```text
clear ip route *
```

**Note:** The command above was used in the lab environment. Clearing routes can temporarily remove entries while the routing protocols relearn them, so the routing table should be checked again afterward.

### 6. Saved the Configuration

After the route change, I saved the running configuration to startup configuration so the change would persist after a reload:

```text
copy running-config startup-config
```

On platforms that support it, `write memory` can be used as an alternative.

## Verification and Results

After the routing table reconverged, I repeated the reachability test to the DNS server. The ping succeeded.

I then returned to the Admin PC and tested access to `networks.com`. I also checked from another PC. The website was reachable, and the lab's completion condition was satisfied.

**Result:** Connectivity to `networks.com` was restored without changing any interface's administrative state, adding or removing static routes, modifying OSPF, changing ACLs, enabling protocols, or configuring NAT.

## Root Cause

The DNS network was unreachable because the preferred static route led traffic toward Router3, whose interface connected to the DNS server was administratively down. The resulting forwarding path contributed to a routing loop between Router1 and Router3.

## Resolution

The administrative distance of the existing static route to `8.8.8.0/28` was increased to `112`. This allowed the OSPF-learned alternative route, with administrative distance `110`, to be preferred. Connectivity to the DNS server and the website was then restored.

## Key Takeaways

- Verify connectivity layer by layer before changing configurations.
- A successful ping to the default gateway does not guarantee reachability to remote networks.
- Routing tables and `traceroute` are useful for identifying routing loops and unexpected paths.
- Administrative distance determines which route source is preferred when routes to the same destination prefix are available.
- An existing static route can be made less preferred without adding or removing routes.
- Troubleshooting constraints matter: the correct fix must restore service while respecting the permitted scope of changes.

## Evidence

| Evidence | Description |
|---|---|
| [Topology of the network](images/topology.png) | Shows the topology. |
| [Image before the fix](images/Before.png) | Network state before connectivity was restored. |
| [Lab completion evidence](images/Lab-complete.png) | Screenshot showing the successful result or completed-lab status. |



## Files in This Repository

```text
.
├── README.md
├── configs
│   └── troubleshooting-commands.txt
└── images
    ├── topology.png
    ├── Before.png
    └── Lab-complete.png
```

The configuration notes file contains the key commands used during troubleshooting. It is a record of the commands relevant to this fix, not a complete export of every device's running configuration.

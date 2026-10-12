# CCNA Networking Field Guide & Troubleshooting Labs

A practical networking reference and hands-on troubleshooting portfolio built after completing **Jeremy's IT Lab — Free CCNA 200-301 course**. This repository brings together concise concept summaries, Cisco IOS command references, lab configurations, and documented troubleshooting cases.

> **Learning resource credit:** The course that guided my studies is [Jeremy's IT Lab — Free CCNA 200-301 playlist](https://www.youtube.com/watch?v=H8W9oMNSuwo&list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ). This is an independent learning project and is not affiliated with or endorsed by Jeremy's IT Lab or Cisco.

## Purpose

My goal is to turn my handwritten course notes and practical lab experience into a clear, reusable technical reference—and to demonstrate that I can apply networking concepts, troubleshoot faults methodically, explain root causes, and verify fixes.

## What's in this repository?

- **Networking field guide:** Topic summaries, key concepts, common Cisco IOS commands, example configurations, and verification tips.
- **Troubleshooting labs:** Intentionally broken Packet Tracer topologies investigated and repaired through a structured troubleshooting process.
- **Configuration examples:** Relevant snippets from lab solutions, with explanations of what each change does.
- **Evidence:** Topology screenshots and verification results showing the final working state.
- **Lessons learned:** Notes on root causes, diagnostic commands, failed hypotheses, and what I would check first next time.

## Topics covered

The guide and labs are organised around the CCNA-level topics I studied, including:

- Network fundamentals, network devices, interfaces, cabling, and the OSI/TCP-IP models
- Cisco IOS CLI, basic device configuration, and secure device management
- Ethernet switching, MAC address learning, VLANs, access ports, and trunking
- IPv4 addressing, subnetting, VLSM concepts, and IPv6 fundamentals
- Inter-VLAN routing, static routing, and dynamic routing concepts including OSPF
- Spanning Tree Protocol (STP), EtherChannel, and Layer 2 redundancy
- DHCP, DHCP relay, DNS, and common IP services
- NAT, PAT, and access control lists (ACLs)
- Network security fundamentals and switch security features
- Wireless networking, network architectures, and QoS fundamentals
- Virtualisation, cloud concepts, and network automation/programming fundamentals

The exact commands and features available depend on the device model, IOS/IOS XE version, and lab environment. Configurations are tested or described in the context of the specific lab rather than presented as universal production templates.

## How I troubleshoot

For each troubleshooting case, I aim to document the work in a consistent way:

1. **Scenario:** Describe the topology and expected network behaviour.
2. **Symptoms:** Record what fails, which devices are affected, and the tests that reproduce the issue.
3. **Investigation:** Show the relevant commands and observations.
4. **Hypotheses and attempts:** Explain what I suspected, what I tested, and what each result revealed.
5. **Root cause:** Identify the actual configuration or design issue.
6. **Fix:** Include the necessary configuration changes and explain why they work.
7. **Verification:** Show successful connectivity tests, command output, or a screenshot of the working topology.
8. **Lessons learned:** Summarise the diagnostic approach and useful takeaways.

I try to make the **smallest justified change**, verify the result, and keep the documentation clear enough for another learner to reproduce the investigation.

## Repository structure

The repository will grow as I organise the guide and publish completed troubleshooting projects. The intended structure is:

```text
ccna-networking-field-guide/
├── README.md
├── notes/
│   ├── network-fundamentals.md
│   ├── ipv4-subnetting.md
│   ├── ipv6.md
│   ├── switching-vlans-trunking.md
│   ├── stp-etherchannel.md
│   ├── routing-ospf.md
│   ├── dhcp-dns.md
│   ├── nat-acls.md
│   └── security-wireless-automation.md
├── configs/
│   └── topic-based-configurations/
└── labs/
    ├── lab-01/
    │   ├── README.md
    │   ├── topology.png
    │   └── configs/
    └── lab-02/
        ├── README.md
        ├── topology.png
        └── configs/
```

*This is the planned organisation; folders and files will be added as the material is published.*

## Tools and technologies

- Cisco Packet Tracer
- Cisco IOS-style command-line configuration
- IPv4 and IPv6 addressing
- Routing and switching concepts
- Git and GitHub for version control and documentation
- Screenshots and CLI evidence for lab verification

## Learning resource

- [Jeremy's IT Lab — Free CCNA 200-301 course playlist](https://www.youtube.com/watch?v=H8W9oMNSuwo&list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ)

The course provided the learning structure. The summaries, lab investigations, explanations, and project documentation in this repository are my own work unless otherwise credited.

## Disclaimer

This repository is for learning and lab practice. Example IP addresses, passwords, interface names, and configurations may need to be adapted to a particular topology or platform. Do not deploy a lab configuration into a production network without reviewing the design, security implications, and platform-specific requirements. No real credentials, private keys, or sensitive network details should be committed here.

---

**Focus:** Learn the concepts, configure the network, troubleshoot the fault, verify the fix, and document the evidence.

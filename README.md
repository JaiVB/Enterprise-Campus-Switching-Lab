# Multi-Tier Enterprise Campus Switching Architecture

![Topology Diagram](topologies/diagrams/campus-topology.png)

## Overview
This repository contains the design, configuration, and verification artifacts for a high-availability, three-tier enterprise campus switching network built inside Cisco Packet Tracer. The network implements a deterministic hierarchical design with active-standby redundancy across Core, Distribution, and Access layers.

---

## Architectural Highlights

### 1. Spanning Tree Optimization (PVST+ / Rapid-PVST)
* **Deterministic Root Bridges:** Bridge priorities (`spanning-tree vlan <id> priority 4096 / 8192`) configured to guarantee `SW-CORE-01` as primary root for odd VLANs and `SW-CORE-02` as primary root for even VLANs.
* **Edge Protection:** `spanning-tree portfast` and `spanning-tree bpduguard enable` applied across all user-facing access ports to eliminate Layer 2 loop injection and TCN storms.

### 2. High Availability & Link Aggregation (LACP)
* Multi-link IEEE 802.3ad LACP EtherChannels (`channel-group mode active`) provisioned between Core and Distribution switches.
* Bundled bandwidth with automatic failover preventing STP convergence delays during single-link physical faults.

### 3. VLAN Segmentation & Inter-VLAN Routing
* IEEE 802.1Q trunking configured with custom Native VLAN (VLAN 999) to mitigate VLAN hopping attacks.
* Layer 3 routing handled via Switch Virtual Interfaces (SVIs) on the Core/Distribution layer with route summarization.

| VLAN ID | Subnet | Purpose | Gateway |
|:---|:---|:---|:---|
| **10** | `10.10.10.0/24` | Management | `10.10.10.1` |
| **20** | `10.10.20.0/24` | Engineering | `10.10.20.1` |
| **30** | `10.10.30.0/24` | Sales / Corporate | `10.10.30.1` |
| **999** | N/A | Native / Blackhole | N/A |

### 4. Switch Hardening & Access Control
* **Port Security:** Access ports restricted using `switchport port-security maximum 2` and `switchport port-security mac-address sticky` with `violation shutdown`.
* **Administrative Isolation:** Unused ports administratively shut down (`shutdown`) and assigned to an isolated parking-lot VLAN (`switchport access vlan 666`).
* **Secure Management:** SSHv2 enforced with 2048-bit RSA keys, Telnet disabled across `line vty 0 15` via `transport input ssh`, and local AAA authentication with encrypted secrets (`enable secret`).

---

## Verification & Validation Runbook

Run these commands on the CLI to verify active redundancy and security policies:

```bash
# Verify spanning tree root status and port states
show spanning-tree brief
show spanning-tree root

# Verify EtherChannel bundling and LACP status
show etherchannel summary

# Check trunk links and native VLAN enforcement
show interfaces trunk

# Verify port security state and sticky MAC entries
show port-security
show port-security address
topologies/: Contains .pkt simulation files and export diagrams.

configs/: Clean running configurations categorized by hierarchical tier.

docs/: Supplementary IP address allocation charts and protocol verification logs.

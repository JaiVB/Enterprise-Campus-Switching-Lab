# Multi-Tier Enterprise Campus Switching Architecture

> [!WARNING]
> ### 🚧 Work in Progress / Under Active Construction 🚧
> This portfolio project is currently undergoing active configuration, baseline validation, and documentation updates. Device configurations and verification outputs are being committed iteratively.

[![Project Status: In Progress](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge&logo=git)](https://github.com/)
[![Network Simulator: Cisco Packet Tracer](https://img.shields.io/badge/Simulator-Packet%20Tracer%208.x-blue?style=for-the-badge&logo=cisco)](https://www.netacad.com/courses/packet-tracer)
[![Design Tier: Core-Dist-Access](https://img.shields.io/badge/Architecture-3--Tier%20Hierarchical-darkgreen?style=for-the-badge)](https://en.wikipedia.org/wiki/Hierarchical_internetworking_model)

---

## 📌 Project Overview
This repository contains the architecture, running configurations, and verification artifacts for a resilient, three-tier enterprise campus Local Area Network (LAN). 

Modeled in Cisco Packet Tracer, the topology implements high availability, loop-free Layer 2 paths via deterministic Spanning Tree tuning, multi-link capacity via LACP EtherChannels, clean VLAN separation with Layer 3 SVIs, and hardened switch access controls.

```
       +--------------------+          +--------------------+
       |    SW-CORE-01      |<========>|    SW-CORE-02      |   CORE TIER
       +--------------------+   LACP   +--------------------+
          \\        //                    \\        //
           \\      //                      \\      //
            \\    //                        \\    //
       +--------------------+          +--------------------+
       |    SW-DIST-01      |<========>|    SW-DIST-02      |   DISTRIBUTION TIER
       +--------------------+   LACP   +--------------------+
          ||        ||                    ||        ||
          ||        ||                    ||        ||
       +--------------------+          +--------------------+
       |    SW-ACC-01       |          |    SW-ACC-02       |   ACCESS TIER
       +--------------------+          +--------------------+
         [Workstations/PCs]              [Workstations/PCs]
```

---

## 🎯 Implementation Goals & Progress Tracker

- [x] Hierarchical 3-tier physical cabling and port map schema
- [x] 802.1Q trunking with custom native VLAN isolation (VLAN 999)
- [x] LACP Link Aggregation (`channel-group mode active`) on inter-switch uplinks
- [/] **In Progress:** PVST+ Root Bridge deterministic election (Odd/Even primary split)
- [/] **In Progress:** Inter-VLAN Routing via Distribution Layer SVIs & default gateways
- [ ] Port Security enforcement (`sticky` MACs, max limit 2, violation shutdown)
- [ ] Switch hardening (SSHv2, parking lot VLAN 666 for unused ports, BPDU Guard, PortFast)
- [ ] Full topology testing (failover drills, TCN suppression, link degradation scenarios)

---

## 📐 Network Architecture & Protocol Design

### 1. Spanning Tree Optimization (PVST+ / Rapid-PVST)
* **Deterministic Root Bridge Tuning:** Avoids default election roulette by explicitly setting priorities:
  * `SW-CORE-01`: Primary root for Odd VLANs (`priority 4096`), Secondary for Even VLANs (`priority 8192`).
  * `SW-CORE-02`: Primary root for Even VLANs (`priority 4096`), Secondary for Odd VLANs (`priority 8192`).
* **Edge Loop & Topology Protection:** Access ports running `spanning-tree portfast` and `spanning-tree bpduguard enable` to immediately drop unauthorized BPDUs and avoid unnecessary Spanning Tree TCNs.

### 2. Link Aggregation (IEEE 802.3ad LACP)
* Inter-switch uplinks are combined into multi-port port channels using standard LACP (`channel-group <id> mode active`).
* Provides load sharing, increased aggregate uplink capacity, and sub-second failover without triggering STP reconvergence when individual physical members fail.

### 3. VLAN Schema & Segmentation
To mitigate VLAN-hopping and double-tagging exploits, Native VLAN 1 is removed from active service.

| VLAN ID | Name | Subnet | Gateway (SVI) | Purpose |
|:---|:---|:---|:---|:---|
| **10** | Management | `10.10.10.0/24` | `10.10.10.1` | Switch In-Band Management / SSH |
| **20** | Engineering | `10.10.20.0/24` | `10.10.20.1` | Engineering Endpoints |
| **30** | Corporate | `10.10.30.0/24` | `10.10.30.1` | General Staff & Workstations |
| **666** | Parking_Lot | Unrouted | None | Disabled/Unused Port Quarantine |
| **999** | Native_Trunk | Unrouted | None | 802.1Q Native Trunk Encapsulation |

### 4. Switch Hardening & Access Control
* **Port Security:** Bound to host-facing ports:
  ```ios
  switchport mode access
  switchport port-security
  switchport port-security maximum 2
  switchport port-security mac-address sticky
  switchport port-security violation shutdown
  ```
* **Attack Surface Reduction:** All unused access ports mapped to `vlan 666` and issued `shutdown`.
* **Administrative Security:**
  * Console and VTY access encrypted (`transport input ssh`, local AAA user lookup).
  * RSA key pair generated at 2048 bits for SSHv2 enforcement.
  * Secrets protected with `service password-encryption` and `enable secret`.

---

## 🛠️ Verification & Runbook

Validate state and failover readiness via CLI:

```bash
# Verify Layer 2 Root Bridge and STP topology states
show spanning-tree brief
show spanning-tree root

# Verify LACP state and member interface bundling
show etherchannel summary
show etherchannel port-channel

# Validate trunk encapsulation, allowed VLANs, and active Native VLAN
show interfaces trunk

# Check port-security status, violations, and dynamically learned sticky MACs
show port-security
show port-security address
```

---

## 📂 Repository Structure

```text
├── .gitignore
├── README.md
├── topologies/
│   ├── enterprise-switching-campus.pkt      <-- Packet Tracer lab file
│   └── diagrams/
│       └── campus-topology.png              <-- Topology diagram
├── configs/
│   ├── core/
│   │   ├── SW-CORE-01.cfg
│   │   └── SW-CORE-02.cfg
│   ├── distribution/
│   │   ├── SW-DIST-01.cfg
│   │   └── SW-DIST-02.cfg
│   └── access/
│       ├── SW-ACC-01.cfg
│       └── SW-ACC-02.cfg
├── docs/
│   ├── ip-schema.md
│   └── verification-commands.md
└── scripts/
    └── Core-Switch-Configs.txt

```
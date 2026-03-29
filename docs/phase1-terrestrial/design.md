# Design Document — Phase 1: Terrestrial Testbed

| | |
|---|---|
| Document ID | ION-DTN-DES-P1-001 |
| Version | 0.1.0 |
| Status | Draft |
| Classification | Public |
| Licence | Apache License, Version 2.0 |

## Document Control

### Authorship

| Role | Name | Organisation |
|------|------|-------------|
| Lead Author | [name] | AMSAT-UK |
| Contributing Author | [name] | AMSAT-DL |
| Technical Review | [name] | [organisation] |

### Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-03-28 | [name] | Initial draft — Phase 1 design extracted from master design document |

### Distribution

This document is released under the Apache License, Version 2.0 and may be freely distributed.

### Review and Approval

| Action | Name | Date | Signature |
|--------|------|------|-----------|
| Prepared by | [name] | | |
| Reviewed by | [name] | | |
| Approved by | [name] | | |

---

## Introduction

This document describes the Phase 1 (Terrestrial Testbed) design for the ION-DTN Cislunar Demonstration project. Phase 1 is the active, unconditional phase that delivers a ground-based ION-DTN network using COTS hardware and open-source software.

This document should be read in conjunction with the [Master Design Document](../master/design.md), which contains the full system architecture, all 12 component designs with interfaces, all data models, all 65 correctness properties, error handling tables, and the complete testing strategy. This phase-specific document covers deployment details, CLA usage, link emulation profiles, and testing considerations unique to Phase 1.

## Phase 1 Deployment

### Docker Compose Topologies

Phase 1 uses Docker Compose for reproducible testbed deployment on COTS hardware (commodity PCs, Raspberry Pi).

#### 2-Node Point-to-Point Topology

The testbed starts with a 2-node point-to-point configuration mirroring ION's `configs/2node-stcp/` example. This validates basic Bundle Protocol operation, bundle exchange, and store-and-forward behaviour before expanding to multi-node topologies.

```
docker/
├── docker-compose.2node.yml     # Point-to-point (Phase 1 start)
```

- Node 1: `dtn://G4DPZ-1` — ION container with STCP/TCP CLA
- Node 2: `dtn://DL1XYZ-1` — ION container with STCP/TCP CLA
- Inter-node link: TCP/UDP over Docker bridge network

#### 3-Node Multi-Hop Topology

After validating point-to-point, the testbed expands to 3+ nodes to exercise Contact Graph Routing multi-hop routing, similar to ION's `configs/3node-stcp-ltp/` example.

```
docker/
├── docker-compose.3node.yml     # Multi-hop CGR (Phase 1 expanded)
├── docker-compose.emulated.yml  # With link emulation profiles
```

- Node 1: `dtn://G4DPZ-1` — Source node
- Node 2: `dtn://DL1XYZ-1` — Relay node (CGR forwarding)
- Node 3: `dtn://F5DEF-1` — Destination node
- Link Emulator container applies `tc`/`netem` profiles to inter-node links

### Point-to-Point First Approach

Phase 1 follows a deliberate progression:

1. **2-node STCP**: Validate basic bundle exchange, store-and-forward, BPSec integrity
2. **2-node with disruption**: Add link disruption simulation, verify bundle storage and forwarding on restoration
3. **3-node CGR**: Add relay node, validate multi-hop Contact Graph Routing
4. **3-node with emulation**: Layer link emulation profiles (LEO, HEO, cislunar) on the 3-node topology
5. **AX.25 CLA**: Add AX.25 FM/AFSK 1200 baud links between nodes (terrestrial packet radio simulation)

## Phase 1 Link Emulation Profiles

Phase 1 provides four pre-built link emulation profiles stored as TOML files:

### `terrestrial_afsk.toml`
- Delay: <1ms
- Bandwidth: 1.2 kbps (FM/AFSK 1200 baud)
- Jitter: minimal
- Loss: 0.5%
- Disruption: none (continuous link)
- Purpose: Simulates terrestrial VHF packet radio links between amateur stations

### `leo_pass.toml`
- Delay: 5–15ms
- Bandwidth: 9.6 kbps (BPSK 9K6)
- Jitter: 2ms
- Loss: 1%
- Disruption: 10-minute pass with 80-minute gap (LEO orbital period)
- Purpose: Simulates a low Earth orbit amateur satellite pass

### `heo_pass.toml`
- Delay: 50–500ms (variable with orbital geometry)
- Bandwidth: 4.8 kbps
- Jitter: 10ms
- Loss: 2%
- Disruption: Long pass with variable geometry, periodic gaps
- Purpose: Simulates a highly elliptical orbit pass (e.g., Molniya-type)

### `cislunar.toml`
- Delay: 1300ms (one-way light delay to Moon)
- Bandwidth: 1 kbps (BPSK)
- Jitter: 20ms
- Loss: 0.5%
- Disruption: Periodic 45-minute lunar occlusion windows
- Purpose: Simulates cislunar link conditions for protocol validation before Phase 3

## Phase 1 Convergence Layer Adapters

Phase 1 uses the following CLAs:

| CLA | Protocol | Data Rate | Use Case |
|-----|----------|-----------|----------|
| TCP (STCP) | TCP/IP | Network speed | Primary inter-node transport in Docker testbed |
| UDP | UDP/IP | Network speed | Alternative transport for testing |
| AX.25 (FM/AFSK) | AX.25 v2.2 | 1200 baud | Terrestrial packet radio simulation (VHF 2m band) |
| Proximity-1 | CCSDS 211.0 | Configurable | Optional — short-range relay simulation |

The AX.25 CLA in Phase 1 uses FM/AFSK modulation at 1200 baud, simulating terrestrial VHF packet radio links. This exercises the custom AX.25 v2.2 CLA (C extension) and bundle fragmentation for constrained links before the higher-rate BPSK 9K6 mode is used in Phase 2.

## Phase 1 Testing

Phase 1 integration tests focus on:

### 2-Node Integration Tests
- Bundle exchange over STCP between two Docker containers
- Store-and-forward: send bundle during link disruption, verify delivery after restoration
- BPSec integrity verification end-to-end
- Configuration round-trip: TOML → ION native → start nodes → exchange bundles
- Bundle fragmentation over simulated AFSK 1200 baud link

### 3-Node Integration Tests
- Multi-hop CGR routing: bundle from Node 1 → Node 2 (relay) → Node 3
- CGR path selection with multiple available routes
- Link emulation profile application and measurement (delay, loss, bandwidth)
- Contact plan scheduling: bundles queued during no-contact, forwarded when contact opens
- Telemetry collection from all three nodes via aggregator

All Phase 1 tests run in Docker containers on COTS hardware without requiring physical radio equipment or space segment access.

For component interfaces, data models, correctness properties, and the full testing strategy, refer to the [Master Design Document](../master/design.md).

---

## Licence

Copyright 2026 AMSAT-UK & AMSAT-DL

Licensed under the Apache License, Version 2.0 (the "Licence"); you may not use this file except in compliance with the Licence. You may obtain a copy of the Licence at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the Licence is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the Licence for the specific language governing permissions and limitations under the Licence.

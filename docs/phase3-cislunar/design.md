# Design Document — Phase 3: Cislunar Payload

| | |
|---|---|
| Document ID | ION-DTN-DES-P3-001 |
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
| 0.1.0 | 2026-03-28 | [name] | Initial draft — Phase 3 design extracted from master design document |

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

This document describes the Phase 3 (Cislunar Payload) design for the ION-DTN Cislunar Demonstration project. Phase 3 is a proposal subject to ESA ARTES (Advanced Research in Telecommunications Systems) programme review and approval. All Phase 3 design elements are conditional on ESA approval and ARTES funding.

This document should be read in conjunction with the [Master Design Document](../master/design.md), which contains the full system architecture, all 12 component designs with interfaces, all data models, all 65 correctness properties, error handling tables, and the complete testing strategy. This phase-specific document covers deployment details, CLA usage, resource constraints, CGR with lunar occlusion, and testing considerations unique to Phase 3.

## Phase 3 Deployment

### Cislunar Payload on Radiation-Tolerant COTS

The cislunar payload runs ION-DTN on radiation-tolerant COTS hardware, designed as a rideshare or secondary payload opportunity on ESA or European commercial missions.

- **Platform:** Radiation-tolerant COTS flight computer
- **Operating System:** Linux (embedded, radiation-hardened kernel configuration)
- **ION-DTN:** Compiled from source for the target architecture
- **Addressing:** `ipn:200` scheme for compact, bandwidth-efficient addressing
- **Communication:** S-band or L-band amateur radio allocation (subject to IARU coordination)

## Phase 3 Convergence Layer Adapters

| CLA | Protocol | Data Rate | Use Case |
|-----|----------|-----------|----------|
| LTP | Licklider Transmission Protocol | ~1 kbps | Cislunar link, calibrated for ~2.5s RTT |

The LTP CLA in Phase 3 is calibrated for the cislunar round-trip time of approximately 2.5 seconds. Retransmission timers are set to account for the current RTT (Property 32), ensuring reliable transport over the long-delay link.

## Phase 3 Resource Constraints

The cislunar environment imposes the most demanding constraints in the project:

| Resource | Constraint | Design Impact |
|----------|-----------|---------------|
| Power | 10W average | Consistent with secondary payload or rideshare power allocations; DTN operations must stay within budget |
| Storage | 4 GB non-volatile minimum | Bundle buffering during lunar occlusion periods (up to 45 minutes) |
| Radiation | Cislunar transit and lunar orbit environment | Radiation-tolerant COTS hardware or software-based error mitigation required |
| SEU handling | Single-event upsets in memory | Error correction initiated on detection; event logged (Requirement 7.5) |
| Link delay | ~1.3s one-way, ~2.5s RTT | LTP retransmission timers calibrated to RTT; CGR accounts for propagation delay |
| Link occlusion | Periodic 45-minute lunar occlusion | Bundles stored during occlusion, forwarded within 30s of reacquisition (Property 34) |

### SEU Handling

When the cislunar payload detects a single-event upset (SEU) in memory:
1. Initiate error correction (ECC or software-based scrubbing)
2. Log the SEU event with timestamp, memory address, and correction status
3. If correction fails, isolate the affected memory region and continue operation in degraded mode
4. Report SEU events via telemetry to ground stations

## Phase 3 CGR with Lunar Occlusion Windows

Contact Graph Routing in Phase 3 must account for periodic link occlusion by the Moon:

- The contact plan includes lunar occlusion windows as no-contact periods
- CGR shall not schedule bundle forwarding during occlusion periods (Property 33)
- Bundles destined for the cislunar payload are stored at the last relay node (ground station or CubeSat) during occlusion
- On link reacquisition after occlusion, forwarding resumes within 30 seconds (Property 34)
- The unified contact plan spans terrestrial, CubeSat, and cislunar segments, with occlusion windows modelled as gaps in the cislunar contact schedule

## Phase 3 Testing Considerations

Phase 3 testing builds on Phase 1 and Phase 2 infrastructure:

- **Cislunar emulation profile:** Use the `cislunar.toml` emulation profile in the Phase 1 testbed to simulate cislunar conditions (1300ms delay, 20ms jitter, 1 kbps, 45-minute occlusion windows)
- **LTP RTT calibration testing:** Verify LTP retransmission timers are correctly calibrated for ~2.5s RTT (Property 32)
- **Lunar occlusion testing:** Verify CGR respects occlusion windows and does not schedule forwarding during occlusion (Property 33)
- **Store and resume testing:** Verify bundles stored during occlusion are forwarded within 30s of reacquisition (Property 34)
- **End-to-end multi-segment testing:** Test bundle delivery from terrestrial node → CubeSat relay → cislunar payload using emulated links for all three segments
- **Power constraint testing:** Verify DTN operations stay within 10W power budget
- **Radiation tolerance testing:** Software-based SEU injection and recovery testing

All Phase 3 tests can be run in the Phase 1 terrestrial testbed using the cislunar emulation profile, without requiring actual cislunar hardware or live radio links.

For component interfaces, data models, correctness properties, and the full testing strategy, refer to the [Master Design Document](../master/design.md).

---

## Licence

Copyright 2026 AMSAT-UK & AMSAT-DL

Licensed under the Apache License, Version 2.0 (the "Licence"); you may not use this file except in compliance with the Licence. You may obtain a copy of the Licence at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the Licence is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the Licence for the specific language governing permissions and limitations under the Licence.

# Design Document — Phase 2: CubeSat (LEO)

| | |
|---|---|
| Document ID | ION-DTN-DES-P2-001 |
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
| 0.1.0 | 2026-03-28 | [name] | Initial draft — Phase 2 design extracted from master design document |

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

This document describes the Phase 2 (CubeSat) design for the ION-DTN Cislunar Demonstration project. Phase 2 is conditional on identifying and partnering with a suitable CubeSat project host willing to accommodate the ION-DTN payload. All Phase 2 design elements are conditional on securing such a host.

This document should be read in conjunction with the [Master Design Document](../master/design.md), which contains the full system architecture, all 12 component designs with interfaces, all data models, all 65 correctness properties, error handling tables, and the complete testing strategy. This phase-specific document covers deployment details, CLA usage, resource constraints, federation considerations, and testing unique to Phase 2.

## Phase 2 Deployment

### CubeSat Flight Computer

The CubeSat runs ION-DTN on a COTS flight computer. The ION-DTN payload is a software package running on the host CubeSat's flight computer with minimal additional hardware.

- **Platform:** COTS flight computer available to amateur satellite programmes
- **Operating System:** Linux (embedded)
- **ION-DTN:** Compiled from source for the target ARM/RISC-V architecture
- **Addressing:** `ipn:100` scheme for compact, bandwidth-efficient addressing

### Ground Station Bare Metal

Ground stations run ION-DTN on bare metal (Raspberry Pi or commodity PC) with amateur radio transceiver hardware:

- **Platform:** Raspberry Pi 4/5 or equivalent COTS PC
- **Transceiver:** UHF amateur radio transceiver (70 cm band, 430–440 MHz)
- **TNC:** Hardware or software TNC for AX.25 packet radio
- **Addressing:** `dtn://` scheme with callsign-SSID (e.g., `dtn://G4DPZ-2`)

## Phase 2 Convergence Layer Adapters

| CLA | Protocol | Data Rate | Use Case |
|-----|----------|-----------|----------|
| AX.25 v2.2 (BPSK 9K6) | AX.25 v2.2 | 9600 bps | CubeSat UHF uplink/downlink (70 cm band) |
| LTP | Licklider Transmission Protocol | Configurable | Ground-to-space reliable transport |

The AX.25 CLA in Phase 2 uses BPSK modulation at 9600 bps (9K6), the standard for modern amateur CubeSat communication. LTP provides reliable transport with retransmission management for the ground-to-space link.

## Phase 2 Resource Constraints

The CubeSat flight environment imposes strict resource constraints:

| Resource | Constraint | Design Impact |
|----------|-----------|---------------|
| Memory | 256 MB maximum | ION Bundle Agent memory footprint must stay within 256 MB under all workloads (Property 60) |
| Storage | 1 GB non-volatile minimum | Bundle store-and-forward buffer; must survive power cycles |
| Power | Variable (solar + battery) | Bundle Agent must recover gracefully after power loss (Property 61); checkpoint in-progress transfers |
| Link availability | ~10 min pass / ~80 min gap | Store-and-forward is primary operating mode; custody transfer ensures no bundle loss during intermittent contacts |

### Power Recovery

When the CubeSat loses power during a bundle transfer, the Bundle Agent must:
1. Persist transfer state to non-volatile storage at regular checkpoints
2. On power restoration, detect incomplete transfers
3. Resume pending transfers from the last checkpoint without data loss
4. Log the power loss event and recovery status

## Phase 2 Federation and Ground Station Coordination

Phase 2 introduces real multi-ground-station federation:

- Ground stations worldwide register with the Federation Service
- Pass predictions computed from TLE/OMM orbital elements for the CubeSat
- Pass assignments coordinated to avoid redundant transmissions when multiple stations have overlapping visibility
- Transfer logs maintained at each ground station recording bundle identifiers, timestamps, transfer status, and byte counts per pass
- Ground station setup documented with bill of materials and configuration guide reproducible by amateur radio operators

## Phase 2 Testing Considerations

Phase 2 testing builds on Phase 1 infrastructure:

- **Emulated CubeSat testing:** Use the `leo_pass.toml` emulation profile in the Phase 1 testbed to simulate CubeSat pass conditions (9.6 kbps, 10-minute pass, 80-minute gap)
- **Memory constraint testing:** Verify ION Bundle Agent stays within 256 MB under load (Property 60)
- **Power loss simulation:** Simulate power loss during bundle transfer, verify checkpoint and recovery (Property 61)
- **Custody transfer testing:** Verify custody acceptance signals, bundle retention, and retransmission on timeout (Properties 19–22)
- **AX.25 BPSK 9K6 testing:** Test the AX.25 CLA at 9600 bps with bundle fragmentation for constrained links
- **Ground station integration:** Test ground station → CubeSat bundle exchange using emulated UHF link
- **Federation testing:** Test multi-station pass coordination and handoff

All Phase 2 tests can be run in the Phase 1 terrestrial testbed using emulation profiles, without requiring actual CubeSat hardware or live radio links.

For component interfaces, data models, correctness properties, and the full testing strategy, refer to the [Master Design Document](../master/design.md).

---

## Licence

Copyright 2026 AMSAT-UK & AMSAT-DL

Licensed under the Apache License, Version 2.0 (the "Licence"); you may not use this file except in compliance with the Licence. You may obtain a copy of the Licence at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the Licence is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the Licence for the specific language governing permissions and limitations under the Licence.

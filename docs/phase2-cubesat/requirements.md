# Requirements Document — Phase 2: CubeSat (LEO)

| | |
|---|---|
| Document ID | ION-DTN-REQ-P2-001 |
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
| 0.1.0 | 2026-03-28 | [name] | Initial draft — Phase 2 requirements extracted from master requirements document |

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

This document defines the Phase 2 (CubeSat) requirements for the ION-DTN Cislunar Demonstration project. Phase 2 is conditional on identifying and partnering with a suitable CubeSat project host willing to accommodate the ION-DTN payload. All Phase 2 requirements are conditional on securing such a host. Phase 1 (Terrestrial Testbed) is not dependent on Phase 2 and will proceed independently.

This document should be read in conjunction with the [Master Requirements Document](../master/requirements.md), which contains the full project introduction, glossary, cross-cutting requirements (Requirements 9–19, 21–27, 34), optional requirements (Requirements 28–33), appendices, and risk register. All cross-cutting requirements from the master document also apply to Phase 2.

---

## Requirements

### Requirement 4: CubeSat ION-DTN Integration (Conditional on CubeSat Host)

**User Story:** As a user, I want to integrate ION-DTN onto a CubeSat platform using COTS components, so that I can demonstrate delay-tolerant networking from low Earth orbit on an IARU-coordinated amateur radio frequency allocation.

#### Acceptance Criteria

1. THE CubeSat SHALL run a Bundle_Agent capable of sending, receiving, and forwarding bundles via the Bundle_Protocol using Open_Source_Software.
2. THE CubeSat Bundle_Agent SHALL operate within a memory footprint of 256 MB or less, compatible with COTS_Hardware flight computers available to amateur satellite programmes.
3. THE CubeSat SHALL support Licklider_Transmission_Protocol and AX.25 as Convergence_Layer_Adapters for ground-to-space links on Amateur_Radio_Band allocations (e.g., UHF 70 cm band per IARU band plan).
4. WHEN the CubeSat has line-of-sight to a Ground_Station, THE CubeSat SHALL establish a link session and begin bundle exchange within 10 seconds of link acquisition.
5. THE CubeSat SHALL store bundles in non-volatile storage with a capacity of at least 1 GB for Store_And_Forward operations.
6. IF the CubeSat loses power during a bundle transfer, THEN THE Bundle_Agent SHALL recover and resume pending transfers from the last checkpoint after power restoration.
7. THE CubeSat flight software and configuration SHALL be published under the Apache License, Version 2.0 to enable community replication and experimentation.

### Requirement 5: Ground Station Communication (Conditional on CubeSat Host)

**User Story:** As a user, I want to communicate with the CubeSat from my amateur ground station, so that I can exchange DTN bundles during orbital passes using standard amateur radio equipment.

#### Acceptance Criteria

1. THE Ground_Station SHALL run a Bundle_Agent configured to communicate with the CubeSat via Licklider_Transmission_Protocol or AX.25 on Amateur_Radio_Band frequencies.
2. WHEN a CubeSat orbital pass begins, THE Ground_Station SHALL detect link availability and initiate bundle exchange within 15 seconds.
3. WHILE the CubeSat is within communication range, THE Ground_Station SHALL sustain a minimum data throughput of 9.6 kbps for bundle transfers, achievable with commonly available amateur radio transceivers.
4. WHEN an orbital pass ends, THE Ground_Station SHALL complete or checkpoint all in-progress bundle transfers within 5 seconds of link loss detection.
5. THE Ground_Station SHALL maintain a transfer log recording bundle identifiers, timestamps, transfer status, and byte counts for each pass.
6. THE Ground_Station setup SHALL be documented with a bill of materials and configuration guide reproducible by amateur radio operators holding a valid amateur radio licence from their national administration.

### Requirement 6: CubeSat Custody Transfer (Conditional on CubeSat Host)

**User Story:** As a user, I want custody transfer support on the CubeSat, so that I can ensure bundles are not lost during intermittent amateur satellite orbital contacts.

#### Acceptance Criteria

1. THE CubeSat Bundle_Agent SHALL support Custody_Transfer for bundles marked with custody-requested delivery options.
2. WHEN the CubeSat accepts custody of a bundle, THE Bundle_Agent SHALL send a custody acceptance signal to the previous custodian.
3. WHILE the CubeSat holds custody of a bundle, THE Bundle_Agent SHALL retain the bundle in non-volatile storage until custody is transferred to the next hop or the bundle is delivered.
4. IF a custody acceptance signal is not received within 120 seconds, THEN THE sending Bundle_Agent SHALL retransmit the bundle.
5. THE Bundle_Agent SHALL log all custody transfer events including bundle identifier, timestamp, and transfer outcome.

---

## Cross-Cutting Requirements

The following cross-cutting requirements from the [Master Requirements Document](../master/requirements.md) also apply to Phase 2:

- Requirement 9: End-to-End Multi-Phase Interoperability
- Requirement 10: Configuration Serialization and Parsing
- Requirement 11: Monitoring and Telemetry
- Requirement 12: Security and Authentication
- Requirement 13: Open Access and Community Collaboration
- Requirement 14: Network Control / Mission Ops Suite
- Requirement 15: Public Downlink Telemetry Protocols/APIs
- Requirement 16: No Service Level Agreement
- Requirement 17: Worldwide Adoption and International Participation
- Requirement 18: Multi-Ground-Station Federation
- Requirement 19: DTN Gateway Bridging
- Requirement 21: STEAM Educational Enablement
- Requirement 22: Public Dashboard and Live Visualisation
- Requirement 23: Data Archive and Experiment Replay
- Requirement 24: Test-Driven Development
- Requirement 25: Continuous Integration
- Requirement 26: Continuous Deployment
- Requirement 27: ION-DTN Baseline Test Suite Compliance
- Requirement 34: Bundle Fragmentation for Constrained Links

---

## Licence

Copyright 2026 AMSAT-UK & AMSAT-DL

Licensed under the Apache License, Version 2.0 (the "Licence"); you may not use this file except in compliance with the Licence. You may obtain a copy of the Licence at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the Licence is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the Licence for the specific language governing permissions and limitations under the Licence.

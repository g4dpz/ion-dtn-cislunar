# Requirements Document — Phase 3: Cislunar Payload

| | |
|---|---|
| Document ID | ION-DTN-REQ-P3-001 |
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
| 0.1.0 | 2026-03-28 | [name] | Initial draft — Phase 3 requirements extracted from master requirements document |

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

This document defines the Phase 3 (Cislunar Payload) requirements for the ION-DTN Cislunar Demonstration project. Phase 3 is a proposal subject to ESA ARTES (Advanced Research in Telecommunications Systems) programme review and approval. All Phase 3 requirements are conditional on ESA approval and ARTES funding. Phases 1 (Terrestrial Testbed) and 2 (CubeSat) are not dependent on Phase 3 approval and will proceed independently.

This document should be read in conjunction with the [Master Requirements Document](../master/requirements.md), which contains the full project introduction, glossary, cross-cutting requirements (Requirements 9–19, 21–27, 34), optional requirements (Requirements 28–33), appendices, and risk register. All cross-cutting requirements from the master document also apply to Phase 3.

---

## Requirements

### Requirement 7: Cislunar Payload Hardware Integration (Conditional on ESA ARTES Approval)

**User Story:** As a user, I want to integrate ION-DTN onto a cislunar-capable payload using COTS components, so that I can demonstrate delay-tolerant networking at lunar distances as an educational and community research mission.

#### Acceptance Criteria

1. THE Cislunar_Payload SHALL run a Bundle_Agent supporting Bundle_Protocol and Licklider_Transmission_Protocol using Open_Source_Software.
2. THE Cislunar_Payload SHALL operate within a power budget of 10 watts average during DTN operations, consistent with secondary payload or rideshare power allocations on ESA or European commercial launch opportunities.
3. THE Cislunar_Payload SHALL provide at least 4 GB of non-volatile storage for bundle buffering using COTS_Hardware storage components.
4. THE Cislunar_Payload SHALL tolerate a radiation environment consistent with cislunar transit and lunar orbit, using radiation-tolerant COTS_Hardware or software-based error mitigation.
5. IF the Cislunar_Payload detects a single-event upset in memory, THEN THE Cislunar_Payload SHALL initiate error correction and log the event.
6. THE Cislunar_Payload hardware design and software SHALL be published under the Apache License, Version 2.0 with sufficient documentation for academic replication by university research groups worldwide.

### Requirement 8: Cislunar Delay-Tolerant Communication (Conditional on ESA ARTES Approval)

**User Story:** As a user, I want to demonstrate DTN communication at cislunar distances, so that I can validate protocol performance with round-trip delays of approximately 2.5 seconds and contribute experimental data to the amateur and academic communities.

#### Acceptance Criteria

1. THE Cislunar_Payload SHALL exchange bundles with a Ground_Station over links with a one-way light delay of up to 1.5 seconds, operating on amateur radio frequency allocations coordinated through IARU and the relevant national administration.
2. WHEN a bundle is sent from the Ground_Station to the Cislunar_Payload, THE Licklider_Transmission_Protocol SHALL manage retransmission timers calibrated to the current Round_Trip_Time.
3. THE Cislunar_Payload SHALL sustain a minimum bundle throughput of 1 kbps over the cislunar link under nominal conditions.
4. WHILE the Cislunar_Payload is in lunar orbit, THE Contact_Graph_Routing algorithm SHALL account for periodic link occlusion by the Moon.
5. WHEN a cislunar link is disrupted due to lunar occlusion, THE Bundle_Agent SHALL store bundles and resume forwarding within 30 seconds of link reacquisition.

---

## Cross-Cutting Requirements

The following cross-cutting requirements from the [Master Requirements Document](../master/requirements.md) also apply to Phase 3:

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

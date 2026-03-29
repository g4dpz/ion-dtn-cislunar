# Requirements Document — Phase 1: Terrestrial Testbed

| | |
|---|---|
| Document ID | ION-DTN-REQ-P1-001 |
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
| 0.1.0 | 2026-03-28 | [name] | Initial draft — Phase 1 requirements extracted from master requirements document |

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

This document defines the Phase 1 (Terrestrial Testbed) requirements for the ION-DTN Cislunar Demonstration project. Phase 1 is the active, unconditional phase that delivers a ground-based ION-DTN network using COTS hardware and open-source software.

This document should be read in conjunction with the [Master Requirements Document](../master/requirements.md), which contains the full project introduction, glossary, cross-cutting requirements (Requirements 9–19, 21–27, 34), optional requirements (Requirements 28–33), appendices, and risk register. All cross-cutting requirements from the master document also apply to Phase 1.

---

## Requirements

### Requirement 1: Terrestrial Testbed Deployment

**User Story:** As a user, I want to deploy a terrestrial ION-DTN testbed using COTS hardware and open-source software, starting with a simple point-to-point configuration and progressing to a multi-node network, so that I can validate Bundle Protocol operations incrementally before progressing to space-based phases.

#### Acceptance Criteria

1. THE Terrestrial_Testbed SHALL initially be deployed as a point-to-point configuration consisting of two ION_DTN Nodes to demonstrate basic Bundle_Protocol operation, bundle exchange, and store-and-forward behaviour.
2. AFTER successful point-to-point validation, THE Terrestrial_Testbed SHALL be expanded to a minimum of three ION_DTN Nodes to demonstrate multi-hop routing and Contact_Graph_Routing.
3. ALL Terrestrial_Testbed Nodes SHALL be built from COTS_Hardware (e.g., Raspberry Pi or equivalent single-board computers) running Open_Source_Software.
4. WHEN a Node in the Terrestrial_Testbed is started, THE Bundle_Agent on that Node SHALL initialize and register with the local ION_DTN configuration within 30 seconds.
5. THE Terrestrial_Testbed SHALL support TCP, UDP, and AX.25 Convergence_Layer_Adapters for inter-node communication.
6. WHEN a bundle is sent from a source Node to a destination Node, THE Terrestrial_Testbed SHALL deliver the bundle to the destination application within 10 seconds under nominal link conditions.
7. IF a Node in the Terrestrial_Testbed fails to start its Bundle_Agent, THEN THE Node SHALL log a descriptive error message indicating the failure reason.
8. THE Terrestrial_Testbed SHALL include setup documentation sufficient for an amateur radio operator or undergraduate student to reproduce both the point-to-point and multi-node configurations independently.

### Requirement 2: Terrestrial Link Disruption Simulation

**User Story:** As a user, I want to simulate link disruptions in the terrestrial testbed, so that I can verify store-and-forward behavior representative of real-world amateur satellite pass conditions before deploying to space.

#### Acceptance Criteria

1. THE Terrestrial_Testbed SHALL provide a mechanism to simulate Link_Disruption between any pair of Nodes for a configurable duration.
2. WHEN a Link_Disruption is active between two Nodes, THE Bundle_Agent on the sending Node SHALL store bundles destined for the disrupted link in persistent storage.
3. WHEN a simulated Link_Disruption ends, THE Bundle_Agent SHALL forward all stored bundles to the destination Node within 60 seconds of link restoration.
4. WHILE a Link_Disruption is active, THE Bundle_Agent SHALL continue to accept new bundles from local applications without data loss.
5. THE Terrestrial_Testbed SHALL log the start time, end time, and affected Node pair for each simulated Link_Disruption.

### Requirement 3: Contact Graph Routing Validation

**User Story:** As a user, I want to validate Contact Graph Routing in the terrestrial testbed, so that I can confirm correct multi-hop forwarding using predicted amateur satellite pass schedules before orbital deployment.

#### Acceptance Criteria

1. THE Terrestrial_Testbed SHALL support Contact_Graph_Routing with configurable contact plans defining scheduled link availability windows representative of amateur satellite orbital passes.
2. WHEN a contact plan is loaded, THE Bundle_Agent SHALL compute forwarding routes based on the time-varying contact graph within 5 seconds.
3. WHEN a bundle requires multi-hop delivery, THE Contact_Graph_Routing algorithm SHALL select the path with the earliest projected delivery time.
4. WHEN a scheduled contact window opens, THE Bundle_Agent SHALL begin forwarding queued bundles for that contact within 2 seconds.
5. IF no valid route exists for a bundle, THEN THE Bundle_Agent SHALL retain the bundle in storage and re-evaluate routing when the contact plan changes.

### Requirement 20: Delay and Disruption Emulation Profiles

**User Story:** As a user, I want pre-built link emulation profiles that simulate realistic LEO, HEO, and cislunar conditions in the terrestrial testbed, so that I can evaluate DTN performance under representative space link characteristics beyond simple on/off disruption.

#### Acceptance Criteria

1. THE Terrestrial_Testbed SHALL support configurable link emulation profiles that model variable latency, jitter, bandwidth constraints, and packet loss in addition to binary Link_Disruption.
2. THE project SHALL provide at least three pre-built emulation profiles: a LEO amateur satellite pass profile, a highly elliptical orbit (HEO) profile, and a cislunar link profile with realistic propagation delay and occlusion patterns.
3. WHEN a link emulation profile is active, THE Terrestrial_Testbed SHALL apply the configured delay, jitter, bandwidth, and loss parameters to all bundles traversing the emulated link.
4. THE link emulation profiles SHALL be defined in a documented, editable format so that users can create custom profiles for their own experimental scenarios.
5. THE link emulation software and profile definitions SHALL be released under the Apache License, Version 2.0.

---

## Cross-Cutting Requirements

The following cross-cutting requirements from the [Master Requirements Document](../master/requirements.md) also apply to Phase 1:

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

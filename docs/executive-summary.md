# ION-DTN Cislunar Demonstration

| | |
|---|---|
| Document ID | ION-DTN-EXEC-001 |
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

### Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-03-28 | [name] | Initial draft |

### Distribution

This document is released under the Apache License, Version 2.0 and may be freely distributed.

### Review and Approval

| Action | Name | Date | Signature |
|--------|------|------|-----------|
| Prepared by | [name] | | |
| Reviewed by | [name] | | |
| Approved by | [name] | | |

---

## Executive Summary

### Project Overview

This project delivers a phased demonstration of Delay-Tolerant Networking (DTN) using NASA JPL's open-source ION implementation, progressing from a terrestrial testbed through a CubeSat in low Earth orbit to a cislunar payload operating at lunar distances. Led by AMSAT-UK and AMSAT-DL, the project is designed for the worldwide amateur radio, amateur satellite, and academic research communities.

DTN is a networking architecture designed for environments where continuous end-to-end connectivity cannot be guaranteed — exactly the conditions encountered in space communication. Bundles of data are stored at intermediate nodes and forwarded when links become available, enabling reliable communication across disrupted, delayed, and intermittent links.

The project wraps the ION-DTN reference implementation with modern tooling for configuration management, network operations, telemetry, and public engagement, making space-grade networking protocols accessible to amateur operators, students, and researchers worldwide.

### Why This Matters

Current space communication relies on point-to-point links scheduled by mission control. As humanity expands its presence beyond Earth orbit — to the Moon, Mars, and beyond — a store-and-forward networking layer becomes essential. DTN provides this layer, and it needs to be tested, validated, and understood by a broad community before it becomes critical infrastructure.

This project brings DTN out of the laboratory and into the hands of the amateur radio and academic communities, who have a 40-year track record of pioneering store-and-forward satellite communication. By building on existing amateur infrastructure and culture, the project creates a globally distributed testbed for cislunar networking at a fraction of the cost of a traditional space mission.

### Three-Phase Approach

**Phase 1 — Terrestrial Testbed (Active)**
A ground-based network of ION-DTN nodes built from commodity hardware (Raspberry Pi, amateur radio transceivers) validates Bundle Protocol operations, Contact Graph Routing, and store-and-forward behaviour. The testbed starts with a simple two-node point-to-point link and expands to a multi-node network with simulated space link conditions (delay, jitter, disruption). A web-based mission operations suite and public dashboard provide real-time visibility. Phase 1 proceeds independently and delivers standalone value.

**Phase 2 — CubeSat (Conditional)**
An ION-DTN payload hosted on an amateur CubeSat in low Earth orbit demonstrates delay-tolerant networking over real space links. Ground stations communicate with the CubeSat via UHF amateur radio using AX.25 v2.2 at 9600 bps (BPSK). A federated network of amateur ground stations worldwide provides pass coverage. Phase 2 is conditional on identifying a suitable CubeSat project host.

**Phase 3 — Cislunar Payload (Proposal)**
A payload operating at lunar distances demonstrates DTN communication with round-trip delays of approximately 2.5 seconds and periodic link occlusion by the Moon. This phase validates protocol performance under the most challenging conditions in the Earth-Moon system. Phase 3 is a proposal subject to ESA ARTES programme review and approval.

### Key Differentiators

| Factor | This Project | Traditional Approach |
|--------|-------------|---------------------|
| Cost | COTS hardware, volunteer ground stations, open-source software | Bespoke hardware, dedicated ground infrastructure, proprietary software |
| Community | Worldwide amateur radio and academic participation | Single agency or contractor team |
| Accessibility | Any licensed amateur operator can participate | Restricted to mission partners |
| Reproducibility | All designs, software, and data published under Apache 2.0 | Proprietary or restricted access |
| Ground coverage | Federated global amateur ground station network | Limited dedicated ground stations |
| Education | STEAM enablement platform for schools and universities | Minimal outreach component |
| Incremental value | Each phase delivers standalone results | All-or-nothing mission success criteria |

### Technical Highlights

- Wraps NASA JPL's ION-DTN reference implementation — the same software stack used on the International Space Station and deep space missions
- Bundle Protocol v7 (RFC 9171) with Contact Graph Routing for scheduled multi-hop delivery
- BPSec integrity verification compliant with amateur radio regulations (no payload encryption)
- CCSDS File Delivery Protocol (CFDP) for reliable file transfer over disrupted links
- Hybrid addressing: callsign-SSID based `dtn://` endpoints for terrestrial nodes (familiar to amateur operators), compact `ipn:` addressing for space nodes
- FM/AFSK at 1200 baud for terrestrial links, BPSK at 9600 bps for CubeSat links
- Bundle fragmentation for bandwidth-constrained amateur radio links
- Full CI/CD pipeline with property-based testing and ION baseline test suite compliance

### Community and Outreach

The project is designed to maximise community participation:

- All software released under the Apache License, Version 2.0
- Hardware designs published under open licences
- Public dashboard with live satellite tracking, ground station map, and network health metrics
- Publicly documented telemetry protocols and APIs enabling independent decoder development
- Data archive with five-year retention and replay capability for research use
- STEAM educational enablement — tools and APIs for educators to build their own teaching materials
- Gateway bridging to existing amateur packet networks (APRS, Winlink) for interoperability with established infrastructure
- No service level agreement — the network is experimental and best-effort, consistent with the amateur radio ethos

### Technology Stack

| Layer | Technology |
|-------|-----------|
| Protocol | ION-DTN (C), Bundle Protocol v7, LTP, CGR, BPSec, CFDP |
| Tooling & Web | Deno, Oak, Handlebars, Bootstrap, TypeScript |
| Storage | SQLite |
| Radio | AX.25 v2.2, FM/AFSK (1200 baud), BPSK (9600 bps) |
| Infrastructure | Docker, GitHub Actions CI/CD |
| Hardware | Raspberry Pi, SDR dongles, amateur radio transceivers |

### Risk Summary

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| No CubeSat host found (Phase 2) | Medium | High | Early engagement with AMSAT organisations and university CubeSat programmes; Phase 1 delivers standalone value |
| ESA ARTES not approved (Phase 3) | Medium | High | Phases 1 and 2 independent; alternative rideshare opportunities explored |
| Frequency coordination delayed | Medium | High | Begin IARU-SAT coordination during Phase 1 |
| Insufficient ground station coverage | Medium | Medium | Build federation early; engage SatNOGS and AMSAT communities |
| Community engagement insufficient | Medium | Medium | Publish early, release often; present at conferences; engage university supervisors |

### Budget Considerations

The project is designed for minimal cost:

- Phase 1 requires only commodity hardware (Raspberry Pi units, amateur radio transceivers, SDR dongles) and volunteer effort. Estimated hardware cost per testbed node: under £200.
- Phase 2 leverages an existing CubeSat mission as a hosted payload, avoiding standalone launch costs. The ION-DTN payload is a software package running on the host CubeSat's flight computer with minimal additional hardware.
- Phase 3, if approved under ESA ARTES, would be funded through that programme as a secondary/rideshare payload.
- Ground station infrastructure is provided by volunteer amateur radio operators using their existing equipment, supplemented by turnkey documentation and configuration guides.
- All development is open-source, enabling contribution from the global amateur and academic community.

### Project Leadership

The project is led by AMSAT-UK and AMSAT-DL, two of Europe's leading amateur satellite organisations with decades of experience in designing, building, and operating amateur satellites. The project welcomes participation from amateur radio operators, university research groups, and space agencies worldwide.

### Contact

For further information, partnership enquiries, or funding discussions, please contact the project leads at AMSAT-UK or AMSAT-DL.

---

*This document summarises the ION-DTN Cislunar Demonstration project. Full requirements (34 requirements across 3 phases) and technical design (65 correctness properties, 12 software components) are available in the accompanying Requirements Document and Design Document.*

---

## Licence

Copyright 2026 AMSAT-UK & AMSAT-DL

Licensed under the Apache License, Version 2.0 (the "Licence"); you may not use this file except in compliance with the Licence. You may obtain a copy of the Licence at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the Licence is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the Licence for the specific language governing permissions and limitations under the Licence.

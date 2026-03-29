# Implementation Tasks: ION-DTN Cislunar Demonstration

## Overview

This plan implements the ION-DTN Cislunar Demonstration project in phased order: Phase 1 (Terrestrial Testbed, active/unconditional) first, then cross-cutting components, then Phase 2 (CubeSat, conditional on host), then Phase 3 (Cislunar, conditional on ESA ARTES). Optional requirements (28–33) are marked as optional tasks. The tech stack is TypeScript/Deno for all tooling and C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

## Execution Order

1. **Phase 1** (Tasks 1–10) — unconditional, complete first
2. **Master cross-cutting** (Tasks 11–24) — builds on Phase 1 infrastructure
3. **Phase 2** (Tasks 25–27) — conditional on CubeSat host
4. **Phase 3** (Tasks 28–29) — conditional on ESA ARTES approval
5. **Optional** (Tasks 30–36) — can be done at any point after Phase 1
6. **Final checkpoint** (Task 37) — full system integration

## Summary

| Section | Tasks | Requirements | Status |
|---------|-------|-------------|--------|
| Phase 1 — Terrestrial | 1–10 | 1, 2, 3, 20 | Active, Unconditional |
| Master — Cross-Cutting | 11–24, 30–37 | 9–19, 21–34 | Builds on Phase 1 |
| Phase 2 — CubeSat | 25–27 | 4, 5, 6 | Conditional (CubeSat host) |
| Phase 3 — Cislunar | 28–29 | 7, 8 | Conditional (ESA ARTES) |

---

## Phase 1 — Terrestrial Testbed Tasks

Requirements: 1, 2, 3, 20 | Status: Active, Unconditional

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

### Tasks

- [ ] 1. Project scaffolding and ION-DTN build infrastructure
  - [ ] 1.1 Create project directory structure and Deno workspace configuration
    - Create top-level directories: `config_manager/`, `link_emulator/`, `mission_ops/`, `telemetry/`, `federation/`, `gateway/`, `archive/`, `dashboard/`, `pass_predictor/`, `ci/`, `docker/`, `ion_extensions/`, `tests/`
    - Create `tests/unit/`, `tests/property/`, `tests/integration/`, `tests/e2e/` subdirectories
    - Create `deno.json` workspace config with import maps and task definitions
    - Install fast-check as a dev dependency for property-based testing
    - _Requirements: 24.1, 13.4_

  - [ ] 1.2 Create Dockerfile for ION-DTN node base image
    - Write `docker/Dockerfile.ion-node` that builds ION-DTN from source (C compilation)
    - Ensure the image includes ION's `bpv7/`, `ltp/`, `ici/`, `cfdp/` modules
    - Include ION's baseline test suite (`tests/alltests`) in the image
    - Verify the image builds on Linux x86_64 and ARM (Raspberry Pi)
    - _Requirements: 1.3, 27.1, 27.2_

  - [ ] 1.3 Create ION-DTN baseline test suite CI gate
    - Write GitHub Actions workflow step that builds ION and runs `tests/alltests`
    - Ensure the step blocks the pipeline on any test failure
    - _Requirements: 27.1, 27.2, 27.3, 25.3_

- [ ] 2. Configuration Manager — TOML parser, validator, and ION compiler
  - [ ] 2.1 Implement TOML configuration parser and NodeConfig types
    - Define TypeScript interfaces: `NodeConfig`, `Endpoint`, `Contact`, `Range`, `Protocol`, `Plan`, `Scheme`, `SecurityConfig`, `CLAConfig` as specified in the master design
    - Implement `parseToml(tomlStr: string): NodeConfig` using a TOML parsing library
    - Support both `dtn://` (callsign-SSID) and `ipn:` (numeric) endpoint schemes
    - Support per-CLA `max_fragment_bytes` configuration for constrained links
    - _Requirements: 10.1, 10.2, 34.3_

  - [ ] 2.2 Implement configuration validator
    - Implement `validateConfig(config: NodeConfig): ValidationError[]`
    - Validate required fields (node number, name), contact time ranges (start < end), known CLA types, endpoint scheme consistency
    - Return descriptive errors with line number and violation description for invalid configs
    - _Requirements: 10.3_

  - [ ] 2.3 Implement ION native config compiler (TOML → .ionrc, .bprc, .ipnrc, .rc)
    - Implement `compileToIon(config: NodeConfig): Record<string, string>`
    - Generate `.ionrc` (node init, contacts, ranges), `.bprc` (schemes, endpoints, protocols, plans), `.ipnrc` (IPN routing rules), `.rc` (combined startup script)
    - Handle both `dtn://` scheme (dtn2admin/dtn2fw) and `ipn:` scheme (ipnadmin/ipnfw) directives
    - _Requirements: 10.1, 10.2_

  - [ ] 2.4 Implement TOML printer and ION native parser for round-trip
    - Implement `printToml(config: NodeConfig): string`
    - Implement `parseIonNative(files: Record<string, string>): NodeConfig`
    - _Requirements: 10.4, 10.5_

  - [ ]* 2.5 Write property test for configuration round-trip
    - **Property 1: Configuration round-trip (parse → print → parse)**
    - Create fast-check arbitrary for valid `NodeConfig` objects
    - Assert `parseToml(printToml(config))` equals original config
    - **Validates: Requirements 10.1, 10.4, 10.5**

  - [ ]* 2.6 Write property test for invalid configuration error reporting
    - **Property 2: Invalid configuration produces descriptive error**
    - Create fast-check arbitrary for invalid TOML strings (missing fields, bad syntax, invalid ranges)
    - Assert parser returns error with line number and description, never a valid NodeConfig
    - **Validates: Requirements 10.3**

  - [ ]* 2.7 Write property test for ION native config generation
    - **Property 3: ION native config generation produces valid ION config**
    - For any valid NodeConfig, compile to ION native, parse back, assert equivalence
    - **Validates: Requirements 10.2, 10.4**

- [ ] 3. Checkpoint — Configuration Manager
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 4. Docker Testbed — 2-node point-to-point topology
  - [ ] 4.1 Create Docker Compose 2-node configuration
    - Write `docker/docker-compose.2node.yml` with two ION node containers (Node 1: `dtn://G4DPZ-1`, Node 2: `dtn://DL1XYZ-1`)
    - Use STCP/TCP CLA over Docker bridge network, mirroring ION's `configs/2node-stcp/` pattern
    - Write `docker/scripts/entrypoint.sh` that loads generated ION config and starts the Bundle Agent
    - Write `docker/scripts/healthcheck.sh` for Bundle Agent health checking
    - _Requirements: 1.1, 1.3, 1.5_

  - [ ] 4.2 Create sample TOML configs for 2-node topology and generate ION native configs
    - Write TOML config files for Node 1 and Node 2 with STCP CLA, BPSec enabled
    - Use Config Manager to compile TOML → ION native config files
    - _Requirements: 1.1, 10.1, 12.1_

  - [ ]* 4.3 Write property test for Bundle Agent startup time bound
    - **Property 11: Bundle Agent startup within time bound**
    - Assert Bundle Agent initializes and registers within 30 seconds for any valid config
    - **Validates: Requirements 1.4**

  - [ ]* 4.4 Write property test for bundle delivery under nominal conditions
    - **Property 12: Bundle delivery under nominal conditions within time bound**
    - Assert bundle delivered to destination within 10 seconds under nominal link
    - **Validates: Requirements 1.6**

  - [ ]* 4.5 Write property test for failed startup logging
    - **Property 13: Failed Bundle Agent startup produces descriptive log**
    - Assert invalid config produces descriptive error log on startup failure
    - **Validates: Requirements 1.7**

- [ ] 5. Link Emulator — delay, jitter, bandwidth, loss, and disruption profiles
  - [ ] 5.1 Implement EmulationProfile types and TOML profile loader
    - Define TypeScript interfaces: `EmulationProfile`, `DisruptionWindow` as specified in design
    - Implement `loadProfile(path: string): Promise<EmulationProfile>` to parse TOML profile files
    - _Requirements: 20.1, 20.4_

  - [ ] 5.2 Implement tc/netem wrapper for applying and clearing profiles
    - Implement `applyProfile(iface: string, profile: EmulationProfile): Promise<void>` wrapping Linux `tc`/`netem` commands
    - Implement `clearProfile(iface: string): Promise<void>`
    - Implement `scheduleDisruptions(iface: string, windows: DisruptionWindow[]): Promise<void>` for timed link on/off
    - _Requirements: 20.1, 20.3_

  - [ ] 5.3 Create pre-built emulation profile TOML files
    - Write `profiles/terrestrial_afsk.toml` — <1ms delay, 1.2 kbps, 0.5% loss
    - Write `profiles/leo_pass.toml` — 5–15ms delay, 9.6 kbps, 10-min pass / 80-min gap, 1% loss
    - Write `profiles/heo_pass.toml` — 50–500ms delay, 4.8 kbps, variable geometry
    - Write `profiles/cislunar.toml` — 1300ms delay, 1 kbps, 45-min lunar occlusion
    - _Requirements: 20.2, 20.5_

  - [ ] 5.4 Implement link disruption simulation mechanism
    - Implement configurable link disruption between any node pair for a configurable duration
    - Log start time, end time, and affected node pair for each disruption
    - _Requirements: 2.1, 2.5_

  - [ ]* 5.5 Write property test for emulation profile round-trip
    - **Property 15: Emulation profile round-trip (parse → serialize → parse)**
    - Assert parsing a profile TOML, serializing back, and re-parsing produces equivalent object
    - **Validates: Requirements 20.4**

  - [ ]* 5.6 Write property test for emulation profile parameter application
    - **Property 14: Emulation profile applies configured parameters**
    - Assert observed delay within tolerance of configured delay ± jitter, loss rate converges
    - **Validates: Requirements 20.1, 20.3**

  - [ ]* 5.7 Write property test for bundle storage during disruption
    - **Property 4: Bundle storage during link disruption (no data loss)**
    - Assert bundles submitted during disruption are stored without loss, storage count increases
    - **Validates: Requirements 2.2, 2.4**

  - [ ]* 5.8 Write property test for stored bundles forwarded after restoration
    - **Property 5: Stored bundles forwarded after link restoration**
    - Assert all stored bundles forwarded within 60 seconds of link restoration
    - **Validates: Requirements 2.3**

  - [ ]* 5.9 Write property test for disruption event logging
    - **Property 6: Disruption events are logged with required fields**
    - Assert log entry contains start time, end time, affected node pair, all non-empty
    - **Validates: Requirements 2.5**

- [ ] 6. Checkpoint — Link Emulator and Disruption Simulation
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Docker Testbed — 3-node multi-hop CGR topology
  - [ ] 7.1 Create Docker Compose 3-node configuration
    - Write `docker/docker-compose.3node.yml` with three ION containers (Node 1: source, Node 2: relay, Node 3: destination)
    - Configure Contact Graph Routing with a contact plan defining scheduled link windows
    - _Requirements: 1.2, 3.1_

  - [ ] 7.2 Create Docker Compose emulated topology
    - Write `docker/docker-compose.emulated.yml` layering link emulation profiles on the 3-node topology
    - Include Link Emulator container applying `tc`/`netem` to inter-node links
    - _Requirements: 20.1, 20.3_

  - [ ]* 7.3 Write property test for CGR earliest-delivery path selection
    - **Property 7: CGR selects earliest-delivery path**
    - Assert CGR selects path with projected delivery time ≤ all other valid paths
    - **Validates: Requirements 3.3**

  - [ ]* 7.4 Write property test for CGR route computation time bound
    - **Property 8: CGR route computation completes within time bound**
    - Assert route computation completes within 5 seconds of contact plan load
    - **Validates: Requirements 3.2**

  - [ ]* 7.5 Write property test for bundle retention when no route exists
    - **Property 9: Bundle retained when no route exists**
    - Assert bundle retained in storage, becomes eligible when new plan provides route
    - **Validates: Requirements 3.5**

  - [ ]* 7.6 Write property test for contact plan round-trip
    - **Property 10: Contact plan round-trip**
    - Assert serializing contact plan to ION format and parsing back produces equivalent plan
    - **Validates: Requirements 3.1, 10.5**

- [ ] 8. ION C Extensions — AX.25 v2.2 CLA and telemetry hook
  - [ ] 8.1 Implement AX.25 v2.2 Convergence Layer Adapter in C
    - Write CLA supporting FM/AFSK modulation at 1200 baud (Phase 1 terrestrial)
    - Support BPSK modulation at 9600 bps (Phase 2 CubeSat ground-to-space)
    - Integrate with ION's CLA framework (induct/outduct pattern)
    - Support per-CLA `max_fragment_bytes` for bundle fragmentation on constrained links
    - _Requirements: 1.5, 34.1, 34.3_

  - [ ] 8.2 Implement ION telemetry callback hook in C
    - Write callback that emits telemetry events (bundles sent/received/forwarded/expired) to the Deno/TypeScript aggregator
    - Use a lightweight IPC mechanism (Unix socket or named pipe) for C → TypeScript communication
    - _Requirements: 11.1_

  - [ ]* 8.3 Write property test for bundle fragmentation and reassembly round-trip
    - **Property 63: Bundle fragmentation and reassembly round-trip**
    - Assert fragments no larger than configured size, reassembled bundle byte-identical to original
    - Use theft (C property-based testing library) for C-level tests
    - **Validates: Requirements 34.1, 34.2**

  - [ ]* 8.4 Write property test for fragment size per-CLA compliance
    - **Property 64: Fragment size respects per-CLA configuration**
    - Assert all fragments transmitted over a CLA have payload ≤ configured `max_fragment_bytes`
    - **Validates: Requirements 34.3**

  - [ ]* 8.5 Write property test for fragmented bundle integrity verification
    - **Property 65: Fragmented bundle integrity verification**
    - Assert BIB verifies successfully on reassembled fragmented bundle
    - **Validates: Requirements 34.2**

- [ ] 9. BPSec — Bundle integrity verification
  - [ ] 9.1 Configure BPSec with pre-shared symmetric keys for integrity-only operation
    - Configure ION's BPSec module for Block Integrity Block (BIB) using HMAC-SHA256
    - Implement key distribution configuration in TOML (key name, authorised nodes, expiry)
    - Ensure no payload encryption on amateur radio links (integrity only)
    - _Requirements: 12.1, 12.2, 12.5, 12.6_

  - [ ]* 9.2 Write property test for BIB attached to all created bundles
    - **Property 16: BIB attached to all created bundles**
    - Assert every bundle created with BPSec enabled contains a BIB
    - **Validates: Requirements 12.2**

  - [ ]* 9.3 Write property test for BIB verification rejects tampered bundles
    - **Property 17: BIB verification rejects tampered bundles**
    - Assert tampered bundles are rejected, logged, and discarded
    - **Validates: Requirements 12.3, 12.4**

  - [ ]* 9.4 Write property test for integrity-only security on amateur links
    - **Property 18: Integrity-only security on amateur radio links**
    - Assert bundles on amateur links use BIB only, no encryption
    - **Validates: Requirements 12.6**

- [ ] 10. Checkpoint — Phase 1 Core (ION build, Config Manager, Link Emulator, Docker Testbed, AX.25 CLA, BPSec)
  - Ensure all tests pass, ask the user if questions arise.

### Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Tasks 1–10 are unconditional and should be completed first before cross-cutting or later phases
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document
- Checkpoints ensure incremental validation at natural breakpoints


---

## Master — Cross-Cutting, CI/CD, Integration, Optional Tasks

Requirements: 9–19, 21–27, 28–34 | Builds on Phase 1 infrastructure

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

### Cross-Cutting Components (Requirements 9–19, 21–27, 34)

- [ ] 11. Telemetry Aggregator — collection, storage, and export
  - [ ] 11.1 Implement TelemetryRecord and StatusReport types and SQLite schema
    - Define TypeScript interfaces: `TelemetryRecord`, `BundleStats`, `StorageStats`, `StatusReport`, `BundleDeliveryEvent`
    - Create SQLite database with `telemetry`, `bundle_deliveries` tables and indices as specified in design
    - _Requirements: 11.1, 11.5_

  - [ ] 11.2 Implement telemetry collection from ION nodes
    - Implement `collectTelemetry(nodeId: number): Promise<TelemetryRecord>` reading from ION telemetry hook
    - Implement `storeRecord(record: TelemetryRecord): Promise<void>` writing to SQLite
    - Implement configurable collection interval
    - _Requirements: 11.1, 11.2_

  - [ ] 11.3 Implement telemetry query and export API
    - Implement `queryRecords(start, end, nodeId?, phase?): Promise<TelemetryRecord[]>`
    - Implement `exportRecords(records, format): Promise<Uint8Array>` supporting CSV, JSON, Parquet
    - _Requirements: 11.5, 11.6_

  - [ ]* 11.4 Write property test for telemetry statistics at configured interval
    - **Property 23: Telemetry statistics reported at configured interval**
    - Assert telemetry record emitted at each interval tick with required fields
    - **Validates: Requirements 11.1**

  - [ ]* 11.5 Write property test for failure status report fields
    - **Property 24: Failure status report contains required fields**
    - Assert status report contains bundle ID, failure reason, timestamp — all non-empty
    - **Validates: Requirements 11.2**

  - [ ]* 11.6 Write property test for telemetry aggregation completeness
    - **Property 25: Telemetry aggregation covers all reachable nodes**
    - Assert consolidated view contains records from every reachable node
    - **Validates: Requirements 11.3**

  - [ ]* 11.7 Write property test for telemetry bundle prioritisation
    - **Property 26: Telemetry bundles prioritised during contact**
    - Assert telemetry bundles transmitted before regular data bundles during contact
    - **Validates: Requirements 11.4**

  - [ ]* 11.8 Write property test for telemetry export format validity
    - **Property 27: Telemetry export produces valid open format**
    - Assert exported CSV/JSON/Parquet is parseable and contains all original fields
    - **Validates: Requirements 11.6**

- [ ] 12. Mission Operations Suite — web-based network control
  - [ ] 12.1 Create Oak web server with REST API endpoints and Handlebars/Bootstrap UI
    - Set up Deno/Oak server with routes for nodes, contacts, passes, federation, telemetry, bundles, emulation, audit
    - Create Handlebars templates with Bootstrap CSS for responsive UI
    - Implement WebSocket endpoint `/api/telemetry/live` for live telemetry streaming
    - _Requirements: 14.1, 14.2, 14.7_

  - [ ] 12.2 Implement role-based access control (viewer, operator, admin)
    - Define `UserRole` enum and permission matrix
    - Implement authentication middleware with pre-shared keys
    - Enforce role-based permissions on all API endpoints
    - _Requirements: 14.8_

  - [ ] 12.3 Implement remote node configuration push and contact plan management
    - Implement `POST /api/nodes/{id}/config` for pushing configuration updates to reachable nodes
    - Implement `GET /api/contacts` and `POST /api/contacts` for contact plan viewing and upload
    - Display scheduled contact windows integrated with CGR
    - _Requirements: 14.3, 14.4_

  - [ ] 12.4 Implement operator audit logging
    - Create SQLite `audit_log` table as specified in design
    - Log all operator commands, configuration changes, and significant events with timestamps
    - Implement `GET /api/audit/log` endpoint
    - _Requirements: 14.6_

  - [ ]* 12.5 Write property test for audit log completeness
    - **Property 40: Audit log captures all operator actions**
    - Assert every operator action creates an audit entry with timestamp, operator, action, target
    - **Validates: Requirements 14.6**

  - [ ]* 12.6 Write property test for role-based access control enforcement
    - **Property 41: Role-based access control enforcement**
    - Assert each role can only perform authorised actions, others rejected
    - **Validates: Requirements 14.8**

  - [ ]* 12.7 Write property test for remote configuration update
    - **Property 43: Remote configuration update applied**
    - Assert valid config pushed via API is reflected in node's active configuration
    - **Validates: Requirements 14.3**

  - [ ]* 12.8 Write property test for contact plan upload display
    - **Property 44: Contact plan upload reflected in CGR display**
    - Assert uploaded contact plan windows match displayed windows
    - **Validates: Requirements 14.4**

- [ ] 13. Checkpoint — Telemetry Aggregator and Mission Ops Suite
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 14. Federation Service — ground station registration and pass coordination
  - [ ] 14.1 Implement GroundStation and PassAssignment types and SQLite schema
    - Define TypeScript interfaces: `GroundStation`, `GeoLocation`, `PassAssignment`
    - Create SQLite `ground_stations` table as specified in design
    - _Requirements: 18.1_

  - [ ] 14.2 Implement station registration, discovery, and pass assignment
    - Implement `registerStation(station): Promise<string>` — register with UUID, store in SQLite
    - Implement `discoverStations(satellite, timeWindow): Promise<GroundStation[]>` — query stations with visibility
    - Implement `assignPasses(stations, passes): Promise<PassAssignment[]>` — coordinate non-overlapping transmission windows
    - Support stations from any country/region (worldwide registration)
    - _Requirements: 18.1, 18.3, 18.4, 17.5_

  - [ ] 14.3 Implement federation REST API endpoints
    - Implement `GET /api/federation/stations` and `POST /api/federation/register`
    - Integrate with Mission Ops Suite consolidated view
    - _Requirements: 18.2, 18.5_

  - [ ]* 14.4 Write property test for federation registration and discovery
    - **Property 35: Ground station federation registration and discovery**
    - Assert registered station is discoverable when querying for satellite visibility
    - **Validates: Requirements 18.1**

  - [ ]* 14.5 Write property test for federation consolidated view completeness
    - **Property 36: Federation consolidated view completeness**
    - Assert consolidated view includes every registered station with location, capabilities, assignments
    - **Validates: Requirements 18.2**

  - [ ]* 14.6 Write property test for pass assignment avoiding redundant transmissions
    - **Property 37: Pass assignment avoids redundant transmissions**
    - Assert no two stations transmit to same satellite simultaneously
    - **Validates: Requirements 18.3**

  - [ ]* 14.7 Write property test for worldwide ground station registration
    - **Property 57: Worldwide ground station registration**
    - Assert registration succeeds for any valid lat/lon/callsign regardless of country
    - **Validates: Requirements 17.5**

- [ ] 15. Pass Predictor — TLE/OMM orbital mechanics
  - [ ] 15.1 Implement pass prediction using satellite.js or tle.js
    - Implement `predictPasses(tle, station, start, end, minElevation?): Promise<PassPrediction[]>`
    - Compute AOS, LOS, max elevation, duration for each pass
    - Implement `generateContactPlan(passes): Contact[]` to convert predictions to ION contact plan entries
    - Integrate with Mission Ops Suite via `GET /api/passes`
    - _Requirements: 14.5_

  - [ ]* 15.2 Write property test for pass prediction correctness
    - **Property 42: Pass prediction correctness**
    - Assert predicted AOS/LOS within 60 seconds of reference SGP4/SDP4 computation
    - **Validates: Requirements 14.5**

- [ ] 16. Gateway Bridge — DTN ↔ amateur packet network translation
  - [ ] 16.1 Implement DTN-to-APRS and APRS-to-DTN translation
    - Implement `dtnToAprs(bundlePayload, destination): string`
    - Implement `aprsToDtn(aprsMessage, dtnEndpoint): Uint8Array`
    - Map DTN endpoint callsign-SSIDs to APRS callsign-SSIDs transparently
    - _Requirements: 19.1, 19.2, 19.3_

  - [ ] 16.2 Implement gateway bridge event logging
    - Define `BridgeMessage` interface
    - Implement `logBridgeEvent(msg): Promise<void>` logging source/destination protocol, timestamp, delivery status
    - _Requirements: 19.4_

  - [ ]* 16.3 Write property test for gateway protocol translation round-trip
    - **Property 38: Gateway protocol translation round-trip**
    - Assert DTN → APRS → DTN produces payload equivalent to original
    - **Validates: Requirements 19.1, 19.2, 19.3**

  - [ ]* 16.4 Write property test for gateway logging completeness
    - **Property 39: Gateway logs all bridged messages**
    - Assert every bridged message creates log entry with source protocol, destination protocol, timestamp, status
    - **Validates: Requirements 19.4**

- [ ] 17. Data Archive and Experiment Replay
  - [ ] 17.1 Implement ArchiveRecord types and SQLite schema
    - Define `ArchiveRecord` interface with schema versioning
    - Create SQLite `archive` table with indices as specified in design
    - _Requirements: 23.1, 23.2_

  - [ ] 17.2 Implement archive storage, query, and export
    - Implement `storeArchiveRecord(record): Promise<void>`
    - Implement `queryArchive(start, end, nodeId?, phase?, recordType?): Promise<ArchiveRecord[]>`
    - Implement `exportArchive(records, format): Promise<Uint8Array>` supporting CSV, JSON, Parquet
    - _Requirements: 23.1, 23.2, 23.3_

  - [ ] 17.3 Implement replay tool for historical network state reconstruction
    - Implement `replaySession(records, speed?): Promise<void>` feeding archived data through dashboard/mission ops
    - _Requirements: 23.4, 23.5_

  - [ ]* 17.4 Write property test for archive query correctness
    - **Property 48: Archive query returns matching records**
    - Assert query returns exactly records matching all filters, no non-matching records
    - **Validates: Requirements 23.3**

  - [ ]* 17.5 Write property test for archive export format validity
    - **Property 49: Archive records in valid open format with schema**
    - Assert exported data parseable by standard parser, conforms to schema
    - **Validates: Requirements 23.2**

  - [ ]* 17.6 Write property test for archive replay state reconstruction
    - **Property 50: Archive replay reconstructs historical state**
    - Assert replayed records produce network state consistent with original recorded state
    - **Validates: Requirements 23.4**

- [ ] 18. Public Dashboard — real-time web visualisation
  - [ ] 18.1 Create Oak web server with Handlebars/Bootstrap dashboard UI
    - Implement satellite position display on interactive map using TLE/OMM data
    - Display federated ground station locations and online status
    - Display recent bundle deliveries with hop count and latency
    - Display network health metrics (throughput, success rate, active links)
    - _Requirements: 22.1, 22.2, 22.3, 22.4, 22.5_

  - [ ] 18.2 Implement WebSocket channels for live updates
    - Implement `/ws/telemetry`, `/ws/bundles`, `/ws/positions` WebSocket channels
    - Implement REST endpoints: `/api/dashboard/satellites`, `/api/dashboard/stations`, `/api/dashboard/metrics`, `/api/dashboard/deliveries`
    - _Requirements: 22.1, 22.6_

  - [ ]* 18.3 Write property test for dashboard API completeness
    - **Property 47: Dashboard API returns data for all registered entities**
    - Assert API returns data for all stations, deliveries, and health metrics
    - **Validates: Requirements 22.3, 22.4, 22.5**

- [ ] 19. Telemetry Protocol Schemas and Reference Decoder
  - [ ] 19.1 Document and publish telemetry protocol schemas
    - Define frame formats, field definitions, encoding rules, byte-level specifications
    - Publish schema files in the public repository
    - Ensure documentation sufficient for independent decoder implementation
    - _Requirements: 15.1, 15.3_

  - [ ] 19.2 Implement telemetry API and reference decoder
    - Implement documented REST API for accessing decoded telemetry data
    - Implement reference decoder for all downlink telemetry formats
    - _Requirements: 15.2, 15.5_

  - [ ]* 19.3 Write property test for telemetry API schema conformance
    - **Property 45: Telemetry API returns schema-conformant data**
    - Assert API responses conform to published schema with correct field types
    - **Validates: Requirements 15.2**

  - [ ]* 19.4 Write property test for reference decoder correctness
    - **Property 46: Reference decoder correctly parses telemetry frames**
    - Assert decoder parses valid frames and extracts fields matching original data
    - **Validates: Requirements 15.5**

- [ ] 20. Checkpoint — Cross-cutting components (Telemetry, Mission Ops, Federation, Pass Predictor, Gateway, Archive, Dashboard, Telemetry Schemas)
  - Ensure all tests pass, ask the user if questions arise.

### CI/CD and Integration

- [ ] 21. CI/CD Pipeline — build, test, deploy
  - [ ] 21.1 Create GitHub Actions CI workflow
    - Define pipeline stages: lint/static analysis → ION baseline tests → unit tests → property tests → integration tests → coverage → Docker build
    - Configure deno lint for TypeScript, clang-tidy/cppcheck for C
    - Enforce 80% code coverage gate
    - Target total pipeline under 30 minutes
    - _Requirements: 25.1, 25.2, 25.3, 25.4, 25.5, 25.6, 24.2_

  - [ ] 21.2 Create GitHub Actions CD workflow
    - Produce versioned container images and Deno compiled binaries on main branch
    - Support staged rollout to testbed nodes (opt-in auto-update)
    - Generate release notes summarising changes, test results, known issues
    - Ensure CD does NOT auto-deploy to CubeSat or cislunar flight systems
    - _Requirements: 26.1, 26.2, 26.3, 26.4, 26.5, 26.6_

  - [ ]* 21.3 Write property test for CD pipeline flight system exclusion
    - **Property 59: CD pipeline does not auto-deploy to flight systems**
    - Assert deployment target set never includes CubeSat or Cislunar Payload
    - **Validates: Requirements 26.6**

- [ ] 22. End-to-end interoperability and multi-hop integration
  - [ ] 22.1 Implement end-to-end bundle delivery across multi-hop topology
    - Wire Config Manager → Docker Testbed → Link Emulator → Telemetry → Mission Ops → Dashboard
    - Verify bundle delivery from source through relay to destination with CGR
    - Verify delivery confirmation report sent back to source
    - _Requirements: 9.1, 9.4_

  - [ ]* 22.2 Write property test for bundle integrity across multi-hop delivery
    - **Property 28: Bundle integrity preserved across multi-hop delivery**
    - Assert BIB verifies at destination after multi-hop transit
    - **Validates: Requirements 9.2**

  - [ ]* 22.3 Write property test for unified CGR across segments
    - **Property 29: Unified CGR across all segments**
    - Assert CGR computes valid routes spanning multiple segment types
    - **Validates: Requirements 9.3**

  - [ ]* 22.4 Write property test for delivery confirmation report
    - **Property 30: Delivery confirmation report sent to source**
    - Assert destination sends confirmation report referencing correct bundle ID
    - **Validates: Requirements 9.4**

  - [ ]* 22.5 Write property test for end-to-end delivery time bound
    - **Property 31: End-to-end delivery within time bound**
    - Assert total delivery time ≤ sum of segment delays + 120s margin
    - **Validates: Requirements 9.5**

- [ ] 23. Checkpoint — CI/CD and end-to-end integration
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 24. Dockerfile for Mission Ops and Dashboard
  - Write `docker/Dockerfile.mission-ops` packaging Mission Ops Suite and Public Dashboard
    - Include Deno runtime, Oak server, Handlebars templates, Bootstrap assets, SQLite
    - Ensure deployable on COTS hardware (Raspberry Pi, commodity PC)
    - _Requirements: 14.7, 22.6_

### Optional Requirements (Requirements 28–33)

- [ ] 30. (Optional) CCSDS File Delivery Protocol (CFDP) demonstration
  - [ ] 30.1 Enable and configure ION's CFDP implementation
    - Configure CFDP for Class 1 (unreliable) and Class 2 (reliable) file delivery modes
    - Test file segmentation, bundle delivery, and reassembly at destination
    - Test CFDP under simulated link disruption conditions
    - _Requirements: 28.1, 28.2, 28.3, 28.4_

  - [ ]* 30.2 Write property test for CFDP file transfer round-trip
    - **Property 51: CFDP file transfer round-trip**
    - Assert reassembled file byte-identical to original
    - **Validates: Requirements 28.3**

  - [ ]* 30.3 Write property test for CFDP delivery surviving link disruption
    - **Property 52: CFDP delivery survives link disruption**
    - Assert transfer completes after link restoration, file byte-identical
    - **Validates: Requirements 28.4**

- [ ] 31. (Optional) CCSDS Space Packet Protocol telemetry framing
  - [ ] 31.1 Implement CCSDS Space Packet framing for CubeSat/cislunar telemetry
    - Frame telemetry using CCSDS 133.0-B packet structures (version, type, APID, sequence count, data length)
    - Publish APID allocation table and packet data field definitions
    - Implement ground station decoder for CCSDS Space Packets
    - Ensure Space Packet encapsulation inside DTN bundles for store-and-forward
    - _Requirements: 29.1, 29.2, 29.3, 29.4, 29.5_

  - [ ]* 31.2 Write property test for CCSDS Space Packet header fields
    - **Property 53: CCSDS Space Packet header contains required fields**
    - Assert header contains version, type, APID, sequence count, data length — all valid per CCSDS 133.0-B
    - **Validates: Requirements 29.2**

  - [ ]* 31.3 Write property test for CCSDS Space Packet decoder round-trip
    - **Property 54: CCSDS Space Packet decoder round-trip**
    - Assert decode then re-encode produces equivalent packet
    - **Validates: Requirements 29.4**

  - [ ]* 31.4 Write property test for Space Packet DTN bundle encapsulation round-trip
    - **Property 55: Space Packet DTN bundle encapsulation round-trip**
    - Assert encapsulate in bundle then extract produces byte-identical packet
    - **Validates: Requirements 29.5**

- [ ] 32. (Optional) CCSDS Proximity-1 Space Link Protocol demonstration
  - [ ] 32.1 Implement Proximity-1 CLA as C extension
    - Write Proximity-1 convergence layer adapter for short-range relay simulation
    - Integrate with ION's CLA framework
    - Create Phase 1 testbed scenario with Proximity-1 relay segment alongside TCP/UDP/AX.25
    - _Requirements: 30.1, 30.2, 30.3, 30.4, 30.5, 30.6_

  - [ ]* 32.2 Write property test for Proximity-1 CLA bundle forwarding
    - **Property 56: Proximity-1 CLA forwards bundles**
    - Assert bundle delivered to next-hop with payload intact
    - **Validates: Requirements 30.4**

- [ ] 33. (Optional) APRS-style SSID role conventions
  - [ ] 33.1 Define and publish SSID role assignment table
    - Define SSID-to-role mapping (e.g., -1 testbed, -2 ground station, -3 gateway, -9 mobile, -10 relay, -15 telemetry)
    - Publish in project documentation and configuration guide
    - Display node role from SSID in Mission Ops Suite and Public Dashboard
    - Ensure non-conforming nodes are still accepted by the network
    - _Requirements: 31.1, 31.2, 31.3, 31.4_

- [ ] 34. (Optional) DTN node beacon bundles
  - [ ] 34.1 Implement periodic beacon bundle emission
    - Emit beacon bundle at configurable interval (default 10 minutes) containing callsign-SSID, position, capabilities, health status
    - Document beacon format in telemetry protocol schema
    - Consume beacons in Federation Service and Public Dashboard for status/map updates
    - Make beacon interval and content configurable via TOML
    - _Requirements: 32.1, 32.2, 32.3, 32.4, 32.5_

- [ ] 35. (Optional) APRS-compatible message format for Gateway Bridge
  - [ ] 35.1 Implement APRS directed message formatting in Gateway Bridge
    - Format outbound APRS messages using standard directed format (`:{addressee}:{message}`)
    - Parse inbound APRS directed messages for DTN encapsulation
    - Map DTN endpoint callsign-SSIDs to APRS callsign-SSIDs transparently
    - Document APRS message format mapping
    - _Requirements: 33.1, 33.2, 33.3, 33.4_

- [ ] 36. ION-DTN baseline test suite verification
  - [ ] 36.1 Verify ION-DTN alltests passes on all target platforms
    - Run ION-DTN `alltests` on Linux x86_64 and ARM (Raspberry Pi)
    - Ensure no existing ION tests are modified or disabled
    - _Requirements: 27.1, 27.2, 27.4_

  - [ ]* 36.2 Write property test for ION-DTN baseline test suite
    - **Property 62: ION-DTN baseline test suite passes**
    - Assert `alltests` passes on all target platforms without modification
    - **Validates: Requirements 27.1, 27.2**

- [ ] 37. Final checkpoint — Full system integration
  - Ensure all tests pass, ask the user if questions arise.

### Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Cross-cutting tasks (11–24) build on Phase 1 infrastructure and should be completed after Phase 1
- Optional tasks (30–36) correspond to optional requirements 28–33
- Task 37 is the final integration checkpoint across all phases
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document


---

## Phase 2 — CubeSat Tasks (Conditional on CubeSat Host)

Requirements: 4, 5, 6 | Status: Conditional on CubeSat Host

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

### Tasks

- [ ] 25. CubeSat ION-DTN integration and ground station communication
  - [ ] 25.1 Create ION-DTN build configuration for CubeSat flight computer (ARM/RISC-V)
    - Cross-compile ION-DTN for target COTS flight computer architecture
    - Configure `ipn:100` addressing scheme for compact bandwidth-efficient addressing
    - Configure AX.25 v2.2 CLA with BPSK 9K6 modulation and LTP CLA for ground-to-space
    - Ensure memory footprint ≤ 256 MB, non-volatile storage ≥ 1 GB
    - _Requirements: 4.1, 4.2, 4.3, 4.5_

  - [ ] 25.2 Implement ground station Bundle Agent configuration for CubeSat communication
    - Create TOML config templates for ground stations using `dtn://` callsign-SSID addressing
    - Configure AX.25 UHF (70 cm band) and LTP CLAs
    - Implement link acquisition detection (within 10s for CubeSat, 15s for ground station)
    - Implement pass-end checkpoint for in-progress transfers (within 5s of link loss)
    - _Requirements: 5.1, 5.2, 5.4, 4.4_

  - [ ] 25.3 Implement ground station transfer logging
    - Create transfer log recording bundle identifiers, timestamps, transfer status, byte counts per pass
    - _Requirements: 5.5_

  - [ ] 25.4 Implement power loss checkpoint and recovery for CubeSat Bundle Agent
    - Persist transfer state to non-volatile storage at regular checkpoints
    - On power restoration, detect incomplete transfers and resume from last checkpoint
    - Log power loss event and recovery status
    - _Requirements: 4.6_

  - [ ]* 25.5 Write property test for CubeSat memory footprint
    - **Property 60: CubeSat memory footprint within 256 MB**
    - Assert total memory usage ≤ 256 MB under all workloads
    - **Validates: Requirements 4.2**

  - [ ]* 25.6 Write property test for Bundle Agent recovery after power loss
    - **Property 61: Bundle Agent recovery after power loss**
    - Assert recovery and resume from last checkpoint without data loss
    - **Validates: Requirements 4.6**

  - [ ]* 25.7 Write property test for transfer log fields
    - **Property 58: Transfer log contains required fields**
    - Assert log contains bundle ID, timestamp, transfer status, byte count — all non-empty
    - **Validates: Requirements 5.5**

- [ ] 26. CubeSat custody transfer
  - [ ] 26.1 Configure ION custody transfer for CubeSat Bundle Agent
    - Enable custody transfer for bundles marked with custody-requested delivery options
    - Configure custody acceptance signal generation and 120-second retransmission timeout
    - Configure non-volatile storage retention for custodied bundles
    - Implement custody event logging (bundle ID, timestamp, outcome)
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ]* 26.2 Write property test for custody acceptance signal
    - **Property 19: Custody acceptance signal sent on custody acceptance**
    - Assert custodian sends acceptance signal referencing correct bundle ID
    - **Validates: Requirements 6.1, 6.2**

  - [ ]* 26.3 Write property test for custody bundle retention invariant
    - **Property 20: Custody bundle retention invariant**
    - Assert custodied bundle never absent from storage until transferred or delivered
    - **Validates: Requirements 6.3**

  - [ ]* 26.4 Write property test for custody retransmission on timeout
    - **Property 21: Custody retransmission on timeout**
    - Assert bundle retransmitted if custody acceptance not received within 120 seconds
    - **Validates: Requirements 6.4**

  - [ ]* 26.5 Write property test for custody event logging
    - **Property 22: Custody events logged with required fields**
    - Assert log entry contains bundle ID, timestamp, transfer outcome
    - **Validates: Requirements 6.5**

- [ ] 27. Checkpoint — Phase 2 CubeSat integration and custody transfer
  - Ensure all tests pass, ask the user if questions arise.

### Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Phase 2 tasks (25–27) are conditional on identifying a CubeSat host
- Phase 2 depends on Phase 1 infrastructure being complete
- Cross-cutting requirements from the master document also apply to Phase 2
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document


---

## Phase 3 — Cislunar Payload Tasks (Conditional on ESA ARTES)

Requirements: 7, 8 | Status: Conditional on ESA ARTES Approval

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

### Tasks

- [ ] 28. Cislunar payload ION-DTN integration
  - [ ] 28.1 Create ION-DTN build configuration for cislunar radiation-tolerant COTS hardware
    - Cross-compile ION-DTN for target radiation-tolerant flight computer
    - Configure `ipn:200` addressing scheme
    - Configure LTP CLA calibrated for ~2.5s RTT with appropriate retransmission timers
    - Ensure non-volatile storage ≥ 4 GB for bundle buffering during lunar occlusion
    - _Requirements: 7.1, 7.3, 8.1, 8.2_

  - [ ] 28.2 Implement SEU detection and error correction handling
    - Implement software-based SEU detection and ECC/scrubbing
    - Log SEU events with timestamp, memory address, correction status
    - Isolate affected memory on correction failure, continue in degraded mode
    - Report SEU events via telemetry
    - _Requirements: 7.4, 7.5_

  - [ ] 28.3 Implement CGR with lunar occlusion window support
    - Model lunar occlusion windows as no-contact periods in the contact plan
    - Ensure CGR does not schedule forwarding during occlusion periods
    - Implement store-and-resume: bundles stored during occlusion, forwarding resumes within 30s of reacquisition
    - _Requirements: 8.4, 8.5_

  - [ ]* 28.4 Write property test for LTP retransmission timer calibration
    - **Property 32: LTP retransmission timers calibrated to RTT**
    - Assert retransmission timer ≥ current RTT for any LTP session
    - **Validates: Requirements 8.2**

  - [ ]* 28.5 Write property test for CGR respecting lunar occlusion
    - **Property 33: CGR respects lunar occlusion windows**
    - Assert CGR does not schedule forwarding during occlusion periods
    - **Validates: Requirements 8.4**

  - [ ]* 28.6 Write property test for bundle store and resume after occlusion
    - **Property 34: Bundle store and resume after occlusion**
    - Assert bundles stored during occlusion, forwarding resumes within 30s of reacquisition
    - **Validates: Requirements 8.5**

- [ ] 29. Checkpoint — Phase 3 cislunar integration
  - Ensure all tests pass, ask the user if questions arise.

### Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Phase 3 tasks (28–29) are conditional on ESA ARTES approval
- Phase 3 depends on Phase 1 infrastructure and cross-cutting components being complete
- Cross-cutting requirements from the master document also apply to Phase 3
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document

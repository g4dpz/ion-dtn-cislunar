# Implementation Tasks — Master (Cross-Cutting, CI/CD, Integration, Optional)

Requirements: 9–19, 21–27, 28–34 | Builds on Phase 1 infrastructure

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

## Cross-Cutting Components (Requirements 9–19, 21–27, 34)

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

## CI/CD and Integration

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

## Optional Requirements (Requirements 28–33)

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

## Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Cross-cutting tasks (11–24) build on Phase 1 infrastructure and should be completed after Phase 1
- Optional tasks (30–36) correspond to optional requirements 28–33
- Task 37 is the final integration checkpoint across all phases
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document

# Implementation Tasks — Phase 1: Terrestrial Testbed

Requirements: 1, 2, 3, 20 | Status: Active, Unconditional

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

## Tasks

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

## Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Tasks 1–10 are unconditional and should be completed first before cross-cutting or later phases
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document
- Checkpoints ensure incremental validation at natural breakpoints

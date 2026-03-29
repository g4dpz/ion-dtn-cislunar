# Implementation Tasks — Phase 2: CubeSat (LEO)

Requirements: 4, 5, 6 | Status: Conditional on CubeSat Host

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

## Tasks

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

## Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Phase 2 tasks (25–27) are conditional on identifying a CubeSat host
- Phase 2 depends on Phase 1 infrastructure being complete
- Cross-cutting requirements from the master document also apply to Phase 2
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document

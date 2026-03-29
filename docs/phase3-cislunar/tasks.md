# Implementation Tasks — Phase 3: Cislunar Payload

Requirements: 7, 8 | Status: Conditional on ESA ARTES Approval

Tech stack: TypeScript/Deno for tooling, C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

## Tasks

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

## Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Phase 3 tasks (28–29) are conditional on ESA ARTES approval
- Phase 3 depends on Phase 1 infrastructure and cross-cutting components being complete
- Cross-cutting requirements from the master document also apply to Phase 3
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document

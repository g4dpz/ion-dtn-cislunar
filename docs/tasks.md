# Implementation Plan: ION-DTN Cislunar Demonstration

## Overview

This plan implements the ION-DTN Cislunar Demonstration project in phased order: Phase 1 (Terrestrial Testbed, active/unconditional) first, then cross-cutting components, then Phase 2 (CubeSat, conditional on host), then Phase 3 (Cislunar, conditional on ESA ARTES). Optional requirements (28–33) are marked as optional tasks. The tech stack is TypeScript/Deno for all tooling and C for ION extensions. Property-based tests use fast-check (TypeScript) and theft (C).

## Task Documents

Tasks are split across four documents, mirroring the requirements and design structure:

- #[[file:phase1-terrestrial/tasks.md]] — Phase 1: Terrestrial Testbed (Tasks 1–10, Requirements 1, 2, 3, 20) — Active, Unconditional
- #[[file:master/tasks.md]] — Master: Cross-Cutting, CI/CD, Integration, Optional (Tasks 11–24, 30–37, Requirements 9–19, 21–34) — Builds on Phase 1
- #[[file:phase2-cubesat/tasks.md]] — Phase 2: CubeSat (Tasks 25–27, Requirements 4, 5, 6) — Conditional on CubeSat Host
- #[[file:phase3-cislunar/tasks.md]] — Phase 3: Cislunar Payload (Tasks 28–29, Requirements 7, 8) — Conditional on ESA ARTES

## Execution Order

1. **Phase 1** (Tasks 1–10) — unconditional, complete first
2. **Master cross-cutting** (Tasks 11–24) — builds on Phase 1 infrastructure
3. **Phase 2** (Tasks 25–27) — conditional on CubeSat host
4. **Phase 3** (Tasks 28–29) — conditional on ESA ARTES approval
5. **Optional** (Tasks 30–36) — can be done at any point after Phase 1
6. **Final checkpoint** (Task 37) — full system integration

## Summary

| Document | Tasks | Requirements | Status |
|----------|-------|-------------|--------|
| Phase 1 — Terrestrial | 1–10 | 1, 2, 3, 20 | Active, Unconditional |
| Master — Cross-Cutting | 11–24, 30–37 | 9–19, 21–34 | Builds on Phase 1 |
| Phase 2 — CubeSat | 25–27 | 4, 5, 6 | Conditional (CubeSat host) |
| Phase 3 — Cislunar | 28–29 | 7, 8 | Conditional (ESA ARTES) |

## Notes

- Tasks marked with `*` are property-based tests — optional for faster MVP
- Each task references specific requirements for traceability
- Property tests reference specific correctness properties from the master design document
- Checkpoints ensure incremental validation at natural breakpoints
- TypeScript/Deno for all tooling; C for ION extensions and CLAs
- Property-based tests use fast-check (TypeScript) and theft (C)

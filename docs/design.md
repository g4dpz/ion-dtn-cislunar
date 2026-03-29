# Design: ION-DTN Cislunar Demonstration

This spec consolidates design from the following documents:

- #[[file:master/design.md]]
- #[[file:phase1-terrestrial/design.md]]
- #[[file:phase2-cubesat/design.md]]
- #[[file:phase3-cislunar/design.md]]

## Summary

12 software components, 65 correctness properties, full data models and testing strategy.

### Components

1. Configuration Manager — TOML parser, validator, ION compiler
2. Docker Testbed — multi-node ION-DTN topology
3. Link Emulator — delay, jitter, bandwidth, loss, disruption profiles
4. ION C Extensions — AX.25 v2.2 CLA and telemetry hook
5. BPSec — Bundle integrity verification
6. Telemetry Aggregator — collection, storage, export
7. Mission Operations Suite — web-based network control
8. Federation Service — ground station registration and pass coordination
9. Pass Predictor — TLE/OMM orbital mechanics
10. Gateway Bridge — DTN ↔ amateur packet network translation
11. Data Archive and Experiment Replay
12. Public Dashboard — real-time web visualisation

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Protocol | ION-DTN (C), Bundle Protocol v7, LTP, CGR, BPSec, CFDP |
| Tooling & Web | Deno, Oak, Handlebars, Bootstrap, TypeScript |
| Storage | SQLite |
| Radio | AX.25 v2.2, FM/AFSK (1200 baud), BPSK (9600 bps) |
| Infrastructure | Docker, GitHub Actions CI/CD |
| Testing | fast-check (TypeScript), theft (C) |

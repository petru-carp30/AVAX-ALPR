# AVAX ALPR Deferred Scope

**Purpose:** preserve non-essential or non-blocking work intentionally deferred to accelerate completion of the first usable AVAX ALPR application.

This is not a deletion list. Items here remain part of the project backlog and may be resumed after the accelerated MVP/pilot is functional.

## Delivery policy

The accelerated target is a practical first version that delivers the core guard workflow at roughly 80% of the long-term target, prioritizing end-to-end functionality over optimization, polish, broad device coverage, analytics, and advanced administration.

Core MVP path:

```text
CameraX
-> License Plate Detector
-> OCR
-> Plate Normalization
-> Local Room Lookup
-> Access Decision
-> Local Access Log
-> Background Access Log Sync
```

The system remains offline-first. AI still never decides access.

## AI / Dataset — deferred after first working detector/OCR pipeline

- additional detector architectures/challengers after the frozen YOLOX-Nano 512 baseline
- 640x640 detector challenger
- broad hyperparameter searches
- extra detector threshold sweeps after the selected operating point is validated
- FP16/INT8 quantization unless required to make Android latency usable
- TFLite/LiteRT comparison if ONNX Runtime is already acceptable on the target device
- multi-runtime benchmarking beyond the runtime selected for the first mobile integration
- large-scale detector hard-negative expansion beyond the current audited pool unless false positives become a blocker
- `AI-DATA-WP-002 — AVAX Field Domain Adaptation & Validation Dataset`
- private AVAX field-image collection/domain adaptation
- extensive detector error taxonomy beyond what is needed to identify blockers
- additional synthetic-data experiments

## Guard Mobile — deferred after end-to-end automatic ALPR works

- production-quality animated bounding-box overlay
- object tracking between frames
- temporal smoothing/voting across long frame sequences
- sophisticated adaptive frame-rate/inference scheduling
- multi-device performance matrix
- landscape-specific camera tuning unless required by actual use
- UI polish beyond clear access result and basic diagnostics
- automatic background vehicle snapshot synchronization; manual/start-of-shift sync is acceptable for the accelerated MVP if kept explicit
- extensive camera diagnostics once AI integration is stable

## Manager / Admin — deferred

- access-request workflow
- Manager Approve/Deny flow
- push notifications
- administration UI
- dashboards
- analytics
- historical reporting beyond basic access-log availability

These remain part of later project phases and are not blockers for the first Guard ALPR MVP.

## Backend / Database — deferred where not required for the pilot

- incremental Vehicle Sync v2
- `ARCH-001 — Vehicle Identity & Incremental Sync Strategy`
- AVAX_VEHICLES schema cleanup for PK/identity/unique/index technical debt unless a production blocker appears
- access-log retention/automatic deletion policy
- advanced reporting endpoints

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment` is not removed. It is required before production central access-log storage, but it does not block completion of the on-device automatic ALPR pipeline.

## Security / Testing / Deployment — deferred depth, not minimum safety

Deferred:

- exhaustive security hardening beyond the minimum needed for the pilot environment
- broad device/OS compatibility matrix
- long-duration soak/performance testing
- full deployment automation
- production observability/analytics

Not deferred:

- build stability
- basic automated regression tests for changed components
- physical-device validation of the automatic ALPR flow
- preserving offline-first behavior
- no direct Guard-to-SQL connectivity
- no AI-based access decision
- no raw camera-frame persistence/upload unless explicitly authorized

## Re-entry rule

After the first end-to-end automatic Guard ALPR workflow is validated on the target physical device, revisit this file and promote deferred items based on measured problems rather than anticipated complexity.

Priority order after MVP should be driven by:

1. blockers observed on the real gate/device;
2. production data persistence/security requirements;
3. access-request/manager workflow;
4. domain adaptation and AI accuracy improvements;
5. administration/analytics/polish.

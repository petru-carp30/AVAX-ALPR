# AVAX ALPR Project Status

**Status snapshot:** 2026-09-07  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery is operating in **ACCELERATED MVP MODE**.

Goal: finish the first usable Guard ALPR application as quickly as possible, accepting an approximately 80% solution if the core operational flow works reliably.

Deferred/non-essential work is preserved in:

`documentation/DEFERRED_SCOPE.md`

Core accelerated MVP flow:

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

The system remains offline-first. AI does not decide access.

## Current critical path

The detector side is now complete through physical Android integration.

Remaining critical path:

```text
AI-WP-002 — OCR MVP Baseline & Mobile Contract
        ↓
MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline
        ↓
Physical end-to-end MVP validation
```

Parallel execution means work packages may complete at different times. `MOB-AI-WP-001` finishing before the OCR workstream is expected and does not invalidate the parallel plan.

## AI detector status

### AI-DATA-WP-001 — Detector Dataset Foundation

- Priority: `P0`
- Status: `DONE`
- Accepted dataset: `AI/PlateDetector/datasets/derived/baseline_v1`
- 8916 images
- 8412 positives
- 504 audited real negatives
- 11580 plate instances
- TRAIN 7622 / VAL 625 / TEST 669
- source-group leakage 0
- exact-pixel leakage 0
- unresolved near-duplicate candidates 0

### AI-WP-001 — Detector Baseline & Mobile Export Contract

- Priority: `P0`
- Status: `DONE`

Frozen detector:

- architecture: `YOLOX-Nano`
- input: `512x512`
- selected epoch: `60`
- confidence threshold: `0.225`
- NMS threshold: `0.45`

Final TEST one-shot:

- mAP@0.50:0.95: `0.450475`
- mAP@0.50: `0.812652`
- Precision: `0.824503`
- Recall: `0.817287`
- F1: `0.820879`

Accepted ONNX artifact:

`avax_plate_detector_yolox_nano_512_v1.onnx`

- opset: `17`
- size: `3,703,049 bytes`
- SHA256: `B42313FE76EFCD98332430A25FE6BEEC553FC5FE7633957917453836AEA81793`

Reference AI commit:

`5d2e1022d924a8b364b86882b9d4d57c94b652cc`

## MOB-AI-WP-001 — On-device Detector Integration

- Priority: `P0`
- Status: `DONE`
- Target project: AVAX ALPR – Guard Mobile App

Master accepted the mobile detector integration handoff.

Implemented:

- ONNX Runtime Android
- exact accepted detector artifact validation by size/SHA256/input/output metadata
- YUV CameraFrame -> BGR -> rotation -> 512x512 top-left letterbox -> NCHW float32 preprocessing
- confidence `0.225`
- NMS `0.45`
- inverse letterbox/rotation to original CameraFrame coordinates
- `PlateDetection(left, top, right, bottom, confidence)` semantic output
- `PlateDetectorFrameProcessor` connected to existing CameraX `FrameProcessor` boundary
- `ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST` preserved
- fixed inference cadence capability available through `minInferenceIntervalMs`
- minimal detector diagnostics and white bounding-box overlay
- detector failure preserves manual Guard fallback
- no raw camera-frame persistence/upload

Physical target device:

`Google Pixel 6 Pro`

Observed offline performance:

- model load: approximately `107–150 ms`
- model inference: approximately `151–210 ms`
- total preprocessing + inference + postprocessing: approximately `383–554 ms/frame`
- observed cadence: approximately `1.8–2.6 fps`

Representative run:

- load: `107.2 ms`
- detections: `2`
- inference: `210.3 ms`
- total: `497.0 ms`
- cadence: `2.0 fps`

Physical validation passed:

- offline application start
- CameraX preview
- offline model load
- real plate detection
- bounding-box correspondence
- lifecycle/background-foreground recovery
- manual plate verification fallback
- no raw-frame persistence/upload
- responsive application during inference

Automated validation:

- `testDebugUnitTest` — PASS
- `assembleDebug` — PASS

Reference Guard commit:

`4ee903de9ab823312c98263bbb7cd22938277e92`

Known non-blocking mobile detector debt:

- preprocessing is a meaningful part of current end-to-end detector latency
- current approximately 2 fps cadence is accepted for gate-scanning MVP
- production overlay mapping/polish is deferred
- `PlateDetectorFrameProcessor.kt` package/source-tree organization may be cleaned up later

## AI-WP-002 — OCR MVP Baseline & Mobile Contract

- Priority: `P0`
- Status: `TODO / READY TO START OR CONTINUE`
- Target project: AVAX ALPR – AI Model

Objective:

Provide the simplest viable offline Android-capable plate OCR solution and a minimal integration contract.

Prefer an existing lightweight offline Latin OCR solution over custom training if it is usable and legally suitable.

Deferred unless a real blocker appears:

- custom OCR training from scratch
- broad OCR model comparison
- sophisticated grammar engines
- multi-frame OCR voting
- extensive augmentation/benchmark studies

## MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline

- Priority: `P0`
- Status: `TODO / BLOCKED ONLY ON OCR CONTRACT`
- Target project: AVAX ALPR – Guard Mobile App

Satisfied dependency:

- `MOB-AI-WP-001 — DONE`

Remaining dependency:

- usable `AI-WP-002` OCR engine/contract

Target flow:

```text
CameraFrame
-> PlateDetection
-> plate crop
-> OCR
-> PlateNormalizer
-> Room local lookup
-> AccessChecker
-> access result
-> existing local access log
-> existing background sync
```

Manual plate entry remains the fallback if automatic OCR fails.

## Confirmed completed foundation

| ID | Work item | Priority | Status |
|---|---|---:|---|
| BE-001 | Backend Baseline Audit & Build Validation | P0 | DONE |
| SEC-001 | Resolve NU1903 Microsoft.OpenApi Vulnerability | P0 | DONE |
| BE-002 | Validate Existing SQL Schema Relevant to ALPR | P0 | DONE |
| DEVDB-001 | Local SQLite Development Database Baseline | P1 | DONE |
| BE-WP-001 | Local Backend Vehicle Read API Foundation | P0 | DONE |
| BE-WP-002 | Vehicle Snapshot Sync API v1 | P0 | DONE |
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | P0 | DONE |
| MOB-WP-002 | Local Access Logging Foundation | P0 | DONE |
| BE-WP-003 | Access Log Ingestion API v1 | P0 | DONE |
| MOB-WP-003 | Background Access Log Upload | P0 | DONE |
| CAM-WP-001 | CameraX Foundation | P0 | DONE |
| AI-DATA-WP-001 | Detector Dataset Foundation | P0 | DONE |
| AI-WP-001 | Detector Baseline & Mobile Export Contract | P0 | DONE |
| MOB-AI-WP-001 | On-device Detector Integration | P0 | DONE |

## Production follow-up not required to finish app MVP

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Status: `TODO`
- Priority: `P0 before production`
- Not a blocker for finishing the Guard automatic ALPR MVP

## Deferred scope

Deferred work remains documented in:

`documentation/DEFERRED_SCOPE.md`

This includes detector optimization, quantization/runtime comparisons, advanced tracking/overlay, broad device testing, AVAX field-domain adaptation, Manager/Admin workflows, analytics, Sync v2, schema cleanup, and deeper deployment/observability work.

## MVP release gate

The accelerated MVP is functionally complete only after physical-device validation of:

```text
Camera
-> detects plate
-> OCR returns plate text
-> PlateNormalizer normalizes it
-> Room lookup works offline
-> AccessChecker returns local decision
-> result is shown clearly
-> access event is stored locally
-> pending access event can sync when connectivity returns
```

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.
- Minimum physical-device validation of the automatic end-to-end ALPR flow remains mandatory before calling the accelerated MVP complete.

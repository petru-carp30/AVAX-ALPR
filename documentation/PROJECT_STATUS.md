# AVAX ALPR Project Status

**Status snapshot:** 2026-09-07  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery is now operating in **ACCELERATED MVP MODE**.

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

## AI-WP-001 — Detector baseline and mobile export

`AI-WP-001 — License Plate Detector Baseline & Mobile Export Contract`

- Priority: `P0 Critical`
- Status: `DONE`
- Target project: AVAX ALPR – AI Model
- Dataset dependency: `AI-DATA-WP-001 — DONE`

Master accepted the frozen YOLOX-Nano 512 detector handoff for Accelerated MVP integration.

### Frozen detector

Architecture: `YOLOX-Nano`  
Input: `512x512`  
Selected checkpoint epoch: `60`  
Checkpoint SHA256: `9A0C6A9D8ED0B9CAD31212F7ADDECF2C35C4B2CFF932ACFF1D02DF34519746B0`  
Confidence threshold: `0.225`  
NMS threshold: `0.45`

### Final TEST one-shot

Reserved TEST was accessed once after detector freeze. No post-TEST tuning was performed.

- TEST images: `669`
- plate instances: `914`
- negative images: `50`
- mAP@0.50:0.95: `0.450475`
- mAP@0.50: `0.812652`
- TP: `747`
- FP: `159`
- FN: `167`
- Precision: `0.824503`
- Recall: `0.817287`
- F1: `0.820879`
- negative images with false positives: `3`
- negative false-positive detections: `3`

The COCO AP protocol uses its evaluation operating settings; the frozen mobile/runtime operating point remains confidence `0.225` and NMS `0.45`.

### ONNX artifact

Filename:

`avax_plate_detector_yolox_nano_512_v1.onnx`

- format: ONNX
- opset: `17`
- size: `3,703,049 bytes`
- SHA256: `B42313FE76EFCD98332430A25FE6BEEC553FC5FE7633957917453836AEA81793`
- batch: `1`
- dynamic shapes: `NO`
- input: `images [1,3,512,512] float32 NCHW`
- output: `output [1,5376,6]`
- YOLOX grid/stride decode: included in model

ONNX checker, ONNX Runtime load/inference, and PyTorch-vs-ONNX validation passed.

Reference/export comparison on 6 VAL samples:

- raw tensor consistency: PASS
- detection consistency: PASS
- max raw absolute difference: `0.0094757080078125`
- max observed box difference: `0.00006103515625 px`
- max observed confidence difference: `0.00000017881393432617188`

### Frozen Mobile Detector Contract

Preprocessing:

```text
CameraFrame
-> apply rotationDegrees clockwise (0/90/180/270)
-> preserve aspect ratio
-> resize into 512x512 using bilinear interpolation
-> place resized image top-left
-> pad right/bottom with 114
-> HWC -> CHW
-> uint8 -> float32
-> batch dimension
```

No divide-by-255, mean subtraction, or standard-deviation normalization.

Detector semantic output:

```text
PlateDetection
- left: float
- top: float
- right: float
- bottom: float
- confidence: float
```

Coordinates are returned in original `CameraFrame` buffer pixel coordinates after undoing letterbox and rotation.

Detector responsibility remains bounding boxes + confidence only. OCR and access decisions remain separate downstream responsibilities.

### Performance status

Desktop model-forward timing: approximately `1.567 ms/image` on NVIDIA GeForce RTX 5060 Laptop GPU.

This is not an Android benchmark.

Android latency: `NOT YET MEASURED`.

Android runtime latency, thermal behavior and frame-selection behavior move to `MOB-AI-WP-001`.

### Reference commits

Pre-TEST detector freeze:

`82d6dda`

Final AI-WP-001 detector/ONNX handoff:

`5d2e1022d924a8b364b86882b9d4d57c94b652cc`

## Accelerated parallel sequence

Detector research is closed for MVP unless mobile integration exposes a real blocker.

Parallel work is now:

```text
Guard Mobile detector integration
    -> ONNX Runtime Android
    -> physical-device detector benchmark

AI OCR MVP
    -> simplest viable offline OCR baseline/contract

then

Guard Mobile OCR + automatic local verification
```

## Next critical work packages

### MOB-AI-WP-001 — On-device Detector Integration

Status: `TODO / READY TO START`  
Priority: `P0`

Objective:

Connect the accepted detector to the existing CameraX `FrameProcessor` boundary, measure actual target-device latency, and produce valid plate bounding boxes/crops for downstream OCR.

Accelerated runtime decision:

Use ONNX Runtime Android as the first runtime path. Do not evaluate alternative mobile runtimes unless ONNX Runtime fails to provide a usable MVP path.

Minimum success:

- exact accepted ONNX model loads offline on target Android device
- CameraFrame -> model preprocessing matches the frozen AI contract
- detector returns visible plate boxes in original frame coordinates
- confidence `0.225` and NMS `0.45` are applied
- app remains responsive
- actual device latency is measured
- no raw frame upload/persistence
- existing manual plate verification still works

### AI-WP-002 — OCR MVP Baseline & Mobile Contract

Status: `TODO / READY TO START IN PARALLEL`  
Priority: `P0`

Accelerated objective:

Build the simplest usable offline plate OCR path. Prefer an existing lightweight mobile-capable Latin OCR solution before considering custom OCR training.

The objective is useful end-to-end plate recognition, not research-grade OCR optimization.

Deferred unless required by a real blocker:

- large OCR model comparison
- custom OCR training from scratch
- extensive country-specific grammar engines
- sophisticated multi-frame OCR fusion
- exhaustive augmentation studies
- broad benchmark matrix

### MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline

Status: `TODO`  
Priority: `P0`

Dependencies:

- `MOB-AI-WP-001` usable detector integration
- `AI-WP-002` usable OCR contract/runtime recommendation

Objective:

Connect:

```text
PlateDetection
-> Plate Crop
-> OCR
-> PlateNormalizer
-> Room lookup
-> AccessChecker
-> local result
-> existing local access-log flow
```

This work package represents the core first usable automatic ALPR milestone.

## Accepted detector dataset

`AI-DATA-WP-001 — License Plate Detector Dataset Acquisition & Annotation Foundation`

Status: `DONE`

Accepted canonical dataset:

`AI/PlateDetector/datasets/derived/baseline_v1`

- 8916 images
- 8412 positives
- 504 audited real negatives
- 11580 plate instances
- TRAIN 7622
- VAL 625
- TEST 669
- source-group leakage 0
- exact-pixel leakage 0
- unresolved near-duplicate candidates 0
- VAL/TEST real-only

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

## Production follow-up not required to finish the app MVP

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Status: `TODO`
- Priority: `P0 before production`, but not a blocker for completing the on-device Guard ALPR MVP

Production access-log storage must still be completed before production rollout.

## Deferred scope summary

Deferred after first usable MVP unless a measured blocker appears:

- detector challengers/extra training
- AVAX field-domain dataset adaptation
- detector quantization/runtime comparisons
- advanced camera overlays/tracking
- sophisticated frame scheduling
- broad device compatibility matrix
- automatic background vehicle snapshot synchronization
- Manager Approve/Deny
- push notifications
- Admin Dashboard
- analytics
- incremental Vehicle Sync v2
- schema cleanup technical debt
- retention policy
- extensive deployment automation and observability

Full list: `documentation/DEFERRED_SCOPE.md`.

## MVP release gate

The first accelerated MVP is considered functionally complete when, on a physical target device:

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

Manual plate verification remains the fallback if AI confidence/recognition fails.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled; deferred work is preserved in `DEFERRED_SCOPE.md`.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin must not communicate directly with Guard Mobile; server-side communication passes through Backend API.
- Minimum physical-device validation of the automatic ALPR flow is mandatory before calling the accelerated MVP complete.

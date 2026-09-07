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

## Current AI milestone

`AI-WP-001 — License Plate Detector Baseline & Mobile Export Contract`

- Priority: `P0 Critical`
- Status: `IN PROGRESS`
- Target project: AVAX ALPR – AI Model
- Dataset dependency: `AI-DATA-WP-001 — DONE`

TRAIN/VAL detector development is frozen.

### Frozen detector selection

Architecture: `YOLOX-Nano`  
Input: `512x512`  
Selected checkpoint epoch: `60`  
Checkpoint SHA256: `9A0C6A9D8ED0B9CAD31212F7ADDECF2C35C4B2CFF932ACFF1D02DF34519746B0`  
Confidence threshold: `0.225`  
NMS threshold: `0.45`

VAL operating point:

- TP: `682`
- FP: `148`
- FN: `175`
- Precision: `0.8217`
- Recall: `0.7958`
- F1: `0.8085`
- best VAL mAP@0.50:0.95: `0.4250771`

Compared with the previous 416x416 baseline at the same runtime threshold, the selected 512 model produced +30 TP, -30 FN, approximately +3.5 percentage points Recall and approximately +2.2 points F1.

Main observed remaining weakness: small/distant plates, followed by difficult multi-plate scenes, blur, angle, occlusion and some annotation issues.

Selection was frozen before TEST access.

Reference AI commit:

`82d6dda` — Freeze YOLOX Nano 512 detector selection

## Accelerated AI-WP-001 completion scope

To finish the detector quickly, only the following remain P0 for AI-WP-001:

1. run the frozen 512 model once on TEST and record final metrics;
2. export the selected detector to ONNX;
3. validate that ONNX inference is functionally consistent with the selected PyTorch checkpoint;
4. freeze a minimal Mobile Detector Contract sufficient for Guard integration;
5. hand the ONNX artifact/contract to Guard Mobile.

Android device performance validation is moved to the mobile integration work package, where actual device latency matters.

The following are deferred unless they become blockers:

- 640x640 detector challenger
- additional detector architectures
- broad hyperparameter search
- additional detector threshold sweeps
- FP16/INT8 optimization
- TFLite/LiteRT comparison if ONNX Runtime is usable
- extensive model/runtime benchmarking
- AVAX-specific field-domain adaptation
- extended detector error taxonomy

## Accelerated parallel sequence

The project should no longer run Phase 4 strictly serially.

Parallel work is approved:

```text
AI Detector finalization
    -> TEST + ONNX + contract

AI OCR MVP
    -> baseline OCR + mobile artifact/contract

Guard Mobile
    -> integrate detector as soon as ONNX/contract is available
    -> integrate OCR as soon as OCR artifact/contract is available
```

Guard Mobile does not need to wait for additional detector experimentation after the frozen detector artifact is available.

## Next critical work packages

### AI-WP-001 — Detector finalization

Status: `IN PROGRESS`  
Priority: `P0`

Fast-exit acceptance:

- TEST run completed once with frozen settings
- ONNX export loads and runs
- output consistency smoke test passes
- preprocessing/output contract documented
- artifact ready for Guard integration

### AI-WP-002 — OCR MVP Baseline & Mobile Export Contract

Status: `TODO`  
Priority: `P0`

Accelerated objective:

Build the simplest usable plate OCR baseline and export contract. The target is useful end-to-end plate recognition, not research-grade OCR optimization.

Deferred from OCR MVP unless required:

- large OCR model comparison
- extensive country-specific grammar engines
- sophisticated multi-frame OCR fusion
- exhaustive augmentation studies
- broad benchmark matrix

### MOB-AI-WP-001 — On-device Detector Integration

Status: `TODO / START AS SOON AS DETECTOR ONNX IS AVAILABLE`  
Priority: `P0`

Objective:

Connect the selected detector to the existing CameraX `FrameProcessor` boundary, measure actual device latency, and produce plate crops/bounding boxes for OCR.

Minimum success:

- model loads offline on target Android device
- frame preprocessing works
- detector returns visible plate boxes
- no raw frame upload/persistence
- app remains responsive
- inference cadence can be reduced if continuous full-frame inference is too slow

### MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline

Status: `TODO`  
Priority: `P0`

Objective:

Connect detector crop -> OCR -> PlateNormalizer -> Room lookup -> AccessChecker -> local result -> existing access-log flow.

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

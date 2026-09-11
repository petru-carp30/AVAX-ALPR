# AVAX ALPR Project Status

**Status snapshot:** 2026-09-11  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery is operating in **ACCELERATED MVP MODE**.

Core MVP flow:

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

### MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline

- Priority: `P0 Critical`
- Status: `IN PROGRESS / MASTER REVIEW — CHANGES REQUIRED`
- Target project: AVAX ALPR – Guard Mobile App

Reference Guard commit under review:

`4babeba79f995cd81632f1cc17f77be2025a76e6`

Commit message:

`feat(ocr): integrate OCR pipeline and native crop quality gate`

### Accepted implementation from current handoff

The following behavior is accepted as implemented/tested evidence:

- original-frame plate crop;
- native crop quality gate;
- `native crop height < 32 px` rejected as low quality;
- `32 <= native crop height < 64 px` resized to 128 px height preserving aspect ratio;
- native crop `>= 64 px` used without required resize;
- Google ML Kit Latin OCR;
- 2-of-3 normalized OCR confirmation;
- no authoritative automatic verification/log when OCR does not confirm;
- `AutomaticScanRearmGate` with approximately 1500 ms no-detection re-arm;
- existing 10-second same-normalized-plate cooldown preserved;
- lifecycle reset of scan/OCR confirmation state while preserving duplicate cooldown;
- manual verification fallback unchanged;
- no API, database, backend, or access-decision rule changes.

Physical evidence accepted from Pixel 6 Pro:

- sub-32 px crop correctly rejected without automatic event;
- 32–63 px crop correctly upscaled and confirmed;
- >=64 px crop correctly processed natively and confirmed;
- manual verification fallback PASS;
- background/minimize/screen-lock/resume PASS;
- duplicate automatic-log suppression PASS.

### Master review blockers before DONE

`MOB-AI-WP-002` is **not DONE yet** for three concrete reasons.

#### 1. ML Kit dependency scope must support non-debug builds

At reference commit `4babeba79f995cd81632f1cc17f77be2025a76e6`, `app/build.gradle.kts` declares:

```text
debugImplementation("com.google.mlkit:text-recognition:16.0.1")
```

The production `main` source set contains the ML Kit OCR implementation, so the OCR runtime dependency must not be debug-only.

Required correction:

```text
implementation("com.google.mlkit:text-recognition:16.0.1")
```

Then validate at minimum:

```text
./gradlew testDebugUnitTest
./gradlew assembleDebug
./gradlew assembleRelease
```

A release/pilot build must compile with OCR available.

#### 2. Required 10-presentation OCR field validation is still missing

Master previously required a controlled test using one clearly visible plate across **10 independent presentations** after the 2-of-3 confirmation and quality-gate changes.

Required report:

- OCR candidates per presentation;
- final confirmed result;
- correct / incorrect / unconfirmed;
- total correct confirmations;
- total incorrect confirmations;
- total unconfirmed.

Accelerated MVP target:

`>= 8 / 10 correct confirmed scans`

This is a field acceptance target, not an AI research benchmark.

If result is below 8/10, continue only the already approved narrow OCR stabilization work.

#### 3. Final offline-to-online access-log sync acceptance must be demonstrated

The final automatic workflow must be physically demonstrated as:

```text
Internet OFF
-> automatic confirmed plate
-> Room local lookup
-> AccessChecker local decision
-> exactly one local access event stored
-> Internet ON
-> pending event synchronizes through existing background sync
```

No new backend work is required; this is regression/acceptance validation of the existing path.

## Approved OCR safety architecture

```text
PlateDetection
-> original-frame native crop
-> native quality gate
-> optional resize
-> ML Kit OCR
-> normalize OCR candidate
-> 2-of-3 confirmation
-> PlateNormalizer
-> Room lookup
-> AccessChecker
-> access result
-> exactly one local access log
-> existing background sync
```

Safety rules:

- sub-32 px native crop cannot create authoritative automatic verification/logging;
- unconfirmed OCR cannot create automatic `UNKNOWN VEHICLE` or access log;
- OCR never decides access;
- no fuzzy lookup, grammar, character substitutions, custom OCR, or OCR retraining are authorized for MVP.

## CAM-WP-002 — Manual Camera Zoom Controls

- Priority: `P1 High`
- Status: `DONE`

Accepted reference Guard commit:

`b48300345f86212efaf4a94b2f42f7280a24818d`

Implemented/validated:

- CameraX `CameraControl.setZoomRatio(...)`;
- `CameraInfo.zoomState`;
- pinch-to-zoom;
- compact slider and current zoom indication;
- `1x` reset;
- dynamic min/max zoom handling;
- detector/OCR continue under zoom;
- bounding-box alignment remains correct;
- physical Pixel 6 Pro validation passed.

Observed tested maximum zoom: approximately `13.5x`.

## Completed AI/mobile foundation

| ID | Work item | Priority | Status |
|---|---|---:|---|
| AI-DATA-WP-001 | Detector Dataset Foundation | P0 | DONE |
| AI-WP-001 | Detector Baseline & Mobile Export Contract | P0 | DONE |
| MOB-AI-WP-001 | On-device Detector Integration | P0 | DONE |
| AI-WP-002 | OCR MVP Baseline & Mobile Contract | P0 | DONE |
| CAM-WP-001 | CameraX Foundation | P0 | DONE |
| CAM-WP-002 | Manual Camera Zoom Controls | P1 | DONE |
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | P0 | DONE |
| MOB-WP-002 | Local Access Logging Foundation | P0 | DONE |
| MOB-WP-003 | Background Access Log Upload | P0 | DONE |

## Production follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Status: `TODO`
- Priority: `P0 before production`
- Does not block completion of the on-device Guard MVP.

## MVP release gate

The accelerated Guard MVP is accepted only after physical validation of:

```text
Camera
-> detector
-> native crop quality gate
-> confirmed OCR
-> PlateNormalizer
-> Room lookup offline
-> AccessChecker local decision
-> clear result
-> exactly one local event
-> background sync after connectivity returns
```

Manual plate entry remains fallback for uncertain OCR or insufficient native crop quality.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.

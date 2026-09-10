# AVAX ALPR Project Status

**Status snapshot:** 2026-09-10  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery is operating in **ACCELERATED MVP MODE**.

Goal: finish the first usable Guard ALPR application as quickly as possible, accepting an approximately 80% solution if the core operational flow works reliably.

Deferred/non-essential work is preserved in `documentation/DEFERRED_SCOPE.md`.

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

`MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline`

- Priority: `P0 Critical`
- Status: `IN PROGRESS / OCR STABILITY FIX REQUIRED`
- Target project: AVAX ALPR – Guard Mobile App

Physical-device testing confirmed that single-frame OCR is not reliable enough to be treated as authoritative for automatic local verification/logging. A stable detector can produce different OCR strings for the same visible plate across presentations.

The previously implemented scene re-arm gate is accepted:

- one visible physical plate no longer creates repeated access events;
- automatic scanning re-arms after approximately 1500 ms continuously without plate detections;
- existing 10-second same-normalized-plate cooldown remains active.

## Master decision — minimal multi-frame OCR confirmation

A narrowly scoped exception to the previously deferred temporal OCR logic is **APPROVED** because physical testing demonstrated a real correctness blocker.

Approved MVP behavior:

```text
Plate detected
-> collect up to 3 usable OCR results over approximately 1.2–1.5 seconds
-> normalize each OCR candidate
-> accept only when the same normalized plate appears at least 2 times
-> perform exactly one local verification
-> create exactly one access log
-> lock the current scene
```

If 3 usable OCR results are obtained without 2-of-3 agreement:

```text
OCR uncertain
-> do not create automatic UNKNOWN VEHICLE
-> do not create an automatic access log
-> do not lock the scene as successfully verified
-> continue/retry or allow manual entry
```

Existing controls remain:

- re-arm after approximately 1500 ms continuously without detections;
- 10-second same-normalized-plate cooldown as secondary duplicate protection.

This exception is intentionally narrow. It does not authorize fuzzy database lookup, plate grammar, O/0-B/8-S/5 substitutions, object tracking, OCR retraining, custom OCR, perspective correction, API changes, or database changes.

## Master decision — native OCR crop quality gate

Physical stabilization diagnostics confirmed that native crop resolution is a material OCR-quality boundary.

Observed controlled results for plate `B208GAB`:

- native crop `79x40` -> OCR input `253x128` -> correct;
- native crop `94x50` -> OCR input `241x128` -> correct;
- native crop `142x74` -> no upscale -> correct;
- native crop `57x28` -> OCR input `261x128` -> incorrect;
- native crop around `47x25` -> detector still sees the plate but OCR is not reliably confirmable.

Current narrow preprocessing under test:

```text
native crop height < 64 px
-> upscale to 128 px height preserving aspect ratio
```

Master decision:

**APPROVED for Accelerated MVP**

Authoritative automatic OCR must now enforce this minimum quality gate before verification/logging:

```text
native crop height < 32 px
-> OCR input considered LOW QUALITY
-> do not perform authoritative automatic local verification
-> do not create automatic access log
-> continue scanning
-> prompt operator to move closer or use manual zoom
```

Important semantics:

- the threshold uses **native plate crop height before any upscale**;
- upscaling does not convert a sub-32 px native crop into a trusted crop;
- detector success alone is not sufficient to authorize OCR-based access verification;
- this quality gate applies before 2-of-3 OCR confirmation;
- sub-32 px native crops must not participate as authoritative confirmation candidates;
- manual plate entry remains available;
- manual camera zoom is the preferred operational aid for distant plates.

The approved safety sequence is now:

```text
PlateDetection
-> native crop
-> if native height < 32 px: LOW QUALITY / retry / zoom / manual fallback
-> otherwise apply current OCR resize rule where needed
-> ML Kit OCR
-> 2-of-3 normalized confirmation
-> local verification
-> exactly one access log
```

This decision does not authorize character substitutions, fuzzy lookup, country grammar, access-rule changes, custom OCR, OCR-engine changes, API changes, or database changes.

## OCR quality decision

The quality gate and 2-of-3 confirmation protect correctness, but physical validation must still measure whether the automatic flow is operationally useful.

For a clearly visible test plate under reasonable conditions, target at least 8 correct automatic confirmations across 10 independent presentations. This is an Accelerated MVP field target, not a research benchmark.

If correct confirmation remains below that target for native crops at or above the quality gate, continue only the already approved narrow OCR stabilization work. Broad OCR research remains deferred.

## CAM-WP-002 — Manual Camera Zoom Controls

- Priority: `P1 High`
- Status: `TODO / APPROVED FOR ACCELERATED MVP`
- Target project: AVAX ALPR – Guard Mobile App

Field testing on the Pixel 6 Pro showed that manual camera zoom is operationally useful when license plates are distant and occupy too few pixels in the frame.

Master decision:

- include manual camera zoom in the Accelerated MVP;
- implement it as a separate focused mobile feature;
- do not mix its logic with OCR stabilization;
- it may be implemented in parallel while `MOB-AI-WP-002` OCR stabilization continues;
- it must not block the current OCR correctness fix unless integration reveals a real regression.

Required MVP behavior:

- pinch-to-zoom directly on CameraX preview;
- provide a simple `1x` reset control;
- an optional `2x` quick control is allowed only if trivial and uncluttered;
- use CameraX `CameraControl` / `CameraInfo.zoomState`;
- respect the actual camera min/max zoom range;
- no detector/OCR image-cropping hack for zoom;
- no auto-zoom;
- detector and OCR continue consuming CameraX analysis frames under zoom;
- bounding-box mapping remains correct;
- autofocus/exposure continue functioning;
- camera remains the central uncluttered UI element.

No changes are authorized to Room, access decisions, access logging, backend/API, PlateNormalizer, OCR confirmation, scene re-arm, duplicate cooldown, detector model, or OCR engine.

Required physical validation on Pixel 6 Pro:

- pinch-to-zoom PASS;
- reset to `1x` PASS where supported by camera zoom range;
- stable preview PASS;
- detector continues detecting PASS;
- bounding-box mapping remains correct PASS;
- OCR continues receiving zoomed analysis frames PASS;
- no regression in automatic verification PASS;
- minimize/resume PASS;
- camera permission handling unchanged PASS.

## Completed AI/mobile foundation

| ID | Work item | Priority | Status |
|---|---|---:|---|
| AI-DATA-WP-001 | Detector Dataset Foundation | P0 | DONE |
| AI-WP-001 | Detector Baseline & Mobile Export Contract | P0 | DONE |
| MOB-AI-WP-001 | On-device Detector Integration | P0 | DONE |
| AI-WP-002 | OCR MVP Baseline & Mobile Contract | P0 | DONE |

Accepted detector:

- YOLOX-Nano 512
- confidence `0.225`
- NMS `0.45`
- ONNX Runtime Android

Accepted OCR baseline:

- Google ML Kit Text Recognition v2 Latin bundled model
- `com.google.mlkit:text-recognition:16.0.1`
- original validation baseline selective upscale: crop height < 32 px -> 96 px height
- current mobile narrow stabilization under test: native crop height < 64 px -> 128 px height
- authoritative automatic verification safety gate: native crop height < 32 px -> reject as low quality

## Existing completed application foundation

| ID | Work item | Status |
|---|---|---|
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | DONE |
| MOB-WP-002 | Local Access Logging Foundation | DONE |
| MOB-WP-003 | Background Access Log Upload | DONE |
| CAM-WP-001 | CameraX Foundation | DONE |

Manual plate verification remains the mandatory fallback while automatic OCR is uncertain or below the native crop quality gate.

## Production follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Status: `TODO`
- Priority: `P0 before production`
- Does not block completion of the on-device Guard MVP.

## MVP release gate

The accelerated MVP is functionally complete only after physical-device validation of:

```text
Camera
-> detects plate
-> native plate crop passes minimum quality gate
-> OCR candidate becomes sufficiently confirmed
-> PlateNormalizer normalizes it
-> Room lookup works offline
-> AccessChecker returns local decision
-> result is shown clearly
-> exactly one access event is stored for the confirmed scan
-> pending event can sync when connectivity returns
```

A sub-32 px native crop or unconfirmed OCR candidate must not create an authoritative automatic UNKNOWN VEHICLE decision/log.

Manual zoom is an approved usability aid for distant plates but does not change access-decision semantics.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.
- Minimum physical-device validation of the automatic end-to-end ALPR flow remains mandatory before calling the accelerated MVP complete.

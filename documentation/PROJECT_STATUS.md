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

## OCR quality decision

The 2-of-3 confirmation gate is required first because it prevents one unstable OCR frame from becoming an authoritative false decision/log.

However, confirmation alone is not assumed to solve OCR quality. After implementing the confirmation gate, physical testing must measure whether the automatic flow is actually usable.

For a clearly visible test plate under reasonable conditions, target at least 8 correct automatic confirmations across 10 independent presentations. This is an Accelerated MVP field target, not a research benchmark.

If the confirmation gate still rarely reaches the correct plate, the next action is a **narrow OCR stabilization patch**, not broad OCR research. The patch may investigate only low-complexity input-quality improvements such as crop selection/padding, crop resolution/upscaling, and simple exposure/contrast handling. Changing OCR engine, custom training, fuzzy lookup, grammar rules, and broad preprocessing experiments remain deferred unless that narrow patch also fails.

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
- selective upscale: crop height < 32 px -> 96 px height

## Existing completed application foundation

| ID | Work item | Status |
|---|---|---|
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | DONE |
| MOB-WP-002 | Local Access Logging Foundation | DONE |
| MOB-WP-003 | Background Access Log Upload | DONE |
| CAM-WP-001 | CameraX Foundation | DONE |

Manual plate verification remains the mandatory fallback while automatic OCR is uncertain.

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
-> OCR candidate becomes sufficiently confirmed
-> PlateNormalizer normalizes it
-> Room lookup works offline
-> AccessChecker returns local decision
-> result is shown clearly
-> exactly one access event is stored for the confirmed scan
-> pending event can sync when connectivity returns
```

An unconfirmed OCR candidate must not create an authoritative automatic UNKNOWN VEHICLE decision/log.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.
- Minimum physical-device validation of the automatic end-to-end ALPR flow remains mandatory before calling the accelerated MVP complete.

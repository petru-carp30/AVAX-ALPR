# AVAX ALPR Project Status

**Status snapshot:** 2026-09-14  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery is operating in **ACCELERATED MVP MODE**.

Core Guard flow:

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

The core automatic Guard ALPR pipeline is now physically validated end to end.

Remaining immediate project step:

```text
CAM-WP-003 formal Master handoff/review
-> Accelerated MVP acceptance / bug-fix only
```

## MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline

- Priority: `P0 Critical`
- Status: `DONE`
- Target project: AVAX ALPR – Guard Mobile App

Master accepted the final physical-device handoff on 2026-09-14.

Final Guard reference commit:

`984d99a9cd3ceb0bd04d41ea8063db4de7954a71`

Commit message:

`fix(ai): improve OCR pipeline runtime performance`

The commit is confirmed as the current `master` head of `petru-carp30/Avax.ALPR.Guard`.

### Final accepted automatic flow

```text
CameraX
-> YOLOX detector
-> original-frame native plate crop
-> native crop quality gate
-> optional OCR resize
-> Google ML Kit OCR
-> normalized OCR candidate
-> 2-of-3 confirmation
-> PlateNormalizer
-> Room local lookup
-> AccessChecker
-> access result
-> durable local access log
-> WorkManager background synchronization
```

Manual plate verification remains available as fallback.

### OCR runtime dependency

Accepted dependency:

`com.google.mlkit:text-recognition:16.0.1`

The dependency is now declared with normal `implementation(...)` scope and is available to non-debug builds.

Final build validation:

- `testDebugUnitTest` — BUILD SUCCESSFUL
- `assembleDebug` — BUILD SUCCESSFUL
- `assembleRelease` — BUILD SUCCESSFUL

### OCR quality and confirmation safety

Native crop rules:

```text
native height < 32 px
-> reject as LOW QUALITY
-> no authoritative automatic verification/logging

32 <= native height < 64 px
-> resize to 128 px height preserving aspect ratio
-> OCR allowed

native height >= 64 px
-> use native crop
-> OCR allowed
```

Automatic OCR must be confirmed using the accepted 2-of-3 rule before local verification/logging.

If no 2-of-3 agreement is reached:

- no authoritative automatic verification;
- no automatic `UNKNOWN VEHICLE` event;
- no automatic access log;
- scanner remains able to retry.

No character substitutions, plate grammar, fuzzy lookup, custom OCR, perspective correction, long-window voting, or object tracking were introduced.

### 10-presentation physical OCR acceptance

Target device:

`Google Pixel 6 Pro`

Physical plate:

`B173AVX`

Result across 10 independent presentations:

- correct confirmations: `10 / 10`
- incorrect confirmations: `0 / 10`
- unconfirmed: `0 / 10`

Accelerated MVP acceptance target was `>= 8 / 10` correct confirmed scans.

Result: `PASS`

The test also demonstrated correct 2-of-3 recovery from occasional bad single-frame OCR candidates without grammar/substitution rules.

### Runtime/performance acceptance

Representative observed performance after runtime improvements:

- detector total processing as low as approximately `256 ms`;
- detector cadence up to approximately `3.9 fps`;
- OCR latency approximately `103–280 ms` in fast cases.

Regression tests passed for:

- same vehicle continuously visible for more than 10–15 seconds without duplicate logs;
- vehicle leaves -> scanner re-arms -> next vehicle processes normally;
- minimize/resume without stale scene state;
- 2-of-3 confirmation;
- approximately 1500 ms scene re-arm;
- existing 10-second same-normalized-plate cooldown;
- native crop quality gate;
- crop/upscale behavior.

Further performance optimization is not required for the current Accelerated MVP scope.

### Offline-first end-to-end acceptance

Physical acceptance scenario:

```text
Internet OFF
-> Camera
-> Detector
-> native crop quality gate
-> OCR
-> 2-of-3 confirmation
-> PlateNormalizer
-> Room local lookup
-> AccessChecker
-> GRANTED local decision
-> local access event PENDING

Internet ON
-> existing WorkManager sync
-> local event SYNCED
-> exactly one server row for mobileEventId
```

Accepted result:

- local access decision without server dependency: `PASS`
- local acceptance event count: `1`
- initial state: `PENDING`
- background synchronization: `PASS`
- final local state: `SYNCED`
- central row count for the event UUID: `1`
- duplicate count: `0`

This validates the intended offline-first Guard architecture through the existing backend ingestion path.

### Architecture impact

No API contract changes.

No database schema changes.

No backend contract changes.

No access-rule changes.

OCR only determines whether recognized text is reliable enough to submit to the existing application logic.

Access authority remains:

```text
confirmed plate
-> PlateNormalizer
-> Room
-> AccessChecker
-> access result
```

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

## CAM-WP-003 — Focus Assist After Zoom

- Priority: `P1 High`
- Status: `IN PROGRESS / IMPLEMENTED — FORMAL HANDOFF PENDING`
- Target project: AVAX ALPR – Guard Mobile App

Implementation is present in Guard commit:

`5fb84a887e9ed1747378d3cb2fd78e6da1a0ca84`

Commit message:

`feat(camera): add focus assist after zoom`

Implemented code includes:

- CameraX `FocusMeteringAction`;
- center refocus after zoom settles;
- tap-to-focus using `PreviewView.meteringPointFactory`;
- AF metering with AE where supported;
- 3-second auto-cancel;
- no detector/OCR/access-logic coupling.

The 10-presentation OCR acceptance was performed after the autofocus/runtime fixes, providing positive integration evidence. CAM-WP-003 still requires its own formal Master handoff before being marked `DONE`.

## Completed AI/mobile foundation

| ID | Work item | Priority | Status |
|---|---|---:|---|
| AI-DATA-WP-001 | Detector Dataset Foundation | P0 | DONE |
| AI-WP-001 | Detector Baseline & Mobile Export Contract | P0 | DONE |
| MOB-AI-WP-001 | On-device Detector Integration | P0 | DONE |
| AI-WP-002 | OCR MVP Baseline & Mobile Contract | P0 | DONE |
| MOB-AI-WP-002 | OCR + Automatic Local Verification Pipeline | P0 | DONE |
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

## Accelerated MVP acceptance state

The core automatic offline-first Guard ALPR flow has now passed physical end-to-end acceptance on the target device.

Before declaring the overall Accelerated MVP milestone fully closed, finish the formal CAM-WP-003 review and then operate in bug-fix-only mode unless a measured blocker justifies reopening scope.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.

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

The core automatic Guard ALPR pipeline and camera usability foundation are now physically validated on the target device.

Next immediate work package:

`MVP-E2E-WP-001 — Accelerated MVP End-to-End Acceptance & Blocker Fixes`

Mode:

`FINAL ACCEPTANCE / BUG FIX ONLY`

No new feature scope should be introduced unless a measured blocker requires Master approval.

## MOB-AI-WP-002 — OCR + Automatic Local Verification Pipeline

- Priority: `P0 Critical`
- Status: `DONE`
- Target project: AVAX ALPR – Guard Mobile App

Master accepted the final physical-device handoff on 2026-09-14.

Final Guard reference commit:

`984d99a9cd3ceb0bd04d41ea8063db4de7954a71`

Commit message:

`fix(ai): improve OCR pipeline runtime performance`

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

The dependency is declared with normal `implementation(...)` scope and is available to non-debug builds.

Final build validation:

- `testDebugUnitTest` — BUILD SUCCESSFUL
- `assembleDebug` — BUILD SUCCESSFUL
- `assembleRelease` — BUILD SUCCESSFUL

### OCR quality and confirmation safety

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

Target device: `Google Pixel 6 Pro`

Physical plate: `B173AVX`

Result:

- correct confirmations: `10 / 10`
- incorrect confirmations: `0 / 10`
- unconfirmed: `0 / 10`

Accelerated MVP acceptance target was `>= 8 / 10` correct confirmed scans.

Result: `PASS`

### Runtime/performance acceptance

Representative observed performance after runtime improvements:

- detector total processing as low as approximately `256 ms`;
- detector cadence up to approximately `3.9 fps`;
- OCR latency approximately `103–280 ms` in fast cases.

Regression tests passed for:

- same vehicle continuously visible without duplicate logs;
- vehicle leaves -> scanner re-arms -> next vehicle processes normally;
- minimize/resume without stale scene state;
- 2-of-3 confirmation;
- approximately 1500 ms scene re-arm;
- existing 10-second same-normalized-plate cooldown;
- native crop quality gate;
- crop/upscale behavior.

Further performance optimization is not required for the current Accelerated MVP scope.

### Offline-first end-to-end acceptance

Validated physical scenario:

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

No API contract, database schema, backend contract, or access-rule changes were introduced by MOB-AI-WP-002.

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
- Status: `DONE`
- Target project: AVAX ALPR – Guard Mobile App

Master accepted the formal handoff on 2026-09-14.

Reference implementation commit:

`5fb84a887e9ed1747378d3cb2fd78e6da1a0ca84`

Commit message:

`feat(camera): add focus assist after zoom`

The implementation remains present in the later Guard baseline headed by:

`984d99a9cd3ceb0bd04d41ea8063db4de7954a71`

Accepted implementation:

- CameraX `FocusMeteringAction` integration;
- center refocus after zoom interaction settles;
- tap-to-focus using `PreviewView.meteringPointFactory`;
- AF metering;
- AE metering when supported by the device;
- 3-second auto-cancel so focus is not permanently locked;
- normal CameraX continuous autofocus behavior preserved;
- Preview and ImageAnalysis remain bound;
- no detector/OCR/access-logic coupling.

Physical validation on Google Pixel 6 Pro confirmed:

- zoom remains functional;
- detector continues processing during/after focus operations;
- bounding boxes remain mapped correctly;
- OCR continues receiving native plate crops;
- no preview freeze caused by focus assist;
- `1x` reset remains functional;
- minimize/resume remains stable;
- later automatic OCR acceptance/runtime validation passed with focus assist already integrated;
- final automatic OCR field validation achieved `10/10` correct independent plate confirmations.

Build evidence:

- `testDebugUnitTest` — BUILD SUCCESSFUL
- `assembleDebug` — BUILD SUCCESSFUL
- later final Guard baseline `assembleRelease` — BUILD SUCCESSFUL

Accepted limitation:

At extreme zoom, visible degradation can be caused by optical/digital zoom limits and cannot be fully corrected by autofocus. This is not considered an MVP blocker.

No custom Camera2 focus algorithm, manual focus-distance control, permanent focus lock, auto-zoom, or object tracking was introduced.

API contract changes: `NONE`

Database changes: `NONE`

Backend changes: `NONE`

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
| CAM-WP-003 | Focus Assist After Zoom | P1 | DONE |
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | P0 | DONE |
| MOB-WP-002 | Local Access Logging Foundation | P0 | DONE |
| MOB-WP-003 | Background Access Log Upload | P0 | DONE |

## Production follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Status: `TODO`
- Priority: `P0 before production`
- Does not block completion of the on-device Guard MVP.

## Accelerated MVP acceptance state

All currently identified implementation work packages required for the Guard automatic MVP pipeline are Master-accepted as `DONE`.

The next step is `MVP-E2E-WP-001`, a final acceptance/regression pass in **BUG FIX ONLY** mode. Its purpose is to verify the assembled MVP as a whole, not to introduce new functionality.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.

# AVAX ALPR Project Status

**Status snapshot:** 2026-09-15  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery has completed the **ACCELERATED MVP** acceptance cycle for the Guard Mobile application.

The core Guard flow is now Master-accepted on the physical target device:

```text
CameraX
-> Manual Zoom / Focus Assist
-> YOLOX Plate Detector
-> Original-frame Plate Crop
-> Native Crop Quality Gate
-> Google ML Kit OCR
-> 2-of-3 OCR Confirmation
-> PlateNormalizer
-> Room Local Lookup
-> AccessChecker
-> Access Result
-> Local Access Log
-> WorkManager Background Sync
-> ASP.NET Core API
-> SQL Server
```

The system remains offline-first. AI does not decide access.

## MVP-E2E-WP-001 — Accelerated MVP End-to-End Acceptance & Blocker Fixes

- Priority: `P0 Critical`
- Status: `DONE`
- Target project: AVAX ALPR – Guard Mobile App
- Master acceptance date: `2026-09-15`

Final Guard baseline:

`984d99a9cd3ceb0bd04d41ea8063db4de7954a71`

Commit message:

`fix(ai): improve OCR pipeline runtime performance`

The repository was reported clean and synchronized with `origin/master`; no additional code changes were required during final acceptance.

### Physical acceptance target

- Device: `Google Pixel 6 Pro`
- Android: `17`
- API: `37`

### Build acceptance

All final build gates passed:

- `testDebugUnitTest` — BUILD SUCCESSFUL
- `assembleDebug` — BUILD SUCCESSFUL
- `assembleRelease` — BUILD SUCCESSFUL

### Final physical acceptance scenarios

All required A–N scenarios passed:

- clean application start;
- known vehicle / Granted;
- known vehicle / Denied;
- genuine unknown vehicle after confirmed OCR;
- low-quality distant crop rejection;
- manual CameraX zoom;
- focus assist after zoom;
- same vehicle continuously visible without repeated logs;
- vehicle leaves / next vehicle re-arm flow;
- minimize / resume recovery;
- offline access decision;
- connectivity recovery and background synchronization;
- offline application restart;
- manual plate-entry fallback.

No unresolved P0/P1 blocker was identified during the final acceptance pass.

### Accepted OCR/access behavior

Examples from final physical validation:

- `B173AVX` -> confirmed OCR -> Room -> `GRANTED`;
- `B30KRP` -> OCR candidates `B30KRA / B30KRP / B30KRP` -> confirmed `B30KRP` -> `DENIED`;
- `SV5660C` -> confirmed 2-of-3 -> local lookup miss -> `UNKNOWN VEHICLE`;
- native crop approximately `35x17 px` -> `Plate too far - move closer or zoom` -> no authoritative automatic decision/log.

The 2-of-3 gate therefore demonstrated both positive confirmation and rejection/recovery from individual bad OCR candidates without character substitution or grammar rules.

### Duplicate protection acceptance

A confirmed plate remained continuously visible for more than one minute without repeated automatic access-log spam.

Accepted protections remain:

- scene lock after successful automatic processing;
- re-arm after approximately 1500 ms continuously without plate detections;
- existing 10-second same-normalized-plate cooldown as secondary protection.

### Zoom/focus acceptance

Manual zoom increased native plate crop size materially during field testing, for example from approximately `35x17` at 1x to approximately `202x96` at 5.6x.

Focus assist remained stable and did not freeze Preview, detector, or OCR processing.

A small visual delay at the start of slider movement is accepted as non-blocking MVP behavior.

### Offline-first acceptance

Final physical acceptance confirmed that access decisions continue without backend connectivity and after application restart.

Observed flow:

```text
Backend unavailable
-> Camera / Manual Input
-> OCR / PlateNormalizer
-> Room
-> AccessChecker
-> local result
-> local access event PENDING
```

After connectivity/backend recovery:

```text
PENDING
-> WorkManager upload
-> SYNCED
```

Final Room inspection reported:

- `183` synchronized access-log rows;
- duplicate query by `localLogId` returned `0 rows`.

A separate server-side count for a specific `mobileEventId` was not rerun during this final acceptance round. This is accepted as non-blocking because the same Guard/backend baseline had already passed explicit server-side idempotency validation during the preceding MOB-AI-WP-002 acceptance, including one central row for the tested event UUID.

### Data and architecture safety

Accepted boundaries remain:

- raw camera frames are not part of normal persistence;
- raw camera frames are not uploaded;
- OCR crops are not persisted as normal application data;
- Guard Mobile does not connect directly to SQL Server;
- AI does not decide access;
- `AccessChecker` remains the local access authority;
- Backend API remains the server-side synchronization boundary.

## Accelerated Guard MVP milestone

**Status: ACCEPTED / CLOSED**

The first automatic offline-first AVAX ALPR Guard MVP is now validated end to end on the primary physical target device.

The Guard application now enters **BUG FIX ONLY** mode for the accepted MVP baseline unless Master explicitly opens a new feature work package.

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
| MVP-E2E-WP-001 | Accelerated MVP End-to-End Acceptance & Blocker Fixes | P0 | DONE |

## Next project priority

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Status: `TODO`
- Priority: `P0 before production`
- Target project: AVAX ALPR – Backend & Database

This is the next production-readiness blocker after Guard MVP acceptance.

After BE-WP-004, continue with production/security/deployment preparation and then the deferred Manager / Access Request flow according to Master priority.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.
- New Guard features require a new Master-approved work package; the accepted MVP baseline is now bug-fix only.

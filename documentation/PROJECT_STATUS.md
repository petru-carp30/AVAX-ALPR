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

## BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment

- Priority: `P0 if central production access-log persistence is required`
- Status: `BLOCKED / DEFERRED TO HOST INTEGRATION`
- Target project: AVAX ALPR – Backend & Database

Reference backend commit:

`fee402af683f8c740663dd0cf33075965b092f30`

Implementation prepared and locally validated:

- SQL Server persistence support for `POST /api/access-logs`;
- `Microsoft.Data.SqlClient` integration;
- `SqlServerAccessLogConnectionFactory`;
- environment-based `ConnectionStrings:AccessLogSqlServer` configuration;
- deployment script `ALPR/Database/BE-WP-004_Create_AVAX_ALPR_ACCESS_LOGS.sql`;
- database-level duplicate handling for SQL Server duplicate-key errors;
- existing `201 Stored` / `200 AlreadyStored` / `409 Conflict` API semantics preserved;
- automated backend suite: `43/43` passed;
- package vulnerability scan: no vulnerable packages reported.

Not completed:

- deployment against the manager-controlled central SQL Server;
- write/DDL validation in the target environment;
- central smoke test proving first insert / identical retry / conflicting duplicate against the real SQL Server table.

Reason:

The current developer access to the central database is read-only. No unauthorized DDL or test writes are permitted.

Master decision:

- do not treat this as forgotten unfinished work;
- preserve the implementation and deployment script;
- resume the controlled SQL Server deployment only when the ALPR feature is integrated into the larger host application and the approved backend/service account, connection string, and SQL permissions are available;
- if the host application does not require central ALPR access-log storage, that requirement must be explicitly re-evaluated during integration rather than silently deploying an unused table.

BE-WP-004 is **not DONE** because the required target-environment validation has not occurred.

## Host application integration ownership

The future integration of the accepted AVAX ALPR Guard capability into the larger Android application is **outside the current implementation scope of this project**.

Integration execution is intentionally handed off to the user's manager / owner of the larger host application.

Therefore:

- `MOB-INT-WP-001 — Extract Guard ALPR as Host-Integrable Android Feature` is **not opened as an active project task**;
- no integration refactor should be performed now;
- no module extraction, package moves, host navigation changes, host DI changes, or host database changes should be made in the standalone Guard repository for speculative integration;
- the accepted standalone Guard baseline must remain available as the reference implementation;
- `documentation/GUARD_CODEBASE_MAP.md` and `documentation/GUARD_INTEGRATION_GUIDE.md` are handoff/reference material for the manager who performs the integration later;
- `BE-WP-004` should be revisited by that integration owner only if the final host architecture still requires the standalone central ALPR log persistence path.

## Current project direction

There is currently **no mandatory active implementation work package** after the accepted Guard MVP within the user's current execution scope.

The project remains in:

```text
Guard MVP: ACCEPTED / CLOSED
Guard baseline: BUG FIX ONLY
Host integration: HANDED OFF TO MANAGER
BE-WP-004 target deployment: DEFERRED TO HOST INTEGRATION IF STILL REQUIRED
```

New development should begin only for:

- a real bug discovered in the accepted Guard baseline;
- a newly approved requirement;
- a measured field issue that justifies reopening deferred AI/mobile work;
- a future request from the host-integration owner.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Deferred does not mean cancelled.
- Blocked work must preserve its blocker explicitly rather than being treated as complete.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.
- New Guard features require a new Master-approved work package; the accepted MVP baseline is now bug-fix only.

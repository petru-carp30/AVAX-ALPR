# AVAX ALPR Project Status

**Status snapshot:** 2026-09-16  
**Source of truth:** AVAX ALPR Master Plan & Current Status

## Delivery mode

Project delivery has completed the **ACCELERATED MVP** acceptance cycle for the Guard Mobile application.

The core Guard flow is Master-accepted on the physical target device:

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

Final accepted standalone Guard baseline:

`984d99a9cd3ceb0bd04d41ea8063db4de7954a71`

Commit message:

`fix(ai): improve OCR pipeline runtime performance`

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

### Duplicate protection acceptance

Accepted protections remain:

- scene lock after successful automatic processing;
- re-arm after approximately 1500 ms continuously without plate detections;
- existing 10-second same-normalized-plate cooldown as secondary protection.

### Zoom/focus acceptance

Manual zoom materially increased native plate crop size during field testing. Focus assist remained stable and did not freeze Preview, detector, or OCR processing.

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

A separate server-side count for a specific `mobileEventId` was not rerun during this final acceptance round. This is accepted as non-blocking because the same Guard/backend baseline had already passed explicit server-side idempotency validation during the preceding MOB-AI-WP-002 acceptance.

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

The first automatic offline-first AVAX ALPR Guard MVP is validated end to end on the primary physical target device.

The accepted `master` baseline remains the safe reference. New presentation work is isolated on a dedicated feature branch until explicitly merged.

## MOB-UX-WP-001 — Camera-First Guard Operator UI

- Priority: `P1 High`
- Status: `DONE`
- Target project: AVAX ALPR – Guard Mobile App
- Master acceptance date: `2026-09-16`
- Guard branch: `feature/guard-operator-ui`
- Reference commit: `4c8b2a4a2dcd7d158c1f6773c3cc652a775a9eb9`
- Commit message: `feat(ui): add camera-first guard operator view`
- Merge status: `NOT MERGED`

Master review confirmed that the feature branch is exactly one commit ahead of the accepted `master` baseline `984d99a9cd3ceb0bd04d41ea8063db4de7954a71` and modifies only the expected Android UI/camera composition files.

### Accepted operator UX

Implemented and physically validated:

- camera-first Operator View;
- compact access-area selector over the camera;
- debug-only `DEV` entry preserving the existing Developer View;
- shared editable license-plate field for OCR and manual correction;
- OCR does not overwrite the plate field while the operator is actively editing;
- manual Search reuses the existing local verification path;
- `Edit Plate` and `Continue` workflows;
- operator result overlay for Granted / Denied / Unknown outcomes;
- vehicle brand/model, color and notes shown where available;
- unknown vehicles do not fabricate vehicle information;
- current access-status color semantics preserved;
- fullscreen camera permission fallback;
- existing detector overlay, pinch zoom, 1x reset, tap-to-focus and focus assist preserved;
- compact vertical zoom control for fullscreen operator use;
- access-area change preserves the current plate candidate for explicit re-check without creating an unintended automatic access event.

### Physical validation

Device:

`Google Pixel 6 Pro`

Confirmed:

- fullscreen camera remains operational;
- vertical and pinch zoom work;
- focus behavior remains functional;
- Granted / Denied / Unknown presentation works;
- access-area selector works;
- Developer View remains accessible in debug workflow;
- background/resume works;
- OCR -> Edit Plate -> manual correction -> Search works;
- OCR does not overwrite active manual input;
- same continuously visible vehicle remains protected from duplicate automatic events;
- removing the plate rearms scanning;
- reintroducing the plate allows a new automatic verification;
- area change does not create an unintended automatic event.

Observed manual-edit isolation example:

```text
Automatic: B30KRP -> DENIED
Manual edit while plate remains visible: B30KRR -> UNKNOWN VEHICLE
Plate removed and presented again: B30KRP -> DENIED
```

### Build validation

- `testDebugUnitTest` — PASS
- `assembleDebug` — PASS
- `assembleRelease` — PASS
- `git diff --check` — PASS

### Architecture impact

No new architecture layer was introduced. Operator View is built on the existing `GuardViewModel` and repository/domain verification pipeline.

No changes were made to:

- detector model or thresholds;
- OCR engine or algorithm;
- crop quality gate;
- 2-of-3 confirmation;
- `PlateNormalizer`;
- Room lookup rules;
- `AccessChecker`;
- scene re-arm or cooldown semantics;
- access logging;
- WorkManager synchronization;
- backend APIs;
- Room schema/migrations.

Driver metadata/photos remain intentionally out of scope and are candidates for separate follow-up work.

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
| MOB-UX-WP-001 | Camera-First Guard Operator UI | P1 | DONE |

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

- preserve the implementation and deployment script;
- resume controlled SQL Server deployment only during host integration if that architecture still requires central ALPR log storage;
- do not treat the target deployment as complete until target-environment validation exists.

## Host application integration ownership

The future integration of the accepted AVAX ALPR Guard capability into the larger Android application is outside the current implementation scope and is intentionally handed off to the user's manager / owner of the larger host application.

Therefore:

- `MOB-INT-WP-001` is not opened as an active project task;
- no speculative integration refactor should be performed now;
- the standalone Guard implementation remains the reference;
- `documentation/GUARD_CODEBASE_MAP.md` and `documentation/GUARD_INTEGRATION_GUIDE.md` remain integration handoff/reference material.

## Current project direction

The active presentation branch is now:

```text
Guard master baseline: ACCEPTED REFERENCE
Operator UI branch: feature/guard-operator-ui
MOB-UX-WP-001: DONE / NOT MERGED
Host integration: HANDED OFF TO MANAGER
BE-WP-004 target deployment: DEFERRED TO HOST INTEGRATION IF STILL REQUIRED
```

Potential next work, if approved, is driver display metadata and opportunistic driver-photo loading for the operator result overlay. This must remain offline-first for authoritative data; photo loading must never block access verification or result display.

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Feature-branch acceptance does not imply merge approval.
- Deferred does not mean cancelled.
- Blocked work must preserve its blocker explicitly rather than being treated as complete.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin server-side communication passes through Backend API.
- Any driver-photo enhancement must remain non-authoritative and asynchronous.

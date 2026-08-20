# AVAX ALPR Project Status

**Status snapshot:** 2026-08-20  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, and changelog information is maintained in the dedicated documentation files.

## Current milestone

`CAM-WP-001 — CameraX Foundation` is confirmed `DONE`.

Validated camera foundation:

```text
Camera
  -> CameraX Preview
  -> ImageAnalysis
  -> CameraFrameAnalyzer
  -> FrameProcessor
  -> future Plate Detector
```

Camera operation is on-device and does not require backend connectivity. No plate detector, OCR, AI inference, camera image upload, or automatic camera-driven access decision was introduced in CAM-WP-001.

## Next work package

`AI-WP-001 — License Plate Detector Baseline & Mobile Export Contract`

- Priority: `P0 Critical`
- Status: `TODO`
- Target project: AVAX ALPR – AI Model
- Dependency: `CAM-WP-001 — DONE`
- Objective: establish the first evaluated license-plate detector baseline and a stable mobile-consumable export contract that can later attach to the confirmed `FrameProcessor` boundary.

This starts Phase 4 of the roadmap: detector, OCR, and on-device AI integration.

The detector must only locate license plates. It must not decide vehicle access.

## Production persistence follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Priority: `P0 before production`
- Status: `TODO`
- Target project: AVAX ALPR – Backend & Database
- Dependency: approved `dbo.AVAX_ALPR_ACCESS_LOGS` design and `BE-WP-003 — DONE`

The production access-log table and SQL Server runtime persistence adapter are not yet deployed. This does not block AI/mobile development, but it blocks production central access-log storage.

## Confirmed completed work

| ID | Work item | Priority | Status |
|---|---|---:|---|
| BE-001 | Backend Baseline Audit & Build Validation | P0 | DONE |
| SEC-001 | Resolve NU1903 Microsoft.OpenApi Vulnerability | P0 | DONE |
| BE-002 | Validate Existing SQL Schema Relevant to ALPR | P0 | DONE |
| DEVDB-001 | Local SQLite Development Database Baseline | P1 | DONE |
| BE-WP-001 | Local Backend Vehicle Read API Foundation | P0 | DONE |
| BE-003 through BE-009 | BE-WP-001 implementation subtasks | P0 | DONE |
| BE-WP-002 | Vehicle Snapshot Sync API v1 | P0 | DONE |
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | P0 | DONE |
| MOB-WP-002 | Local Access Logging Foundation | P0 | DONE |
| BE-WP-003 | Access Log Ingestion API v1 | P0 | DONE |
| MOB-WP-003 | Background Access Log Upload | P0 | DONE |
| CAM-WP-001 | CameraX Foundation | P0 | DONE |

## Backend status

Confirmed endpoints:

- `GET /api/vehicles`
- `GET /api/vehicles/by-plate/{plate}`
- `GET /api/sync/vehicles`
- `POST /api/access-logs`

Access Log API v1 remains idempotent on the mobile-generated UUID:

- new event -> `201 Created`, `Stored`
- identical retry -> `200 OK`, `AlreadyStored`
- same UUID with different logical event data -> `409 Conflict`

Backend reference commit for Access Log API v1:

`88005bf7cd231fc79708f767552499e29fc8da9f`

## Guard Mobile status

Confirmed offline-first capabilities include:

- Vehicle Snapshot Sync v1 client
- transactional Room vehicle cache
- local plate normalization
- local ParkingLot / Site / Camp access verification
- local access logging
- background access-log synchronization with WorkManager
- application-start synchronization recovery
- Android 17 Local Network Protection support
- CameraX rear-camera preview
- CameraX ImageAnalysis
- lifecycle-safe camera binding
- runtime Camera permission handling
- bounded frame analysis using `STRATEGY_KEEP_ONLY_LATEST`
- clean `FrameProcessor` boundary for future AI integration

### CameraX foundation

Confirmed CameraX version:

`1.6.1`

Confirmed CameraX components:

- `ProcessCameraProvider`
- `Preview`
- `PreviewView`
- `ImageAnalysis`
- `CameraSelector.DEFAULT_BACK_CAMERA`

The preview is integrated with Compose through `AndroidView` and isolated through a dedicated `CameraSession`.

Frame-analysis flow:

```text
ImageAnalysis
  -> CameraFrameAnalyzer
  -> FrameProcessor
```

Current processor:

`DevelopmentFrameProcessor`

It provides development frame diagnostics only and performs no AI inference.

Confirmed analysis properties:

- `ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST`
- one dedicated single-thread analysis executor per camera session
- no per-frame executor creation
- no uncontrolled coroutine creation per frame
- no required full-frame Bitmap conversion
- frame width/height/rotation/timestamp/format metadata exposed
- `ImageProxy.close()` guaranteed through cleanup/finally handling
- raw camera frames are not retained after processing
- raw camera frames are not persisted or uploaded

### CAM-WP-001 validation

Confirmed automated validation:

- unit tests: `64/64 PASS`
- instrumentation tests: `21/21 PASS`
- total: `85/85 PASS`
- Android build: PASS

Confirmed physical-device validation:

- Camera permission grant flow -> PASS
- rear-camera preview -> PASS
- portrait preview -> PASS
- continuous ImageAnalysis frame delivery -> PASS
- background/foreground and lock/unlock lifecycle recovery -> PASS
- offline camera operation -> PASS
- manual local verification while offline -> PASS
- permission-denied/manual fallback -> PASS

Observed development diagnostics included continuous frame counts above 1900 frames, `640x480` analysis resolution, and `90°` rotation on the validated portrait device configuration.

The application is currently effectively single-screen, so a separate in-app navigation screen-switch camera test was not applicable. Equivalent lifecycle leave/return behavior was validated through background/foreground, lock/unlock, and application re-entry.

CameraX implementation reference commit:

`35172b5695577866dc2129895b525f8e4386f267`

## Access-log synchronization status

Confirmed mobile synchronization states:

- `Pending`
- `Synced`
- `Conflict`
- `Rejected`

WorkManager uses a connected-network constraint, exponential backoff, unique queue-draining work, oldest-first Pending processing, application-start recovery, and original UUID reuse for idempotent retry.

MOB-WP-003 reference commit:

`ad0788a`

## Approved production access-log design

Master-approved table:

`dbo.AVAX_ALPR_ACCESS_LOGS`

The approved design includes database-enforced uniqueness on `mobileEventId`, explicit area/decision CHECK constraints, server-controlled `receivedAtUtc`, and no foreign key to `AVAX_VEHICLES` at this stage.

The table has not yet been deployed by confirmed production implementation work.

## Open technical debt and follow-up

Existing production technical debt remains open:

- `TD-001` — `AVAX_VEHICLES` lacks enforced primary identity
- `TD-002` — license plate uniqueness is not enforced
- `TD-003` — no incremental synchronization marker
- `TD-004` — no vehicle lookup index in production

Additional follow-up:

- production access-log table deployment pending
- production SQL Server access-log persistence adapter/configuration pending
- no access-log retention policy defined
- automatic access-log deletion not implemented
- controlled physical `409 Conflict` mobile validation not performed; automated coverage exists
- one pre-existing non-CameraX Kotlin compiler warning remains in `GuardDatabaseMigrations.kt`; it did not block CAM-WP-001

## Architecture decision status

`ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1`

Status: `PROPOSED`

`ARCH-001 — Vehicle Identity & Incremental Sync Strategy` remains `TODO / P1` before a future incremental Sync v2.

The dedicated vehicle access-log storage and UUID-based idempotency semantics were explicitly accepted by Master for Access Log Contract v1 and should remain represented as an accepted architecture decision in the ADR set.

The current camera-to-AI integration boundary is implementation evidence only. No detector/OCR model architecture has yet been accepted.

## Explicitly not confirmed as implemented

- production SQL Server `dbo.AVAX_ALPR_ACCESS_LOGS` deployment
- production SQL Server access-log persistence adapter/configuration
- automatic background vehicle snapshot synchronization
- license plate detector
- OCR
- on-device AI inference
- automatic camera plate lookup
- automatic camera-generated access decision
- automatic camera-generated access logging
- camera frame persistence/upload
- access-request workflow
- Manager Approve/Deny workflow
- push notifications
- Admin Dashboard
- automatic access-log deletion or retention policy
- production authentication and authorization
- production deployment

## Reference commits

Backend:

- `6b9e61e` — Microsoft.OpenApi security remediation
- `861ab991` — Vehicle Snapshot Sync API v1
- `88005bf7cd231fc79708f767552499e29fc8da9f` — Access Log Ingestion API v1

Guard Mobile:

- `0d3732f3` — domain access logic and transactional Room cache foundation
- `4eb09213` — snapshot networking and validated Room synchronization
- `046fc8ac` — manual offline verification and Android 17 local-network support
- `ad0788a` — background access-log synchronization
- `35172b5695577866dc2129895b525f8e4386f267` — CameraX foundation

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Unconfirmed architecture decisions remain `PROPOSED`.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin must not communicate directly with Guard Mobile; server-side communication passes through the Backend API.

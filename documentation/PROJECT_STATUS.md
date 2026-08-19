# AVAX ALPR Project Status

**Status snapshot:** 2026-08-19  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, and changelog information is maintained in the dedicated documentation files.

## Current milestone

`MOB-WP-003 — Background Access Log Upload` is confirmed `DONE`.

The validated offline-first access-log flow is now:

```text
Local Access Decision
  -> Local Access Log persisted in Room
  -> Pending
  -> WorkManager
  -> POST /api/access-logs
  -> Stored / AlreadyStored
  -> Synced
```

The Guard App still performs normal vehicle verification and access logging locally without backend connectivity. Access-log synchronization happens separately in the background when connectivity becomes available.

## Next work package

`CAM-WP-001 — CameraX Foundation`

- Priority: `P0 Critical`
- Status: `TODO`
- Target project: AVAX ALPR – Guard Mobile App
- Objective: establish a stable CameraX preview and frame-analysis foundation without introducing detector/OCR logic yet.

This starts Phase 3 of the development roadmap.

## Production persistence follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Priority: `P0 before production`
- Status: `TODO`
- Target project: AVAX ALPR – Backend & Database
- Dependency: approved `dbo.AVAX_ALPR_ACCESS_LOGS` design and BE-WP-003 API contract

The production access-log table and SQL Server runtime persistence adapter are still not deployed. This does not block CameraX or continued mobile development, but it blocks production central access-log storage.

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

## Backend status

Confirmed endpoints:

- `GET /api/vehicles`
- `GET /api/vehicles/by-plate/{plate}`
- `GET /api/sync/vehicles`
- `POST /api/access-logs`

Access Log API v1 semantics:

- new event -> `201 Created`, `Stored`
- identical retry -> `200 OK`, `AlreadyStored`
- same UUID with different logical data -> `409 Conflict`
- mobile `eventTimestampUtc` is preserved
- `receivedAtUtc` is persistence-controlled
- `VehicleNotFound` is valid with `sourceVehicleId = NULL`
- historical decisions are not recalculated or overwritten

Backend implementation reference commit:

`88005bf7cd231fc79708f767552499e29fc8da9f`

## Guard Mobile status

Confirmed offline-first mobile capabilities now include:

- Vehicle Snapshot Sync v1 client
- transactional Room vehicle cache
- local plate normalization
- local access verification for Parking Lot, Site, and Camp
- local access logging
- background access-log synchronization
- application-start recovery
- Android 17 Local Network Protection support

### Access-log synchronization states

Confirmed states:

- `Pending`
- `Synced`
- `Conflict`
- `Rejected`

Semantics:

- `Pending` -> eligible for background upload/retry
- `Synced` -> backend returned `Stored` or `AlreadyStored`
- `Conflict` -> backend returned `409`; preserved locally and excluded from normal retry
- `Rejected` -> permanent contract/request rejection; preserved locally and excluded from infinite retry

Transient failures such as no network, timeout, connection failure, or HTTP 5xx keep the event `Pending`.

### WorkManager behavior

Confirmed implementation:

- network constraint: `NetworkType.CONNECTED`
- exponential retry backoff
- unique queue-draining work
- oldest-first Pending processing
- application-start recovery
- scheduling after successful local log persistence
- partial success is preserved
- successfully acknowledged events are not rolled back because a later event fails

No Room migration was required for MOB-WP-003. Room database version remains `2`.

### MOB-WP-003 validation

Confirmed automated results:

- JVM unit tests: `52/52 PASS`
- Android instrumented tests: `21/21 PASS`
- total automated: `73/73 PASS`
- Android build: PASS

Confirmed physical-device scenarios:

- offline local event creation -> PASS
- Pending persistence -> PASS
- automatic upload after backend/network restoration -> PASS
- app restart recovery -> PASS
- idempotent retry using original UUID -> PASS
- backend row count after retry remained exactly one logical event

A controlled physical `409 Conflict` scenario was not performed. This was optional in the work package and the `409 -> Conflict` behavior is covered by automated tests.

Guard Mobile implementation reference commit:

`ad0788a` — Implement background access log synchronization

## Approved production access-log design

Master-approved table:

`dbo.AVAX_ALPR_ACCESS_LOGS`

Approved fields:

- `id BIGINT IDENTITY(1,1) NOT NULL`
- `mobileEventId UNIQUEIDENTIFIER NOT NULL`
- `eventTimestampUtc DATETIME2(3) NOT NULL`
- `receivedAtUtc DATETIME2(3) NOT NULL`
- `licensePlate NVARCHAR(32) NOT NULL`
- `normalizedPlate NVARCHAR(32) NOT NULL`
- `sourceVehicleId INT NULL`
- `accessArea VARCHAR(16) NOT NULL`
- `accessDecision VARCHAR(24) NOT NULL`

Approved constraints:

- primary key on `id`
- UNIQUE on `mobileEventId`
- CHECK for `ParkingLot`, `Site`, `Camp`
- CHECK for `Granted`, `Denied`, `NotYetValid`, `Expired`, `VehicleNotFound`

No foreign key to `AVAX_VEHICLES` is approved at this stage.

This approved production table has not yet been deployed by the confirmed implementation work.

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

## Architecture decision status

`ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1`

Status: `PROPOSED`

`ARCH-001 — Vehicle Identity & Incremental Sync Strategy` remains `TODO / P1` before a future incremental Sync v2.

The dedicated vehicle access-log storage and UUID-based idempotency semantics were explicitly accepted by Master for Access Log Contract v1 and should remain represented as an accepted architecture decision in the ADR set.

## Explicitly not confirmed as implemented

- production SQL Server `dbo.AVAX_ALPR_ACCESS_LOGS` deployment
- production SQL Server access-log persistence adapter/configuration
- automatic background vehicle snapshot synchronization
- CameraX
- plate detector
- OCR
- on-device AI inference
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

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Unconfirmed architecture decisions remain `PROPOSED`.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin must not communicate directly with Guard Mobile; server-side communication passes through the Backend API.

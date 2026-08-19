# AVAX ALPR Project Status

**Status snapshot:** 2026-08-19  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, and changelog information is maintained in the dedicated documentation files.

## Current milestone

`BE-WP-003 — Access Log Ingestion API v1` is confirmed `DONE` for the development/API-contract scope.

Confirmed development vertical slice:

```text
POST /api/access-logs
  -> AccessLogsController
  -> AccessLogService
  -> AccessLogRepository
  -> development SQLite AVAX_ALPR_ACCESS_LOGS
```

The API provides idempotent access-log ingestion based on the mobile-generated UUID, deterministic retry behavior, historical event preservation, concurrency-safe duplicate protection, sanitized errors, and OpenAPI coverage.

Production SQL Server remained read-only during BE-WP-003. The approved `dbo.AVAX_ALPR_ACCESS_LOGS` production design has NOT yet been physically deployed, and the current runtime access-log persistence adapter is development SQLite only.

## Next work packages

### Mobile — next P0

`MOB-WP-003 — Background Access Log Upload`

- Priority: `P0 Critical`
- Status: `TODO`
- Dependency: `BE-WP-003 — DONE`
- Objective: upload locally `Pending` access logs to `POST /api/access-logs`, retry safely under unstable connectivity, and mark a local event `Synced` only after `Stored` or `AlreadyStored` acknowledgement.

### Production persistence follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Priority: `P0 before production`
- Status: `TODO`
- Dependency: approved `dbo.AVAX_ALPR_ACCESS_LOGS` design and BE-WP-003 API contract
- Objective: deploy the approved production table through a controlled SQL Server change and implement/configure the production SQL Server persistence adapter without changing the public Access Log API v1 contract.

The lack of production deployment does not block mobile development against the validated v1 API contract, but it blocks production central access-log storage.

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

## Backend status

Confirmed backend project: `Avax.ALPR.Api` on `.NET 10.0`.

Confirmed endpoints:

- `GET /api/vehicles`
- `GET /api/vehicles/by-plate/{plate}`
- `GET /api/sync/vehicles`
- `POST /api/access-logs`

### Access Log Ingestion API v1

Confirmed request concepts:

- `mobileEventId`
- `eventTimestampUtc`
- `licensePlate`
- `normalizedPlate`
- nullable `sourceVehicleId`
- `accessArea`
- `accessDecision`

Accepted access areas:

- `ParkingLot`
- `Site`
- `Camp`

Accepted access decisions:

- `Granted`
- `Denied`
- `NotYetValid`
- `Expired`
- `VehicleNotFound`

Rejected as access-log events:

- `InvalidInput`
- `DataUnavailable`

Confirmed HTTP semantics:

- new event -> `201 Created`, status `Stored`
- identical retry -> `200 OK`, status `AlreadyStored`
- same UUID with different logical event data -> `409 Conflict`

`receivedAtUtc` is persistence-controlled and is not accepted as authoritative client data. `eventTimestampUtc` preserves the original mobile event time and is canonicalized to millisecond precision for compatibility with the approved future SQL Server `DATETIME2(3)` contract.

Historical decisions are never recalculated or overwritten during ingestion.

`VehicleNotFound` is valid with `sourceVehicleId = NULL` and does not require an existing central vehicle row.

### Idempotency and concurrency

`mobileEventId` is the canonical idempotency identity.

Logical comparison includes:

- `mobileEventId`
- `eventTimestampUtc`
- `licensePlate`
- `normalizedPlate`
- `sourceVehicleId`
- `accessArea`
- `accessDecision`

Server-generated `id` and `receivedAtUtc` are excluded.

The development persistence layer enforces UNIQUE `mobileEventId`. Concurrent duplicate handling was tested and confirmed to leave exactly one logical row.

### BE-WP-003 validation

Confirmed:

- `39/39` automated tests passed
- `28` Access Log API tests
- existing `11` Vehicle Sync tests remain green
- build passed with `0` errors
- three xUnit analyzer `xUnit2031` warnings remain in existing test style
- vulnerability scan reports no vulnerable packages
- `/openapi/v1.json` returns HTTP 200 and contains the access-log POST contract
- manual identical retry produced `201 Stored`, then `200 AlreadyStored`, with exactly one row for the UUID

Backend implementation reference commit:

`88005bf7cd231fc79708f767552499e29fc8da9f`

### Development environment

A development-only LAN launch profile now listens on:

`http://0.0.0.0:5079`

Existing localhost profiles remain available. No production listener/deployment configuration was changed.

The current development SQLite path is:

`../../DevelopmentData/avax_alpr_dev.db`

## Guard Mobile status

Confirmed mobile baseline includes:

- Kotlin / Jetpack Compose
- Room / SQLite
- Vehicle Snapshot Sync v1 client
- transactional snapshot replacement
- synchronization metadata
- deterministic `PlateNormalizer`
- local `AccessChecker`
- manual Room lookup
- Parking Lot, Site, and Camp evaluation
- persistence across application restart
- Android 17 Local Network Protection support on API 37+

### Local access logging

Confirmed local access-log implementation:

- dedicated Room `access_logs` storage independent from the vehicle cache
- locally generated UUID identity
- local ISO-8601 UTC event timestamp
- explicit Parking Lot / Site / Camp area
- explicit access result
- synchronization states `Pending` and `Synced`
- all new logs start as `Pending`
- pending logs can be queried for future synchronization
- Room migration `1 -> 2` preserves vehicles and sync metadata and creates access-log storage
- no destructive migration
- recent local access-event UI for functional visibility
- access logs persist across application restart
- access logs remain preserved across vehicle snapshot replacement/resynchronization

Master-confirmed access-log semantics:

- `Granted` -> logged
- `Denied` -> logged
- `NotYetValid` -> logged
- `Expired` -> logged
- `VehicleNotFound` -> logged
- `InvalidInput` -> not logged
- `DataUnavailable` -> not logged

Confirmed MOB-WP-002 validation:

- unit tests: `22/22 PASS`
- Android instrumentation tests: `21/21 PASS`
- total: `43/43 PASS`
- physical-device offline logging validated on Google Pixel 6 Pro

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

`receivedAtUtc` must be generated by server/database logic, preferably with UTC database default behavior such as `SYSUTCDATETIME()`.

This approved production table was NOT created during BE-WP-003 because production SQL Server remained under read/discovery-only execution policy.

## Production database findings and safety

The production `AVAX_VEHICLES` audit confirmed:

- no declared `PRIMARY KEY`
- no `IDENTITY`
- no declared `FOREIGN KEY`
- no indexes
- no `UNIQUE` constraint on `licPlate`
- no `CHECK` constraints
- no triggers
- no SQL Server Change Tracking
- no `rowversion`, `updatedAt`, or `modifiedAt` synchronization marker

Access-log discovery confirmed:

- `dbo.AccessEntries` is for person access logs and is not suitable for ALPR vehicle events
- no existing dedicated production vehicle access-log table was identified

During BE-WP-003:

- `AVAX_VEHICLES` production schema modified: NO
- `dbo.AccessEntries` modified: NO
- People / CompanyStructures modified: NO
- production rows modified: NO
- production DDL applied: NONE
- development SQLite schema modified: YES

## Open technical debt and follow-up

Existing production technical debt remains open:

- `TD-001` — `AVAX_VEHICLES` lacks enforced primary identity
- `TD-002` — license plate uniqueness is not enforced
- `TD-003` — no incremental synchronization marker
- `TD-004` — no vehicle lookup index in production

Additional follow-up:

- production access-log table deployment is pending
- production SQL Server access-log persistence adapter/configuration is pending
- no access-log retention policy is defined
- automatic access-log deletion is not implemented
- three existing xUnit analyzer warnings remain in backend test style

## Architecture decision status

`ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1`

Status: `PROPOSED`

`ARCH-001 — Vehicle Identity & Incremental Sync Strategy` remains `TODO / P1` before a future incremental Sync v2.

The dedicated vehicle access-log storage and UUID-based idempotency semantics were explicitly approved by Master for Access Log Contract v1. The Architecture documentation/ADR set should record this as an accepted decision.

## Explicitly not confirmed as implemented

- production SQL Server `dbo.AVAX_ALPR_ACCESS_LOGS` deployment
- production SQL Server access-log persistence adapter/configuration
- WorkManager access-log upload
- automatic background access-log synchronization
- automatic background vehicle synchronization
- automatic access-log deletion or retention policy
- CameraX
- plate detector
- OCR
- on-device AI inference
- access-request workflow
- Manager Approve/Deny workflow
- push notifications
- Admin Dashboard
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

## Documentation map

- [Task Board](TASK_BOARD.md)
- [Roadmap](ROADMAP.md)
- [Architecture](ARCHITECTURE.md)
- [ADR register](adr/)
- [API Contract](API_CONTRACT.md)
- [Database](DATABASE.md)
- [Offline Synchronization](OFFLINE_SYNC.md)
- [Mobile Architecture](MOBILE_ARCHITECTURE.md)
- [Security](SECURITY.md)
- [Testing](TESTING.md)
- [Bugs](BUGS.md)
- [Technical Debt](TECHNICAL_DEBT.md)
- [Changelog](CHANGELOG.md)

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Unconfirmed architecture decisions remain `PROPOSED`.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin must not communicate directly with Guard Mobile; server-side communication passes through the Backend API.

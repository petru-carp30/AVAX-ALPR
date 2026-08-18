# AVAX ALPR Project Status

**Status snapshot:** 2026-08-18  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, and changelog information is maintained in the dedicated documentation files.

## Current milestone

`MOB-WP-002 — Local Access Logging Foundation` is confirmed `DONE`.

The validated local-first flow is:

```text
Plate Input
  -> Local Room Lookup
  -> AccessChecker
  -> AccessDecision
  -> Local Access Log
  -> Pending Sync
```

Access logging works without backend connectivity, access logs survive application restart, and vehicle snapshot replacement does not delete access logs. Normal vehicle verification still does not require a backend request and the Guard App does not connect directly to SQL Server.

## Next work packages

### Backend — next P0

`BE-WP-003 — Access Log Ingestion API v1`

- Priority: `P0 Critical`
- Status: `TODO`
- Dependency: `MOB-WP-002 — DONE`
- Objective: define and implement a backend ingestion contract for mobile-generated access logs with idempotent handling based on the mobile log UUID.

### Mobile — follows backend contract

`MOB-WP-003 — Background Access Log Upload`

- Priority: `P0 Critical`
- Status: `BLOCKED`
- Dependency: `BE-WP-003`
- Objective: upload locally pending access logs with retry/background execution after the backend contract exists.

Automatic background vehicle snapshot synchronization remains separate future work.

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

No task is currently Master-confirmed as `IN PROGRESS`.

## Backend status

Confirmed backend project: `Avax.ALPR.Api` on `.NET 10.0`.

Confirmed endpoints:

- `GET /api/vehicles`
- `GET /api/vehicles/by-plate/{plate}`
- `GET /api/sync/vehicles`

Sync Contract v1 is a complete vehicle snapshot and includes:

- `contractVersion = 1`
- UTC snapshot generation timestamp
- vehicle count
- dedicated mobile vehicle payload
- deterministic plate normalization
- fail-safe `409 Conflict` on normalized plate collisions
- sanitized database errors
- read-only data access

The confirmed Sync v1 backend test suite passed `11/11` integration tests.

A backend access-log ingestion contract is not yet implemented.

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

Confirmed decision states:

- `Granted`
- `Denied`
- `NotYetValid`
- `Expired`
- `VehicleNotFound`
- `InvalidInput`
- `DataUnavailable`

Confirmed result colors in the current Guard UI:

- `Granted` -> GREEN
- `Denied` -> RED
- `Expired`, `NotYetValid`, and unknown-type results -> YELLOW

### Local access logging

Confirmed local access-log implementation:

- dedicated Room `access_logs` storage independent from the vehicle cache
- `AccessLogEntity`
- `AccessLogDao`
- `AccessLogStore`
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

Master-confirmed access-log semantics for the current logging contract:

- `Granted` -> logged
- `Denied` -> logged
- `NotYetValid` -> logged
- `Expired` -> logged
- `VehicleNotFound` -> logged
- `InvalidInput` -> not logged
- `DataUnavailable` -> not logged

`InvalidInput` is treated as an invalid operator/input state rather than a completed vehicle access attempt. `DataUnavailable` is treated as a technical/cache availability state rather than a vehicle access decision. Diagnostic/operator telemetry, if required later, must be handled separately from the access-event log.

Confirmed MOB-WP-002 automated validation:

- unit tests: `22/22 PASS`
- Android instrumentation tests: `21/21 PASS`
- total: `43/43 PASS`
- `AccessLogStoreTest`: `8/8 PASS`
- `GuardDatabaseMigrationTest`: `1/1 PASS`
- MOB-WP-001 regression tests remain green

Physical-device offline logging was validated on a Google Pixel 6 Pro. A historical physical Room v1-to-v2 migration was not demonstrated because the device database was empty before synchronization; migration compatibility is confirmed by the automated migration test only.

## Production database findings

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
- zero duplicate non-null `licPlate` groups at the time of the confirmed audit

Logical relationships:

- `AVAX_VEHICLES.pID -> People.pID`
- `AVAX_VEHICLES.dID -> CompanyStructures.csID`

The production database has not been recreated or modified by the confirmed AVAX ALPR implementation work.

## Open technical debt and development follow-up

The following production technical-debt entries remain open:

- `TD-001` — `AVAX_VEHICLES` lacks enforced primary identity
- `TD-002` — license plate uniqueness is not enforced
- `TD-003` — no incremental synchronization marker
- `TD-004` — no vehicle lookup index in the production schema

These entries do not authorize production schema changes.

A separate backend development-environment follow-up is required because the ASP.NET Core development backend currently binds to localhost by default. Physical-device LAN validation succeeded with an explicit temporary `0.0.0.0:5079` binding. This does not block MOB-WP-002 and is not a mobile defect.

No access-log retention policy is currently defined. Automatic access-log deletion is not implemented.

## Architecture decision status

`ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1`

Status: `PROPOSED`

The full-snapshot architecture is implemented and physically validated, but implementation evidence does not change the ADR to `ACCEPTED`. Master confirmation is required for an ADR status change.

`ARCH-001 — Vehicle Identity & Incremental Sync Strategy` remains `TODO / P1` and is required before a future incremental Sync v2. Incremental synchronization is not implemented.

The use of the mobile-generated access-log UUID as the future backend idempotency key is the intended direction for `BE-WP-003`; the backend contract itself is not yet implemented or confirmed.

## Explicitly not confirmed as implemented

- backend access-log ingestion/upload
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

Guard Mobile:

- `0d3732f3` — domain access logic and transactional Room cache foundation
- `4eb09213` — snapshot networking and validated Room synchronization
- `046fc8ac` — manual offline verification and Android 17 local-network support

MOB-WP-002 reference commit is not yet recorded in Master because the handoff reports it as pending local commit.

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

# AVAX ALPR Project Status

**Status snapshot:** 2026-08-18  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, and changelog information is maintained in the dedicated documentation files.

## Current milestone

`MOB-WP-001 — Offline Vehicle Cache & Manual Access Verification` is confirmed `DONE`.

The validated offline-first flow is:

```text
Backend Sync API
  -> Vehicle Snapshot Sync v1
  -> Room transactional cache
  -> Internet/backend unavailable
  -> Application restart
  -> Manual plate entry
  -> Local normalized lookup
  -> Local access decision
```

This flow was validated on a physical Android device. Normal vehicle verification does not require a backend request and the Guard App does not connect directly to SQL Server.

## Next work package

`MOB-WP-002 — Local Access Logging Foundation`

- Priority: `P0 Critical`
- Status: `TODO`
- Dependency: `MOB-WP-001 — DONE`

Automatic background vehicle synchronization is not part of the confirmed implementation and remains future work.

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

## Open technical debt

The following remain open:

- `TD-001` — `AVAX_VEHICLES` lacks enforced primary identity
- `TD-002` — license plate uniqueness is not enforced
- `TD-003` — no incremental synchronization marker
- `TD-004` — no vehicle lookup index in the production schema

These entries do not authorize production schema changes.

## Architecture decision status

`ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1`

Status: `PROPOSED`

The full-snapshot architecture is implemented and physically validated, but implementation evidence does not change the ADR to `ACCEPTED`. Master confirmation is required for an ADR status change.

`ARCH-001 — Vehicle Identity & Incremental Sync Strategy` remains `TODO / P1` and is required before a future incremental Sync v2. Incremental synchronization is not implemented.

## Explicitly not confirmed as implemented

- automatic background vehicle synchronization
- local access logging
- access-log upload
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

# AVAX ALPR Task Board

**Board scope:** backlog, task status, priorities, dependencies, current/next work packages, and confirmed milestones.

**Last Master-confirmed status update:** 2026-08-17

## Governance

Statuses:

- `TODO`
- `IN PROGRESS`
- `BLOCKED`
- `DONE`

Priorities:

- `P0` — Critical
- `P1` — High
- `P2` — Medium
- `P3` — Low

A task must not be marked `DONE` without explicit Master confirmation that implementation and testing/validation are complete.

## Current task board

| ID | Task | Priority | Status | Dependencies / notes |
|---|---|---:|---|---|
| BE-001 | Backend Baseline Audit & Build Validation | P0 | DONE | Master-confirmed |
| SEC-001 | Resolve NU1903 Microsoft.OpenApi Vulnerability | P0 | DONE | Master-confirmed |
| BE-002 | Validate Existing SQL Schema Relevant to ALPR | P0 | DONE | Master-confirmed |
| DEVDB-001 | Local SQLite Development Database Baseline | P1 | DONE | Master-confirmed development/testing asset |
| BE-WP-001 | Local Backend Vehicle Read API Foundation | P0 | DONE | Master-confirmed |
| BE-003 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-004 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-005 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-006 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-007 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-008 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-009 | BE-WP-001 subtask | P0 | DONE | Master-confirmed as part of BE-WP-001 |
| BE-WP-002 | Vehicle Snapshot Sync API v1 | P0 | DONE | Depends on BE-WP-001; Master-confirmed |
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | P0 | DONE | Depends on BE-WP-001 and BE-WP-002; Master-confirmed on physical Android device |
| MOB-WP-002 | Local Access Logging Foundation | P0 | TODO | Next P0 work package; depends on MOB-WP-001 |
| ARCH-001 | Vehicle Identity & Incremental Sync Strategy | P1 | TODO | Required before a future incremental Sync v2; does not block current full-snapshot MVP |

## Current and next work

No task has been Master-confirmed as `IN PROGRESS` or `BLOCKED` in the planning history recorded here.

The next confirmed P0 work package is:

`MOB-WP-002 — Local Access Logging Foundation`

- Priority: `P0`
- Status: `TODO`
- Dependency: `MOB-WP-001 — DONE`

The architectural follow-up is:

`ARCH-001 — Vehicle Identity & Incremental Sync Strategy`

- Priority: `P1`
- Status: `TODO`
- Required before a future incremental Sync v2
- Does not block the current full-snapshot MVP

## Confirmed backend synchronization milestone

`BE-WP-002 — Vehicle Snapshot Sync API v1` is confirmed `DONE`.

Confirmed implementation:

- `GET /api/sync/vehicles`
- full vehicle snapshot
- `contractVersion = 1`
- UTC snapshot generation timestamp
- vehicle count
- dedicated mobile payload
- shared deterministic plate normalization
- normalized plate collisions return `409 Conflict`
- sanitized database errors
- read-only implementation
- 11/11 automated integration tests passed
- OpenAPI validated
- no vulnerable NuGet packages
- reference commit `861ab991` on `master`

## Confirmed mobile offline milestone

`MOB-WP-001 — Offline Vehicle Cache & Manual Access Verification` is confirmed `DONE`.

Validated flow:

`Backend Sync API -> Room -> Internet OFF -> Application Restart -> Manual Plate Entry -> Local Lookup -> Local Access Result`

Confirmed implementation and validation:

- `PlateNormalizer`
- `AccessChecker`
- Parking Lot / Site / Camp access evaluation
- transactional Room vehicle cache
- synchronization metadata
- Vehicle Snapshot Sync v1 client
- `contractVersion` validation
- `vehicleCount` validation
- duplicate normalized plate protection
- DTO-to-Room mapping
- manual offline access verification
- result states: `Granted`, `Denied`, `NotYetValid`, `Expired`, `VehicleNotFound`, `InvalidInput`, `DataUnavailable`
- physical Android-device validation
- offline persistence after application restart
- Android 17 `ACCESS_LOCAL_NETWORK` support

Reference commits:

- `0d3732f3`
- `4eb09213`
- `046fc8ac`

## Explicitly not confirmed as implemented

The following remain outside the confirmed implementation state and must not be treated as complete:

- automatic background vehicle synchronization
- CameraX
- AI detector
- OCR
- access log upload
- Manager functionality

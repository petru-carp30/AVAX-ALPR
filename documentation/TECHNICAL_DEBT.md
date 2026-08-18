# AVAX ALPR Technical Debt and Limitations

**Evidence status date:** 2026-08-18

This document records confirmed technical debt and known limitations affecting AVAX ALPR.

Open technical-debt entries do not authorize production schema or architecture changes. A remediation must be explicitly approved, implemented, and validated before an entry can be considered resolved.

## Open technical debt and limitations

### TD-001 — AVAX_VEHICLES lacks enforced primary identity

- **Type:** Technical debt / Limitation
- **Severity:** Medium
- **Component:** Production SQL Server / Backend synchronization architecture
- **Status:** OPEN
- **Cause:** `AVAX_VEHICLES` has no declared `PRIMARY KEY` and no `IDENTITY` constraint.
- **Mitigation:** Sync Contract v1 uses a complete vehicle snapshot and does not currently depend on incremental change tracking.
- **Resolution:** Not yet decided.
- **Validation:** Production schema audit confirmed the absence of the constraints described above.

### TD-002 — License plate uniqueness is not enforced

- **Type:** Technical debt / Limitation
- **Severity:** High
- **Component:** Production SQL Server / Vehicle synchronization
- **Status:** OPEN
- **Cause:** No `UNIQUE` constraint or unique index exists on `licPlate`.
- **Current evidence:** The production audit found zero duplicate non-null license-plate groups, but uniqueness is not guaranteed by the schema.
- **Mitigation:** Backend deterministic normalization checks for normalized plate collisions. `GET /api/sync/vehicles` fails safely with `409 Conflict` when collisions would make deterministic mobile lookup unsafe.
- **Resolution:** Not yet decided.
- **Validation:** Production schema audit confirmed that uniqueness is not enforced; backend Sync v1 collision handling is confirmed implemented and tested.

### TD-003 — No incremental synchronization marker

- **Type:** Technical debt / Limitation
- **Severity:** Medium
- **Component:** Production SQL Server / Sync architecture
- **Status:** OPEN
- **Cause:** `AVAX_VEHICLES` has no `rowversion`, `updatedAt`, `modifiedAt`, or SQL Server Change Tracking mechanism available for incremental synchronization.
- **Mitigation:** Sync Contract v1 uses a complete snapshot.
- **Resolution:** Future Sync v2 strategy is not yet decided.
- **Validation:** Production schema audit confirmed the absence of the listed synchronization markers and Change Tracking.

### TD-004 — No vehicle lookup index in production schema

- **Type:** Technical debt / Limitation
- **Severity:** Medium
- **Component:** Production SQL Server
- **Status:** OPEN
- **Cause:** No indexes exist on `AVAX_VEHICLES`, including on `licPlate`.
- **Mitigation:** Current development work uses the local SQLite development database, and current scale has not demonstrated a production performance blocker.
- **Resolution:** Requires future performance evidence and an approved production database change.
- **Validation:** Production schema audit confirmed the absence of indexes on `AVAX_VEHICLES`.

## Status and change-control rules

- Do not mark `TD-001` through `TD-004` resolved without confirmed remediation and validation.
- Do not modify the production SQL Server schema solely because these technical-debt entries exist.
- Unconfirmed permanent remediation options remain `PROPOSED`.
- Contract changes caused by a future remediation must be coordinated across every affected AVAX ALPR project.

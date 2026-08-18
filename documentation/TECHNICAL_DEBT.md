# AVAX ALPR Technical Debt, Limitations, and Resolved Issues

**Evidence status date:** 2026-08-18

This document records confirmed technical debt, known limitations, and validated bug fixes that affect AVAX ALPR architecture or implementation.

Open technical-debt entries do not authorize production schema or architecture changes. A remediation must be explicitly approved, implemented, and validated before an entry can be considered resolved.

## Open technical debt and limitations

### TD-001 — AVAX_VEHICLES lacks enforced primary identity

- **Severity:** Medium
- **Component:** Production SQL Server / Backend synchronization architecture
- **Status:** OPEN
- **Cause:** `AVAX_VEHICLES` has no declared `PRIMARY KEY` and no `IDENTITY` constraint.
- **Current mitigation:** Sync Contract v1 uses a complete vehicle snapshot and does not currently depend on incremental change tracking.
- **Resolution:** Not yet decided.

### TD-002 — License plate uniqueness is not enforced

- **Severity:** High
- **Component:** Production SQL Server / Vehicle synchronization
- **Status:** OPEN
- **Cause:** No `UNIQUE` constraint or unique index exists on `licPlate`.
- **Current evidence:** The production audit found zero duplicate non-null license-plate groups, but uniqueness is not guaranteed by the schema.
- **Current mitigation:** Backend deterministic normalization checks for normalized plate collisions. `GET /api/sync/vehicles` fails safely with `409 Conflict` when collisions would make deterministic mobile lookup unsafe.
- **Resolution:** Not yet decided.

### TD-003 — No incremental synchronization marker

- **Severity:** Medium
- **Component:** Production SQL Server / Sync architecture
- **Status:** OPEN
- **Cause:** `AVAX_VEHICLES` has no `rowversion`, `updatedAt`, `modifiedAt`, or SQL Server Change Tracking mechanism available for incremental synchronization.
- **Current mitigation:** Sync Contract v1 uses a complete snapshot.
- **Resolution:** Future Sync v2 strategy is not yet decided.

### TD-004 — No vehicle lookup index in production schema

- **Severity:** Medium
- **Component:** Production SQL Server
- **Status:** OPEN
- **Cause:** No indexes exist on `AVAX_VEHICLES`, including on `licPlate`.
- **Current mitigation:** Current development work uses the local SQLite development database, and current scale has not demonstrated a production performance blocker.
- **Resolution:** Requires future performance evidence and an approved production database change.

## Resolved and validated issues

### Android 17 Local Network Protection

- **Severity:** Not assigned in the confirmed update
- **Component:** AVAX ALPR Guard Mobile App
- **Status:** RESOLVED / VALIDATED
- **Issue:** Physical-device synchronization with the local development backend was blocked by Android 17 Local Network Protection.
- **Cause:** Local network access requires explicit permission behavior on API 37+.
- **Resolution:** Implemented `android.permission.ACCESS_LOCAL_NETWORK` and a runtime permission request on API 37+. Existing `INTERNET`, `ACCESS_NETWORK_STATE`, and debug cleartext HTTP configuration were preserved.
- **Validation:** Physical Android-device synchronization with the local development backend succeeds.
- **Build baseline impact:** No locked Android build baseline changes were required.

## Change-control rules

- Do not mark `TD-001` through `TD-004` resolved without confirmed remediation and validation.
- Do not modify the production SQL Server schema solely because these technical-debt entries exist.
- Unconfirmed permanent remediation options remain `PROPOSED`.
- Contract changes caused by a future remediation must be coordinated across every affected AVAX ALPR project.

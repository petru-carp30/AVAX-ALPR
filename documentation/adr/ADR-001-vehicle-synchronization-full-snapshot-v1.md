# ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1

**Status:** PROPOSED  
**Decision date:** 2026-08-10  
**Implementation evidence updated:** 2026-08-17

## Context

AVAX ALPR is offline-first. The Guard Mobile App must verify normal vehicle access locally and must not contact the backend for every license plate lookup.

The implemented synchronization endpoint is:

`GET /api/sync/vehicles`

Sync Contract v1 returns:

- `contractVersion = 1`;
- a complete vehicle snapshot;
- a UTC generation timestamp;
- vehicle count;
- a dedicated Guard Mobile payload.

The current synchronization architecture is:

`Backend -> Full Snapshot -> Guard App -> Transactional Local Cache Replacement`

During the confirmed production schema audit, `AVAX_VEHICLES` was found to have no reliable database-level primitive for deterministic incremental synchronization:

- no declared `PRIMARY KEY` constraint;
- no `UNIQUE` constraint on `licPlate`;
- no indexes relevant to synchronization;
- no `rowversion`;
- no `updatedAt` / `modifiedAt` marker;
- no SQL Server Change Tracking.

The backend applies deterministic license plate normalization before publishing a snapshot. If multiple source records produce the same normalized plate, snapshot generation fails safely with `409 Conflict`.

Incremental or delta synchronization is not implemented in Sync Contract v1.

## Decision

**PROPOSED:** Sync Contract v1 uses complete vehicle snapshots. The Guard Mobile App downloads the complete snapshot, validates it, and transactionally replaces its local Room vehicle cache.

The intended sequence is:

`Request Snapshot -> Validate Contract -> Receive Complete Dataset -> Transactional Local Cache Replacement -> Continue Offline Operation`

A failed or incomplete synchronization must not replace the previously valid local dataset. The existing cache remains available until a new snapshot has been completely validated and committed.

This decision applies only to Sync Contract v1 and does not establish full-snapshot synchronization as the permanent model for future contract versions.

## Relationship with Offline-First Architecture

Synchronization is separated from normal vehicle verification.

The normal verification path is:

`Plate -> Normalize -> Local Room Lookup -> Local Access Evaluation`

It is not:

`Plate -> Backend Request -> Access Evaluation`

Consequently:

- normal vehicle verification does not contact the backend;
- temporary backend or Internet outages do not prevent local verification when a valid cache exists;
- the Guard App never connects directly to SQL Server;
- the Room database is the operational source for normal vehicle lookup;
- access evaluation runs locally;
- Parking Lot, Site, and Camp are explicit local access areas.

## Alternatives Considered

### Modification timestamp based delta synchronization

Not selected for v1 because the confirmed production schema has no reliable `updatedAt` or `modifiedAt` marker.

### SQL Server Change Tracking

Not selected for v1 because Change Tracking is not enabled. Enabling it would require a separate production database architecture decision. This ADR does not approve such a modification.

### `rowversion` based synchronization

Not selected for v1 because no suitable `rowversion` exists. Introducing one would require a separate production database decision. This ADR does not approve that change.

### Backend-managed change journal

Not selected for v1. It could support future delta synchronization, but it introduces persistent synchronization state, ordering, retention, deletion tracking, and reconciliation rules.

### Hash-based comparison

Not selected for v1. Hashes can detect differences but do not independently provide authoritative change history, deletion tracking, or deterministic synchronization ordering.

### Backend lookup for every vehicle

Rejected because it conflicts directly with the offline-first requirement.

## Consequences

### Positive

- simple and explicit Sync v1 contract;
- deterministic reconstruction of local state;
- missed synchronization cycles do not require delta replay recovery;
- deleted backend vehicles disappear after the next successful full snapshot;
- no approved production SQL Server schema modification is required;
- local access verification remains independent of network availability;
- `contractVersion` provides an explicit contract-evolution boundary.

### Trade-offs

- each successful synchronization transfers the complete Guard Mobile vehicle dataset;
- bandwidth, snapshot generation cost, and local replacement cost grow with dataset size;
- changes after snapshot generation are not visible until a later successful synchronization;
- Sync v1 does not expose change history to the client.

## Risks

### Dataset growth

Full snapshot transfer may become inefficient as the dataset grows. Payload size, generation duration, download duration, cache replacement duration, and failure rate should be observed before deciding whether Sync v2 is necessary.

### Interrupted synchronization

Network or processing failure can occur before a snapshot is committed. The previous valid cache must remain intact until the new snapshot is fully validated and transactionally applied.

### Normalized plate collision

Two source records can normalize to the same plate and make deterministic offline lookup ambiguous. The backend mitigates this by rejecting the snapshot with `409 Conflict`.

### Contract incompatibility

Future backend changes may be incompatible with older Guard App versions. The Guard App must accept only supported `contractVersion` values.

### Stale offline data

An offline device can continue using an older successful snapshot. This is an inherent offline-first trade-off. Any policy that restricts operation based on snapshot age requires a separate confirmed business/security decision.

## Implementation Evidence

`MOB-WP-001 — Offline Vehicle Cache & Manual Access Verification` validated the architecture on a physical Android device.

Confirmed operational flow:

`Backend -> Vehicle Snapshot Sync v1 -> Room Transactional Cache -> Backend/Internet Unavailable -> Application Restart -> Room Local Lookup -> Local Access Evaluation`

Confirmed evidence:

- normal vehicle verification does not contact the backend;
- the Guard App does not connect directly to SQL Server;
- access evaluation runs locally;
- failed snapshot synchronization preserves the previous valid local cache;
- previously synchronized Room data remains usable after application restart while offline;
- Parking Lot, Site, and Camp are explicit access areas;
- mobile plate normalization follows the same conceptual normalization contract as the backend.

The temporary manual synchronization control is development-only. Automatic background vehicle synchronization remains future work.

Implementation evidence does **not** change this ADR from `PROPOSED` to `ACCEPTED`.

## Migration Path Toward Sync v2

A future Sync v2 may introduce incremental synchronization if operational measurements justify the additional complexity.

Potential capabilities include:

- server-defined synchronization cursor or version token;
- explicit `upsert` and `delete` delta operations;
- deterministic change ordering;
- snapshot/version identifiers;
- reconciliation and resume behavior;
- fallback from delta synchronization to a complete snapshot.

A possible future model is:

`Initial Full Snapshot -> Sync Cursor -> Delta Requests -> Transactional Local Apply`

with recovery through:

`Delta State Invalid -> Request New Full Snapshot -> Replace Local Cache`

The exact source of future change information is intentionally undecided. Backend-managed synchronization metadata or database-supported change mechanisms would require separate evaluation and, where applicable, separate architectural approval.

## Affected Projects

- AVAX ALPR – Backend & Database
- AVAX ALPR – Guard Mobile App
- AVAX ALPR – Project Management & Architecture

Any future move from full snapshot to incremental synchronization is a cross-component contract change.

## Status Governance

This ADR remains `PROPOSED` until explicitly accepted by Master. Implementation and successful testing are evidence supporting the proposal, not automatic architectural acceptance.

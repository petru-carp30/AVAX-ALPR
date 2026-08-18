# AVAX ALPR Architecture

## Purpose

AVAX ALPR is an offline-first license-plate access verification system for construction-site gates. Normal vehicle verification must continue when backend or mobile network connectivity is unavailable.

This document describes confirmed architectural boundaries, confirmed implementation evidence, and target architecture. Formal architecture decisions are maintained separately under [`documentation/adr/`](adr/).

## Architecture governance

Architecture documentation distinguishes between two different concepts:

- **Implementation evidence** — behavior that has been explicitly confirmed implemented and tested by Master.
- **Architectural approval** — a formal decision whose ADR status has been explicitly confirmed by Master.

Implementation evidence does not automatically approve an architecture decision. An ADR remains `PROPOSED` until Master explicitly confirms acceptance.

ADR status must therefore be preserved exactly. Unconfirmed decisions are documented as `PROPOSED`; they must not be changed to `ACCEPTED` based only on successful implementation or testing.

## Component boundaries

| Component | Responsibility |
|---|---|
| AI Model | Dataset, plate detection, OCR, evaluation, optimization, and mobile model export |
| Backend & Database | ASP.NET Core API, synchronization contracts, SQL Server integration, access logs, permissions, authentication, access requests, and server-side notification logic |
| Guard Mobile App | Android/Kotlin, local Room cache, offline verification, local access decisions, future CameraX and AI integration |
| Manager & Admin | Future approval workflow, administration, dashboards, and analytics |
| Project Management & Architecture | Cross-component contracts, roadmap, architecture decisions, and official documentation |

## Confirmed architectural boundaries

The following cross-component rules are established project constraints:

- The system is offline-first.
- The Guard App must not contact the backend for normal vehicle verification.
- The Guard App must never connect directly to SQL Server.
- Access evaluation runs locally in the Guard App.
- The AI pipeline supplies recognition results; AI does not decide vehicle access.
- The Manager App does not communicate directly with the Guard App.
- Server-side communication passes through the Backend API.
- Contract changes must identify and coordinate all affected projects.
- The existing SQL Server database remains the authoritative server-side business data source.
- The existing `[DEVS-AVAX-RO].[dbo].[AVAX_VEHICLES]` table must not be recreated without a justified and explicitly accepted decision.

## Confirmed operational architecture

`MOB-WP-001 — Offline Vehicle Cache & Manual Access Verification` validated the first offline-first vertical slice on a physical Android device.

Confirmed operational flow:

```text
ASP.NET Core Backend
        |
        | GET /api/sync/vehicles
        | complete snapshot, contractVersion = 1
        v
Vehicle Snapshot Sync v1 Client
        |
        v
Transactional Room Cache
        |
        | backend/network can become unavailable
        | application can restart
        v
Manual Plate Entry
        |
        v
Deterministic Plate Normalization
        |
        v
Local Room Lookup
        |
        v
Local Parking Lot / Site / Camp Access Evaluation
```

Confirmed implementation evidence:

- `GET /api/sync/vehicles` provides Vehicle Snapshot Sync v1.
- The Guard App stores the synchronized dataset in Room.
- Snapshot replacement is transactional.
- A failed snapshot synchronization does not destroy the previous valid local cache.
- Previously synchronized data remains usable after backend/Internet loss and application restart.
- Normal access verification does not contact the backend.
- The Guard App does not connect directly to SQL Server.
- Access evaluation runs locally.
- Parking Lot, Site, and Camp are explicit access areas.
- Mobile plate normalization follows the same conceptual normalization contract as the backend.
- The backend rejects normalized-plate collisions before a conflicting snapshot can be accepted.
- The temporary manual synchronization control is development-only.

The normal confirmed vehicle-verification path is therefore:

```text
Plate Input
  -> Plate Normalization
  -> Local Room Lookup
  -> Local Access Evaluation
```

and not:

```text
Plate Input
  -> Backend Request
  -> Access Evaluation
```

## Vehicle synchronization architecture

The currently implemented Sync Contract v1 architecture is:

```text
Backend
  -> Full Vehicle Snapshot
  -> Guard App
  -> Transactional Local Cache Replacement
```

Incremental/delta synchronization is not implemented in Sync Contract v1.

The production schema audit established constraints relevant to synchronization strategy:

- no declared `PRIMARY KEY` constraint on `AVAX_VEHICLES`;
- no `UNIQUE` constraint on `licPlate`;
- no indexes relevant to synchronization;
- no `rowversion`;
- no `updatedAt` / `modifiedAt` marker;
- no SQL Server Change Tracking.

These constraints are documented facts. This architecture document does not approve modifications to the production SQL Server schema or configuration.

The formal decision record for the full-snapshot strategy is [ADR-001 — Vehicle Synchronization Strategy: Full Snapshot for Sync Contract v1](adr/ADR-001-vehicle-synchronization-full-snapshot-v1.md).

**ADR-001 status: `PROPOSED`.**

The successful physical-device validation is implementation evidence supporting the proposed architecture; it is not architectural approval. ADR-001 remains `PROPOSED` until explicitly accepted by Master.

## Target architecture

The overall target flow is:

```text
Camera
  -> Plate Detector
  -> OCR
  -> Plate Normalization
  -> Local SQLite Lookup
  -> Access Decision
  -> Access Log
  -> Background Sync
  -> ASP.NET Core API
  -> Existing SQL Server
```

Only synchronization, local Room lookup, plate normalization, and manual local access evaluation are confirmed implemented within this target flow.

The following remain future work and must not be described as implemented:

- automatic background vehicle synchronization;
- CameraX integration;
- license plate detector integration;
- OCR integration;
- access-log upload;
- Manager functionality.

The target access-request flow is:

```text
Guard App -> Backend API -> Manager -> Approve/Deny -> Backend API -> Guard App
```

This flow is not confirmed implemented and remains `PROPOSED`.

## Data ownership and communication rules

- SQL Server owns the authoritative server-side business data.
- The Backend API is the server-side integration boundary.
- The Guard App owns a replaceable local Room snapshot used for offline reads.
- Normal access evaluation uses the local mobile dataset rather than a per-vehicle backend call.
- The AI pipeline provides plate-recognition input but does not own access policy.
- Manager-side workflows must communicate with the Guard App through the backend, not directly.

## Current ADR register

| ADR | Topic | Status | Implementation evidence |
|---|---|---|---|
| [ADR-001](adr/ADR-001-vehicle-synchronization-full-snapshot-v1.md) | Vehicle Synchronization Strategy — Full Snapshot for Sync Contract v1 | `PROPOSED` | Sync v1 and transactional offline cache flow validated on physical Android device |

No other architecture decision from the discussions represented in this backfill is recorded as formally accepted unless a dedicated ADR explicitly states `ACCEPTED` following Master confirmation.

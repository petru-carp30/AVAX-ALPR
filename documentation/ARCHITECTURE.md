# AVAX ALPR Architecture

## Purpose

AVAX ALPR is an offline-first license-plate access verification system for construction-site gates. Normal vehicle verification must continue when backend or mobile network connectivity is unavailable.

## Component boundaries

| Component | Responsibility |
|---|---|
| AI Model | Dataset, plate detection, OCR, evaluation, optimization, and mobile model export |
| Backend & Database | ASP.NET Core API, synchronization contracts, SQL Server integration, access logs, permissions, authentication, access requests, and server-side notification logic |
| Guard Mobile App | Android/Kotlin, local Room cache, offline verification, local access decisions, future CameraX and AI integration |
| Manager & Admin | Future approval workflow, administration, dashboards, and analytics |
| Project Management & Architecture | Cross-component contracts, roadmap, decisions, and official documentation |

## Confirmed operational architecture

The first offline-first vertical slice is implemented and validated:

```text
ASP.NET Core Backend
        |
        | GET /api/sync/vehicles (complete snapshot, contract v1)
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

Confirmed principles:

- Normal access verification does not contact the backend.
- The Guard App does not connect directly to SQL Server.
- Access evaluation runs locally.
- A failed snapshot synchronization does not destroy the previous valid cache.
- Parking Lot, Site, and Camp are explicit access areas.
- The backend rejects normalized-plate collisions before a snapshot is accepted.
- The temporary manual synchronization control is development-only.

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

Only the synchronization, local Room lookup, normalization, and manual local access-evaluation portion is confirmed implemented. CameraX, detector, OCR, access-log upload, and automatic background synchronization remain future work.

The target access-request flow is:

```text
Guard App -> Backend API -> Manager -> Approve/Deny -> Backend API -> Guard App
```

This flow is not confirmed implemented and is therefore `PROPOSED`.

## Data ownership and communication rules

- The existing SQL Server database remains the authoritative business data source.
- The existing `[DEVS-AVAX-RO].[dbo].[AVAX_VEHICLES]` table must not be recreated without a justified and accepted decision.
- The backend is the only component that integrates with server-side data.
- The Guard App owns a replaceable local Room snapshot for offline reads.
- The AI pipeline supplies recognition results; it does not decide access.
- The Manager App does not communicate directly with the Guard App.
- Contract changes must be coordinated between all affected components.

## Current synchronization decision status

Full-snapshot synchronization is the implemented Sync Contract v1 strategy. The corresponding formal ADR remains `PROPOSED` until explicitly accepted by the project owner; implementation evidence must not be interpreted as automatic ADR acceptance.

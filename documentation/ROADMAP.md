# AVAX ALPR Roadmap

**Status date:** 2026-08-17

Statuses used: `TODO`, `IN PROGRESS`, `BLOCKED`, `DONE`.

A task is marked `DONE` only after explicit confirmation that it was implemented and tested.

## Confirmed completed foundations

| Work item | Priority | Status | Confirmed result |
|---|---:|---|---|
| BE-001 — Backend Baseline Audit & Build Validation | P0 | DONE | Backend baseline audited and build validated |
| SEC-001 — Resolve Microsoft.OpenApi vulnerability | P0 | DONE | Vulnerability resolved and validated |
| BE-002 — Validate Existing SQL Schema Relevant to ALPR | P0 | DONE | Production SQL Server schema audited read-only |
| DEVDB-001 — Local SQLite Development Database Baseline | P0 | DONE | Development/test SQLite replica established |
| BE-WP-001 — Local Backend Vehicle Read API Foundation | P0 | DONE | Vehicle list and normalized plate lookup implemented |
| BE-WP-002 — Vehicle Snapshot Sync API v1 | P0 | DONE | Complete snapshot contract implemented and tested |
| MOB-WP-001 — Offline Vehicle Cache & Manual Access Verification | P0 | DONE | Offline-first Android vertical slice implemented and validated on a physical device |

## Current next work

| Work item | Priority | Status | Objective |
|---|---:|---|---|
| MOB-WP-002 — Local Access Logging Foundation | P0 | TODO | Persist local access verification events as the foundation for later synchronization |

## Phased roadmap

### Phase 1 — Backend API, Android foundation, and SQLite synchronization

Core read API, Sync v1 contract, Room cache, and manual offline verification are confirmed complete.

Automatic background vehicle synchronization remains future work.

### Phase 2 — Offline access verification and logs

Manual local verification is complete. Local access logging is next.

Access-log upload and conflict/retry behavior are not yet confirmed.

### Phase 3 — CameraX

`PROPOSED`: camera preview, lifecycle integration, frame analysis, and performance controls.

### Phase 4 — AI detector, OCR, and mobile integration

`PROPOSED`: dataset pipeline, detector, OCR, normalization integration, evaluation thresholds, model export, and on-device runtime integration.

The AI system must produce recognition data only; it must not decide access.

### Phase 5 — Access requests, approval, and notifications

`PROPOSED`: Guard App request, Backend API workflow, Manager Approve/Deny action, and result delivery to the Guard App.

### Phase 6 — Manager, administration, and analytics

`PROPOSED`: manager interface, administration, dashboards, and reporting.

### Phase 7 — Security, testing, deployment, and documentation

Some security and testing foundations already exist. Production authentication, authorization, deployment architecture, operational monitoring, and release readiness are not yet confirmed.

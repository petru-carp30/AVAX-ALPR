# AVAX ALPR

**Offline-first Automatic License Plate Recognition for vehicle access control.**

AVAX ALPR is a modular access-control system designed for gate operators working in environments where mobile connectivity may be unreliable. The Guard application detects and reads license plates on-device, verifies access against a local Room database, records the decision locally, and synchronizes when connectivity is available.

> The AI pipeline recognizes the plate. It does **not** decide access. Access decisions remain deterministic and are made from locally synchronized access data.

## Current status

The first standalone Guard MVP has been implemented and physically validated on a Google Pixel 6 Pro.

| Area | Status |
| --- | --- |
| CameraX live scanning | Implemented |
| YOLOX-Nano plate detection | Implemented |
| Google ML Kit OCR | Implemented |
| 2-of-3 OCR confirmation | Implemented |
| Offline Room vehicle cache | Implemented |
| Local access decision | Implemented |
| Local access logging | Implemented |
| Background log synchronization | Implemented |
| Camera-first operator UI | Implemented and merged |
| SQL Server access-log persistence support | Implemented, target deployment deferred |
| Manager / Admin workflow | Deferred |
| Host-application integration | Handed off to host application owner |

Current Guard `master` includes the camera-first operator interface introduced by merge commit:

```text
d22780d2a220458a1d0fe77a2c78da234eacda2a
merge: add camera-first guard operator UI
```

## Core architecture

```text
CameraX
   ↓
YOLOX Plate Detector
   ↓
Plate Crop
   ↓
Google ML Kit OCR
   ↓
2-of-3 Confirmation
   ↓
Plate Normalization
   ↓
Room / SQLite Local Lookup
   ↓
AccessChecker
   ↓
Granted / Denied / Unknown
   ↓
Local Access Log
   ↓
WorkManager Background Sync
   ↓
ASP.NET Core API
```

The normal vehicle decision path is intentionally **offline-first**. The Guard application does not contact the backend for every vehicle and never connects directly to SQL Server.

## Guard operator workflow

The current operator UI is designed around the camera instead of development diagnostics:

- fullscreen camera-first layout;
- compact access-area selector;
- editable OCR plate field;
- quick manual correction and Search;
- zoom, pinch-to-zoom, tap-to-focus, and focus assist;
- clear Granted / Denied / Unknown result overlay;
- developer diagnostics preserved behind the debug-only `DEV` view.

Typical flow:

```text
Plate enters camera
→ detector finds plate
→ OCR reads plate
→ operator can correct text if needed
→ local Room lookup
→ access decision
→ result shown immediately
→ event stored locally
→ sync happens later when network is available
```

## Technology stack

### Android Guard App

- Kotlin
- Jetpack Compose
- CameraX
- Room / SQLite
- WorkManager
- ONNX Runtime Android
- Google ML Kit Text Recognition v2

### Backend

- C#
- ASP.NET Core Web API
- SQL Server integration
- `Microsoft.Data.SqlClient`

### AI

- YOLOX-Nano plate detector
- ONNX export for Android inference
- Google ML Kit OCR for the current MVP

## Repositories

This repository is the project coordination and documentation root.

| Component | Repository |
| --- | --- |
| Project / Documentation | [petru-carp30/AVAX-ALPR](https://github.com/petru-carp30/AVAX-ALPR) |
| Backend API | [petru-carp30/Avax.ALPR.Api](https://github.com/petru-carp30/Avax.ALPR.Api) |
| Guard Android App | [petru-carp30/Avax.ALPR.Guard](https://github.com/petru-carp30/Avax.ALPR.Guard) |
| AI Model | [petru-carp30/AVAX-ALPR-AI](https://github.com/petru-carp30/AVAX-ALPR-AI) |

The backend and mobile projects are linked here as Git submodules.

## Clone

Clone the project together with its submodules:

```bash
git clone --recurse-submodules https://github.com/petru-carp30/AVAX-ALPR.git
cd AVAX-ALPR
```

If the repository was already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Key design principles

1. **Offline-first** — normal gate access verification must work without network connectivity.
2. **Deterministic access logic** — AI reads the plate; `AccessChecker` decides access.
3. **No direct mobile-to-SQL access** — all server-side communication goes through the backend API.
4. **Local durability** — access events are stored locally before synchronization.
5. **Simple architecture** — no unnecessary microservices, message brokers, or infrastructure layers.
6. **Field usability** — the operator UI prioritizes fast scanning, correction, and clear decisions.

## Access result semantics

The Guard application currently distinguishes the following local outcomes:

- `Granted`
- `Denied`
- `NotYetValid`
- `Expired`
- `VehicleNotFound`
- `InvalidInput`
- `DataUnavailable`

The operator UI presents these using clear status colors while preserving the same underlying access logic used by the validated MVP.

## Backend API surface

Confirmed backend endpoints include:

```text
GET  /api/vehicles
GET  /api/vehicles/by-plate/{plate}
GET  /api/sync/vehicles
POST /api/access-logs
```

Vehicle synchronization uses a full snapshot contract for the current version.

Access-log ingestion supports idempotency through the mobile-generated event UUID.

## SQL Server deployment note

SQL Server persistence support for ALPR access logs has been implemented in the backend, including the prepared `dbo.AVAX_ALPR_ACCESS_LOGS` deployment script.

The final controlled deployment and write validation against the manager-controlled SQL Server environment are intentionally deferred until approved credentials and write/DDL permissions are available.

No unauthorized production schema changes are performed by this project.

## Documentation

Detailed technical documentation is available under [`documentation/`](documentation/).

Useful starting points:

- [Project Status](documentation/PROJECT_STATUS.md)
- [Architecture](documentation/ARCHITECTURE.md)
- [API Contract](documentation/API_CONTRACT.md)
- [Database](documentation/DATABASE.md)
- [Offline Sync](documentation/OFFLINE_SYNC.md)
- [Mobile Architecture](documentation/MOBILE_ARCHITECTURE.md)
- [Security](documentation/SECURITY.md)
- [Testing](documentation/TESTING.md)
- [Technical Debt](documentation/TECHNICAL_DEBT.md)
- [Changelog](documentation/CHANGELOG.md)
- [Guard Codebase Map](documentation/GUARD_CODEBASE_MAP.md)
- [Guard Integration Guide](documentation/GUARD_INTEGRATION_GUIDE.md)

## Project boundaries

The standalone Guard application is the validated reference implementation.

Future integration into the larger host Android application is intentionally owned by the host application's maintainer. The current project does not perform speculative module extraction, navigation changes, DI changes, or host database changes before that integration work begins.

## Development state

The validated Guard MVP remains the functional reference baseline. New changes should be introduced through dedicated work packages and validated before being treated as accepted project behavior.

---

**AVAX ALPR** — practical, offline-first license plate access control for real gate operation.

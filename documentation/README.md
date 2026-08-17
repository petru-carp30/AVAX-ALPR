# AVAX ALPR Documentation

Official technical documentation for the AVAX ALPR system.

**Documentation status date:** 2026-08-17

## Confirmed project state

The following vertical slices are implemented and tested:

- Backend vehicle read API:
  - `GET /api/vehicles`
  - `GET /api/vehicles/by-plate/{plate}`
- Vehicle Snapshot Sync API v1:
  - `GET /api/sync/vehicles`
  - complete snapshot
  - `contractVersion = 1`
  - UTC generation timestamp
  - vehicle count
  - deterministic plate normalization
  - normalized-plate collision rejection with `409 Conflict`
- Guard Mobile offline verification:
  - Vehicle Snapshot Sync v1 client
  - transactional Room cache replacement
  - previous valid cache preserved when synchronization fails
  - manual normalized-plate lookup
  - local access evaluation for Parking Lot, Site, and Camp
  - offline operation after backend/network loss and application restart
  - physical Android-device end-to-end validation
  - Android 17 Local Network Protection support for development synchronization

The next confirmed work package is `MOB-WP-002 — Local Access Logging Foundation`, currently `TODO`.

CameraX, plate detection, OCR, AI integration, access-log upload, automatic background vehicle synchronization, Manager functionality, and production deployment are not confirmed as implemented.

## Documentation index

- [Architecture](ARCHITECTURE.md)
- [Roadmap](ROADMAP.md)
- [API Contract](API_CONTRACT.md)
- [Database](DATABASE.md)
- [Offline Synchronization](OFFLINE_SYNC.md)
- [Mobile Architecture](MOBILE_ARCHITECTURE.md)
- [Security](SECURITY.md)
- [Testing](TESTING.md)

`AI_PIPELINE.md` and `DEPLOYMENT.md` are intentionally not published yet because there is insufficient confirmed implementation to document them accurately.

## Documentation rules

- Document only confirmed implementation and test results as current behavior.
- Mark unconfirmed decisions and future functionality as `PROPOSED`.
- Do not mark work `DONE` without explicit implementation and test confirmation.
- Keep the system offline-first.
- The Guard App never connects directly to SQL Server.
- The Manager App never communicates directly with the Guard App.
- Server-side communication passes through the Backend API.

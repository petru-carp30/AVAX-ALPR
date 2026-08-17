# AVAX ALPR Testing

**Evidence status date:** 2026-08-17

Only confirmed test results are listed as passed.

## Backend testing

### Baseline and vulnerability

Confirmed:

- backend baseline build validation passed
- Microsoft.OpenApi vulnerability remediation validated

### Vehicle read API

Confirmed:

- vehicle list endpoint validated
- normalized plate lookup validated
- missing vehicle behavior validated
- invalid plate input behavior validated
- OpenAPI exposure validated
- read-only development SQLite access validated

### Vehicle Snapshot Sync API v1

Confirmed:

- `GET /api/sync/vehicles`
- `contractVersion = 1`
- UTC generation timestamp
- vehicle count
- complete mobile payload
- deterministic normalization
- normalized collision response `409 Conflict`
- sanitized database failures
- read-only behavior

The backend Sync v1 test suite passed `11/11`.

Reference implementation commit: `861ab991`.

## Guard Mobile automated testing

Confirmed green test areas:

- deterministic `PlateNormalizer`
- local `AccessChecker`
- Parking Lot, Site, and Camp evaluation
- validity-window results
- Room vehicle cache
- Room snapshot replacement
- synchronization contract-version validation
- declared vehicle-count validation
- normalized duplicate protection
- DTO-to-Room mapping
- previous-cache preservation after failed synchronization/import

Previously reported confirmed counts include:

- `RoomVehicleCacheTest`: `3/3`
- `SnapshotReplacementTest`: `2/2`

## Guard Mobile result-state validation

Confirmed states:

- `Granted`
- `Denied`
- `NotYetValid`
- `Expired`
- `VehicleNotFound`
- `InvalidInput`
- `DataUnavailable`

## Physical-device end-to-end validation

The following milestone was completed on a physical Android device:

```text
Backend Sync API
  -> Room
  -> Internet/backend unavailable
  -> Application restart
  -> Manual plate entry
  -> Local lookup
  -> Local access result
```

Confirmed during validation:

- cached data persisted across application restart
- normal lookup worked without backend/network availability
- access flags and validity windows produced the expected states
- failed synchronization preserved the earlier valid snapshot
- Android 17 Local Network Protection permission support allowed development LAN synchronization on API 37+

Reference mobile commits:

- `0d3732f3`
- `4eb09213`
- `046fc8ac`

## Not yet confirmed tested

- local access logging
- access-log synchronization
- automatic background vehicle synchronization
- CameraX
- detector and OCR accuracy
- on-device AI performance
- access-request approval workflow
- notifications
- authentication and authorization
- production deployment, recovery, and load behavior

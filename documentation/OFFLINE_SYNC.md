# AVAX ALPR Offline Synchronization

## Offline-first rule

Normal access verification uses the Guard App's local Room database. It does not perform an HTTP request for every vehicle and does not connect directly to SQL Server.

## Confirmed Sync Contract v1

Endpoint:

`GET /api/sync/vehicles`

Confirmed response characteristics:

- `contractVersion = 1`
- complete vehicle snapshot
- UTC snapshot generation timestamp
- vehicle count
- dedicated mobile vehicle payload
- deterministic normalized license plates
- `409 Conflict` when source rows collide after normalization

Full-snapshot synchronization is used because the audited SQL Server source has no reliable primary identity, uniqueness guarantee, change tracking, row version, or modification timestamp suitable for safe incremental synchronization.

The formal ADR for this strategy remains `PROPOSED` until explicitly accepted.

## Confirmed mobile synchronization flow

1. Request `GET /api/sync/vehicles`.
2. Validate the contract version.
3. Validate the declared vehicle count.
4. Normalize and validate vehicle plates.
5. reject duplicate normalized plates.
6. Map the transport payload to Room entities.
7. Replace vehicle data and synchronization metadata in one database transaction.
8. Commit only when the complete snapshot is valid.

If request, validation, mapping, or database replacement fails, the last valid snapshot remains available.

## Confirmed offline verification flow

```text
Backend Sync API
  -> Room transactional cache
  -> backend/network unavailable
  -> application restart
  -> manual plate entry
  -> local normalized lookup
  -> local access result
```

This flow was validated end-to-end on a physical Android device.

## Local access evaluation

The current mobile implementation evaluates:

- Parking Lot
- Site
- Camp
- the corresponding access flag
- area-specific start and end validity values
- current time

Confirmed result states:

- `Granted`
- `Denied`
- `NotYetValid`
- `Expired`
- `VehicleNotFound`
- `InvalidInput`
- `DataUnavailable`

## Android 17 local development synchronization

Android 17 Local Network Protection support is implemented and validated for synchronization with a backend on the local development network.

Confirmed permissions/behavior:

- `android.permission.INTERNET` remains present
- network-state access remains present
- `android.permission.ACCESS_LOCAL_NETWORK` is declared
- runtime local-network permission is requested on API 37+
- existing offline Room verification remains independent of that permission after a valid snapshot exists

## Current limitation

The synchronization control is currently manual and development-oriented.

Automatic periodic/background vehicle synchronization, scheduling policy, retry/backoff rules, and production transport configuration are future work and are not confirmed implemented.

# AVAX ALPR Guard Mobile Architecture

## Purpose

The Guard Mobile App provides offline-first vehicle access verification. The currently validated vertical slice uses manual plate entry; camera and AI recognition remain future work.

## Confirmed platform baseline

| Item | Confirmed value |
|---|---|
| Package | `com.avax.alpr.guard` |
| Language | Kotlin |
| UI | Jetpack Compose |
| Local persistence | Room / SQLite |
| Min SDK | 26 |
| Compile SDK | 37 |
| Target SDK | 37 |
| Android Gradle Plugin | 9.3.1 |
| Gradle Wrapper | 9.5.0 |
| Gradle daemon JVM | 25 |
| Java source/target | 11 |
| Compose BOM | 2026.02.01 |
| Room | 2.8.4 |
| KSP | 2.3.11 |

AGP 9 built-in Kotlin is used; an external Kotlin Android plugin is not part of the confirmed baseline.

## Confirmed layering

The implementation uses a simple maintainable separation around:

- network synchronization
- Room cache
- domain models and access evaluation
- Compose guard UI

Confirmed model concepts include:

- `AccessArea`
- `AccessDecisionStatus`
- `ValidityWindow`
- `VehicleRecord`

Confirmed data concepts include:

- vehicle Room entity
- synchronization metadata entity
- DAOs
- Guard database
- vehicle cache store
- Vehicle Snapshot Sync v1 API client
- DTO-to-Room mapping

## Confirmed verification behavior

1. The guard enters a license plate.
2. The app performs deterministic normalization.
3. The app looks up the normalized plate in Room.
4. The app evaluates the selected Parking Lot, Site, or Camp access locally.
5. The app displays one of the confirmed result states:
   - `Granted`
   - `Denied`
   - `NotYetValid`
   - `Expired`
   - `VehicleNotFound`
   - `InvalidInput`
   - `DataUnavailable`

The access result is derived from cached business data. AI does not decide access.

## Cache behavior

- Snapshot import is transactional.
- Vehicle data and synchronization metadata are replaced consistently.
- A failed import preserves the previous valid snapshot.
- Cached verification remains available without the backend and after app restart.

## Network behavior

Network access is used to refresh the local snapshot, not to verify every vehicle.

Android 17 local-development synchronization includes explicit local-network permission support on API 37+.

## Not confirmed implemented

- CameraX
- image-frame analysis
- plate detector
- OCR
- on-device AI model integration
- local access logging
- access-log upload
- automatic background vehicle synchronization
- access requests and Manager approval
- push notifications
- Manager/Admin UI

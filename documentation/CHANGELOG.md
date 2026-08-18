# AVAX ALPR Changelog

This changelog records only functionality, fixes, security remediations, and technical changes explicitly confirmed as implemented and tested by the project Master.

Version numbers, releases, release names, and release dates are not recorded unless explicitly confirmed by the project Master.

## Confirmed Implementation History

### Added

#### Backend Foundation

- Validated the ASP.NET Core backend baseline on `.NET 10.0`.
- Added local SQLite development database integration.
- Implemented read-only vehicle data access.
- Implemented asynchronous parameterized vehicle repository reads.
- Implemented vehicle mapping and clean `VehicleDto` output.
- Implemented and tested `GET /api/vehicles`.
- Implemented and tested `GET /api/vehicles/by-plate/{plate}`.
- Implemented and tested deterministic license plate normalization.
- Validated `404 Not Found` behavior and invalid license plate input handling.
- Validated OpenAPI generation.

#### Vehicle Snapshot Sync v1

- Implemented `GET /api/sync/vehicles`.
- Implemented the full vehicle snapshot contract.
- Added `contractVersion = 1`.
- Added UTC snapshot generation metadata.
- Added vehicle count metadata.
- Added a dedicated mobile synchronization payload.
- Implemented deterministic plate normalization for the synchronization contract.
- Implemented normalized plate collision protection with `409 Conflict`.
- Implemented sanitized database failure responses.
- Kept synchronization operations read-only.
- Confirmed `11/11` automated integration tests passing.

#### Guard Mobile — MOB-WP-001: Offline Vehicle Cache & Manual Access Verification

- Implemented a Room-based offline vehicle cache.
- Implemented the Vehicle Snapshot Sync v1 client.
- Implemented transactional snapshot replacement.
- Preserved the previous valid local snapshot when synchronization fails.
- Implemented synchronization contract validation.
- Implemented deterministic local license plate normalization.
- Implemented manual license plate lookup using the local Room cache.
- Implemented local access evaluation for Parking Lot, Site, and Camp.
- Implemented access validity-window evaluation.
- Implemented the following access decision states:
  - `Granted`
  - `Denied`
  - `NotYetValid`
  - `Expired`
  - `VehicleNotFound`
  - `InvalidInput`
  - `DataUnavailable`
- Validated offline access verification after backend/network loss and application restart.
- Completed end-to-end validation on a physical Android device.

### Fixed

#### Guard Mobile — Android 17 Local Network Protection

- Implemented and validated Android 17 Local Network Protection support required for development synchronization with a backend on the local network.

### Security

- Resolved the `NU1903` vulnerability caused by the transitive `Microsoft.OpenApi 2.0.0` dependency.
- Added an explicit `Microsoft.OpenApi 2.7.5` package reference.
- Confirmed vulnerability scans report no known vulnerable packages in the backend project or test project.

### Reference Commits

#### Backend

- Security remediation: `6b9e61e`
- Vehicle Snapshot Sync API v1: `861ab991`

#### Guard Mobile — MOB-WP-001

- `0d3732f3`
- `4eb09213`
- `046fc8ac`

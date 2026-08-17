# AVAX ALPR API Contract

## Scope

This document records only backend API behavior confirmed as implemented and tested. The current backend project is `Avax.ALPR.Api`.

## Confirmed endpoints

### `GET /api/vehicles`

Returns vehicle records from the configured development data source.

Confirmed behavior:

- asynchronous repository read
- vehicle DTO response
- backend data access remains read-only

### `GET /api/vehicles/by-plate/{plate}`

Returns a vehicle using deterministic normalized-license-plate lookup.

Confirmed behavior:

- input plate normalization
- invalid-input handling
- `404 Not Found` when no vehicle matches
- backend data access remains read-only

### `GET /api/sync/vehicles`

Returns the complete vehicle snapshot used to replace the Guard App Room cache.

Confirmed Sync v1 envelope semantics:

| Field or concept | Confirmed behavior |
|---|---|
| Contract version | `contractVersion = 1` |
| Snapshot time | UTC generation timestamp |
| Count | vehicle count included in the response |
| Vehicle data | dedicated mobile vehicle payload |
| Snapshot type | complete snapshot, not an incremental delta |
| Data access | read-only |
| Plate identity | deterministic normalized plate |
| Collision behavior | `409 Conflict` if two source rows collide after normalization |

The payload contains the vehicle and access information required for local verification, including the three explicit access areas:

- Parking Lot
- Site
- Camp

The source fields include access flags and area-specific start/end validity values. This document intentionally does not invent field names beyond those confirmed by the implemented contract.

## Client validation

The confirmed Guard Mobile client validates:

- `contractVersion`
- declared vehicle count
- normalized plate values
- duplicate normalized plates
- DTO-to-Room mapping before accepting the snapshot

A validation or import failure must preserve the previous valid local snapshot.

## Failure semantics

Confirmed behavior includes:

- invalid plate input is rejected by the vehicle lookup flow
- missing vehicle lookup returns `404 Not Found`
- normalized snapshot collisions return `409 Conflict`
- database failures are sanitized and do not expose raw internal database details
- an unsuccessful mobile synchronization does not erase the previous Room cache

## Versioning rule

Any incompatible synchronization payload change requires a new contract version and coordinated updates to:

- Backend & Database
- Guard Mobile App
- testing
- official API and offline-sync documentation

## Not confirmed

The following contracts are not yet documented as implemented:

- authentication and authorization endpoints
- access-log upload
- access-request and Manager approval workflow
- notification delivery
- incremental vehicle synchronization

# AVAX ALPR Database

## Scope and data ownership

AVAX ALPR integrates with an existing SQL Server database. The production schema is authoritative and must not be recreated or changed without a justified and explicitly accepted database decision.

Detailed production server identifiers, full column inventories, and row-level data are intentionally excluded from this public document.

## Relevant logical data

The backend reads existing business records for:

- vehicles and license plates
- people
- company or organizational structures
- access permission flags
- area-specific validity windows
- vehicle descriptive information and access notes

The three confirmed access areas are:

- Parking Lot
- Site
- Camp

The backend maps these records into API-specific DTOs. Mobile clients must depend on the versioned API contract, not on the physical SQL Server schema.

## Confirmed synchronization constraints

The production schema audit did not identify a reliable source mechanism for safe incremental vehicle synchronization.

Relevant confirmed limitations include:

- no enforced vehicle identity suitable for an incremental mobile contract
- no database-enforced unique normalized license plate
- no reliable change-tracking or modification marker for incremental synchronization

Current mitigations:

| Limitation | Current mitigation |
|---|---|
| No reliable incremental identity | Sync Contract v1 transfers a complete snapshot |
| Plate uniqueness not enforced at source | Backend normalization detects collisions and rejects the snapshot with `409 Conflict` |
| No reliable incremental change marker | Guard Mobile replaces the complete Room snapshot transactionally |

Permanent schema resolutions remain `PROPOSED` until explicitly accepted.

## Development data source

A local SQLite development/test data source was established for backend development.

Confirmed characteristics:

- it represents the relevant vehicle, people, and organizational data structures
- it is used only for development and testing
- it does not introduce artificial guarantees that would misrepresent production
- current backend vehicle read and synchronization tests use this local data source
- its filename is not a public or stable application contract

## Guard Mobile Room database

The Guard App owns a separate Room/SQLite offline cache. It does not open the backend development database and never connects directly to SQL Server.

Confirmed mobile persistence includes:

- vehicle entities
- synchronization metadata
- DAO-based normalized-plate lookup
- transactional complete-snapshot replacement
- preservation of the previous valid snapshot when import fails

Room is a replaceable offline projection of the backend Sync v1 contract, not the system of record.

## Change-management rules

- Physical production-schema changes belong to Backend & Database work.
- Mobile schema changes must preserve offline upgrade and cache-safety requirements.
- API DTOs isolate mobile code from physical database changes.
- Contract-breaking changes require a new synchronization contract version and coordinated backend/mobile tests.

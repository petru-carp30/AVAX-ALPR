# AVAX ALPR Security

## Scope

This document separates confirmed security behavior from requirements that are not yet implemented or validated.

## Confirmed controls and boundaries

### Data access

- The Guard App never connects directly to SQL Server.
- Server-side data access passes through the ASP.NET Core Backend API.
- Current backend vehicle and snapshot operations are read-only.
- The existing production database is not recreated or modified by the mobile synchronization flow.

### Offline integrity

- The Guard App validates the synchronization contract version.
- It validates the declared vehicle count.
- It validates normalized plates and rejects duplicate normalized plates.
- Snapshot replacement is transactional.
- A failed synchronization preserves the previous known-valid local snapshot.
- The backend rejects normalized-plate collisions with `409 Conflict`.

These controls prevent a partial or ambiguous snapshot from replacing valid offline data.

### Error handling

Database failures are sanitized by the backend so raw internal database details are not exposed to API clients.

### Android network permissions

For Android 17 local-development synchronization:

- `INTERNET` access remains declared
- network-state access remains declared
- `ACCESS_LOCAL_NETWORK` is declared
- local-network runtime permission is requested on API 37+

The local-network permission is required for development backend access on the LAN. Existing Room data continues to support offline verification without network access.

### Dependency remediation

The confirmed `Microsoft.OpenApi` vulnerability was resolved by explicitly using `Microsoft.OpenApi 2.7.5` alongside `Microsoft.AspNetCore.OpenApi 10.0.10`.

## Not confirmed for production

The following items are required before production but are not confirmed implemented:

- user authentication
- role-based authorization
- device enrollment or trust
- production TLS and certificate policy
- secure secret provisioning
- encryption-at-rest policy for mobile cached data
- audit-log retention and protection
- access-request authorization
- Manager approval authorization
- push-notification security
- abuse/rate-limit controls
- production privacy and data-retention rules
- production monitoring and incident response

Any chosen solution for these items remains `PROPOSED` until explicitly accepted, implemented, and tested.

## Security invariants

- AI output must never directly grant or deny access.
- The backend remains the server-side trust boundary.
- The Manager App must not communicate directly with the Guard App.
- Snapshot validation must complete before cached data is replaced.
- Internal exceptions and connection details must not be returned to clients.

# AVAX ALPR Bugs and Vulnerabilities

**Evidence status date:** 2026-08-18

This document records confirmed bugs and vulnerabilities affecting AVAX ALPR.

An issue is marked `RESOLVED / VALIDATED` only when both remediation implementation and validation are confirmed. A proposed fix alone is not sufficient.

## Resolved and validated issues

### SEC-001 — Microsoft.OpenApi NU1903 vulnerability

- **Type:** Vulnerability
- **Severity:** High
- **Component:** AVAX ALPR Backend & Database / .NET dependencies
- **Status:** RESOLVED / VALIDATED
- **Cause:** The backend dependency graph included transitive `Microsoft.OpenApi 2.0.0`, reported through `NU1903`.
- **Mitigation:** No separate temporary mitigation was recorded before remediation.
- **Resolution:** Added an explicit `Microsoft.OpenApi 2.7.5` package reference alongside `Microsoft.AspNetCore.OpenApi 10.0.10`.
- **Validation:** Vulnerability scanning was confirmed to report no known vulnerable packages in the backend project or test project after remediation.
- **Reference commit:** `6b9e61e`

### Android 17 Local Network Protection

- **Type:** Bug / Platform compatibility
- **Severity:** Not assigned in the confirmed update
- **Component:** AVAX ALPR Guard Mobile App
- **Status:** RESOLVED / VALIDATED
- **Cause:** On Android 17 / API 37+, local network access requires explicit Local Network Protection permission behavior. This blocked physical-device synchronization with the local development backend.
- **Mitigation:** Existing `INTERNET`, `ACCESS_NETWORK_STATE`, and debug cleartext HTTP configuration were preserved while the permission fix was added.
- **Resolution:** Added `android.permission.ACCESS_LOCAL_NETWORK` and implemented the runtime permission request on API 37+.
- **Validation:** Physical Android-device synchronization with the local development backend succeeds after the change.
- **Build baseline impact:** No locked Android build baseline changes were required.

## Open bugs and vulnerabilities

No additional open bug or vulnerability is recorded here from the confirmed history available for this initial backfill.

## Status rules

- A possible or proposed fix does not change an issue to `RESOLVED`.
- `RESOLVED / VALIDATED` requires confirmed implementation and confirmed validation.
- If validation later fails, the issue must be reopened.
- Unconfirmed causes, severities, or remediation details must not be invented.

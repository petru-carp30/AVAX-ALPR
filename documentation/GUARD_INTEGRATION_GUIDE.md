# AVAX ALPR Guard — Integration Guide

**Status:** PROPOSED integration plan based on the accepted Guard MVP baseline.  
**Current Guard baseline:** `petru-carp30/Avax.ALPR.Guard` `master` at `984d99a9cd3ceb0bd04d41ea8063db4de7954a71`.

## Goal

Prepare the validated AVAX ALPR Guard capability so it can be integrated into another Android application without losing the accepted offline-first behavior or forcing the host application to understand internal detector/OCR/storage details.

This guide does not authorize an integration refactor yet. It defines the recommended boundary and migration order.

## Current situation

The Guard project is currently a single Gradle `:app` module. The internal packages are separated by responsibility, but the feature is still owned as a standalone application through:

- `GuardApplication`
- `AppContainer`
- `MainActivity`
- `GuardViewModel`
- `GuardScreen`

The current host wiring creates Room, Retrofit APIs, repositories, WorkManager scheduling, and the Guard UI directly inside the Guard application lifecycle.

For another application to consume this capability cleanly, integration should happen through a small public feature boundary rather than by directly calling DAOs, detector classes, OCR processors, or `GuardViewModel` internals.

## Recommended integration strategy

### Phase A — Preserve the validated baseline

Before refactoring:

1. tag or otherwise preserve the accepted Guard MVP baseline;
2. keep `984d99a9cd3ceb0bd04d41ea8063db4de7954a71` as the reference behavior;
3. retain the current physical acceptance evidence;
4. run the same unit/debug/release build gates after every integration-oriented refactor.

Do not combine integration refactoring with detector/OCR/access-rule changes.

### Phase B — Introduce a host-facing feature boundary

PROPOSED public surface:

```kotlin
interface AlprGuardFeature {
    suspend fun synchronizeVehicles(): VehicleSyncOutcome
    suspend fun verifyPlate(
        plate: String,
        area: AccessArea
    ): AccessVerificationOutcome

    fun schedulePendingLogUpload()
}
```

For UI integration, expose a Compose entry point instead of requiring the host to instantiate `GuardViewModel` directly:

```kotlin
@Composable
fun AlprGuardScreen(
    config: AlprGuardConfig,
    onClose: () -> Unit = {}
)
```

The exact names and signatures are **PROPOSED**. They must be finalized only when the host application's navigation, DI and configuration model are known.

### Phase C — Move implementation behind that boundary

The host-facing API should delegate to the existing accepted internals:

```text
Host App
-> AlprGuardFeature / AlprGuardScreen
-> internal Guard composition
-> Camera / AI / Room / AccessChecker / Logs / Sync
```

The host app should not need to know about:

- `VehicleDao`
- `AccessLogDao`
- `OnnxPlateDetector`
- `AutomaticPlateOcrProcessor`
- `MlKitPlateOcr`
- `AccessLogUploadWorker`
- internal DTOs
- Room migration internals

Those remain implementation details of the ALPR feature.

## Proposed module boundary

The safest long-term target is a reusable Android library module, for example:

```text
:alpr-guard
```

with the host application consuming it:

```text
:app
  -> implementation(project(":alpr-guard"))
```

Inside `:alpr-guard`, keep the existing package responsibilities:

```text
alpr-guard
├── ai
│   ├── detector
│   └── ocr
├── camera
├── domain
├── data
│   ├── local
│   ├── network
│   └── repository
├── background
├── ui
└── integration
```

The new `integration` package would contain only the host-facing contracts/factories/configuration.

This module split is **PROPOSED** and should be done only after reviewing the target application's existing Gradle structure and dependency injection approach.

## Configuration that should move out of standalone app ownership

The host application should provide configuration rather than the ALPR feature reading assumptions from a standalone application shell.

PROPOSED configuration object:

```kotlin
data class AlprGuardConfig(
    val apiBaseUrl: String,
    val defaultAccessArea: AccessArea? = null
)
```

Potential future host-supplied values:

- API base URL;
- authentication/token provider once production auth is defined;
- selected/default access area;
- feature enablement flags;
- logging/diagnostic policy;
- environment identifier.

Do not hard-code host navigation or host authentication into detector/OCR/domain packages.

## Ownership boundaries after integration

### Host application should own

- top-level navigation;
- authenticated user/session if the host already has one;
- application-wide theme if desired;
- production environment selection;
- permission routing at the application level where appropriate;
- opening/closing the ALPR feature.

### ALPR feature should continue to own

- CameraX scanning lifecycle while the ALPR screen is active;
- detector model/runtime;
- OCR and quality/confirmation gates;
- plate normalization;
- offline Room vehicle cache unless an explicit shared-data architecture is approved;
- `AccessChecker` behavior;
- durable local access logs;
- WorkManager access-log synchronization;
- ALPR-specific diagnostics.

## Database integration decision

Current Guard uses its own Room database through `GuardDatabase`.

Recommended first integration approach:

**Keep the ALPR Room database isolated.**

Reasons:

- already validated offline-first;
- avoids coupling host schema migrations to ALPR migrations;
- minimizes risk during first integration;
- easier rollback and testing.

Only merge databases if there is a concrete requirement for shared transactions/data ownership. A shared database would require a separate architecture decision and migration plan.

## WorkManager integration

Current access-log recovery is started from `GuardApplication` through `AccessLogUploadScheduler.ensureRecoveryWork()`.

When embedded in another application, this application-owned behavior must be moved behind an explicit initialization boundary.

PROPOSED pattern:

```kotlin
class AlprGuardInitializer(
    private val scheduler: AccessLogUploadScheduler
) {
    fun initialize() {
        scheduler.ensureRecoveryWork()
    }
}
```

The host calls it from its own `Application`/DI startup.

Do not require the host to subclass `GuardApplication`.

## UI integration

The current standalone path is:

```text
MainActivity
-> GuardViewModel
-> GuardScreen
```

Recommended embedded path:

```text
Host Navigation
-> AlprGuardScreen
-> internally owned GuardViewModel / feature state
```

The host should not depend on `GuardUiState` details unless there is a deliberate shared UI contract.

If the host application already has a design system, visual theming may be adapted later, but access colors/meanings and safety behavior must remain unchanged unless explicitly approved.

## Camera permission integration

The host application may already own permission flows.

Integration must preserve:

- CAMERA permission;
- INTERNET;
- ACCESS_NETWORK_STATE;
- Android 17 `ACCESS_LOCAL_NETWORK` where local development/server access requires it.

The reusable feature should expose permission requirements clearly and avoid silently assuming that `MainActivity` owns all permission UX.

## Model/assets integration

The ONNX detector asset remains part of the ALPR feature implementation.

Do not make the host application manually interpret detector tensor contracts.

The feature must continue to validate the detector artifact expected by the accepted runtime contract.

ML Kit OCR remains a feature dependency:

```text
com.google.mlkit:text-recognition:16.0.1
```

## Backend boundary

The host application must not replace offline local verification with direct server verification.

Accepted normal decision path remains:

```text
confirmed plate
-> local Room lookup
-> AccessChecker
-> result
```

Backend is used for:

- vehicle snapshot synchronization;
- access-log upload;
- future server-side features.

The Guard/host application still must not connect directly to SQL Server.

## Integration migration order

Recommended order:

1. **Document/freeze current baseline** — DONE for MVP behavior.
2. **Inspect host application architecture** — Gradle modules, navigation, DI, auth, database, theme.
3. **Define host-facing integration contract** — configuration + entry point + lifecycle responsibilities.
4. **Create reusable module boundary** or equivalent feature package boundary.
5. **Move `AppContainer` responsibilities behind a feature factory/container** without changing behavior.
6. **Move `GuardApplication` recovery startup into explicit feature initialization.**
7. **Expose Compose screen through the integration boundary.**
8. **Integrate host navigation and permission handling.**
9. **Run regression tests/builds.**
10. **Repeat physical A–N acceptance subset focused on integration regressions.**

## Minimum regression gates after integration

Required at minimum:

```text
./gradlew testDebugUnitTest
./gradlew assembleDebug
./gradlew assembleRelease
```

Physical checks:

- camera opens from host application;
- zoom/focus works;
- detector works;
- OCR 2-of-3 works;
- Granted / Denied / Unknown paths work;
- low-quality crop still blocks authoritative decision/logging;
- manual fallback works;
- offline decision works;
- pending log survives restart;
- pending log later syncs;
- no duplicate access-log spam;
- host navigation away/back does not leave camera resources bound or scanner state corrupted.

## Changes that require extra care

### Changing package names

May affect:

- Android manifest references;
- Room schema/migrations;
- WorkManager worker class names if serialized/referenced by framework state;
- tests;
- asset/resource paths.

Avoid mass package renames during first integration unless necessary.

### Changing Room database ownership/name

Treat as data migration work. Do not silently create a second empty database and lose the cached vehicle snapshot/pending logs.

### Changing WorkManager initialization

Must retain recovery of pending access logs after process/application restart.

### Reusing host Retrofit/OkHttp

Possible, but only through an explicit contract. Preserve current endpoint semantics and offline behavior.

### Reusing host dependency injection

Recommended if the host already uses Hilt/Koin/manual DI, but adapt the composition root, not the domain/AI internals.

## Recommended integration task

When the target host application is available, create a dedicated work package:

```text
MOB-INT-WP-001 — Extract Guard ALPR as Host-Integrable Android Feature
```

Do not begin implementation until the host application's repository/architecture has been reviewed. The extraction contract depends on that architecture.

## Definition of a successful integration

Integration is successful when the host application can open the ALPR Guard feature and the feature behaves like the accepted standalone MVP without the host directly depending on internal AI, Room, or repository implementation classes.

The integration should make future ALPR changes local and understandable rather than spreading AVAX ALPR internals throughout the host application.

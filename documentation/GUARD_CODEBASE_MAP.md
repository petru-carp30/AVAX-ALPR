# AVAX ALPR Guard — Codebase Map

**Baseline reviewed:** `petru-carp30/Avax.ALPR.Guard` `master` at `984d99a9cd3ceb0bd04d41ea8063db4de7954a71`  
**Purpose:** make the Guard application faster to read, change, debug, and later integrate into another Android application.

This document describes the current codebase. It does not change architecture or behavior.

## Runtime flow

```text
CameraX
-> camera.FrameAnalyzer
-> YOLOX ONNX detector
-> original-frame plate crop
-> native crop quality gate
-> ML Kit OCR
-> 2-of-3 OCR confirmation
-> PlateNormalizer
-> Room local lookup
-> AccessChecker
-> access result
-> local access log
-> WorkManager upload
-> Backend API
```

The normal access decision is offline-first. AI produces plate information; `AccessChecker` remains responsible for the access decision.

## Top-level package map

| Path | Responsibility | Start reading here |
|---|---|---|
| `com.avax.alpr.guard` | Application composition and Android entry point | `AppContainer.kt`, `GuardApplication.kt`, `MainActivity.kt` |
| `ai/detector` | YOLOX ONNX detector, preprocessing, inference and postprocessing | `PlateDetector.kt`, `OnnxPlateDetector.kt`, `DetectorModelConfig.kt` |
| `ai/ocr` | Crop, crop-quality gate, ML Kit OCR and temporal confirmation | `AutomaticPlateOcrProcessor.kt`, `PlateCropQualityGate.kt`, `AutomaticOcrConfirmationGate.kt` |
| `camera` | CameraX session, frame transport, zoom and focus helpers | `CameraSession.kt`, `CameraFrameAnalyzer.kt`, `CameraFocusAssist.kt`, `CameraZoomMath.kt` |
| `domain` | Business rules that should remain independent from UI/network | `PlateNormalizer.kt`, `AccessChecker.kt`, `AutomaticScanRearmGate.kt` |
| `data/local` | Room database, DAOs, entities, migrations and local stores | `GuardDatabase.kt`, `VehicleDao.kt`, `AccessLogDao.kt` |
| `data/network` | Retrofit/API boundary and connectivity abstraction | `VehicleSyncApi.kt`, `AccessLogApi.kt`, `NetworkClientFactory.kt` |
| `data/repository` | Orchestration between local storage, domain rules and network | `VehicleAccessRepository.kt`, `VehicleSyncRepository.kt`, `AccessLogUploadRepository.kt` |
| `background` | WorkManager scheduling and pending access-log upload | `AccessLogUploadScheduler.kt`, `AccessLogUploadWorker.kt` |
| `ui/guard` | Guard screen state, ViewModel and Compose screen | `GuardViewModel.kt`, `GuardUiState.kt`, `GuardScreen.kt` |

## Composition root

`AppContainer.kt` is the current manual dependency-composition root.

It creates and wires:

```text
GuardDatabase
-> VehicleCacheStore / AccessLogStore

BuildConfig.API_BASE_URL
-> VehicleSyncApi / AccessLogApi

VehicleSyncApi + Room
-> VehicleSyncRepository

Room + AccessChecker + AccessLogStore + WorkManager scheduler
-> VehicleAccessRepository

AccessLogApi + Room
-> AccessLogUploadRepository
```

`GuardApplication.kt` owns the `AppContainer` and ensures recovery work for pending access-log uploads.

`MainActivity.kt` retrieves the container from `GuardApplication`, creates `GuardViewModel.Factory`, observes `GuardUiState`, and renders `GuardScreen`.

This is the first place to inspect when moving the feature into another host application.

## Where to change what

| Change needed | Primary files / package | Notes |
|---|---|---|
| Detector model input/threshold/runtime contract | `ai/detector/DetectorModelConfig.kt`, `OnnxPlateDetector.kt` | Do not change frozen MVP values without a new approved AI/mobile task. |
| Detector preprocessing | `ai/detector/PlateDetectorPreprocessor.kt` | Camera-frame orientation and detector tensor creation live here. |
| Detector postprocessing/NMS | `ai/detector/PlateDetectorPostprocessor.kt` | Keep semantic output as plate bounds/confidence only. |
| OCR engine | `ai/ocr/MlKitPlateOcr.kt`, `PlateOcr.kt` | OCR must not decide access. |
| OCR crop rules | `ai/ocr/PlateCropper.kt`, `PlateCropQualityGate.kt` | Native crop height is evaluated before OCR resize. |
| 2-of-3 confirmation | `ai/ocr/AutomaticOcrConfirmationGate.kt` | Reliability gate before authoritative automatic verification. |
| Automatic OCR orchestration | `ai/ocr/AutomaticPlateOcrProcessor.kt` | Connects detection, crop, OCR and confirmation. |
| Plate normalization | `domain/PlateNormalizer.kt` | Shared by automatic and manual verification paths. |
| Access rules | `domain/AccessChecker.kt` | This is the local authority for access decisions. |
| Scene re-arm | `domain/AutomaticScanRearmGate.kt` | Separate from OCR confirmation and duplicate cooldown. |
| Camera lifecycle | `camera/CameraSession.kt` | Preview + ImageAnalysis binding. |
| Zoom behavior | `camera/CameraZoomMath.kt`, camera UI integration | Uses CameraX zoom; do not replace with fake crop zoom. |
| Focus behavior | `camera/CameraFocusAssist.kt` | CameraX AF/AE focus/metering boundary. |
| Vehicle cache schema/access | `data/local/VehicleEntity.kt`, `VehicleDao.kt`, `VehicleCacheStore.kt` | Preserve offline-first behavior. |
| Access-log persistence | `data/local/AccessLogEntity.kt`, `AccessLogDao.kt`, `AccessLogStore.kt` | Local durable log exists before upload. |
| Room schema/migrations | `data/local/GuardDatabase.kt`, `GuardDatabaseMigrations.kt` | Any schema change needs migration coverage. |
| Vehicle snapshot API | `data/network/VehicleSyncApi.kt` + DTOs | Backend contract boundary. |
| Access-log API | `data/network/AccessLogApi.kt` + DTOs | Backend contract boundary. |
| Vehicle sync behavior | `data/repository/VehicleSyncRepository.kt`, `SnapshotValidator.kt` | Full snapshot v1; validate before replacement. |
| Local access verification orchestration | `data/repository/VehicleAccessRepository.kt` | Combines Room lookup, AccessChecker and local logging. |
| Pending log upload | `data/repository/AccessLogUploadRepository.kt`, `background/*` | WorkManager owns retry/recovery path. |
| Guard screen behavior | `ui/guard/GuardViewModel.kt`, `GuardUiState.kt`, `GuardScreen.kt` | UI layer; do not move access rules here. |
| Application wiring | `AppContainer.kt`, `GuardApplication.kt`, `MainActivity.kt` | Main integration hotspot for another host app. |

## Automatic scan path

```text
CameraSession
-> CameraFrameAnalyzer
-> PlateDetectorFrameProcessor
-> OnnxPlateDetector
-> PlateDetection[]
-> AutomaticPlateOcrProcessor
-> PlateCropper
-> PlateCropQualityGate
-> MlKitPlateOcr
-> AutomaticOcrConfirmationGate
-> GuardViewModel automatic-recognition callback
-> VehicleAccessRepository
-> PlateNormalizer / Room / AccessChecker
-> AccessLogStore
-> AccessLogUploadScheduler
```

The detector/OCR path must not query the vehicle database to alter recognition output.

## Manual verification path

```text
GuardScreen manual input
-> GuardViewModel
-> VehicleAccessRepository
-> PlateNormalizer
-> Room vehicle lookup
-> AccessChecker
-> local access log
-> background upload scheduling
```

Manual verification is the operational fallback and should remain available after integration.

## Offline data path

Vehicle data:

```text
Backend snapshot API
-> VehicleSyncRepository
-> SnapshotValidator
-> transactional Room replacement
-> VehicleDao
```

Access events:

```text
Access decision
-> AccessLogStore
-> Room access_logs (PENDING)
-> AccessLogUploadScheduler / Worker
-> AccessLogUploadRepository
-> Backend API
-> local state SYNCED / CONFLICT / REJECTED as applicable
```

The host application must not bypass Room and call the backend for every plate.

## Safe reading order for a new developer

1. `documentation/GUARD_INTEGRATION_GUIDE.md`
2. `AppContainer.kt`
3. `MainActivity.kt`
4. `ui/guard/GuardViewModel.kt`
5. `data/repository/VehicleAccessRepository.kt`
6. `domain/AccessChecker.kt` and `PlateNormalizer.kt`
7. `ui/camera` / `camera` plus `ai/detector`
8. `ai/ocr`
9. `data/local`, `data/network`, `background`

## Modification rules

- Keep `domain` free from UI and Retrofit/Room implementation details where practical.
- Keep AI responsible only for detection/recognition quality, never access authorization.
- Keep the normal access decision offline-first.
- Keep local access logging durable before upload.
- Treat API DTOs, Room schema, detector/OCR contracts, and host-integration interfaces as change boundaries.
- When changing a boundary, update all affected components and tests together.
- Avoid unrelated cleanup while fixing a production defect; make integration/refactor work a separate reviewed change.

## Integration hotspot summary

The current project is a single Gradle `:app` module. The business/data/AI/camera packages are reasonably separated, but application ownership is still centralized in `GuardApplication`, `AppContainer`, `MainActivity`, and `GuardViewModel`.

For integration into another application, the lowest-risk approach is to preserve the validated internals and first create a stable host-facing boundary around those four ownership points instead of copying individual classes into the host project.

See `GUARD_INTEGRATION_GUIDE.md` for the proposed migration approach.

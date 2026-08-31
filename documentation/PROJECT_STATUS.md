# AVAX ALPR Project Status

**Status snapshot:** 2026-08-31  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, licensing, and changelog information is maintained in the dedicated documentation files.

## Current AI milestone

`AI-DATA-WP-001 — License Plate Detector Dataset Acquisition & Annotation Foundation`

- Priority: `P0 Critical`
- Status: `DONE`
- Target project: AVAX ALPR – AI Model

Master accepted the materialized canonical detector dataset baseline:

`AI/PlateDetector/datasets/derived/baseline_v1`

The dataset foundation is accepted for the first detector baseline.

**READY TO RESUME AI-WP-001**

`AI-WP-001 — License Plate Detector Baseline & Mobile Export Contract`

- Priority: `P0 Critical`
- Status: `TODO / READY TO RESUME`
- Target project: AVAX ALPR – AI Model
- Dataset blocker: CLEARED

Detector training may now begin against the accepted `baseline_v1` dataset. Model architecture/runtime/export choices that were previously only `PROPOSED` remain to be validated by AI-WP-001 and are not automatically accepted by this dataset decision.

## Accepted canonical detector dataset — baseline_v1

Materialized baseline:

- total images: `8916`
- positive images: `8412`
- real audited negatives: `504`
- license-plate instances: `11580`

Splits:

- TRAIN: `7622`
- VAL: `625`
- TEST: `669`

Validation confirmed:

- image/label pairing complete
- YOLO normalized XYWH labels valid
- negative label files empty
- excluded samples not materialized
- raw source datasets unchanged
- source-group leakage: `0`
- exact decoded-pixel leakage: `0`
- unresolved near-duplicate candidate pairs: `0`
- TRAIN_ONLY samples outside TRAIN: `0`
- synthetic samples in VAL: `0`
- synthetic samples in TEST: `0`

## Training composition

TRAIN composition:

- REAL: `5717`
- SYNTHETIC: `1905`
- synthetic ratio: `24.9934%`

First-baseline policy result:

- REAL >= 75%: PASS
- SYNTHETIC <= 25%: PASS

VAL and TEST contain real samples only.

## Accepted source composition

### Romanian public license-plate dataset

Role: **PRIMARY ROMANIAN / EU REAL-DOMAIN SOURCE**

Raw:

- 534 images
- 652 plate instances

Excluded:

- 2 all-black duplicate/conflicting frames

Final eligible:

- 532 images
- 649 plate instances

Canonical sequence assignment:

- `dayride_type1_001.mp4` -> TRAIN: 384 images
- `dayride_type1_003.mp4` -> TRAIN: 28 images
- `dayride_type1_002.mp4` -> VAL: 38 images
- `nightride_type3_001.mp4` -> TEST: 82 images

No Romanian source sequence crosses AVAX splits.

### Open Images derived detector source

Role: **ADDITIONAL REAL-WORLD DETECTOR SOURCE**

Included:

- 5368 images
- 7852 plate instances

Canonical split:

- TRAIN: 4294 images / 6224 instances
- VAL: 537 images / 812 instances
- TEST: 537 images / 816 instances

All audited ImageIDs retain upstream provenance metadata including license, original URLs, author and author-profile fields.

The Kaggle wrapper license is not used to replace upstream image-level attribution.

192 tiny source bounding-box overshoots are corrected only in derived canonical annotations through deterministic clipping to valid normalized bounds. Raw labels remain untouched.

### Kaggle `plate_license_recognition`

Role: **FILTERED SUPPLEMENTAL REAL TRAIN-ONLY SOURCE**

Raw detector-positive candidates:

- 1539 images
- 2224 instances
- 909 source groups

Manual filtering:

- accepted training positives before duplicate cleanup: 609
- reject plate/OCR crop: 640
- reject mosaic/collage: 290
- unsure: 0

After exact-duplicate annotation adjudication:

- final eligible: 607 images
- final instances: 633
- usage: TRAIN ONLY

Upstream split organization is not reused.

### ELPD

Role: **SYNTHETIC EUROPEAN SUPPLEMENTAL TRAIN-ONLY SOURCE**

Raw:

- 2329 images

Excluded from derived training pool:

- 4 corrupt images
- 39 incomplete empty-label images

Usable positive pool:

- 2286 images
- 2947 plate instances

Included in baseline_v1 due to synthetic cap:

- 1905 images
- 2446 plate instances

Remaining eligible ELPD samples are preserved but not materialized into baseline_v1.

ELPD remains TRAIN_ONLY and SYNTHETIC.

### Open Images real negative pool

Role: **REAL DETECTOR NEGATIVES**

Accepted audited negatives:

- 504 unique ImageIDs
- 0 plate instances

Category distribution:

- vehicle without visible plate candidates: 172
- road/gate scenes: 159
- heavy-equipment candidates: 128
- barrier/background: 24
- people: 15
- text-like objects: 6

Canonical split:

- TRAIN: 404
- VAL: 50
- TEST: 50

A sample is treated as a detector negative only after audit confirms no visible license plate.

## Canonical normalization

Canonical class:

`0 = license_plate`

Canonical annotation format:

`YOLO normalized XYWH`

The use of YOLO annotation format does not by itself lock AVAX to a specific detector architecture or runtime.

Canonical pre-filter pool:

- 9299 images
- 8795 positives
- 504 negatives
- 12083 plate instances
- REAL: 7013
- SYNTHETIC: 2286

Images are copied without re-encoding. Raw source datasets are not modified.

## Duplicate / leakage safety

Canonical images analyzed: `9299`

- unique decoded-pixel SHA256 hashes: 9297
- leakage source groups: 8510
- exact duplicate groups: 2
- images in exact duplicate groups: 4
- exact groups crossing source-group boundaries: 0

Perceptual duplicate analysis:

- pHash threshold <= 8
- dHash threshold <= 10
- aspect-ratio relative difference <= 0.15
- unresolved near-duplicate candidate pairs: 0

After manual exact-duplicate annotation adjudication:

- eligible candidate pool: 9297 images
- eligible positive images: 8793
- negatives: 504
- plate instances: 12081

Final split leakage validation: PASS.

## Licensing / provenance status

Third-party dataset attribution and license preservation are maintained in `documentation/THIRD_PARTY_NOTICES.md` and AI provenance artifacts.

Confirmed source-level handling includes:

- Romanian public LP: MIT repository license preserved
- Open Images image-level metadata: CC BY 2.0 attribution/provenance retained per accepted ImageID
- Open Images annotation provenance/attribution retained where applicable
- ELPD: CC BY 4.0 attribution retained; synthetic/derived status preserved
- Kaggle `plate_license_recognition`: Kaggle data card currently reports Apache 2.0; source is used TRAIN_ONLY and its attribution/license record must be retained

External redistribution of raw datasets remains separate from model distribution and requires preservation of all applicable source notices and attribution.

## Known domain limitation

The accepted public baseline still has limited dedicated AVAX construction-site coverage for:

- mud
- heavy dust
- fixed gate-camera geometry
- AVAX-specific vehicle approaches
- severe construction-site occlusion
- site-specific night/headlight conditions

This does not block the first detector baseline.

Private AVAX field imagery has not been incorporated without explicit Master authorization.

Future follow-up remains:

`AI-DATA-WP-002 — AVAX Field Domain Adaptation & Validation Dataset`

- Status: `PROPOSED`
- Priority: `P1 High before production AI sign-off`

## Next work package

`AI-WP-001 — License Plate Detector Baseline & Mobile Export Contract`

Immediate sequence:

1. select/justify the first lightweight detector baseline;
2. create a reproducible training configuration against `baseline_v1`;
3. train and select checkpoint using TRAIN/VAL only;
4. evaluate final selected checkpoint on TEST without tuning on TEST;
5. report Precision, Recall, mAP@0.5 and mAP@0.5:0.95;
6. perform failure analysis including false positives on the audited negative pool;
7. select confidence/NMS behavior from validation evidence;
8. export at least one mobile-consumable artifact;
9. validate exported inference against reference inference;
10. finalize the Mobile Detector Contract.

Do not use TEST to tune hyperparameters or confidence thresholds.

## Confirmed completed work

| ID | Work item | Priority | Status |
|---|---|---:|---|
| BE-001 | Backend Baseline Audit & Build Validation | P0 | DONE |
| SEC-001 | Resolve NU1903 Microsoft.OpenApi Vulnerability | P0 | DONE |
| BE-002 | Validate Existing SQL Schema Relevant to ALPR | P0 | DONE |
| DEVDB-001 | Local SQLite Development Database Baseline | P1 | DONE |
| BE-WP-001 | Local Backend Vehicle Read API Foundation | P0 | DONE |
| BE-WP-002 | Vehicle Snapshot Sync API v1 | P0 | DONE |
| MOB-WP-001 | Offline Vehicle Cache & Manual Access Verification | P0 | DONE |
| MOB-WP-002 | Local Access Logging Foundation | P0 | DONE |
| BE-WP-003 | Access Log Ingestion API v1 | P0 | DONE |
| MOB-WP-003 | Background Access Log Upload | P0 | DONE |
| CAM-WP-001 | CameraX Foundation | P0 | DONE |
| AI-DATA-WP-001 | License Plate Detector Dataset Acquisition & Annotation Foundation | P0 | DONE |

## Backend / production follow-up

`BE-WP-004 — SQL Server Access Log Persistence & Controlled Deployment`

- Priority: `P0 before production`
- Status: `TODO`

Production `dbo.AVAX_ALPR_ACCESS_LOGS` deployment and the production SQL Server persistence adapter are not yet confirmed implemented.

## Guard Mobile status

Confirmed mobile foundation includes:

- offline Room vehicle cache
- local access verification
- local access logging
- background access-log upload
- CameraX rear-camera preview
- ImageAnalysis with bounded backpressure
- lifecycle-safe camera binding
- clean `FrameProcessor` boundary for future on-device AI

No detector or OCR integration is yet confirmed implemented.

## Open technical debt / limitations

- `TD-001` — `AVAX_VEHICLES` lacks enforced primary identity
- `TD-002` — license plate uniqueness is not enforced
- `TD-003` — no incremental synchronization marker
- `TD-004` — no vehicle lookup index in production
- production access-log persistence deployment pending
- no access-log retention policy defined
- AVAX field-domain detector data remains limited

## Explicitly not confirmed as implemented

- trained license-plate detector
- finalized detector architecture/runtime choice
- OCR
- ONNX/TFLite mobile detector integration
- automatic camera plate lookup
- automatic camera-generated access decision
- automatic camera-generated access logging
- AVAX field-domain adaptation dataset
- production SQL Server access-log persistence
- Manager Approve/Deny workflow
- push notifications
- Admin Dashboard
- production authentication/authorization
- production deployment

## Governance

- Only Master-confirmed implementation and validation may be marked `DONE`.
- Unconfirmed architecture decisions remain `PROPOSED`.
- AI never decides access.
- Guard Mobile never connects directly to SQL Server.
- Manager/Admin must not communicate directly with Guard Mobile; server-side communication passes through the Backend API.

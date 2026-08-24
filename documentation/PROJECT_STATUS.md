# AVAX ALPR Project Status

**Status snapshot:** 2026-08-24  
**Source of truth:** AVAX ALPR Master Plan & Current Status

This file is the concise cross-project status snapshot. Detailed planning, architecture, API, database, testing, technical-debt, licensing, and changelog information is maintained in the dedicated documentation files.

## Current milestone

`AI-DATA-WP-001 — License Plate Detector Dataset Acquisition & Annotation Foundation`

- Priority: `P0 Critical`
- Status: `IN PROGRESS`
- Target project: AVAX ALPR – AI Model

`AI-WP-001 — License Plate Detector Baseline & Mobile Export Contract` remains `BLOCKED / DATASET REQUIRED` until the canonical detector dataset is constructed and accepted.

## Master-approved detector dataset direction

The canonical AVAX detector dataset will be built from four audited sources with distinct roles.

### 1. Romanian public license-plate dataset

Role: **PRIMARY ROMANIAN / EU REAL-DOMAIN SOURCE**

Confirmed audit:

- 534 images
- 652 license-plate instances
- Pascal VOC
- 0 corrupt images
- 0 invalid annotations
- 0 invalid bounding boxes
- only 4 source video sequences
- all 4 sequences leak across the upstream train/validation split

Master decision:

- ACCEPT
- raw data remains untouched
- upstream split must not be reused for AVAX evaluation
- sequence identity must be preserved
- every sequence must belong to exactly one AVAX split
- MIT attribution/license preservation remains required and is documented in `THIRD_PARTY_NOTICES.md`

### 2. Existing Kaggle `plate-license-recognition-dataset`

Role: **FILTERED SUPPLEMENTAL REAL TRAINING SOURCE**

Confirmed detector subset:

- 1539 detector images
- 2224 license-plate instances
- 291 multi-plate images
- 909 detector source groups
- 384 source groups with multiple image variants
- 271 multi-variant groups with annotation disagreement
- original split has source-group leakage
- significant mosaic/collage-style content exists
- approximately 2500 non-detector samples are primarily OCR/character content and must not be treated as detector negatives

Master decision:

- ACCEPT FOR TRAINING AFTER FILTERING
- TRAIN only for the first canonical baseline
- preserve source-group identity
- all variants from one source group stay in one split
- exclude/isolate mosaic/collage samples
- exclude OCR/character crops from detector-negative logic
- do not reuse upstream train/validation/test

### 3. Open-Images-derived Kaggle detector dataset

Role: **ADDITIONAL REAL-WORLD DETECTOR SOURCE**

Confirmed audit:

- 5368 images
- 7852 plate instances
- 1609 multi-plate images
- 0 corrupt images
- 0 missing labels
- 0 orphan labels
- 0 exact duplicate groups
- 0 identical local train/validation ImageIDs
- all 5368 filenames preserve valid Open Images ImageIDs
- 192 boxes have tiny boundary overshoot only; maximum observed normalized overshoot approximately 0.002252
- all 5368 local ImageIDs matched official Open Images metadata
- all local samples originate from the upstream Open Images train pool
- image-level provenance metadata is available for every image
- image-level license metadata is CC BY 2.0 for all audited images
- Kaggle-local train/validation is not the upstream Open Images split

Master decision:

- ACCEPT
- eligible for TRAIN
- eligible for VALIDATION after canonical filtering/group rules
- eligible for TEST after canonical filtering/group rules
- validation/test should prioritize realistic full-vehicle/full-scene images rather than heavily cropped plate scenes
- preserve per-image provenance/attribution metadata
- Kaggle uploader-level CC0 must not replace upstream image-level licensing information
- raw labels remain untouched
- tiny boundary overshoot may be deterministically clipped only in derived canonical annotations

Before final canonical splitting, run near-duplicate/similarity grouping sufficient to prevent visually equivalent frames from crossing splits.

### 4. European License Plate Dataset — ELPD

Role: **SYNTHETIC EUROPEAN SUPPLEMENTAL TRAINING SOURCE**

Confirmed audit:

- 2329 raw images
- 2948 annotated plate instances
- 628 multi-object images
- 4 corrupt source images
- 2325 usable images after corrupt exclusion
- 2286 usable positive images
- 2947 usable annotated plate instances
- 39 usable empty-annotation images contain visible unannotated plates and are therefore not valid detector negatives
- synthetic European-like road scenes with varied weather, distance, traffic, night, glare, and some trucks
- CC BY 4.0

Master decision:

- ACCEPT — TRAIN-ONLY
- exclude the 4 corrupt source samples from derived AVAX data
- exclude the 39 incomplete empty-label samples unless they are later manually re-annotated
- do not repair or modify raw source data
- do not use ELPD for primary AVAX validation/test metrics
- preserve source attribution, license reference, and derivative/modification notice where applicable
- keep samples explicitly marked as synthetic in provenance metadata

## Canonical dataset composition decision

Approved first-baseline composition strategy:

### Training

REAL sources:

- sequence-safe Romanian data assigned to training
- Open-Images-derived real detector data
- filtered clean samples from the existing Kaggle detector subset

SYNTHETIC source:

- ELPD usable positive samples only

Initial sampling rule:

- target at least 75% REAL samples
- cap synthetic ELPD contribution at approximately 25% of detector training samples for the first baseline

This is a first-baseline sampling policy, not a permanent architecture constraint. It may be revised later from measured detector error analysis.

### Validation

Primary validation must be REAL.

Eligible:

- sequence-safe Romanian real data
- filtered realistic Open Images scenes

Exclude from primary validation:

- ELPD
- mosaic/collage samples
- OCR/character crops
- source-leaking variants

### Test

Primary test benchmark should be 100% REAL where practical.

Priority:

1. sequence-held-out Romanian/EU real samples
2. realistic full-vehicle/full-scene Open Images samples

No train source group / video sequence / near-duplicate group may appear in the primary test set.

ELPD must not contribute to the primary AVAX test metric.

## Detector-negative strategy — Master decision

The current positive datasets do not provide a trustworthy real detector-negative pool.

Master decision: **curate a dedicated real negative subset from provenance-safe Open Images data.**

This is not a search for another license-plate dataset. It is a targeted negative-frame collection task.

Required negative categories should include, where available:

- vehicles with no visible plate
- partial vehicles / vehicle fragments
- empty road and gate-like scenes
- people
- construction machinery / heavy equipment
- signs and text-like objects
- barriers / fences / gate infrastructure
- background scenes likely to create plate-like false positives

Negative acceptance rule:

- an image is a detector negative only if no license plate is visibly present after audit
- absence of a `Vehicle registration plate` annotation alone is not sufficient proof of a true negative
- source ImageID and per-image provenance/license metadata must be retained
- raw Open Images data must remain untouched

Initial negative-pool target:

- curate approximately 500–1000 audited REAL negatives for the first baseline
- prefer diversity over volume
- do not exceed this range merely to increase dataset size before first model error analysis

After the first detector baseline, false-positive analysis may justify expanding or rebalancing the negative pool.

## AVAX field-domain limitation

Construction-site, heavy-equipment, security-gate, muddy-site, and fixed gate-camera representation is currently limited.

Master decision:

- this does NOT block the first public-data detector baseline
- it must be documented as an explicit domain limitation
- do not acquire/use private AVAX field imagery without explicit authorization
- create a later dedicated domain-adaptation / field-validation work package before production detector sign-off if authorized field data becomes available

Proposed future follow-up:

`AI-DATA-WP-002 — AVAX Field Domain Adaptation & Validation Dataset`

- Status: `PROPOSED`
- Priority: `P1 High before production AI sign-off`
- Not a dependency for the first AI-WP-001 baseline

## Immediate next execution

AI-DATA-WP-001 should now:

1. curate and audit the real Open Images negative pool;
2. normalize all accepted source annotations into one derived canonical detector format;
3. preserve per-sample source, provenance, synthetic/real flag, and source-group/sequence identity;
4. perform near-duplicate/group analysis before splitting;
5. construct leakage-safe canonical train/validation/test splits under the approved source-role rules;
6. produce factual final split counts and real/synthetic/negative composition;
7. validate the derived dataset without modifying raw sources;
8. return `MASTER HANDOFF — AI-DATA-WP-001` for acceptance.

Do NOT begin detector training until Master accepts the final canonical dataset handoff.

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
- canonical detector dataset not yet finalized
- AVAX field-domain detector data is limited

## Explicitly not confirmed as implemented

- final canonical detector dataset
- detector training
- trained license-plate detector
- OCR
- ONNX/TFLite mobile detector runtime integration
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

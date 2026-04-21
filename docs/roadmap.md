# Roadmap

## Phase 1 — Project scaffold ✅

- Flutter project created
- GitHub Actions CI (Flutter APK builds)
- `.gitignore` (excludes `last.pt`, datasets, large media)
- `planogram.json` schema defined
- App directory layout settled

## Phase 2 — Core inference scaffold ✅

- `CameraService` (live preview, frame streaming)
- `InferenceIsolate` (Dart Isolate + TFLite runtime)
- `ComplianceEngine` scaffold (stub logic, real comparison comes in Phase 3)
- Android configuration (min SDK 21, permissions, GPU delegate option)
- `best_fp16.tflite` ready for integration into `assets/models/`

Status: files generated as ZIPs, waiting for push to private GitHub repo.

## Phase 3 — Full Compliance Engine (NEXT)

- Shelf grouping: cluster detections by y-coordinate into rows
- Compliance comparison: for each row, match detected layout against planogram
- `CustomPainter` overlay: green box = correct, red box = wrong SKU / missing / extra
- Handle edge cases: occlusion, angle, partial boxes
- Tests: golden-file comparison on a fixed set of shelf photos

Unblocks: real user-facing demo of the core value prop.

## Phase 3.5 — Embedding knowledge base

- Integrate MobileNetV3 TFLite (~6MB) for per-crop embeddings
- Build knowledge base: label → centroid embedding → threshold
- Fallback logic: if classifier confidence < 0.85, query embedding KB
- Saving path:
  - Conf 0.65–0.85 → save crop to `/pending_review/`
  - Conf < 0.65 → same, but flagged as "unknown"
- Nightly/weekly: DBSCAN on unknowns to discover new clusters (new SKUs?)

**Why this matters:** adding a new energy drink becomes "annotate ~20 crops, recompute centroid" instead of "retrain the whole model".

## Phase 4 — Review UI

- In-app screen for `/pending_review/` items
- Swipe-to-label: accept suggested class / pick from list / mark as new class
- Bulk export of relabelled crops → push to Roboflow `new-way-energy`
- Weekly retrain loop closes: new labels → new dataset version → new model → replaces on-device TFLite

## Phase 5 — Multi-category expansion (speculative)

- Generalise beyond energy drinks: sparkling water, juices, etc.
- Each category gets its own Stage 2 classifier
- Stage 1 detector stays unchanged (still trained on SKU110K)
- Planogram schema extended with `category` field

## Cross-cutting work (ongoing)

- Dataset quality: class name cleanup (e.g. merging `adrenaline_250ml` + `adrenaline_original_250ml`), manual review of auto-labels, growing validation split
- Documentation: keeping this repo + Obsidian vault in sync
- CI/CD: automatic APK upload to a GitHub release per tag

## Timeline (aspirational, not committed)

| When | Milestone |
|---|---|
| Q2 2026 | Phase 3 complete, first demo-able build |
| Q3 2026 | Phase 3.5 rolling out, first live retrain loop |
| Q4 2026 | Phase 4 review UI shipped, closed-loop dataset growth |

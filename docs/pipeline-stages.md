# Pipeline stages

Per-stage breakdown of the ShelfVision pipeline. Complements [architecture.md](architecture.md).

## Stage 1 — Generic detector

**Model:** YOLOv8s trained on SKU110K.
**Classes:** 1 (`object`).
**Input:** 640×640 RGB tensor from camera frame.
**Output:** List of `(x, y, w, h, confidence)` bounding boxes.

**Why single class:** the detector's job is to *find product-shaped things*. The 1.7M labelled boxes in SKU110K teach it shelf-context cues — spacing, lighting, typical can/bottle silhouettes — that a small brand-specific dataset can't match. We don't want the detector second-guessing itself on "is this a can?" vs "is this Hell Energy?"

**Runtime:** ~30ms per frame on Snapdragon 865 GPU delegate.

## Stage 2 — Brand classifier

**Model (current):** YOLOv8s fine-tuned on `new-way-energy` (34 classes, ~129 annotated shelf images).
**Model (alternative, Phase 3.5):** MobileNetV3 + nearest-neighbour over embeddings.
**Input:** Each Stage-1 bbox, cropped from the original frame.
**Output:** Per-bbox class label + confidence.

**Current status:** first training run hit mAP@50 ~28.6% on a 3-image validation set — noisy metric, per-image detection was actually solid. Dataset has since grown to ~129 images with ~8,059 auto-labelled boxes.

**Phase 3.5 alternative:** the classifier runs first; if confidence < 0.85, we compute the embedding and look up nearest class centroid. This lets us:
1. Fall back gracefully when the classifier is unsure.
2. Add new SKUs by recomputing a centroid, not retraining.

## Stage 3 — Shelf grouping

**Input:** Stage-2 labelled boxes.
**Output:** `List<ShelfRow>` — boxes clustered into horizontal shelf rows.

**How:**
- Sort boxes by y-centre.
- Greedy pass: if the next box's y-centre is within `k * median_box_height` of the current row's mean, add it to the row; else start a new row.
- `k ≈ 0.6` works for typical shelf photos; tune on real data.

**Why this matters:** the planogram is defined as "shelf 1 left→right: A, B, C; shelf 2 left→right: D, E, F". You can only check compliance if you know which shelf each detected product belongs to.

## Stage 4 — Planogram comparison + overlay

**Input:** `List<ShelfRow>` + `planogram.json`.
**Output:** Per-box compliance result (`correct` / `wrong_sku` / `missing` / `extra`).

**Rules:**
- For each planogram row, align left-to-right with detected row.
- Detected SKU matches expected → **green**.
- Detected SKU does not match → **red** with expected label.
- Expected SKU not detected at that position → **red outline at expected position**.
- Detected SKU at a position with no planogram entry → **amber** (configurable).

**Rendering:** `CustomPainter` draws on top of the live camera preview. 2–3 FPS throttle matches inference speed.

## Phase 3.5 — Embedding knowledge base

Not a pipeline stage per se — a *sidecar* to Stage 2.

- **Embedding model:** MobileNetV3-Small TFLite (~6MB, ~8ms/crop on Snapdragon 865).
- **Knowledge base:** `Map<class_label, embedding_centroid>`, updated weekly.
- **Fallback trigger:** Stage-2 classifier confidence < 0.85.
- **Auto-save buckets:**
  - 0.65–0.85 → `/pending_review/known_low_conf/`
  - < 0.65 → `/pending_review/unknown/`
- **DBSCAN pass:** weekly, over `unknown/` embeddings. Clusters with > N members are candidate new SKUs. John reviews, names, seeds a centroid.

## Phase 4 — Review UI (sidecar)

- Lists crops from `/pending_review/`.
- Swipe actions: **Accept** (confirm predicted label) / **Relabel** (pick from 34 classes) / **New class** (enter name → adds to a queue for Roboflow upload).
- Bulk export: relabelled crops → ZIP → upload to `new-way-energy` dataset → new version → retrain.

This closes the active-learning loop.

# Architecture

ShelfVision uses a **hybrid multi-stage pipeline** inspired by `albertferre/shelf-product-identifier`. The design prioritises independence between stages, so a new energy drink SKU can be added without retraining the generic detector, and the generic detector stays trained on the massive SKU110K dataset (11,762 images, 1.7M boxes) without ever being contaminated by brand-specific labels.

## Pipeline at a glance

```
 ┌───────────────┐    ┌────────────────┐    ┌─────────────────┐    ┌──────────────────────┐
 │ Stage 1       │    │ Stage 2        │    │ Stage 3         │    │ Stage 4              │
 │ Generic       │──▶│ Brand          │──▶│ Shelf grouping  │──▶│ Planogram comparison │
 │ detector      │    │ classifier     │    │ (row clustering)│    │ + CustomPainter      │
 │ (SKU110K,     │    │ (34-class      │    │                 │    │ overlay (green/red)  │
 │ 1 class)      │    │ `energys`)     │    │                 │    │                      │
 └───────────────┘    └────────────────┘    └─────────────────┘    └──────────────────────┘
        │                    │
        │                    │ OR (Phase 3.5)
        │                    ▼
        │          ┌─────────────────────────────┐
        │          │ MobileNetV3 embeddings +    │
        └─────────▶│ knowledge-base lookup       │
                   │ (fast new-SKU addition      │
                   │  without retraining)        │
                   └─────────────────────────────┘
```

## Why this split?

**The 1-class detector handles all the visual diversity of "a drink can on a shelf" — it has seen 1.7M boxes.** Retraining it for every new brand would be wasteful and would dilute its generalisation. Instead, we let it be the world expert on *finding things*, and defer *identifying what the thing is* to Stage 2.

Stage 2 can be swapped between:

- **Option A (current)**: a 34-class YOLOv8 classifier (`energys`). Fast and accurate, but requires retraining for each new SKU.
- **Option B (Phase 3.5)**: MobileNetV3 → embedding vector → nearest-neighbour lookup in a knowledge base. Slower per query, but adding a new SKU means adding a row to a CSV, not retraining a model.

The plan is to run both in parallel, use the classifier when it's confident, fall back to embeddings otherwise, and route the genuinely unknown cases into a review queue (Phase 4).

## On-device constraints

Target: **Poco F2 Pro** (Snapdragon 865, Adreno 650 GPU, 6GB RAM).

- Model size: `best_fp16.tflite` is ~12MB. Comfortably fits.
- Inference: 2–3 FPS throttled on a Dart Isolate to keep the UI thread free.
- INT8 export was attempted but failed because no `data.yaml` calibration file was available. FP16 is the current compromise — quality > speed here.
- MobileNetV3 embedding step (planned) adds ~8ms/crop on this hardware.

## Component boundaries

| Component | Responsibility | Tech |
|---|---|---|
| `CameraService` | Frame capture, downscale, hand off to isolate | Flutter `camera` package |
| `InferenceIsolate` | Run TFLite, return boxes + labels | Dart Isolate + TFLite |
| `ShelfGrouper` | Cluster boxes by y-coordinate into shelf rows | Pure Dart |
| `ComplianceEngine` | Compare detected layout vs planogram JSON | Pure Dart |
| `OverlayPainter` | Draw green/red boxes on top of camera preview | `CustomPainter` |
| `PendingReviewStore` | Save low-confidence crops + embeddings for later | SQLite + file system |

Each component is independently testable. The isolate boundary also means a hung inference cannot freeze the camera preview.

## Data flow

1. Camera frame → `ImageStream` → downscale to 640×640 tensor.
2. Tensor → `InferenceIsolate` → bounding boxes (x, y, w, h, class, confidence).
3. Boxes → `ShelfGrouper` → `List<ShelfRow>`.
4. Rows + `planogram.json` → `ComplianceEngine` → `List<ComplianceResult>`.
5. Results → `OverlayPainter` → green/red boxes on screen.
6. Low-confidence crops (conf 0.65–0.85) → `PendingReviewStore` → weekly review → back to Roboflow as new labelled data.

See [pipeline-stages.md](pipeline-stages.md) for per-stage details.

# Model training

The end-to-end workflow for training and iterating the `energys` classifier, plus notes on the Stage 1 SKU110K detector.

## Overview

The training loop is a **bootstrap active-learning workflow**:

```
Manually label seed set (Roboflow)
        │
        ▼
Train in Colab (T4 GPU, YOLOv8s)
        │
        ▼
Export best.pt + best_fp16.tflite
        │
        ▼
Auto-label next batch of shelf photos (script using best.pt)
        │
        ▼
Upload annotated ZIP to Roboflow
        │
        ▼
Human review/correction inside Roboflow
        │
        ▼
Create new dataset version → back to top (retrain w/ transfer learning)
```

## Roboflow setup

- **Workspace:** `segment-anything-hw7w8`
- **Project:** `new-way-energy`
- **Plan:** Free
- **Current dataset:** ~129 annotated images, growing
- **Split:** 50/30/20 train/valid/test (small-dataset-safe)

### Upload format

ZIP must be **flat** (no subdirectories), containing:

- `img_001.jpg`, `img_002.jpg`, …
- `img_001.txt`, `img_002.txt`, … (YOLO-format annotations, one box per line)
- `classes.txt` (one class name per line, order matches class indices)

**Images must be JPEG.** WebP uploads have caused annotation-matching failures — even though Roboflow nominally accepts WebP, the matching logic has been inconsistent. Keep it simple: convert WebP → JPEG before upload.

## Colab training

Notebook runs on T4 GPU. Key parameters:

```python
from ultralytics import YOLO

model = YOLO("best.pt")  # transfer-learning from previous run
results = model.train(
    data="data.yaml",
    epochs=150,
    patience=30,
    imgsz=640,
    batch=16,
    optimizer="AdamW",
    lr0=0.001,
    device=0,
)
```

**Notes:**
- `patience=30` stops training if validation mAP hasn't improved in 30 epochs.
- `optimizer="AdamW"` chosen empirically — works well on small datasets.
- `lr0=0.001` is the Ultralytics default for AdamW.

## Auto-labelling (Label Assist workaround)

Free-plan Roboflow doesn't support workspace-model Label Assist, so we do it externally:

```python
# Colab, after training
from ultralytics import YOLO
model = YOLO("runs/detect/train/weights/best.pt")

import os, glob
for img_path in glob.glob("new_shelf_images/*.jpg"):
    results = model.predict(img_path, conf=0.25, save_txt=True)
    # save_txt writes YOLO-format annotations next to each image
```

Then zip the images + `.txt` files + `classes.txt` and upload back to Roboflow.

Last run: ~105 images → ~8,059 bounding boxes, ~40% high confidence.

## Export

```python
model.export(format="tflite", half=True)   # FP16
# INT8 attempted but fails without a data.yaml calibration file
```

Current export: `best_fp16.tflite` (~12MB, 34 classes).

## Upload best.pt to Roboflow for future Label Assist

The `roboflow` Python SDK supports `workspace.deploy_model()` — this is how we can register `best.pt` as an internal model, potentially unlocking Label Assist for the next tier. Needs the **Private** API key (not Publishable).

## Stage 1 detector (SKU110K)

- **Source:** SKU110K public dataset (11,762 images, 1.7M boxes, 1 class).
- **Training:** YOLOv8s, standard hyperparameters, trained once and not retrained per-iteration.
- **Output:** Stage 1 `.tflite` used for generic "find any product" detection.
- **Do not merge** with `energys`. Ever. This is the single most important training-side decision.

## Validation metrics interpretation

- **mAP@50 on small datasets:** take with a grain of salt. With 3 validation images, one FN tanks mAP by 33%.
- **Per-image detection quality:** visually inspect predictions on held-out shelf photos. That's the ground truth for "is the model working".
- **Expected trajectory after class cleanup + 129-image retrain:** mAP@50 80–90%.

## Common failure modes to watch for

| Symptom | Likely cause | Fix |
|---|---|---|
| mAP drops after adding data | Class imbalance, duplicate labels | Review classes.txt; inspect per-class AP |
| Auto-label confusion between similar cans | Stage 2 genuinely can't distinguish; not a training bug | Collect more examples of the pair; consider embedding fallback |
| Model sees products but wrong box sizes | Training imgsz mismatch with inference | Both must be 640 |
| Slow convergence | LR too low, or dataset too small | Try lr0=0.003; increase augmentation |

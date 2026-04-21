# Decisions log

A running list of technical decisions, each with the reason and consequences. New entries go on top.

---

## 2026-04-21 — `adrenaline_original_250ml` is the canonical class name

**Context:** Roboflow had both `adrenaline_250ml` and `adrenaline_original_250ml` as separate classes, but they are the same product.

**Decision:** Merge → `adrenaline_original_250ml`. Review all 34 classes for similar duplicates before the next retrain.

**Why:** Duplicate classes fragment the training signal and hurt mAP. Catching these before retraining saves a training cycle.

---

## 2026-04-10 — Use Colab bootstrap instead of Roboflow Label Assist

**Context:** Roboflow Free plan does not allow Label Assist with user-trained workspace models.

**Decision:** Train in Colab (T4 GPU) → export `best.pt` → auto-label externally → upload annotated ZIPs back to Roboflow.

**Why:** Free-plan limitation workaround. Colab is free, T4 is fast enough, and it keeps the workflow within our budget.

**Consequence:** Label Assist step is an external scripted process, not a one-click thing inside Roboflow. Extra glue code but fully automated.

---

## 2026-04 — Upload as JPEG, not WebP

**Context:** Planohero exports WebP by default, and WebP is smaller/higher quality than JPEG. But Roboflow failed to match WebP uploads against their corresponding annotation files.

**Decision:** Convert to JPEG before uploading to Roboflow.

**Why:** YOLOv8 converts all inputs to 640×640 tensors internally anyway — format is irrelevant to training quality. JPEGs work reliably for annotation matching. WebP doesn't.

**Consequence:** Slight storage overhead during upload; zero impact on training quality.

---

## 2026-04 — FP16 export, not INT8

**Context:** Wanted INT8 quantisation for speed and smaller model size.

**Decision:** Stuck with FP16 (`best_fp16.tflite`, ~12MB).

**Why:** INT8 export failed because the calibration step needs `data.yaml`, which was not available/compatible at export time.

**Consequence:** Larger model, slower inference than INT8 would be — but still well within Snapdragon 865 budget. Revisit once calibration data is sorted.

---

## 2026-04 — SKU110K detector stays separate; never merged into `energys`

**Context:** Tempting to train one big model that both detects and classifies.

**Decision:** SKU110K remains a standalone Stage 1 single-class ("drink") detector. The 34-class brand work stays in `energys`. Do **not** merge them.

**Why:**
- SKU110K has 11,762 images and 1.7M boxes of pure "generic retail product" data. Mixing in 129 brand-labelled images would dilute it with no benefit.
- Adding a new energy drink brand should not require retraining the generic detector.
- Separation of concerns: Stage 1 = "where are the products", Stage 2 = "what is each product".

**Consequence:** Two models ship in the app, orchestrated by the pipeline. Slightly more glue code, much more flexibility.

---

## 2026-04 — 50/30/20 split for small datasets

**Context:** Default Roboflow split gave only 3 validation images. First training run reported mAP@50 ~28.6%, which looked catastrophic — but per-image detection was actually fine.

**Decision:** Use 50/30/20 train/valid/test until the dataset is large enough to absorb a 70/20/10 split reliably.

**Why:** With 3 val images, a single false negative tanks mAP by 33%. Misleadingly low numbers are a bad signal for decision-making.

**Consequence:** Smaller training set, but trustworthy metrics.

---

## 2026-04 — Use Private API key (not Publishable) for Roboflow SDK

**Context:** Roboflow provides two keys; the `rf_`-prefixed "Publishable" key was tried first and failed.

**Decision:** Use the **Private** API key for `roboflow` Python SDK authentication.

**Why:** Publishable keys are read-only; SDK operations (upload, version creation, deploy) need write access.

**Consequence:** Key is stored only in Colab secrets / local env, never committed.

---

## Guiding principles (meta-decisions)

These are not specific incidents but the principles behind many of the above.

- **Dataset quality > training time.** A clean 500-image dataset beats a noisy 5000-image one for small-project economics.
- **Iterative active learning.** Collect → auto-label → review → retrain. Never a single "big bang" training.
- **Architecture before code.** Explain the *why* first, then write the code.
- **Phased delivery.** Numbered phases with clear handoffs prevent scope creep.
- **Transfer learning by default.** Every new run starts from the previous `best.pt`, never from COCO weights.

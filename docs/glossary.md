# Glossary

Short, practical definitions of every term that shows up in ShelfVision docs.

**Active learning** — A machine learning loop where the model picks the data it wants labelled next. In ShelfVision: we use the current model to auto-label new shelf photos, a human reviews/fixes the predictions, and the corrected data goes back into training. Opposite of passive learning (label everything upfront).

**Bounding box (bbox)** — A rectangle `(x, y, width, height)` around a detected object. YOLO outputs one bbox per detection plus a class label and a confidence.

**Class** — A category label in an object detection model. In `energys`, each of the 34 classes is a specific SKU (e.g. `adrenaline_original_250ml`, `hell_classic_500ml`). In SKU110K, there is only one class: `object`.

**Colab / Google Colab** — Free cloud Jupyter notebook environment with GPU access (T4 in our case). Where ShelfVision models are trained.

**Compliance** — Whether the detected shelf layout matches the planogram. A product is "compliant" if the right SKU is in the right position.

**CustomPainter** — Flutter's low-level 2D drawing API. Used to paint the green/red overlay on top of the camera preview.

**DBSCAN** — A clustering algorithm that groups nearby points without needing to know the number of clusters upfront. Used in Phase 3.5 to discover new-SKU clusters from unknown embeddings.

**Embedding** — A fixed-length vector (e.g. 1280 floats from MobileNetV3) that represents an image's visual content. Similar-looking crops have embeddings that are close in vector space. Similarity is measured with cosine distance.

**FP16 / FP32 / INT8** — Numeric precision of model weights. FP32 is the training default (4 bytes/weight), FP16 halves it (2 bytes), INT8 quarters it (1 byte). Smaller precision = smaller model + faster inference, but needs calibration. ShelfVision uses FP16.

**Isolate (Dart Isolate)** — Dart's concurrency primitive — a separate memory space with its own event loop. ML inference runs on an isolate so it doesn't block the UI thread.

**Label Assist** — Roboflow feature that pre-fills bounding boxes using a model, so humans only correct instead of drawing from scratch. Requires a paid plan if using a custom workspace model; the Colab bootstrap is our free-plan workaround.

**mAP (mean Average Precision)** — The standard object-detection metric. `mAP@50` means "the mAP computed at IoU=0.5", i.e. a detection counts as correct if it overlaps a ground-truth box by at least 50%. Ranges 0–100%. Small-dataset mAP is noisy and not a reason to panic.

**MobileNetV3** — A small, mobile-friendly CNN. ShelfVision uses the small variant (~6MB TFLite) for per-crop embeddings in Phase 3.5.

**`new-way-energy`** — The Roboflow project containing the 34-class energy drink dataset. Part of the `segment-anything-hw7w8` workspace.

**Pending review** — Crops with confidence in the 0.65–0.85 band. Saved to `/pending_review/` in-app; John reviews weekly.

**Planogram** — A diagram that specifies which SKU goes where on a shelf. Stored as `planogram.json` in-app.

**Roboflow** — Dataset management platform for computer vision. Upload images, annotate, create dataset versions, export in YOLO format.

**SKU (Stock Keeping Unit)** — A unique product identifier. In retail, each variant (size, flavour, can vs bottle) is a separate SKU.

**SKU110K** — A public dataset of 11,762 retail shelf images with 1.7M bounding boxes, all labelled as one class (`object`). The Stage 1 generic detector in ShelfVision is trained on it.

**TFLite (TensorFlow Lite)** — On-device ML runtime. Converts and runs models on phones with optional GPU/NNAPI delegates.

**Transfer learning** — Starting a training run from a previously trained model's weights instead of random initialisation. Every `energys` retrain starts from the previous `best.pt`.

**YOLOv8** — The current generation of the YOLO family of object detectors. `YOLOv8s` is the "small" variant (~22M parameters) — the size ShelfVision uses.

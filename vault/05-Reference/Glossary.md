# Glossary

Personal shorthand glossary. Public version lives at [docs/glossary.md](../../docs/glossary.md).

- **SKU** — stock keeping unit. Each variant (size / flavour / can vs bottle) is its own SKU.
- **Planogram** — shelf layout spec: which SKU in which position.
- **Compliance** — does the shelf match the planogram?
- **mAP@50** — standard detection metric. Noisy on small val sets.
- **Bbox** — bounding box (x, y, w, h).
- **Embedding** — fixed-length vector representing an image. Cosine similarity compares them.
- **DBSCAN** — density-based clustering, doesn't need k upfront. Used for discovering new-SKU clusters.
- **Isolate** — Dart's concurrency unit. Inference runs on one so UI isn't blocked.
- **FP16 / INT8** — model weight precision. We use FP16.
- **SKU110K** — public dataset: 11,762 retail shelf images, 1.7M boxes, 1 class. Stage 1 detector is trained on it.
- **`energys`** — my Roboflow project, 34 energy drink classes.
- **Active learning** — loop: predict → human corrects → retrain.
- **Label Assist** — Roboflow feature that pre-fills boxes with model predictions.
- **Transfer learning** — train from previous best weights, not random init.

---
Tags: #reference

# ShelfVision

> Flutter mobile app for retail shelf planogram compliance in Uzbekistan — energy drink SKU detection with real-time green/red overlay.

Live site: **https://firststeptogithub.github.io/shelfvision-docs/**

---

## What is this repo?

This is the **documentation and knowledge base** for the ShelfVision project. It serves two purposes:

1. **Static documentation website** (GitHub Pages, served from `/index.html`) — publicly browsable roadmap, architecture, decisions log, glossary.
2. **Obsidian vault** (`/vault`) — personal knowledge graph for daily notes, ideas, deep-dive references. Uses [obsidian-git](https://github.com/Vinzent03/obsidian-git) for sync.

The code repo for the Flutter app and ML pipeline is separate (private).

---

## Quick links

- [Architecture](docs/architecture.md) — hybrid multi-stage pipeline (SKU110K generic detector → brand classifier → shelf grouping → compliance overlay)
- [Roadmap](docs/roadmap.md) — Phase 1 → 2 → 3 → 3.5 → 4
- [Decisions log](docs/decisions-log.md) — why INT8 export failed, why SKU110K stays separate, why FP16, etc.
- [Glossary](docs/glossary.md) — SKU, planogram, mAP, DBSCAN, active learning, label assist, etc.
- [Pipeline stages](docs/pipeline-stages.md) — detailed breakdown of Stage 1–4
- [Model training](docs/model-training.md) — Roboflow + Colab bootstrap workflow
- [Flutter app](docs/flutter-app.md) — Phase 1/2 status, TFLite integration, Isolates
- [Obsidian setup](docs/obsidian-setup.md) — how to clone this repo as a vault + enable git sync

---

## Tech stack (short)

| Layer | Tech |
|---|---|
| Mobile | Flutter, Dart Isolates, TFLite |
| ML | YOLOv8s (Ultralytics), MobileNetV3 (planned) |
| Data | Roboflow (`segment-anything-hw7w8` / `new-way-energy`) |
| Training | Google Colab (T4 GPU) |
| Target device | Poco F2 Pro (Snapdragon 865, Adreno 650) |
| Classes | 34 energy drink SKUs (`energys` project) |
| Detector | SKU110K-trained single-class "drink" (Stage 1) |
| DevOps | GitHub, GitHub Actions (APK builds) |

---

## Status snapshot (2026-04-21)

- Dataset: ~129 annotated images, ~8,059 auto-labeled bounding boxes
- First training run: mAP@50 ~28.6% (3 val images — expected low)
- Model: `best_fp16.tflite` (~12MB, 34 classes)
- App: Phase 1 + Phase 2 scaffold complete, awaiting Phase 3 start
- Active work: class name cleanup → retrain → Phase 3 Compliance Engine

See [docs/roadmap.md](docs/roadmap.md) for the full picture.

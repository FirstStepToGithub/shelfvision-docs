# ShelfVision — Home

> Flutter app for retail shelf planogram compliance. Energy drink SKU detection. Target: Poco F2 Pro.

## Quick nav

- [[01-Architecture/System overview]]
- [[02-Roadmap/Current sprint]]
- [[03-Decisions/Index]]
- [[04-Daily-Notes/Latest|Today's note]]
- [[05-Reference/Glossary]]
- [[05-Reference/Links]]

## Status (2026-04-21)

- **Phase:** Phase 2 scaffold done → Phase 3 next (Compliance Engine)
- **Active work:** class name cleanup in Roboflow, then retrain
- **Dataset:** ~129 annotated images, ~8,059 auto-labelled boxes
- **Model:** `best_fp16.tflite` (~12MB, 34 classes)

## This week

- [ ] Merge `adrenaline_250ml` → `adrenaline_original_250ml` in Roboflow
- [ ] Review rest of 34 classes for similar duplicates
- [ ] Manual review pass over 105 auto-labelled images
- [ ] Push Flutter Phase 1+2 files to private repo
- [ ] Integrate `best_fp16.tflite` into `assets/models/`

## Pinned decisions

- SKU110K stays separate from `energys` ([[03-Decisions/SKU110K separation]])
- FP16 export, not INT8 ([[03-Decisions/FP16 export]])
- JPEG uploads to Roboflow, not WebP ([[03-Decisions/JPEG uploads]])

## Links

- **Live docs site:** https://firststeptogithub.github.io/shelfvision-docs/
- **Docs repo:** https://github.com/FirstStepToGithub/shelfvision-docs
- **Roboflow project:** `segment-anything-hw7w8` / `new-way-energy`
- **Reference repo:** [albertferre/shelf-product-identifier](https://github.com/albertferre/shelf-product-identifier)

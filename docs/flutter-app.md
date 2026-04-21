# Flutter app

## Target

- **Primary device:** Poco F2 Pro (Snapdragon 865, Adreno 650, 6GB RAM)
- **Min Android SDK:** 21
- **Orientation:** portrait
- **Camera:** rear only (shelf-facing)

## Status

- **Phase 1 (scaffold):** ✅ done. Project structure, GitHub Actions CI, `.gitignore`, `planogram.json` schema.
- **Phase 2 (core inference):** ✅ scaffolded. `CameraService`, `InferenceIsolate`, compliance engine stubs, Android config. Files exist as ZIPs, awaiting push.
- **Phase 3 (full compliance):** queued — starts once repo is up and `best_fp16.tflite` is integrated.
- **Phase 3.5, 4, 5:** see [roadmap.md](roadmap.md).

## Module layout

```
lib/
├── main.dart
├── screens/
│   ├── camera_screen.dart        # Live preview + overlay
│   └── review_screen.dart        # Phase 4: pending-review list
├── services/
│   ├── camera_service.dart       # Wraps camera package, streams frames
│   ├── inference_isolate.dart    # Dart Isolate running TFLite
│   ├── shelf_grouper.dart        # Phase 3
│   └── compliance_engine.dart    # Phase 3
├── models/
│   ├── detection.dart            # Bbox + class + confidence
│   ├── shelf_row.dart
│   └── planogram.dart            # Parses planogram.json
├── painters/
│   └── overlay_painter.dart      # CustomPainter: green/red overlay
└── stores/
    └── pending_review_store.dart # Phase 3.5: SQLite + file system

assets/
├── models/
│   ├── stage1_detector.tflite    # SKU110K single-class
│   └── best_fp16.tflite          # 34-class energys
├── planograms/
│   └── demo.json
└── (icons, fonts)

android/
└── app/src/main/AndroidManifest.xml  # Camera permission
```

## Isolate strategy

- One background isolate for inference; main isolate handles UI and camera.
- Communication via `SendPort` — send a downscaled RGB byte buffer, receive `List<Detection>`.
- Throttle: new frame only sent when isolate is idle (2–3 FPS effective).
- Cold-start: isolate loads both TFLite models once at app start. Warm inference ~30ms for Stage 1, ~15ms for Stage 2 classifier pass.

## Planogram schema

```json
{
  "shelf_id": "demo-shelf-01",
  "rows": [
    {
      "row_index": 0,
      "positions": [
        { "x": 0, "sku": "adrenaline_original_250ml" },
        { "x": 1, "sku": "adrenaline_original_250ml" },
        { "x": 2, "sku": "hell_classic_500ml" },
        { "x": 3, "sku": "hell_classic_500ml" },
        { "x": 4, "sku": "hell_classic_500ml" }
      ]
    },
    {
      "row_index": 1,
      "positions": [
        { "x": 0, "sku": "red_bull_250ml" },
        { "x": 1, "sku": "red_bull_250ml" },
        { "x": 2, "sku": "monster_500ml" }
      ]
    }
  ]
}
```

Positions are integer slots, not pixel coordinates — the matching logic aligns detected boxes left-to-right within each row.

## CI/CD

- GitHub Actions workflow: `flutter build apk --release` on push to `main`, artefact uploaded.
- Future: tag-based GitHub Release with signed APK.

## Known issues / open questions

- **GPU delegate stability:** TFLite GPU delegate sometimes crashes on older Adreno drivers. Fallback to CPU is wired but untested at scale.
- **Low-light shelves:** Stage 1 confidence drops noticeably under poor lighting. May need a post-processing pass or a small low-light-augmented fine-tune.
- **Multi-shelf scenes:** Stage 3 shelf grouping assumes a single visible shelf. Multi-shelf framing is future work.

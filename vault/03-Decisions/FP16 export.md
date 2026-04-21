# FP16 export (not INT8)

**Status:** accepted (revisit later)
**Area:** training / export

## Context

Wanted INT8 quantisation for smaller model + faster inference.

## Decision

Stuck with FP16 (`best_fp16.tflite`, ~12MB).

## Why

INT8 export failed because the calibration step needs `data.yaml`, which was not available / compatible at export time.

## Consequences

- Larger model, slower inference than INT8 — but still well within Snapdragon 865 budget.
- Revisit once calibration data is sorted.

---
Tags: #decision #training

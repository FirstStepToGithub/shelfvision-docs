# Colab bootstrap instead of Roboflow Label Assist

**Status:** accepted
**Area:** training workflow

## Context

Roboflow Free plan doesn't allow Label Assist with user-trained workspace models.

## Decision

Train in Colab (T4 GPU) → export `best.pt` → auto-label externally → upload annotated ZIPs back to Roboflow.

## Why

- Free-plan limitation workaround.
- Colab is free, T4 is fast enough.
- Keeps the workflow within budget.

## Consequences

- Label Assist is an external scripted process, not one-click in Roboflow.
- Extra glue code but fully automated.

---
Tags: #decision #training

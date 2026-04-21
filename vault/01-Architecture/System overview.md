# System overview

> Companion to the public [architecture.md](../../docs/architecture.md). This note is for personal notes, sketches, and questions as they come up.

## Pipeline at a glance

1. **Stage 1** — generic detector (SKU110K, 1 class) — "where are the products?"
2. **Stage 2** — brand classifier (`energys`, 34 classes) — "what is each product?"
3. **Stage 3** — shelf grouping — "which row does each product belong to?"
4. **Stage 4** — planogram comparison → green/red overlay

## Phase 3.5 sidecar

- MobileNetV3 embeddings on low-confidence crops
- Nearest-neighbour lookup in local knowledge base
- Auto-save to `/pending_review/` for weekly review
- DBSCAN on unknowns to discover new SKUs

## Things to think about later

- 

## Open sketches

- 

---
Tags: #architecture

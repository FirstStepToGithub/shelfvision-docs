# SKU110K separation

**Status:** accepted
**Area:** architecture

## Context

Tempting to train one big model that both detects and classifies products.

## Decision

SKU110K remains a standalone Stage 1 single-class ("drink") detector. The 34-class brand work stays in `energys`. Do **not** merge them.

## Why

- SKU110K has 11,762 images and 1.7M boxes of pure "generic retail product" data. Mixing in 129 brand-labelled images would dilute it with no benefit.
- Adding a new energy drink brand should not require retraining the generic detector.
- Separation of concerns: Stage 1 = "where are the products", Stage 2 = "what is each product".

## Consequences

- Two models ship in the app, orchestrated by the pipeline.
- Slightly more glue code, much more flexibility.

---
Tags: #decision #architecture

# JPEG uploads to Roboflow (not WebP)

**Status:** accepted
**Area:** data

## Context

Planohero exports WebP by default. WebP is smaller, higher quality, no compression artefacts.

## Decision

Convert to JPEG before uploading to Roboflow.

## Why

- YOLOv8 converts all inputs to 640×640 tensors internally. Format is irrelevant to training quality.
- WebP uploads caused annotation-matching failures in Roboflow. JPEGs work reliably.

## Consequences

- Slight storage overhead during upload; zero impact on training quality.

---
Tags: #decision #data

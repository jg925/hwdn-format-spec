# Contributing

Thanks for helping shape HWDN.

This repository is currently a specification scaffold. Contributions should focus on clarifying the single-canvas format, adding examples, improving schemas, and documenting compatibility behavior.

## Suggested Workflow

1. Update the prose specification in `docs/specification.md`.
2. Update the JSON Schemas in `schemas/` when fields or validation rules change.
3. Add or update examples in `examples/`.
4. Note compatibility implications in `docs/versioning.md`.

## Drafting Principles

- Prefer explicit field names over compact encodings in the baseline format.
- Keep unknown fields forward-compatible where possible.
- Tie interpreted text back to canvas coordinates and source strokes when available.
- Preserve enough stroke data to redraw notes without relying on raster images.
- Document whether each new field is required, optional, or recommended.

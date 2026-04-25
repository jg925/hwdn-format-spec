# HWDN Format Specification

HWDN is a proposed file format for handwritten digital notes. It is designed to preserve the information needed to reproduce ink on a page, attach OCR results to the written content, and carry standard document metadata such as file name, creation date, and optional version history.

This repository contains the scaffolding for defining the `.hwdn` format. The initial draft treats an HWDN file as a ZIP package with JSON documents inside it, which keeps the format portable, inspectable, and easy to validate.

## Repository Layout

- [docs/specification.md](docs/specification.md): human-readable format specification draft.
- [docs/versioning.md](docs/versioning.md): versioning and compatibility policy.
- [schemas/manifest.schema.json](schemas/manifest.schema.json): JSON Schema for package metadata.
- [schemas/note.schema.json](schemas/note.schema.json): JSON Schema for pages, strokes, OCR, and history.
- [examples/basic-note](examples/basic-note): unpacked example `.hwdn` package contents.

## Format Snapshot

A `.hwdn` file is a ZIP archive containing:

```text
manifest.json
note.json
assets/
```

`manifest.json` describes the file-level package metadata and points to the note payload. `note.json` stores pages, stroke geometry, drawing attributes, OCR detections, and optional version history. The `assets/` directory is reserved for embedded backgrounds, thumbnails, or attachments.

## Design Goals

- Reproduce handwritten pages faithfully from stored stroke data.
- Preserve OCR text and positional confidence without requiring OCR to be rerun.
- Keep metadata explicit, including file name, creation date, modification date, author, app identity, and format version.
- Allow version history when an application chooses to retain edit snapshots.
- Keep the baseline format easy to parse with common ZIP and JSON tooling.

## Current Status

This is an early scaffold, not a finalized standard. Field names, required properties, validation rules, and packaging details should be reviewed before being treated as stable.

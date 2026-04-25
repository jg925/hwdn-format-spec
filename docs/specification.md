# HWDN File Format Specification

Status: Draft

Version: 0.1.0

## 1. Overview

HWDN defines a package format for handwritten digital notes. A conforming reader should be able to reproduce the visible handwritten page from stored stroke data and expose OCR detections associated with that page.

The format separates package metadata from note content:

- `manifest.json` identifies the package, format version, creator, timestamps, and payload paths.
- `note.json` stores pages, strokes, OCR results, document metadata, and optional version history.
- `assets/` stores optional binary resources referenced from JSON payloads.

## 2. Package Structure

An `.hwdn` file MUST be a ZIP archive. The following entries are reserved:

```text
manifest.json
note.json
assets/
```

`manifest.json` and `note.json` MUST be UTF-8 encoded JSON files. Unknown files MAY be present for forward compatibility. Readers SHOULD ignore unknown files unless a required feature declares them necessary.

## 3. Coordinate System

Page and stroke coordinates use page-local units. The default unit is CSS pixel unless `document.units` specifies another supported unit.

The origin is the top-left corner of the page. The positive x-axis points right. The positive y-axis points down.

Stroke points are ordered by capture time. A renderer SHOULD connect adjacent points using the stroke's declared interpolation strategy.

## 4. Manifest

The manifest describes the package, not the handwritten content itself.

Required fields:

- `format`: MUST be `hwdn`.
- `formatVersion`: semantic version of the HWDN format used by the package.
- `fileName`: display name for the file.
- `createdAt`: ISO 8601 timestamp for package creation.
- `modifiedAt`: ISO 8601 timestamp for last package modification.
- `payload`: path to the primary note JSON document.

Recommended fields:

- `id`: stable package identifier.
- `createdBy`: app or service that created the package.
- `thumbnail`: path to a package thumbnail.
- `features`: feature flags required or used by the payload.

See [manifest.schema.json](../schemas/manifest.schema.json) for the draft validation schema.

## 5. Note Document

The note document contains the handwritten document model.

Top-level fields:

- `document`: document-level metadata and defaults.
- `pages`: ordered page list.
- `history`: optional version history entries.

### 5.1 Document Metadata

`document` SHOULD include:

- `id`
- `title`
- `createdAt`
- `modifiedAt`
- `units`
- `defaultPageSize`
- `authors`
- `sourceApplication`

### 5.2 Pages

Each page has:

- `id`: stable page identifier.
- `index`: zero-based page order.
- `size`: page width and height.
- `background`: optional page color, ruled paper description, or asset reference.
- `strokes`: ordered stroke list.
- `ocr`: OCR detections for the page.

### 5.3 Strokes

A stroke captures pen input. Each stroke MUST include:

- `id`
- `tool`
- `color`
- `width`
- `points`

Each point MUST include:

- `x`
- `y`

Each point MAY include:

- `t`: timestamp offset in milliseconds from stroke start.
- `pressure`: normalized pressure from `0` to `1`.
- `tiltX`
- `tiltY`
- `azimuth`
- `altitude`
- `width`

Stroke rendering MAY be influenced by:

- `opacity`
- `blendMode`
- `smoothing`
- `interpolation`

### 5.4 OCR

OCR results are page-local detections. OCR entries MAY be organized at multiple levels such as block, line, word, or character.

Each OCR detection SHOULD include:

- `id`
- `type`
- `text`
- `confidence`
- `bounds`
- `strokeRefs`

`strokeRefs` links recognized text back to the source stroke IDs when known.

### 5.5 Version History

`history` is optional. When present, each entry SHOULD include:

- `version`
- `createdAt`
- `author`
- `summary`
- `changeSet`

Applications MAY store lightweight summaries or richer operation logs. The baseline schema leaves `changeSet` open so apps can evolve history storage without changing the core page model.

## 6. Assets

Assets MAY include images, page backgrounds, thumbnails, or attachments. JSON payloads reference assets by relative package paths such as `assets/page-1-background.png`.

Readers SHOULD validate that asset paths remain inside the package and MUST NOT interpret paths as filesystem paths.

## 7. Compatibility

Readers SHOULD reject packages where `format` is not `hwdn`. Readers MAY open packages with newer `formatVersion` values if all required `features` are supported.

Writers SHOULD preserve unknown fields when performing non-destructive edits.

## 8. Validation

Draft JSON Schemas are provided for validation:

- [manifest.schema.json](../schemas/manifest.schema.json)
- [note.schema.json](../schemas/note.schema.json)

Schemas are intended as executable scaffolding for the prose spec. The prose specification should be treated as the source of truth when the two disagree during draft development.

# HWDN File Format Specification

Status: Draft

Version: 0.1.0

## 1. Overview

HWDN defines a package format for handwritten digital notes. Version `0.1.0` is intentionally scoped to one file containing one writable canvas. A conforming reader should be able to reproduce the visible handwritten canvas from stored stroke data and expose OCR detections associated with that canvas.

The format separates package metadata from note content:

- `manifest.json` identifies the package, format version, creator, timestamps, and payload paths.
- `note.json` stores one canvas, strokes, OCR results, and document metadata.
- `assets/` stores optional binary resources referenced from JSON payloads.

## 2. Package Structure

An `.hwdn` file MUST be a ZIP archive. The following entries are reserved:

```text
manifest.json
note.json
assets/
```

`manifest.json` and `note.json` MUST be UTF-8 encoded JSON files. Unknown files MAY be present for forward compatibility. Readers SHOULD ignore unknown files unless a required feature declares them necessary.

## 3. Version 0.1 Scope

Version `0.1.0` supports:

- One `.hwdn` package.
- One `note.json` payload.
- One `canvas` in that payload.
- An ordered list of handwritten strokes.
- Optional OCR detections for recognized text such as `hello world`.
- Standard metadata for the file and note.

Version `0.1.0` does not define multiple pages, layers, version history, collaboration, streaming ink, encryption, imported PDF mapping, or partial-stroke erasing.

## 4. Coordinate System

Canvas and stroke coordinates use canvas-local units. The default unit is CSS pixel unless `document.units` specifies another supported unit.

The origin is the top-left corner of the canvas. The positive x-axis points right. The positive y-axis points down.

Stroke points are ordered by capture time. A renderer MUST draw strokes in array order. Later strokes appear above earlier strokes.

## 5. Manifest

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

## 6. Note Document

The note document contains the handwritten single-canvas model.

Top-level fields:

- `document`: document-level metadata and defaults.
- `canvas`: the writable area, strokes, and OCR detections.

### 6.1 Document Metadata

`document` SHOULD include:

- `id`
- `title`
- `createdAt`
- `modifiedAt`
- `units`
- `canvasSize`
- `authors`
- `sourceApplication`

### 6.2 Canvas

The canvas has:

- `id`: stable canvas identifier.
- `size`: canvas width and height.
- `background`: optional canvas color, ruled paper description, or asset reference.
- `strokes`: ordered stroke list.
- `ocr`: optional OCR detections for the canvas.

### 6.3 Strokes

A stroke captures one continuous pen-down to pen-up gesture. Each stroke MUST include:

- `id`
- `tool`
- `brush`
- `points`

`tool` MUST be one of:

- `pen`
- `pencil`
- `marker`
- `highlighter`
- `eraser`

`brush` MUST include:

- `color`: sRGB hex color such as `#111111`.
- `baseWidth`: nominal stroke width in document units.

`brush` MAY include:

- `opacity`: normalized opacity from `0` to `1`.
- `pressureCurve`: pressure-to-width curve. Version `0.1.0` defines `none` and `linear`.

Each point MUST include:

- `x`
- `y`
- `t`: timestamp offset in milliseconds from stroke start.

Each point MAY include:

- `pressure`: normalized pressure from `0` to `1`.
- `tiltX`
- `tiltY`
- `azimuth`
- `altitude`

For version `0.1.0`, a reader MUST support drawing `pen` strokes as a continuous path through the point sequence using round caps and round joins. If `pressureCurve` is `linear`, the rendered width at a point is `brush.baseWidth * pressure`, clamped to at least 10% of `brush.baseWidth`. If `pressure` is absent or `pressureCurve` is `none`, the rendered width is `brush.baseWidth`.

Writers SHOULD store raw captured points. They MAY also store smoothed points only if those are the points the app intends readers to render.

### 6.4 OCR

OCR results are canvas-local detections. OCR entries MAY be organized at multiple levels such as block, line, word, or character.

Each OCR detection SHOULD include:

- `id`
- `type`
- `text`
- `confidence`
- `bounds`
- `strokeRefs`

`strokeRefs` links recognized text back to the source stroke IDs when known.

## 7. Hello World Example

The example package in [examples/basic-note](../examples/basic-note) stores `hello world` as two handwritten strokes and one OCR line. The structure is:

```json
{
  "document": {
    "id": "doc-hello-world",
    "title": "Hello World",
    "createdAt": "2026-04-25T17:00:00Z",
    "modifiedAt": "2026-04-25T17:05:00Z",
    "units": "px",
    "canvasSize": {
      "width": 1200,
      "height": 800
    }
  },
  "canvas": {
    "id": "canvas-001",
    "size": {
      "width": 1200,
      "height": 800
    },
    "background": {
      "color": "#ffffff",
      "style": "blank"
    },
    "strokes": [
      {
        "id": "stroke-hello-001",
        "tool": "pen",
        "brush": {
          "color": "#111111",
          "baseWidth": 5,
          "opacity": 1,
          "pressureCurve": "linear"
        },
        "points": [
          {
            "x": 120,
            "y": 260,
            "t": 0,
            "pressure": 0.42
          }
        ]
      }
    ],
    "ocr": [
      {
        "id": "ocr-line-001",
        "type": "line",
        "text": "hello world",
        "confidence": 0.92,
        "bounds": {
          "x": 120,
          "y": 180,
          "width": 420,
          "height": 90
        },
        "strokeRefs": []
      }
    ]
  }
}
```

The snippet above is abbreviated to one stroke point for readability. A real handwritten note MUST store enough points to reproduce the visible writing.

## 8. Assets

Assets MAY include images, canvas backgrounds, thumbnails, or attachments. JSON payloads reference assets by relative package paths such as `assets/canvas-background.png`.

Readers SHOULD validate that asset paths remain inside the package and MUST NOT interpret paths as filesystem paths.

## 9. Compatibility

Readers SHOULD reject packages where `format` is not `hwdn`. Readers MAY open packages with newer `formatVersion` values if all required `features` are supported.

Writers SHOULD preserve unknown fields when performing non-destructive edits.

## 10. Validation

Draft JSON Schemas are provided for validation:

- [manifest.schema.json](../schemas/manifest.schema.json)
- [note.schema.json](../schemas/note.schema.json)

Schemas are intended as executable scaffolding for the prose spec. The prose specification should be treated as the source of truth when the two disagree during draft development.

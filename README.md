# HWDN Format Specification

> **Open standard · MIT License · Anyone may implement**

HWDN is an open file format for handwritten digital notes. Version `0.2.0` is
scoped to one file containing one writable canvas. It is designed to preserve
the information needed to reproduce ink on that canvas, attach a structured
text interpretation of the handwritten contents, and carry standard document
metadata such as file name and creation date.

Anyone may build a reader, writer, importer, exporter, or converter for `.hwdn`
files — in any language, in any product, commercial or otherwise — without fee,
royalty, or permission. See [IMPLEMENTING.md](IMPLEMENTING.md) for conformance
rules and [IMPLEMENTATIONS.md](IMPLEMENTATIONS.md) to register your project.

---

## Repository layout

| Path | Purpose |
|------|---------|
| [`docs/specification.md`](docs/specification.md) | Human-readable format specification draft |
| [`docs/versioning.md`](docs/versioning.md) | Versioning and compatibility policy |
| [`schemas/manifest.schema.json`](schemas/manifest.schema.json) | JSON Schema for package metadata |
| [`schemas/note.schema.json`](schemas/note.schema.json) | JSON Schema for one canvas, strokes, and text interpretation |
| [`examples/basic-note/`](examples/basic-note/) | Unpacked canonical `.hwdn` example |
| [`IMPLEMENTING.md`](IMPLEMENTING.md) | Conformance rules for readers and writers |
| [`IMPLEMENTATIONS.md`](IMPLEMENTATIONS.md) | Registry of known implementations |
| [`LICENSE`](LICENSE) | MIT License |

---

## Format snapshot

A `.hwdn` file is a ZIP archive containing:

```text
manifest.json   <- package identity, version, metadata
note.json       <- canvas geometry, ink strokes, text interpretation
assets/         <- reserved for backgrounds, thumbnails, attachments
```

`manifest.json` describes the file-level package metadata and points to the note
payload. `note.json` stores one canvas, stroke geometry, drawing attributes, and
a structured interpretation that other services can read without processing the
ink points directly. The `assets/` directory is reserved for embedded
backgrounds, thumbnails, or attachments.

---

## Design goals

- Reproduce a handwritten canvas faithfully from stored stroke data.
- Preserve readable text, structure, positional context, and source-stroke links
  without requiring handwriting recognition to be rerun.
- Keep metadata explicit: file name, creation date, modification date, author,
  app identity, and format version.
- Keep the baseline format small enough for a custom note app to write and load a
  single handwritten canvas.
- Keep the baseline format easy to parse with common ZIP and JSON tooling.
- Be extensible: unknown keys are ignored, vendor extensions use the `x-` prefix.

---

## Quick start for implementors

1. Read [`docs/specification.md`](docs/specification.md) for the full field
   reference.
2. Validate against the schemas in [`schemas/`](schemas/).
3. Compare your output against the canonical examples in
   [`examples/basic-note/`](examples/basic-note/).
4. Add your project to [`IMPLEMENTATIONS.md`](IMPLEMENTATIONS.md).

---

## Current status

This is an early draft (v0.2.0), not a finalized standard. Field names, required
properties, validation rules, and packaging details may change before v1.0.0.
Breaking changes will be tracked in [`docs/versioning.md`](docs/versioning.md).

---

## Contributing

Issues and pull requests are welcome. If you are proposing a new field or
behavior change, please open an issue first so the rationale can be discussed
before implementation.

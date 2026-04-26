# Implementing the HWDN Format

The `.hwdn` format is an open standard. Anyone may implement a reader, writer,
importer, exporter, or converter for `.hwdn` files — in any programming language,
in any product, commercial or otherwise — without fee, royalty, or permission.

This document explains what conformance means and what to do once you have built
an implementation.

---

## Conformance levels

### Reader (parser / importer)

A conformant reader MUST:

- Detect the `hwdn` magic string at offset 0 of the ZIP central directory comment
  or in `manifest.json → format.magic`.
- Reject files whose `format.magic` value is not `"hwdn"`.
- Parse `manifest.json` and `note.json` according to their published JSON Schemas
  in [`schemas/`](schemas/).
- Preserve all stroke point coordinates without lossy rounding.
- Silently ignore unknown top-level keys in any JSON document (forward compatibility).

A conformant reader SHOULD:

- Respect the `format.version` field and warn the user when encountering a major
  version higher than the one the reader was built against.
- Surface OCR text from `note.json → ocr_layers` when present.
- Preserve `manifest.json → metadata` fields when round-tripping through an editor.

### Writer (serialiser / exporter)

A conformant writer MUST:

- Produce a valid ZIP archive whose central directory comment begins with `hwdn`.
- Include a `manifest.json` that validates against
  [`schemas/manifest.schema.json`](schemas/manifest.schema.json).
- Include a `note.json` that validates against
  [`schemas/note.schema.json`](schemas/note.schema.json).
- Write `format.magic` as the exact string `"hwdn"`.
- Write `format.version` as a semver string (e.g. `"0.1.0"`).
- Write `format.encoding` as `"utf-8"`.
- Include at least one canvas entry in `note.json → canvas`.

A conformant writer SHOULD:

- Write `metadata.created_at` and `metadata.modified_at` as RFC 3339 timestamps.
- Populate `metadata.app_name` and `metadata.app_version` with the producing
  application's identity.
- Validate output against the published schemas before finalising the file.

---

## Validating your implementation

The [`examples/`](examples/) directory contains canonical sample packages. A
correct reader must be able to parse every example without error. A correct writer
must produce output that passes JSON Schema validation against the files in
[`schemas/`](schemas/).

To validate locally using [`ajv-cli`](https://github.com/ajv-validator/ajv-cli):

```bash
# Validate the manifest
ajv validate -s schemas/manifest.schema.json -d examples/basic-note/manifest.json

# Validate the note payload
ajv validate -s schemas/note.schema.json -d examples/basic-note/note.json
```

---

## Registering your implementation

Once your importer, exporter, or app supports `.hwdn`, open a pull request that
adds an entry to [`IMPLEMENTATIONS.md`](IMPLEMENTATIONS.md). Include:

- Project name and URL
- Language or platform
- Conformance level (reader, writer, or both)
- HWDN spec version supported
- Any notes on partial support or known limitations

There is no approval gate. The list is informational.

---

## Extending the format

If you need fields not covered by the current spec, add them under a
vendor-prefixed key (e.g. `"x-myapp-ink-smoothing": true`). The `x-` prefix
signals a private extension. Conformant readers ignore unknown keys, so
extensions are safe to add without breaking existing tools.

If you believe an extension is broadly useful, open an issue in this repository
to propose adding it to the core spec.

---

## License

This specification is licensed under the
[Apache License 2.0](LICENSE). You may implement, fork, or build
commercial products around the `.hwdn` format without restriction.

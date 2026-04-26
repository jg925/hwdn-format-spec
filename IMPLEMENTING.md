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

- Open the `.hwdn` package as a ZIP archive.
- Reject files whose `manifest.json -> format` value is not `"hwdn"`.
- Parse `manifest.json` and `note.json` according to their published JSON Schemas
  in [`schemas/`](schemas/).
- Preserve all stroke point coordinates without lossy rounding.
- Silently ignore unknown top-level keys in any JSON document (forward compatibility).

A conformant reader SHOULD:

- Respect the `formatVersion` field and warn the user when encountering a major
  version higher than the one the reader was built against.
- Surface text from `note.json -> canvas -> interpretation` when present.
- Preserve manifest and document metadata fields when round-tripping through an editor.

### Writer (serialiser / exporter)

A conformant writer MUST:

- Produce a valid ZIP archive.
- Include a `manifest.json` that validates against
  [`schemas/manifest.schema.json`](schemas/manifest.schema.json).
- Include a `note.json` that validates against
  [`schemas/note.schema.json`](schemas/note.schema.json).
- Write `format` as the exact string `"hwdn"`.
- Write `formatVersion` as a semver string (e.g. `"0.2.0"`).
- Include one canvas object in `note.json -> canvas`.

A conformant writer SHOULD:

- Write `createdAt` and `modifiedAt` values as RFC 3339 timestamps.
- Populate `createdBy` and `document.sourceApplication` with the producing
  application's identity.
- Write `note.json -> canvas -> interpretation` when readable handwritten text is
  known or can be produced by the application.
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

This specification is licensed under the [MIT License](LICENSE). You may
implement, fork, or build commercial products around the `.hwdn` format without
restriction.

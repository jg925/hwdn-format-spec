## Review guidelines

- In Codex reviews, check that all changes align with HWDN file format
  communication standards: the prose specification, JSON Schemas, examples, and
  versioning guidance should describe the same package structure, field
  semantics, compatibility behavior, and reader/writer expectations.
- Flag changes that alter required structure, field meaning, validation rules,
  feature-flag behavior, or reader/writer interoperability unless the
  compatibility impact is explicitly documented.
- Require every breaking file format change to include an associated version tag
  or `formatVersion` update that follows `docs/versioning.md`, with incompatible
  required structure or semantic changes reflected as a `MAJOR` version change. Treat these as P0.

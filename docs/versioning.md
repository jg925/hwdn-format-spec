# Versioning Policy

Status: Draft

HWDN format versions use semantic versioning: `MAJOR.MINOR.PATCH`.

## Version Meaning

- `MAJOR`: incompatible changes to required structure or semantics.
- `MINOR`: backward-compatible fields, features, or clarifications.
- `PATCH`: editorial fixes, schema corrections, or non-semantic clarifications.

## Compatibility Rules

Readers SHOULD support all patch versions for a supported major/minor version.

Readers MAY open newer minor versions when unsupported fields can be ignored and no unsupported required feature is declared in `manifest.json`.

Writers SHOULD set `formatVersion` to the lowest version that can represent the written file.

## Feature Flags

`manifest.features` can advertise optional or required capabilities. A feature entry has:

- `name`
- `required`
- `version`

Readers MUST NOT silently open a package when a required feature is unsupported and ignoring that feature would change document meaning.

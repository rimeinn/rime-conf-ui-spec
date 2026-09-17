# Format-1 conformance fixtures

An implementation should accept every file in `valid/` and reject every file in `invalid/`.

## Valid fixtures

| File | Requirement exercised |
| --- | --- |
| `minimal.ini` | Required metadata, implicit `general` group, one scalar field. |
| `grouped.ini` | Explicit group and declaration order. |
| `equals-and-markers.ini` | Only the first `=` separates a property; `#` and `;` remain literal inside values. |
| `list-row-fallback.ini` | An invalid `rows` hint falls back to 3 rather than invalidating the manifest. |
| `file.ini` | Empty and Rime-data-relative file defaults plus non-binding extension hints. |

## Invalid fixtures

| File | Reason for rejection |
| --- | --- |
| `unsupported-format.ini` | `[meta] format` is not `1`. |
| `duplicate-key.ini` | A section repeats a property key. |
| `missing-group.ini` | A field refers to an undeclared explicit group. |
| `unsafe-path.ini` | A field path contains a Rime patch operator. |
| `wrong-specialized-path.ini` | `key_binding_list` is used outside `key_binder/bindings`. |
| `complex-default.ini` | A specialized complex type declares an unsupported default. |
| `absolute-file-default.ini` | A `file` default is an absolute Windows path. |
| `traversing-file-default.ini` | A `file` default contains a `..` path segment. |
| `colon-file-default.ini` | A `file` default contains host-specific colon syntax. |
| `invalid-file-extensions.ini` | A `file` extension hint contains an invalid leading dot. |

These fixtures test the portable manifest format. Host-specific manifest filename discovery is outside this directory.

# Rime Config UI Manifest Specification

This repository defines an implementation-independent INI format for describing configuration user interfaces for
Rime schemas. It is extracted from the `format = 1` schema-settings manifests implemented by
[Rabbit](https://github.com/rimeinn/rabbit).

本仓库将玉兔毫现有的 `<schema_id>.rabbit.ini` 提炼为与编程语言和 GUI 工具包无关的格式规范。清单只描述可编辑的
Rime 配置路径、控件语义、分组和显示文本；它不包含用户的配置值，也不直接包含 YAML patch。

> [!IMPORTANT]
> This is an independent draft based on Rabbit's existing behavior. It is not currently an official Rime project
> standard.

## What the format describes

- ordered pages or groups;
- scalar fields: `boolean`, `integer`, `number`, `string`, and `enum`;
- ordered string lists and Rime key bindings;
- Rime punctuation maps and recognizer patterns;
- Rime switches and the four engine component lists;
- optional defaults, numeric bounds, descriptions, and list row hints.

The normative definition is [SPEC.md](SPEC.md). The specification deliberately defines the INI format itself; it does
not require JSON or any other intermediate representation.

## Minimal example

```ini
[meta]
format = 1
title = Translator settings

[field.enable_completion]
path = translator/enable_completion
type = boolean
label = Enable completion
default = true
```

## Repository layout

- `SPEC.md` — normative format specification;
- `examples/` — complete INI examples, including Rabbit's generic fallback manifest;
- `conformance/` — valid and invalid INI fixtures for implementations;
- `PROVENANCE.md` — source implementation and extraction notes.

## Versioning

The value of `[meta] format` is the format's compatibility boundary. This repository currently specifies only
`format = 1`. Additive clarifications that do not change accepted documents may be published without changing the
format number. Incompatible syntax or semantics require a new format number.

## License

The specification, examples, and conformance material are licensed under the GNU General Public License version 3.
See [LICENSE](LICENSE).

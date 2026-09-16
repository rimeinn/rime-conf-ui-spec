# Rime Config UI Manifest Specification

Status: Draft

Format version: 1

## 1. Scope

A Rime Config UI Manifest is a UTF-8 INI document that describes which values from one effective Rime schema
configuration a graphical application may expose for editing. A manifest describes presentation metadata and value
types. It does not contain the user's current values and does not itself contain a Rime YAML patch.

This specification defines:

- the lexical form of the INI document;
- ordered groups and fields;
- field types and their configuration-value shapes;
- requirements for reading, writing, and restoring declared values;
- the Rabbit format-1 compatibility profile.

This specification does not prescribe a GUI toolkit, visual layout, localization system, or storage API. It also does
not define general editing for arbitrary YAML values.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** describe conformance
requirements.

## 2. Terms

- **producer**: software or a person that creates a manifest;
- **consumer**: software that parses a manifest and presents configuration controls;
- **host**: the application containing the consumer and performing Rime configuration I/O;
- **effective configuration**: the schema configuration after Rime has applied defaults and user customizations;
- **field path**: a slash-separated path in the effective Rime configuration;
- **restore**: removal of the user-owned override and related patch operations so that the schema value becomes
  effective again.

## 3. Encoding and lexical grammar

### 3.1 Encoding

A manifest MUST be encoded as UTF-8. A UTF-8 byte order mark MAY be present. Lines MAY use LF or CRLF endings.

### 3.2 Lines

Before interpretation, a consumer trims leading and trailing whitespace from each line.

- An empty line is ignored.
- A line whose first non-whitespace character is `;` or `#` is a comment and is ignored.
- A section header has the form `[section-name]`.
- A property has the form `key = value` and belongs to the most recent section.
- The first `=` separates the key from the value. Later `=` characters are part of the value.
- Keys and values are trimmed.
- Inline comments and line continuations are not supported. Consequently, `;` and `#` are ordinary characters when
  they occur after the beginning of a property value.

A property before the first section is invalid. An empty section name or property key is invalid. A section MUST NOT
occur more than once, and a key MUST NOT occur more than once within one section.

Producers MUST use the lowercase standard section and property names shown in this specification. Consumers SHOULD
ignore unknown sections and unknown properties so that compatible extensions can be introduced. Consumers MUST reject
an unknown value of the `type` property.

## 4. Document structure

Every manifest MUST contain exactly one `[meta]` section and at least one `[field.<id>]` section. It MAY contain
`[group.<id>]` sections.

The section identifier `<id>` MUST match:

```text
[A-Za-z0-9_-]+
```

Group order is the declaration order of `[group.<id>]` sections. Field order is the declaration order of
`[field.<id>]` sections. A consumer MUST preserve both orders.

If at least one explicit group exists, every field MUST name an existing group. If no explicit group exists, the
consumer creates one implicit group with the identifier `general`; its displayed label is host-defined.

### 4.1 `[meta]`

| Property | Required | Meaning |
| --- | --- | --- |
| `format` | yes | MUST be the exact decimal string `1`. |
| `title` | no | Window or page title. The host supplies a default when absent. |
| `description` | no | Introductory text displayed before the groups. |

### 4.2 `[group.<id>]`

| Property | Required | Meaning |
| --- | --- | --- |
| `label` | yes | Non-empty displayed group label. |
| `description` | no | Explanatory text displayed with the group. |

### 4.3 `[field.<id>]`

| Property | Required | Meaning |
| --- | --- | --- |
| `group` | with explicit groups | Identifier of a declared group. |
| `path` | yes | Path in the effective Rime schema configuration. |
| `type` | yes | One of the field types in section 6. |
| `label` | yes | Displayed field label. |
| `description` | no | Explanatory text displayed with the field. |
| `default` | type-dependent | Fallback used only when the effective configuration does not contain the path. |
| `min` | numeric fields only | Inclusive minimum. |
| `max` | numeric fields only | Inclusive maximum. |
| `options` | `enum` only | `|`-separated enumeration values. |
| `rows` | list-like fields only | Preferred number of visible rows. |

## 5. Field paths and ownership

A field path MUST match:

```text
[A-Za-z0-9_.-]+(?:/[A-Za-z0-9_.-]+)*
```

The path is relative to the root of the effective schema configuration. Empty segments, leading or trailing slashes,
and Rime patch operators such as `+`, `-`, `@0`, or `@after` are forbidden.

Two fields in the same manifest MUST NOT declare the same path. Their paths also MUST NOT be ancestors or descendants
of one another. This avoids ambiguous ownership when a host replaces or restores a complete value. Consumers SHOULD
diagnose such conflicts.

Except where a specialized field type says otherwise, changing a field writes a complete value at its path. A host
MUST preserve configuration outside paths owned by the changed field.

## 6. Field types

Format 1 defines these field types:

| Type | Effective Rime value | Suggested control |
| --- | --- | --- |
| `boolean` | boolean | checkbox or switch |
| `integer` | integer | integer input |
| `number` | integer or floating-point number | numeric input |
| `string` | string | single-line text input |
| `enum` | string | select or combo box |
| `list` | ordered list of strings | ordered list editor |
| `key_binding_list` | `key_binder/bindings` list | Rime key-binding editor |
| `punctuator_map` | punctuation definition map | punctuation-map editor |
| `recognizer_patterns` | recognizer pattern map | tag/pattern editor |
| `switch_list` | Rime `switches` list | switch and option-group editor |
| `engine_lists` | four Rime engine component lists | four ordered list editors |

The suggested control is informative. A conforming consumer MAY use any interaction that preserves the specified value
semantics.

### 6.1 Scalar types

#### `boolean`

The effective value is a Rime boolean. A default, when present, MUST be the lowercase string `true` or `false`.

#### `integer`

The effective value is an integer. Textual defaults and edited values MUST match `^-?\d+$`.

`min` and `max` use the number grammar below and are inclusive. If both exist, `min` MUST NOT be greater than `max`.

#### `number`

The effective value is an integer or floating-point number. Textual defaults, bounds, and edited values MUST match:

```text
^-?(?:\d+(?:\.\d*)?|\.\d+)$
```

Exponent notation, `NaN`, and infinities are not supported. Bounds are inclusive.

#### `string`

The effective value is a string. `default` MAY be empty. Because INI values are trimmed, format 1 cannot express a
default whose leading or trailing whitespace is significant.

#### `enum`

The effective value is a string. `options` is REQUIRED. It is split at `|`; every option is trimmed and MUST be
non-empty and unique. A default, when present, MUST equal one declared option.

Example:

```ini
[field.shift_action]
path = ascii_composer/switch_key/Shift_L
type = enum
label = Left Shift
options = noop|inline_ascii|commit_text|commit_code|clear
default = inline_ascii
```

### 6.2 `list`

The effective value is an ordered list of strings. Reordering is significant. An empty default list MAY be expressed as
an empty property:

```ini
default =
```

Format 1 cannot express a non-empty list default in the manifest.

### 6.3 `key_binding_list`

`key_binding_list` is valid only for the exact path `key_binder/bindings`. The effective value is an ordered list of
maps. Each editable binding contains:

- a non-empty `accept` value;
- an optional `when` value;
- one action key with a non-empty value.

Common action keys are `send`, `toggle`, `select`, `send_sequence`, `set_option`, and `unset_option`. Consumers SHOULD
allow other action keys and MUST preserve unknown map members that they do not edit.

### 6.4 `punctuator_map`

`punctuator_map` is valid only for these exact paths:

- `punctuator/full_shape`;
- `punctuator/half_shape`;
- `punctuator/symbols`.

The effective value is a map from a trigger string to one punctuation definition. Keys in `full_shape` and `half_shape`
MUST be one printable ASCII character from U+0020 through U+007E. Keys in `symbols` MUST be non-empty strings.

A punctuation definition is one of:

- a scalar direct output;
- a non-empty list of scalar candidates;
- a map containing `pair`, whose value is exactly two scalars;
- a map containing `commit`, whose value is a scalar;
- another map preserved as a custom definition.

Editing writes the complete map. Restoring removes the exact override and nested patch operations for that map.

### 6.5 `recognizer_patterns`

`recognizer_patterns` is valid only for the exact path `recognizer/patterns`. The effective value is a map from
non-empty tag strings to scalar pattern strings. Tags MUST NOT contain CR or LF.

Consumers do not need to compile or otherwise validate the regular expressions. Their final syntax is interpreted by
librime's regular-expression implementation. Unknown or unsupported patterns MUST remain representable as text.

Editing writes the complete map. Restoring removes the exact override and nested patch operations for that map.

### 6.6 `switch_list`

`switch_list` is valid only for the exact path `switches`. Its effective value is an ordered list of maps. Each item is
exactly one of:

- a toggle containing a non-empty string `name`; or
- an option group containing a non-empty `options` list of distinct, non-empty strings.

Both forms MAY contain:

- `states`: a list of display strings;
- `abbrev`: a list of abbreviated display strings;
- `reset`: an integer default-state index, or `-1` for no default.

For a toggle, a non-negative `reset` MUST be `0` or `1`. For an option group, it MUST be less than the number of
options. Unknown map members MUST be preserved when an item is edited.

Editing writes the complete `switches` list. Restoring removes the exact override and nested or ordered patch
operations for the list.

### 6.7 `engine_lists`

`engine_lists` is valid only for the exact path `engine`. It represents these four ordered string lists together:

- `engine/processors`;
- `engine/segmentors`;
- `engine/translators`;
- `engine/filters`.

Each entry is a string and MAY use Rime's `<component_type>@<component_name>` expression. A consumer presents and
tracks the four lists independently. Changing or restoring one list MUST NOT rewrite or restore another list. Restoring
a list removes its exact override and its `+`, `-`, and ordered `@...` patch operations.

## 7. Common field properties

### 7.1 `default`

When a path is absent from the effective configuration, a consumer uses `default` if the field type supports it. If the
path is absent and no default exists, the field cannot be loaded and the host SHOULD report a configuration error.

Format 1 supports defaults for `boolean`, `integer`, `number`, `string`, `enum`, and an empty `list`. A producer MUST
NOT specify `default` for the specialized complex types.

### 7.2 `rows`

`rows` applies only to `list`, `key_binding_list`, and `engine_lists`. It is an integer from 1 through 10 inclusive and
is a presentation hint, not a value constraint. For `engine_lists`, the same row count applies to all four lists.

If `rows` is absent, malformed, or outside the range, a format-1 consumer uses 3. Other field types ignore `rows`.

### 7.3 Labels and descriptions

`title`, `label`, and `description` are literal display strings. Format 1 has no built-in translation key syntax or
locale negotiation. A distribution that needs localized manifests must select an appropriate localized file outside
this format or use a future compatible extension.

## 8. Reading, saving, and restoring

A consumer reads each declared field from the effective schema configuration according to its field type. A host MUST
write user changes to a customization layer rather than modifying the installed schema definition.

For librime hosts, the recommended target is `<schema_id>.custom.yaml` under `patch`. A host replacing a list or map
SHOULD remove obsolete exact, nested, append, remove, and ordered patch operations owned by that field before writing
the complete replacement.

Restore is a staged user action: it takes effect when changes are saved. A restore MUST remove only the override and
patch operations owned by the selected field or engine sub-list.

## 9. Errors and forward compatibility

A consumer MUST reject a manifest when any REQUIRED structural or semantic rule in this specification fails. A host
SHOULD report the source file and the failing section or property without exposing an unsafe path as an executable or
filesystem operation.

For forward compatibility, consumers SHOULD ignore unknown sections and unknown properties in known sections.
Consumers MUST reject an unsupported `format` value and an unknown field `type` because their value semantics are not
defined.

Manifests are data, not code. Consumers MUST NOT evaluate property values as commands, expressions, or filesystem paths.

## 10. Rabbit format-1 compatibility profile

Rabbit discovers a manifest for schema identifier `<schema_id>` in this order:

1. `<user_data_dir>/<schema_id>.rabbit.ini`;
2. `<shared_data_dir>/<schema_id>.rabbit.ini`;
3. `<shared_data_dir>/schema.rabbit-fallback.ini`.

The first existing file is authoritative; manifests are not merged. Rabbit accepts schema identifiers matching:

```text
[A-Za-z0-9][A-Za-z0-9_.-]*
```

The filename and discovery rules are a host profile, not part of the core INI syntax. Other applications MAY use a
different suffix or discovery mechanism while consuming the same manifest contents.

Rabbit's original format-1 implementation is the compatibility reference for this draft. See [PROVENANCE.md](PROVENANCE.md).

# Provenance

The initial version of this specification was extracted from Rabbit commit
`583d235f0be7f6d3883a7af443cf2eb379b246cc`:

- repository: <https://github.com/rimeinn/rabbit>;
- manifest parser: `Lib/RabbitSchemaSettingsManifest.ahk`;
- configuration model: `Lib/RabbitSchemaSettingsModel.ahk`;
- specialized value models: `RabbitEngineLists.ahk`, `RabbitPunctuatorMap.ahk`,
  `RabbitRecognizerPatterns.ahk`, and `RabbitSwitchList.ahk`;
- source manifest: `schemas/schema.rabbit-fallback.ini`;
- behavior tests: `tests/component/RabbitSchemaSettingsManifestTest.ahk`.

The specification separates Rabbit's filename/discovery convention from the portable INI contents. It also turns two
existing producer recommendations into explicit requirements: declared field paths must not overlap, and standard
section/property names use their documented lowercase spelling.

The copy of Rabbit's generic fallback manifest under `examples/` is retained as a GPL-3.0 example and compatibility
fixture.

While format 1 remains a draft, the specification and Rabbit may continue to evolve together. The `file` field type
was added after the initial extraction to describe a portable Rime-data-relative path. It resolves from user data with
shared data as a fallback, while only interactive selections under user data are accepted.

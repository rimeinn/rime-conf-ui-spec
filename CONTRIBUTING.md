# Contributing

Changes should preserve interoperability between independent implementations.

- While format 1 remains a draft, syntax and semantics may evolve together with the Rabbit reference implementation.
- After format 1 is declared stable, clarifications must not change which format-1 documents are valid, and new syntax
  or incompatible semantics require a new `[meta] format` value.
- A behavioral change should add or update an INI fixture under `conformance/`.
- Examples must remain valid UTF-8 format-1 manifests.
- The normative specification is `SPEC.md`.

Run basic repository checks before committing:

```powershell
git diff --check
git status --short
```

Parser implementations are encouraged to run every file under `conformance/valid` as an accepted document and every
file under `conformance/invalid` as a rejected document.

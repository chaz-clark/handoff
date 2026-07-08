<!--
Template — contract-change handoff
Use when: your repo needs another repo to change a contract you both depend on
(schema, return shape, function signature, file layout, config key, etc.).

Fill in the placeholders below, then DELETE this comment block.
See CONVENTION.md → Required Metadata Header for field semantics.
-->

**Date:** YYYY-MM-DD
**Author:** <your name or agent + originating repo>
**Direction:** request
**Status:** draft
**Origin:** <one-sentence: what discovered this need (a wiring task, a type mismatch, a downstream consumer breakage)>
**Origin-Commit:** <git-sha in your repo whose state demonstrates the need>
**Topic:** <kebab-case slug matching this filename's <topic> portion>
<!-- Optional:
**Sensitivity:** standard
**Companions:**
- <related handoff or issue> — <relationship>: <one-line note>
-->

---

# Contract change: <one-line summary>

## What contract is changing

<Name the artifact: schema field, function name, return type, config key, file path, etc. Cite the producer's current state by commit hash or path.>

## Current shape (producer)

```
<paste or describe the current contract>
```

## Proposed shape (after this handoff applies)

```
<paste or describe the new contract — diff-style if it helps>
```

## Why this change

<What does your repo need that the current shape can't deliver? Be specific — name the downstream code path or use case in your repo that requires the new shape.>

## Backwards compatibility

<Will existing callers/data still work? If breaking: who else needs to update, and is this a coordinated cutover or an additive change?>

## Suggested apply order

1. <first step the producer should take>
2. <second>
3. …

## Acceptance check

<How will both repos verify the change worked? A test, a type-check, a successful sync, etc.>

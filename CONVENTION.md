# CONVENTION

The handoff convention — file structure, lifecycle, and required metadata for cross-repo design coordination via markdown.

**Version:** 0.1 (working draft)
**Last updated:** 2026-05-13
**Status:** baseline schema covering `Direction`, `Status`, and the required metadata header. Extensions for `Sensitivity` ([#8](https://github.com/chaz-clark/handoff/issues/8)), `Companions` ([#9](https://github.com/chaz-clark/handoff/issues/9)), `REPO_CARD.md` ([#5](https://github.com/chaz-clark/handoff/issues/5)), and `AGENTS_snippet.md` ([#7](https://github.com/chaz-clark/handoff/issues/7)) are deferred to v0.2+. A handoff doc satisfying v0.1's schema will remain valid in v0.2+ — extensions are additive.

---

## Scope

This file codifies:
- What a handoff doc IS (file + lifecycle, not tool).
- WHO authors one (`Direction:` enum — `request` or `deliver`).
- WHERE it lives during each lifecycle stage.
- WHAT structured metadata every handoff must declare (`Status`, `Direction`, `Origin`, `Origin-Commit`, etc.).
- HOW it's named in each repo it touches.

It does NOT yet codify (deferred to v0.2+):

| Deferred issue | What it adds |
|---|---|
| [#8](https://github.com/chaz-clark/handoff/issues/8) `Sensitivity:` | A field + exclusion guidance for what NOT to include (credentials, PII, internal-only data). |
| [#9](https://github.com/chaz-clark/handoff/issues/9) `Companions:` | A typed-cross-link schema between related handoffs/issues. |
| [#5](https://github.com/chaz-clark/handoff/issues/5) `REPO_CARD.md` | Producer capability surface (what handoff types accepted, owner, freeze state). |
| [#7](https://github.com/chaz-clark/handoff/issues/7) `AGENTS_snippet.md` | Recommended prompt prefix teaching consumer agents how to recognize and act on handoff docs. |

---

## Concepts (refresher from `AGENTS.md`)

- **consumer repo** — the repo that ultimately holds the canonical handoff record.
- **producer repo** — the repo that makes the change requested or delivered.
- **handoff doc** — the single markdown artifact describing the coordination, with a defined lifecycle.

---

## Direction enum

Every handoff doc declares a `Direction:` — one of:

| Direction | Initiator | Use case | Canonical analog |
|---|---|---|---|
| `request` | Consumer's maintainer (or its agent) | Consumer needs producer to make a change. The handoff requests that change with the consumer's design context attached. | OpenAI Agents SDK handoff (parent-initiates-transfer) |
| `deliver` | Producer's maintainer (or its agent) | Producer did work that affects consumer and is delivering findings/artifacts/recommendations for the consumer to apply. | Anthropic subagent result (subagent-delivers-to-parent); Google ADK A2A consume-result |

The direction shapes the lifecycle (next section).

---

## Lifecycle by direction

Both directions share four canonical stages — **author → drop → apply → archive** — but the locations and actors differ.

### `request` lifecycle

1. **author** — Consumer authors `handoffs/HANDOFF_<topic>.md` in its own `handoffs/` folder. `Status: draft`.
2. **drop** — Consumer copies the file into producer's root as `<CONSUMER>_HANDOFF_<topic>.md`. `Status: delivered`.
3. **apply** — Producer reviews, applies the change, commits in producer. `Status: applying` → `applied` on the dropped copy.
4. **archive** — Producer deletes the dropped copy. Consumer keeps the canonical record in `handoffs/` forever and updates its own copy's `Status: archived`.

### `deliver` lifecycle

1. **author** — Producer authors a handoff describing the change/finding/artifact it's delivering. Initial location can be ephemeral (producer session output, draft branch); the file's eventual home is the consumer.
2. **drop** — Producer (or its agent) places the file into consumer's `handoffs/` as `<YYYY-MM-DD>_<topic>.md`. `Status: delivered`. Optionally also drops a visibility copy at consumer's root as `<PRODUCER>_DELIVERS_<topic>.md` for next-session visibility.
3. **apply** — Consumer reviews, applies the change in consumer, commits. `Status: applying` → `applied`.
4. **archive** — Consumer deletes the root-level visibility copy if one was dropped. Canonical lives in consumer's `handoffs/` forever. `Status: archived`.

In the `deliver` direction, the consumer is both recipient AND canonical keeper — the producer doesn't retain a copy.

---

## Status enum

The `Status:` field is one of six values:

| Status | Meaning |
|---|---|
| `draft` | Author is still composing; NOT yet visible/delivered to the other repo. |
| `delivered` | Dropped at the destination (producer's root for `request`; consumer's `handoffs/` for `deliver`). Awaiting recipient review. |
| `applying` | Recipient is actively working through the change. |
| `applied` | The change has landed in the receiving repo; the work portion of the handoff is done. |
| `archived` | Handoff is fully settled; transient/dropped copies have been deleted, the canonical record persists in consumer's `handoffs/`. |
| `superseded` | This handoff was replaced by a newer one. The replacement is named in `Companions:` (when codified in v0.2) or in a closing prose note. |

When a handoff's `Status` changes, update the file in place. An optional `## Lifecycle` section at the bottom of the file may log the prior Status + date for audit history.

---

## Required Metadata Header

Every handoff doc opens with a bold-labeled metadata header. Bold-labeled (not YAML frontmatter) is the canonical form — it renders inline in GitHub/IDE previews without requiring a frontmatter parser.

### Required fields (v0.1)

```markdown
**Date:** YYYY-MM-DD
**Author:** <human or agent + originating repo>
**Direction:** request | deliver
**Status:** draft | delivered | applying | applied | archived | superseded
**Origin:** <one-sentence: what produced this handoff (event, design pass, bug)>
**Origin-Commit:** <git-sha in the authoring repo>
**Topic:** <kebab-case slug matching the filename's <topic> portion>
```

### Field guidance

- **Date** — the date the handoff was authored. Not last-updated; use Status transitions or a `## Lifecycle` section for tracking updates.
- **Author** — concrete attribution: `Chaz Clark (canvas-toolbox)`, `make_AGENTS agent (Make-AI-Agents)`, etc. Both human and agent attribution are valid.
- **Direction** — see Direction enum.
- **Status** — see Status enum. Initial value is `draft` until dropped at the destination.
- **Origin** — the EVENT that produced the handoff. Example: "Discovered missing field while wiring AgentJ's `submission_types` round-trip."
- **Origin-Commit** — git SHA of the commit in the AUTHORING repo whose state produced the handoff. Lets the receiver inspect the exact code/design that motivated the work. Analogous to Anthropic's `parent_tool_use_id` (traces a result back to its invocation).
- **Topic** — short kebab-case slug. Must match the `<topic>` portion of the filename so the file is self-locating.

### Reserved-for-v0.2 fields (optional in v0.1)

These may appear in a handoff but their schemas are not yet fixed:

- **Companions:** — list of related handoffs/issues. v0.1 accepts a free-form bullet list; v0.2 will codify a structured schema (per [#9](https://github.com/chaz-clark/handoff/issues/9)).
- **Sensitivity:** — `standard | restricted | internal-only`. v0.1 accepts the field; v0.2 will codify exclusion guidance (per [#8](https://github.com/chaz-clark/handoff/issues/8)). Defaults to `standard` if absent.

### Example header (v0.1, required fields only)

```markdown
**Date:** 2026-05-13
**Author:** make_AGENTS agent (Make-AI-Agents)
**Direction:** deliver
**Status:** delivered
**Origin:** WAVE-2 skill audit found 10 knowledge files lacking the standard companions header; this handoff packages the retrofits + audit findings for canvas-toolbox to apply.
**Origin-Commit:** 8f3a2c1
**Topic:** knowledge-retrofit-and-agent-audit
```

---

## Filename Conventions

| Location | Direction | Filename pattern |
|---|---|---|
| Consumer's `handoffs/` (canonical) | `request` | `HANDOFF_<topic>.md` |
| Producer's root (dropped, transient) | `request` | `<CONSUMER>_HANDOFF_<topic>.md` |
| Consumer's `handoffs/` (canonical) | `deliver` | `<YYYY-MM-DD>_<topic>.md` |
| Consumer's root (visibility copy, transient, optional) | `deliver` | `<PRODUCER>_DELIVERS_<topic>.md` |

`<CONSUMER>` and `<PRODUCER>` are the repo names in capitalized form (e.g., `AGENTJ`, `CANVAS-TOOLBOX`). `<topic>` is kebab-case and matches the `Topic:` field in the metadata header.

---

## Tooling implications

The v0.1 schema enables:

- **Status-grouped views** — `grep -l "Status: delivered" handoffs/` shows what's awaiting application at a glance.
- **Direction filtering** — see who initiated what.
- **Origin-Commit traceability** — clone the originating repo at the named SHA to reproduce the design state.
- **Validator scripts** — a future `handoff_status_check.sh` can validate that every doc in `handoffs/` has the required fields and valid `Status`/`Direction` enum values.

No companion tooling is required for the convention to work; the schema is the contract.

---

## References

The schema and lifecycle are grounded in existing agent-handoff precedent:

- **OpenAI Agents SDK — Handoffs** ([docs](https://openai.github.io/openai-agents-python/handoffs/)): `HandoffInputData` provides a fixed-schema metadata structure for handoff transitions. `Direction: request` mirrors parent-initiates-transfer.
- **Anthropic Claude Agent SDK — Subagents** ([docs](https://code.claude.com/docs/en/agent-sdk/subagents)): Subagent context isolation + `parent_tool_use_id` for traceability inform the `Origin-Commit` field. `Direction: deliver` mirrors subagent-result-returned-to-parent.
- **Google ADK — A2A Protocol** ([intro](https://adk.dev/a2a/intro/)): `AgentCard` at well-known path is the precedent for the future `REPO_CARD.md` ([#5](https://github.com/chaz-clark/handoff/issues/5)).

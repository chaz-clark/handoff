# CONVENTION

The handoff convention — file structure, lifecycle, and required metadata for cross-repo design coordination via markdown.

**Version:** 0.2 (working draft)
**Last updated:** 2026-05-13
**Status:** v0.2 extends v0.1 with `Sensitivity` ([#8](https://github.com/chaz-clark/handoff/issues/8)) and `Companions` ([#9](https://github.com/chaz-clark/handoff/issues/9)) schemas plus exclusion guidance. Both fields are optional and additive — v0.1 docs without them remain valid (Sensitivity defaults to `standard`, Companions defaults to absent). Still deferred to v0.3+: `REPO_CARD.md` ([#5](https://github.com/chaz-clark/handoff/issues/5)) and `AGENTS_snippet.md` ([#7](https://github.com/chaz-clark/handoff/issues/7)).

---

## Scope

This file codifies:
- What a handoff doc IS (file + lifecycle, not tool).
- WHO authors one (`Direction:` enum — `request` or `deliver`).
- WHERE it lives during each lifecycle stage.
- WHAT structured metadata every handoff must declare (`Status`, `Direction`, `Origin`, `Origin-Commit`, etc.).
- HOW it's named in each repo it touches.

It does NOT yet codify (deferred to v0.3+):

| Deferred issue | What it adds |
|---|---|
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

## Sensitivity enum

The optional `Sensitivity:` field declares what content the handoff includes and how recipients should handle it. Defaults to `standard` if omitted.

| Sensitivity | Meaning | Recipient responsibility |
|---|---|---|
| `standard` (default) | Public-shareable. Names projects, design decisions, refactors, public bug context. | None beyond normal review. |
| `restricted` | Names internal services, account IDs, host names, or other not-for-public-publication details. | Do NOT publish or commit to a public branch as-is; sanitize first. |
| `internal-only` | Contains credential-shaped strings, PII, real production data, or content that must never leave private contexts. | MUST sanitize before any commit. Treat with the same care as a `.env` file. |

### Exclusion guidance

Independent of the declared sensitivity, every handoff should avoid carrying content that has no business crossing repo boundaries. Default exclusion list:

- **No credentials.** No API keys, tokens, bearer strings, or passwords — even masked. Masking is a smell, not a fix; the surrounding context that requires the credential to be there is the actual problem.
- **No PII.** No real names, emails, IDs, or addresses of identifiable people. Use placeholders (`user@example.com`, `Alice`, `<student-id>`) when an example is needed.
- **No raw transcripts.** Don't paste full agent/user conversations. Extract the decisions, omit the dialog. Transcripts both leak data and bury the signal in noise.
- **No binary attachments or generated outputs.** Link to the location or describe the artifact; don't embed.
- **No internal hostnames or URLs** that aren't safe to publish. Replace with placeholders if a structural example is needed.

If a handoff genuinely needs to cite restricted content (a specific internal URL, an account ID, etc.), declare `Sensitivity: restricted` (or `internal-only` for credential-shaped or PII data) and rely on the recipient's responsibility column above. The field is the boundary marker.

---

## Companions schema

The optional `Companions:` field cross-links related handoffs, issues, or design docs. Each entry is a bullet with three parts:

```
- <path-or-link> — <relationship>: <one-line note>
```

`<path-or-link>` is a repo-relative path, a cross-repo path, an issue reference (`owner/repo#N`), or a URL. `<relationship>` is one of:

| Relationship | Meaning |
|---|---|
| `supersedes` | This handoff replaces an earlier one (the linked one is now stale). The linked handoff's `Status` should become `superseded`. |
| `superseded-by` | This handoff was replaced by a later one (the link points forward to the replacement). `Status` of this doc should be `superseded`. |
| `prerequisite` | The linked work must land before this handoff can be applied. |
| `follow-up` | The linked handoff/issue extends or builds on this one (one-way pointer; the follow-up may or may not exist yet). |
| `related-discussion` | The link is informational context — relevant prior thinking but not a hard dependency. |

### Example

```markdown
**Companions:**
- canvas-toolbox/handoffs/2026-04-29_three_phase_audit_skill.md — prerequisite: the audit design this handoff builds on
- chaz-clark/Make-AI-Agents#9 — related-discussion: the public issue that motivated the original schema work
- handoff/handoffs/2026-06-01_convention_v0.3_proposals.md — follow-up: planned convention changes informed by this handoff
```

Companions are optional (most handoffs won't have any). When a handoff is `superseded`, the `superseded-by` companion entry is the recipient's canonical pointer to the replacement.

---

## Required Metadata Header

Every handoff doc opens with a bold-labeled metadata header. Bold-labeled (not YAML frontmatter) is the canonical form — it renders inline in GitHub/IDE previews without requiring a frontmatter parser.

### Required fields (v0.2)

```markdown
**Date:** YYYY-MM-DD
**Author:** <human or agent + originating repo>
**Direction:** request | deliver
**Status:** draft | delivered | applying | applied | archived | superseded
**Origin:** <one-sentence: what produced this handoff (event, design pass, bug)>
**Origin-Commit:** <git-sha in the authoring repo>
**Topic:** <kebab-case slug matching the filename's <topic> portion>
```

### Optional fields (v0.2)

```markdown
**Sensitivity:** standard | restricted | internal-only        # defaults to `standard` if omitted
**Companions:**                                                # zero or more entries
- <path-or-link> — <relationship>: <one-line note>
```

### Field guidance

- **Date** — the date the handoff was authored. Not last-updated; use Status transitions or a `## Lifecycle` section for tracking updates.
- **Author** — concrete attribution: `Chaz Clark (canvas-toolbox)`, `make_AGENTS agent (Make-AI-Agents)`, etc. Both human and agent attribution are valid.
- **Direction** — see [Direction enum](#direction-enum).
- **Status** — see [Status enum](#status-enum). Initial value is `draft` until dropped at the destination.
- **Origin** — the EVENT that produced the handoff. Example: "Discovered missing field while wiring AgentJ's `submission_types` round-trip."
- **Origin-Commit** — git SHA of the commit in the AUTHORING repo whose state produced the handoff. Lets the receiver inspect the exact code/design that motivated the work. Analogous to Anthropic's `parent_tool_use_id` (traces a result back to its invocation).
- **Topic** — short kebab-case slug. Must match the `<topic>` portion of the filename so the file is self-locating.
- **Sensitivity** — see [Sensitivity enum](#sensitivity-enum). Optional; defaults to `standard` if omitted. Include explicitly when content is `restricted` or `internal-only` so the recipient knows the boundary.
- **Companions** — see [Companions schema](#companions-schema). Optional; include the field only when relationships to other handoffs/issues exist.

### Example header (v0.2)

```markdown
**Date:** 2026-05-13
**Author:** make_AGENTS agent (Make-AI-Agents)
**Direction:** deliver
**Status:** delivered
**Origin:** Skill audit found 10 knowledge files lacking the standard companions header; this handoff packages the retrofits + audit findings for canvas-toolbox to apply.
**Origin-Commit:** 8f3a2c1
**Topic:** knowledge-retrofit-and-agent-audit
**Sensitivity:** standard
**Companions:**
- canvas-toolbox/handoffs/2026-04-29_three_phase_audit_skill.md — prerequisite: the audit design this handoff builds on
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

The v0.2 schema enables:

- **Status-grouped views** — `grep -l "Status: delivered" handoffs/` shows what's awaiting application at a glance.
- **Direction filtering** — see who initiated what.
- **Origin-Commit traceability** — clone the originating repo at the named SHA to reproduce the design state.
- **Sensitivity filtering** — `grep -L "Sensitivity:" handoffs/` finds docs that omitted the field; `grep -l "Sensitivity: restricted" handoffs/` finds ones flagged for handling care.
- **Companions traversal** — chase chains of related handoffs (supersession, prerequisites, follow-ups) by following the bullet entries.
- **Validator scripts** — a future `handoff_status_check.sh` can validate that every doc in `handoffs/` has the required fields and valid enum values for `Status`, `Direction`, `Sensitivity`, and Companion `<relationship>` tokens. A `handoff_credential_scan.sh` could grep for common credential patterns (`sk-`, `AKIA`, `ghp_`, `Bearer `, etc.) and flag docs missing a `Sensitivity` declaration that's stricter than `standard`.

No companion tooling is required for the convention to work; the schema is the contract.

---

## References

The schema and lifecycle are grounded in existing agent-handoff precedent:

- **OpenAI Agents SDK — Handoffs** ([docs](https://openai.github.io/openai-agents-python/handoffs/)): `HandoffInputData` provides a fixed-schema metadata structure for handoff transitions. `Direction: request` mirrors parent-initiates-transfer.
- **Anthropic Claude Agent SDK — Subagents** ([docs](https://code.claude.com/docs/en/agent-sdk/subagents)): Subagent context isolation + `parent_tool_use_id` for traceability inform the `Origin-Commit` field. `Direction: deliver` mirrors subagent-result-returned-to-parent.
- **Google ADK — A2A Protocol** ([intro](https://adk.dev/a2a/intro/)): `AgentCard` at well-known path is the precedent for the future `REPO_CARD.md` ([#5](https://github.com/chaz-clark/handoff/issues/5)).

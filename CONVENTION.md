# CONVENTION

The handoff convention — file structure, lifecycle, and required metadata for cross-repo design coordination via markdown.

**Version:** 0.3 (working draft)
**Last updated:** 2026-05-13
**Status:** v0.3 closes the originally-deferred extensions. Adds `REPO_CARD.md` ([#5](https://github.com/chaz-clark/handoff/issues/5)) — a producer-side capability surface modeled on Google ADK's `AgentCard`. Adds `AGENTS_snippet.md` ([#7](https://github.com/chaz-clark/handoff/issues/7)) — a paste-into-consumer-AGENTS.md snippet teaching receiving agents to recognize handoff docs as a structured kind, modeled on OpenAI's `RECOMMENDED_PROMPT_PREFIX`. Both are additive to v0.2 (existing handoff docs and consumers without these remain valid).

---

## Scope

This file codifies:
- What a handoff doc IS (file + lifecycle, not tool).
- WHO authors one (`Direction:` enum — `request` or `deliver`).
- WHERE it lives during each lifecycle stage.
- WHAT structured metadata every handoff must declare (`Status`, `Direction`, `Origin`, `Origin-Commit`, etc.).
- HOW it's named in each repo it touches.

No fields or artifacts are currently deferred. Convention is feature-complete relative to issues #1–9; future versions will respond to new issues or real-world usage feedback.

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

## REPO_CARD.md (producer capability surface)

The optional `REPO_CARD.md` is a single-file declaration at a producer repo's root telling consumers what kinds of handoffs the repo accepts, who owns the lifecycle, and where to drop them. Modeled on Google ADK's `AgentCard` (`/.well-known/agent.json`, A2A protocol) but adapted to markdown so it sits next to `AGENTS.md` and `CONVENTION.md` without a separate parser.

A consumer authoring a handoff should read the target producer's `REPO_CARD.md` first — it's the "do you accept what I'm sending?" check before authoring effort is invested.

### Schema (bold-labeled fields, same format as handoff metadata)

```markdown
**Name:** <repo name>
**Description:** <one-sentence summary of what this repo does>
**Owner:** <GitHub handle, team, or `automated` if a bot applies handoffs>
**Status:** accepting | freeze | archived
**Drop-location:** <relative path where `request`-direction handoffs should be dropped; default `./` (repo root)>
**Accepts-handoff-types:** <list of accepted types — see `templates/` names: contract_change, bug_handoff, feature_request, design_proposal>
**Lifecycle-owner:** <who applies handoffs — usually the same as Owner, occasionally different (e.g., `automated` for a bot pipeline)>
**Companion-files:**
- <path> — <one-line note>
```

### Field guidance

- **Name / Description** — concise. Producer-facing summary, not marketing copy. ~1 sentence each.
- **Owner** — concrete (`@chaz-clark`, `@team-foo`). `automated` is valid if a bot applies handoffs without human review.
- **Status** — `accepting` (open), `freeze` (paused during a release or migration), `archived` (no longer maintained; handoffs go elsewhere).
- **Drop-location** — relative path. Default `./` means "drop at repo root." A repo with an `incoming/` folder for queued work would set `Drop-location: incoming/`.
- **Accepts-handoff-types** — template names from `templates/` (Sprint 4). A repo can decline some types (e.g., a stable spec repo might decline `feature_request` but accept `contract_change`). Empty list = accepts nothing right now (often paired with `Status: freeze` or `archived`).
- **Lifecycle-owner** — who's responsible for moving Status `delivered → applying → applied`.
- **Companion-files** — pointers to context the consumer should read alongside the card. Typically `AGENTS.md`, `CONVENTION.md`, repo-specific guides.

### Freeze semantics

When `Status: freeze`, consumers hold handoffs and resume after the freeze lifts. The card may include an optional `**Freeze-until:**` date or condition (e.g., `release 2.0 cut`).

### Example

```markdown
**Name:** canvas-toolbox
**Description:** Local-first Canvas LMS course content sync, audit, and quality-check toolkit.
**Owner:** @chaz-clark
**Status:** accepting
**Drop-location:** handoffs/
**Accepts-handoff-types:**
- contract_change
- bug_handoff
- design_proposal
**Lifecycle-owner:** @chaz-clark
**Companion-files:**
- `AGENTS.md` — project context
- `CONVENTION.md` — handoff convention spec (via `handoff/` clone)
- `lib/agents/knowledge/` — agent knowledge files
```

### Why a separate card instead of inferring from AGENTS.md

`AGENTS.md` is comprehensive — project context, working style, domain terms. A consumer authoring a handoff would have to parse all of that to find "do you accept this type right now?" `REPO_CARD.md` is small, load-bearing for one question, and `grep`-able. Agents can read the card's fields directly without prose parsing.

### Optional JSON variant

Repos preferring JSON (e.g., automated pipelines, A2A-style consumption) may publish `.handoff-card.json` instead with the same field set. Either is valid; `REPO_CARD.md` is the canonical form.

---

## AGENTS_snippet.md (receiving-agent prompt prefix)

The handoff convention is only effective when receiving agents *recognize* handoff docs as a structured artifact with a lifecycle — not as conversation prose. Modeled on OpenAI Agents SDK's `RECOMMENDED_PROMPT_PREFIX` (`agents.extensions.handoff_prompt`), which exists because models without an explicit prefix narrate handoffs in prose ("I'll connect you to billing") instead of acting on them. The cross-repo handoff convention has the same vulnerability: without the prefix, an agent reading a dropped handoff doc may treat it as just-another-markdown.

The canonical snippet lives at `handoff/AGENTS_snippet.md`. Consumers paste it (or a paraphrase preserving its semantics) into their own `AGENTS.md` (Working Style or any load-on-startup location).

### What the snippet teaches receiving agents

1. **Recognition** — files at `handoffs/HANDOFF_*.md`, `handoffs/<YYYY-MM-DD>_*.md`, `<CONSUMER>_HANDOFF_*.md` (at root), or `<PRODUCER>_DELIVERS_*.md` (at root) are HANDOFF DOCUMENTS, not conversation.
2. **Lifecycle awareness** — read `Status:` and `Direction:`. Stages: `draft / delivered / applying / applied / archived / superseded`. Act only on `delivered`.
3. **Surface, don't auto-apply** — summarize the handoff's request/delivery to the human user and get per-decision approval. The convention is per-proposal-approval.
4. **Update Status on apply** — after committing the change, edit the handoff's `Status:` and append a `## Lifecycle marker` entry with the apply date.
5. **Stop on missing referenced artifacts** — if the handoff names files, commits, or agents that don't exist locally, halt and ask. Do not infer; do not fabricate.

### Sixth rule (added in v0.3 with REPO_CARD)

6. **Before authoring an outbound handoff**, read the target producer's `REPO_CARD.md` if present. Confirm `Status: accepting` and that the intended type is in `Accepts-handoff-types`. Drop at `Drop-location`.

### Update cadence

When new `Direction`, `Status`, `Sensitivity`, or filename-pattern values are introduced in future CONVENTION versions, update `handoff/AGENTS_snippet.md` correspondingly so consumer agents recognize the new shapes.

---

## Tooling implications

The v0.3 schema enables:

- **Status-grouped views** — `grep -l "Status: delivered" handoffs/` shows what's awaiting application at a glance.
- **Direction filtering** — see who initiated what.
- **Origin-Commit traceability** — clone the originating repo at the named SHA to reproduce the design state.
- **Sensitivity filtering** — `grep -L "Sensitivity:" handoffs/` finds docs that omitted the field; `grep -l "Sensitivity: restricted" handoffs/` finds ones flagged for handling care.
- **Companions traversal** — chase chains of related handoffs (supersession, prerequisites, follow-ups) by following the bullet entries.
- **REPO_CARD pre-check** — `cat <producer-root>/REPO_CARD.md` before authoring an outbound handoff confirms the producer accepts the type and is currently `accepting` (not `freeze` / `archived`).
- **AGENTS_snippet adoption check** — a consumer's `AGENTS.md` either contains the snippet's five (or six) recognition rules, or the consumer's receiving agent will mishandle handoff docs. `grep -q "handoff document recognition\|Five rules for handling a handoff" <consumer>/AGENTS.md` is a quick adoption probe.
- **Validator scripts** — a future `handoff_status_check.sh` can validate that every doc in `handoffs/` has the required fields and valid enum values for `Status`, `Direction`, `Sensitivity`, and Companion `<relationship>` tokens. A `handoff_credential_scan.sh` could grep for common credential patterns (`sk-`, `AKIA`, `ghp_`, `Bearer `, etc.) and flag docs missing a `Sensitivity` declaration stricter than `standard`. A `repo_card_check.sh` could validate REPO_CARD.md's enum values and required fields.

No companion tooling is required for the convention to work; the schema is the contract.

---

## References

The schema and lifecycle are grounded in existing agent-handoff precedent:

- **OpenAI Agents SDK — Handoffs** ([docs](https://openai.github.io/openai-agents-python/handoffs/)): `HandoffInputData` provides a fixed-schema metadata structure for handoff transitions — informs the v0.1 Required Metadata Header. `RECOMMENDED_PROMPT_PREFIX` (in `agents.extensions.handoff_prompt`) is the direct precedent for v0.3's `AGENTS_snippet.md` — without it, agents narrate handoffs in prose instead of acting on them. `Direction: request` mirrors parent-initiates-transfer.
- **Anthropic Claude Agent SDK — Subagents** ([docs](https://code.claude.com/docs/en/agent-sdk/subagents)): Subagent context isolation + `parent_tool_use_id` for traceability inform the `Origin-Commit` field. `Direction: deliver` mirrors subagent-result-returned-to-parent.
- **Google ADK — A2A Protocol** ([intro](https://adk.dev/a2a/intro/), [consuming quickstart](https://adk.dev/a2a/quickstart-consuming/)): `AgentCard` at well-known path (`/.well-known/agent.json`) is the precedent for v0.3's `REPO_CARD.md` — a single-file capability declaration consumers can read before initiating a handoff.

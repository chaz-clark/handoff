# Handoff — Agile Sprint Plan

Tracks active and completed sprints for the `handoff` convention repo. Updated when stories close on GitHub and the closing commit lands on `main`. Each sprint maps to a discrete deliverable — usually a version of `CONVENTION.md`, occasionally a new artifact (REPO_CARD.md, examples/).

Used by the `gh_issues_agent` skill (step 1 of its workflow): read this file to find the active sprint and identify the next `[ ]` story in order before picking up new work.

---

## Scope

**This repo's stories are design proposals for the handoff convention — not bug tickets.** Bugs found in *other* repos (e.g., `Make-AI-Agents`, `gh-issues-agent`, `canvas-toolbox`) are filed via `gh-issues-agent` against those repos directly — they don't appear here. The split:

| Where it goes | What lives there |
|---|---|
| `chaz-clark/handoff` issues (this file's stories) | Design proposals, schema changes, convention extensions for `CONVENTION.md` and related artifacts. |
| `chaz-clark/<other-repo>` issues (filed via `gh-issues-agent`) | Bugs, contract drift, behaviour mismatches in those other repos discovered while using them. |
| `<repo>/handoffs/HANDOFF_*.md` or `<repo>/handoffs/<date>_*.md` | Cross-repo design coordination — workflow handoffs, NOT bugs. |

If a finding doesn't fit handoff's own design scope, route it via one of the other two channels — don't shoehorn it into this sprint plan.

---

## Sprint Status Legend

- `[ ]` — not started
- `[~]` — in progress
- `[x]` — complete (closed on GitHub + committed to `main`)

---

## Sprint 0 — Bootstrap and Cleanup 🧹

**Goal:** Establish working repo state and resolve internal contradictions before formal design work begins.

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #1 | AGENTS.md contradicts itself: drop "subtreed" language in favor of clone+gitignore | S | `[x]` | c4537a5 |

**Untracked housekeeping** (no GitHub issue; same session):

- Initial seed commit: AGENTS.md, SEED_CONTEXT.md, .gitignore (0449c10)
- Move SEED_CONTEXT.md → archive/, drop stale Active Context bullet (540014a)
- Match gh-issues-agent folder + gitignore to upstream repo rename (37b3885)
- Gitignore .github_issues/ (gh_issues_agent skill local output) (c9c2b5f)

**Lessons:** Inbound tooling pattern = clone + gitignore (NOT `git subtree`). Outbound consumer posture is also clone + gitignore (per #1) — `subtree` is no longer used anywhere in this repo.

---

## Sprint 1 — CONVENTION v0.1 (Direction, Status, Metadata Header) 📋

**Goal:** First formal spec for the handoff convention. Land the schema keystone (#4 metadata header) plus the two fields that live inside it (#3 Status enum, #6 Direction enum).

**Deliverable:** `CONVENTION.md` v0.1

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #4 | Specify required metadata header schema for handoff docs | M | `[x]` | 3f6edb3 |
| #3 | Define Status: enum for handoff doc lifecycle stages | M | `[x]` | 3f6edb3 |
| #6 | Accommodate producer-delivers handoff direction in convention | M | `[x]` | 3f6edb3 |

**Work order:** #4 is the keystone; #3 and #6 are fields/enums inside it. All three authored in a single pass so the schema landed consistent.

**Design decisions made:**

- Header format: bold-labeled (not YAML frontmatter).
- Status enum size: keep all 6 values (`draft / delivered / applying / applied / archived / superseded`).
- Direction names: short form `request` / `deliver` (rather than the originally proposed `consumer-requests` / `producer-delivers`).

---

## Sprint 2 — CONVENTION v0.2 (Sensitivity + Companions) 🔐

**Goal:** Additive schema extensions for content sensitivity and cross-handoff linking. Both optional, additive on v0.1.

**Deliverable:** `CONVENTION.md` v0.2

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #8 | Document input_filter / Sensitivity guidance for handoff docs | M | `[x]` | d805cc8 |
| #9 | Add Companions: cross-link field to handoff metadata for related-handoff chaining | M | `[x]` | d805cc8 |

**Work order:** Bundled — both are optional fields on v0.1's required header; both additive (v0.1 docs without them remain valid).

**Design decisions made:**

- `Sensitivity`: optional with `standard` default (matches issue text).
- `Companions`: 5 relationship types — `supersedes`, `superseded-by`, `prerequisite`, `follow-up`, `related-discussion`.
- Required Metadata Header section split into Required + Optional subsections to accommodate the additions cleanly.

---

## Sprint 2.5 — AGENTS.md Discipline Pointer Fix (delivered by Make-AI-Agents) 🔗

**Goal:** Resolve the `(if present in this repo)` hedge in AGENTS.md Working Style by pointing at the actual clone path. First real-world `deliver` handoff applied under CONVENTION v0.2.

**Deliverable:** One-paragraph rewrite of AGENTS.md Working Style discipline pointer.

**Status:** Complete ✅

**Note: no GitHub story.** This work arrived as a `deliver` handoff from `Make-AI-Agents`, not via the issue tracker. The handoff doc (`handoffs/2026-05-13_AGENTS_md_v3.6_upgrade.md`) is the canonical record.

| Source | Detail | Status | Commit |
|---|---|---|---|
| Handoff: `handoffs/2026-05-13_AGENTS_md_v3.6_upgrade.md` | Rewrite Working Style pointer from `knowledge/behavioral_discipline.md` (broken local resolve) → `Make-AI-Agents/knowledge/behavioral_discipline.md` (resolves via clone) | `[x]` | 3a74db5 |

**Lessons:**

- First applied `deliver` handoff using our own CONVENTION v0.2 lifecycle (dogfood).
- Producer's `AGENTS_updated.md` was generated from an earlier snapshot of `handoff/AGENTS.md` and didn't include in-flight changes (the `knowledge/` Structure tree addition). A straight `mv AGENTS_updated.md AGENTS.md` would have lost the in-flight work. Manual surgical merge via an `AGENTS_temp.md` working copy was the right move.
- The `_temp`-as-working-copy pattern is worth naming as the canonical apply procedure when a delivered handoff competes with local in-flight changes — codify it in CONVENTION v0.3 when REPO_CARD/AGENTS_snippet land.

---

## Sprint 3 — Producer Surface (REPO_CARD + AGENTS_snippet) 🪪

**Goal:** Capability surface so consumers know what a producer accepts before authoring; receiving-agent prompt prefix so agents recognize handoff docs as a structured kind, not prose.

**Deliverable:** `CONVENTION.md` v0.3 (adds REPO_CARD + AGENTS_snippet sections) + new file `handoff/AGENTS_snippet.md` as the paste-ready snippet.

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #5 | Adopt AgentCard-equivalent REPO_CARD.md for producer capability surface | M–L | `[x]` | 7b52a50 |
| #7 | Specify AGENTS_snippet.md content — handoff RECOMMENDED_PROMPT_PREFIX equivalent | M | `[x]` | 7b52a50 |

**Work order:** Both bundled — REPO_CARD is the surface; AGENTS_snippet teaches consumers to read it. Authoring them together let the snippet's sixth rule cite REPO_CARD concretely instead of as a placeholder.

**Design decisions made:**

- REPO_CARD format: bold-labeled `.md` (canonical), with `.handoff-card.json` documented as an optional alternative for repos that prefer JSON.
- REPO_CARD `Status` enum: `accepting / freeze / archived` (three values; sufficient to cover the lifecycle states a producer surface needs).
- AGENTS_snippet length: kept short (~55 lines) so consumers can paste without bloating their `AGENTS.md`. Authored as a real usable file at handoff root, not just a spec.
- AGENTS_snippet semantic count: six rules (the five from issue #7 + a sixth for REPO_CARD pre-check on outbound authoring, naturally added when bundling #5 and #7 together).

---

## Sprint 4 — Examples 📚

**Goal:** Seed `examples/` with real-world handoff docs annotated against the v0.3 schema. Makes the convention concrete.

**Deliverable:** `examples/README.md` cataloging real handoffs in this repo, linking (not copying) to canonical docs in `handoffs/`.

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #2 | Create examples/ folder with first two real-world handoff examples | S | `[x]` | 4ab910e |

**Design decisions made:**

- Strategy: **link, don't copy** — `examples/README.md` points at canonical docs in `handoffs/` rather than duplicating files. Avoids drift; one source of truth per handoff.
- Seeded with the one real-world handoff that exists locally in this repo: `handoffs/2026-05-13_AGENTS_md_v3.6_upgrade.md` (`deliver` direction, applied during Sprint 2.5).
- `request`-direction example marked as pending until a real `request` handoff is authored from/to this repo. Sprint 4 deliberately did NOT fabricate a synthetic example — real > complete.
- The originally-named external examples (canvas-toolbox handoffs, AGENTJ_HANDOFF_*) are out of scope per the project rule: this repo concerns itself with its own state only.

**Lessons:**

- Sprint 4 was tighter in scope than the original issue body proposed because we hewed to the "only worry about our repo" rule. The README's "Adding new examples" section is the canonical hook for organic growth as new handoffs land here.

---

## Completed Sprints (rollup)

| Sprint | Deliverable | Stories closed | Final commit |
|---|---|---|---|
| 0 — Bootstrap and Cleanup | Repo baseline + #1 fix | #1 | c4537a5 |
| 1 — CONVENTION v0.1 | Direction/Status/Metadata schema | #3, #4, #6 | 3f6edb3 |
| 2 — CONVENTION v0.2 | Sensitivity + Companions | #8, #9 | d805cc8 |
| 2.5 — AGENTS.md Discipline Pointer Fix | Working Style pointer rewrite | (delivered via handoff, no GH story) | 3a74db5 |
| 3 — Producer Surface | REPO_CARD spec + AGENTS_snippet.md | #5, #7 | 7b52a50 |
| 4 — Examples | examples/README.md linking to real handoffs in this repo | #2 | 4ab910e |

---

## Open Design Questions (tracked elsewhere)

These live in `archive/SEED_CONTEXT.md` → "Open questions" and `AGENTS.md` Active Context, not as GitHub stories:

- License choice (MIT vs CC0)
- Whether the convention should *require* an `AGENTS.md` reference in consumer repos
- Whether examples use real repo names (lean: yes)
- Whether to add a small ack/archive script

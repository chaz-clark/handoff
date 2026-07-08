---
name: agile-sprint
description: Sprint planning and tracking for handoff convention repo
version: "0.5"
last_updated: 2026-05-22
sprints_completed: 7
current_convention_version: "0.5"
used_by: gh_issues_agent
purpose: Tracks active and completed sprints; read by gh_issues_agent workflow step 1 to find next story
---

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

- Initial seed commit: AGENTS.md, SEED-CONTEXT.md, .gitignore (0449c10)
- Move SEED-CONTEXT.md → archive/, drop stale Active Context bullet (540014a)
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

**Note: no GitHub story.** This work arrived as a `deliver` handoff from `Make-AI-Agents`, not via the issue tracker. The handoff doc (`handoffs/2026-05-13_AGENTS-md-v3-6-upgrade.md`) is the canonical record.

| Source | Detail | Status | Commit |
|---|---|---|---|
| Handoff: `handoffs/2026-05-13_AGENTS-md-v3-6-upgrade.md` | Rewrite Working Style pointer from `knowledge/behavioral_discipline.md` (broken local resolve) → `Make-AI-Agents/knowledge/behavioral_discipline.md` (resolves via clone) | `[x]` | 3a74db5 |

**Lessons:**

- First applied `deliver` handoff using our own CONVENTION v0.2 lifecycle (dogfood).
- Producer's `AGENTS_updated.md` was generated from an earlier snapshot of `handoff/AGENTS.md` and didn't include in-flight changes (the `knowledge/` Structure tree addition). A straight `mv AGENTS_updated.md AGENTS.md` would have lost the in-flight work. Manual surgical merge via an `AGENTS_temp.md` working copy was the right move.
- The `_temp`-as-working-copy pattern is worth naming as the canonical apply procedure when a delivered handoff competes with local in-flight changes — codify it in CONVENTION v0.3 when REPO_CARD/AGENTS-snippet land.

---

## Sprint 3 — Producer Surface (REPO_CARD + AGENTS-snippet) 🪪

**Goal:** Capability surface so consumers know what a producer accepts before authoring; receiving-agent prompt prefix so agents recognize handoff docs as a structured kind, not prose.

**Deliverable:** `CONVENTION.md` v0.3 (adds REPO_CARD + AGENTS-snippet sections) + new file `handoff/AGENTS-snippet.md` as the paste-ready snippet.

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #5 | Adopt AgentCard-equivalent REPO_CARD.md for producer capability surface | M–L | `[x]` | 7b52a50 |
| #7 | Specify AGENTS-snippet.md content — handoff RECOMMENDED_PROMPT_PREFIX equivalent | M | `[x]` | 7b52a50 |

**Work order:** Both bundled — REPO_CARD is the surface; AGENTS-snippet teaches consumers to read it. Authoring them together let the snippet's sixth rule cite REPO_CARD concretely instead of as a placeholder.

**Design decisions made:**

- REPO_CARD format: bold-labeled `.md` (canonical), with `.handoff-card.json` documented as an optional alternative for repos that prefer JSON.
- REPO_CARD `Status` enum: `accepting / freeze / archived` (three values; sufficient to cover the lifecycle states a producer surface needs).
- AGENTS-snippet length: kept short (~55 lines) so consumers can paste without bloating their `AGENTS.md`. Authored as a real usable file at handoff root, not just a spec.
- AGENTS-snippet semantic count: six rules (the five from issue #7 + a sixth for REPO_CARD pre-check on outbound authoring, naturally added when bundling #5 and #7 together).

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
- Seeded with the one real-world handoff that exists locally in this repo: `handoffs/2026-05-13_AGENTS-md-v3-6-upgrade.md` (`deliver` direction, applied during Sprint 2.5).
- `request`-direction example marked as pending until a real `request` handoff is authored from/to this repo. Sprint 4 deliberately did NOT fabricate a synthetic example — real > complete.
- The originally-named external examples (canvas-toolbox handoffs, AGENTJ_HANDOFF_*) are out of scope per the project rule: this repo concerns itself with its own state only.

**Lessons:**

- Sprint 4 was tighter in scope than the original issue body proposed because we hewed to the "only worry about our repo" rule. The README's "Adding new examples" section is the canonical hook for organic growth as new handoffs land here.

---

## Sprint 5 — Public Launch Prep (README + templates + LICENSE + tag) 🚀

**Goal:** Ship the public-facing layer of the convention: a pitch (`README.md`), 4 fill-in scaffolds (`templates/`), a license dedication (`LICENSE`), and the first tag (`v0.3`).

**Deliverable:** `README.md` + `templates/` × 4 + `LICENSE` (CC0 1.0) + git tag `v0.3`.

**Status:** Complete ✅

**Note: no GitHub stories.** Sprint 5 closed out the four non-story items from the Active Context Next steps list. Items 5 and 6 (consumer dogfood, organic example growth) are handled outside the working tree (real-world genchi genbutsu) and as a passive procedure documented in `examples/README.md` respectively.

| Source | Detail | Status | Commit |
|---|---|---|---|
| Next-step item 1 | `README.md` — public pitch + quickstart (clone-and-gitignore adoption, AGENTS-snippet paste, link to spec/templates/examples) | `[x]` | 039e7e8 |
| Next-step item 2 | `templates/` × 4 (contract-change, feature-request, bug-handoff, design-proposal) pre-filled with v0.3 metadata header | `[x]` | 039e7e8 |
| Next-step item 3 | `LICENSE` — CC0 1.0 Universal | `[x]` | 039e7e8 |
| Next-step item 4 | git tag `v0.3` | `[x]` | (tag — points at 039e7e8) |

**Design decisions made:**

- LICENSE: CC0 over MIT. Convention adoption pattern (clone the spec, paste the snippet) benefits from zero attribution friction; MIT would require attribution in every consumer that embeds AGENTS-snippet.
- Template direction: all 4 templates default to `Direction: request` since they're consumer-initiates flows. `deliver`-direction handoffs are producer-authored and template less well — adapt one of the 4 if needed.
- README scope: kept under 70 lines. Lands the pitch, the quickstart, and the layout map; depth lives in `CONVENTION.md`.
- Tag granularity: single `v0.3` covering the full feature-complete state (Sprints 0–5). Did not retro-tag v0.1 and v0.2; the commit history is the audit trail.

**Lessons:**

- Sprint 5 was lighter than Sprints 1–3 because the design decisions were already made; this sprint was mostly authorship + publishing. A "launch prep" sprint at the end of a spec project consistently looks like that.
- `genchi genbutsu` reframing of item 5 (consumer dogfood) is worth remembering: a convention proves itself in real use, not in synthetic test repos. Issues filed against `chaz-clark/handoff` from real consumer agents will be the v0.3 validation signal.

---

## Sprint 6 — CONVENTION v0.4 (authoring guidance from real-session friction) 📝

**Goal:** Bake two session-derived authoring lessons into the spec as doc-only additions, so future spec authors don't re-discover them.

**Deliverable:** `CONVENTION.md` v0.4 — adds `_temp`-as-working-copy apply procedure under `deliver` lifecycle ([#10](https://github.com/chaz-clark/handoff/issues/10)), and a new top-level "Authoring guidance — avoiding self-referential commit hashes" section ([#11](https://github.com/chaz-clark/handoff/issues/11)).

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #10 | Codify the `_temp`-as-working-copy pattern as canonical apply procedure | S | `[x]` | 5e9452d |
| #11 | Document the self-referential commit-hash pattern: prefer two-commit over `--amend` | XS | `[x]` | 5e9452d |

**Work order:** Bundled — both doc-only, both derived from session lessons (#10 from Sprint 2.5, #11 from the same self-referential dance that needed the backfill commit `9228151`). Single commit since neither conflicts with the other and both land in adjacent sections.

**Design decisions made:**

- Insertion placement: #10 as a `####` sub-subsection of `### deliver lifecycle` (it's specifically the deliver-apply procedure, so it lives with the lifecycle). #11 as a new top-level `##` section "Authoring guidance" (it's spec-wide guidance for anyone writing tracked lifecycle records, not deliver-specific).
- Scope tightness: #10 names the procedure as **conflict-only** — for trivial `deliver` applies with no in-flight changes, a straight `mv` is still valid. The `_temp` workflow isn't the default for every apply.
- #11 added three patterns rather than picking one: two-commit (preferred), `--amend` warning (anti-pattern), date-based references (precision-trading fallback). Real-world authoring sometimes wants each.

**Lessons:**

- Sprint 6 felt very natural — the issues were filed from session reflections at v0.3 ship time, then resolved in a single small sprint. Closing-the-loop sprint pattern: ship → reflect → file → resolve in the next sprint.

---

## Sprint 7 — CONVENTION v0.5 (internal handoffs / parking lots) 🅿️

**Goal:** Add a third handoff direction for context that crosses a *session* boundary rather than a repo boundary — a handoff to a future session of the same repo. Formalize the parking-lot pattern that emerged in canvas-toolbox.

**Deliverable:** `CONVENTION.md` v0.5 (adds `Direction: internal`, `Status: parked`, and an "Internal handoffs (self-directed parking lots)" section) + two seeded files `handoffs/parkinglot.md` and `handoffs/long-term-parking.md` + a 7th `AGENTS-snippet.md` recognition rule.

**Status:** Complete ✅

| # | Story | Size | Status | Commit |
|---|---|---|---|---|
| #12 | `parkinglot.md` convention for per-repo internal deferred-work tracking | M | `[x]` | 61896b0 |

**Work order:** Single deliverable commit (61896b0) for spec + snippet + both seed files + AGENTS.md, then this marker commit referencing it (two-commit pattern per v0.4 authoring guidance — no `--amend` self-reference).

**Design decisions made:**

- **Two files, split by ripeness, not date.** `parkinglot.md` = near-term, capacity-gated ("busy now"); `long-term-parking.md` = far/someday, evidence-gated. The user's redefinition mid-design replaced an earlier dated-session-snapshot idea — the active sprint/roadmap state stays in `agile-sprint.md`, so the two parking files are purely the *deferred* tiers of the backlog.
- **`internal` is a real `Direction` value** (not "no Direction" and not a special-case): a self-handoff *is* a handoff, so it declares a Direction like every other doc. Resolved the "sentinel vs absent" fork in favor of an explicit value.
- **`Trigger:` is the load-bearing field** — it's what makes a parking lot ≠ a TODO list (items are condition-gated = Definition of Ready). `Routes-to:` records the graduation destination (active-work | issue | handoff).
- **Melting-pot framing**, agile-forward: agile gives the structure (backlog tiers, refinement, DoR/DoD, retro origin of the "parking lot"); lean/Kaizen gives the restraint (JIT pull, muda avoidance — no fake-WIP inventory); TBP is the problem-solving engine an item enters once triggered; genchi genbutsu grounds triggers in observed need. Deliberately kept OUT: story points, velocity, burndown. Kept IN: "atomic story, not epic."
- **Refinement = backlog grooming**, run as a lean Check-Act review on each `agile-sprint.md` close; `parkinglot.md` churns fast, `long-term-parking.md` churns slow (reviewed for "has evidence arrived?"). Files carry a `Last-refined:` date.
- **Seeded with real deferred work, not padding** (Sprint 4 "real > complete" lesson): `bootstrap_tooling.sh` moved out of AGENTS.md Active Context into `long-term-parking.md` (its proper home); the `handoff_status_check.sh` validator parked near-term.

**Lessons:**

- The design converged through several reframes (kanban → lean → agile-melting-pot → corrected file semantics). Letting the framing settle *before* writing the spec — rather than coding the first proposal — kept the schema cost minimal (two enum values + one section) for a conceptually large addition.
- v0.5 is the first **non-additive-in-spirit** extension: it widens what "handoff" *means* (session boundary, not just repo boundary). Held the line on backward compatibility — existing `request`/`deliver` docs and consumers are untouched.

---

## Completed Sprints (rollup)

| Sprint | Deliverable | Stories closed | Final commit |
|---|---|---|---|
| 0 — Bootstrap and Cleanup | Repo baseline + #1 fix | #1 | c4537a5 |
| 1 — CONVENTION v0.1 | Direction/Status/Metadata schema | #3, #4, #6 | 3f6edb3 |
| 2 — CONVENTION v0.2 | Sensitivity + Companions | #8, #9 | d805cc8 |
| 2.5 — AGENTS.md Discipline Pointer Fix | Working Style pointer rewrite | (delivered via handoff, no GH story) | 3a74db5 |
| 3 — Producer Surface | REPO_CARD spec + AGENTS-snippet.md | #5, #7 | 7b52a50 |
| 4 — Examples | examples/README.md linking to real handoffs in this repo | #2 | 4ab910e |
| 5 — Public Launch Prep | README.md + templates/ × 4 + LICENSE (CC0) + tag v0.3 | (no GH stories — non-story items) | 039e7e8 |
| 6 — CONVENTION v0.4 | _temp apply procedure + commit-hash authoring guidance | #10, #11 | 5e9452d |
| 7 — CONVENTION v0.5 | internal handoffs (parkinglot.md + long-term-parking.md) | #12 | 61896b0 |

---

## Open Design Questions (tracked elsewhere)

These live in `archive/SEED-CONTEXT.md` → "Open questions" and `AGENTS.md` Active Context, not as GitHub stories:

- License choice (MIT vs CC0)
- Whether the convention should *require* an `AGENTS.md` reference in consumer repos
- Whether examples use real repo names (lean: yes)
- Whether to add a small ack/archive script

# Handoff: AGENTS.md v3.6 upgrade proposal (from Make-AI-Agents)

**Date:** 2026-05-13
**Author:** Claude in Make-AI-Agents (delivering to handoff)
**Origin:** Cross-repo AGENTS.md audit sprint, step 3 (regenerate). See `Make-AI-Agents/AGENTS.md` → Active Context → Next likely for the workstream definition. Source commit: `ba6b98a` (Make-AI-Agents).
**Status:** applied (2026-05-13)
**Direction:** deliver — this handoff itself is a real-world test of the `deliver` direction codified in CONVENTION v0.2.
**Sensitivity:** standard.
**Companions:**
- `chaz-clark/handoff` issues #1–9 (filed 2026-05-13) — concrete recommendations for v0.2 of this convention
- `canvas-toolbox/handoffs/2026-05-13_knowledge_retrofit_and_agent_audit.md` — sibling producer-delivers handoff (canvas-toolbox is the second real example after this one)

---

## What this proposes

**One surgical change** to `AGENTS.md`: rewrite the Working Style discipline pointer from the canonical-path form to the cloned-subdirectory form, so the pointer actually resolves to a file that exists locally.

### The change

**Current (line 43):**
> "This project follows the behavioral discipline defined in `knowledge/behavioral_discipline.md` (if present in this repo) or the equivalent discipline loaded via the host tool's skill system."

**Proposed:**
> "This project follows the behavioral discipline defined in `Make-AI-Agents/knowledge/behavioral_discipline.md` — the discipline file lives inside the `Make-AI-Agents/` clone at this repo's root (gitignored per the clone+gitignore convention noted in Project-specific rules below). To refresh the discipline: `cd Make-AI-Agents && git pull`."

The rest of the Working Style paragraph (no-override principles list) is preserved verbatim.

---

## Why

`Make-AI-Agents` raised the make_AGENTS bar on 2026-05-13 with **AGENTS-QC-006** (severity: critical) — every generated AGENTS.md's Working Style pointer must resolve to a file that actually exists locally in the target repo. Three PASS conditions:

- **(a)** Canonical: `<repo>/knowledge/behavioral_discipline.{md,json}` exist at the canonical path
- **(b)** Cloned subdirectory: pointer references a path like `Make-AI-Agents/knowledge/behavioral_discipline.md` AND that path resolves through a full Make-AI-Agents clone at the repo's root
- **(c)** Skill-system: the host tool loads the equivalent discipline at runtime (must be declared explicitly in Working Style)

This repo's current Working Style references condition (a) — `knowledge/behavioral_discipline.md`. But the file does NOT exist at that path here:

```
$ ls knowledge/behavioral_discipline.md
ls: knowledge/: No such file or directory
```

The file DOES exist via the clone:

```
$ ls Make-AI-Agents/knowledge/behavioral_discipline.{md,json}
Make-AI-Agents/knowledge/behavioral_discipline.json
Make-AI-Agents/knowledge/behavioral_discipline.md
```

This repo already documents the clone+gitignore pattern as the canonical pattern (see this repo's own Project-specific rules section). The pointer rewrite aligns the Working Style with the actual local layout.

---

## What this does NOT touch

- **No content rewrite.** Title, Project Purpose, Structure, Active Context, Domain Terms — all preserved verbatim.
- **No discipline-file packaging into `knowledge/`.** The clone already provides the file via condition (b). Packaging a redundant snapshot into `handoff/knowledge/` is technically valid (condition a) but adds a sync burden when the discipline changes upstream. Cleaner to use the existing clone.
- **No project-state changes.** Active Context bullets (lines 80–96) stay as-is; refresh those on your own cadence when project state changes.

---

## Audit verdict (full, applied 2026-05-13)

| Check | Result |
|---|---|
| AGENTS-QC-001 (required sections present, in order) | PASS |
| AGENTS-QC-002 (Working Style discipline pointer present) | PASS — pointer text is canonical, names the four no-override principles |
| AGENTS-QC-003 (Active Context date stamp present) | PASS — `_Last updated: 2026-05-13_` |
| AGENTS-QC-004 (no tool-specific language) | PASS — tool names appear in a clearly tool-agnostic list ("Claude Code, Cursor, Aider, Antigravity") describing audience, not "When using X, do Y" |
| AGENTS-QC-005 (optional sections meet inclusion criteria) | PASS — Domain Terms is warranted (handoff vocabulary is dense: handoff vs handoffs/ vs handoff/, author/drop/apply/archive, producer vs consumer) |
| AGENTS-QC-006 (discipline files co-located) | **FAIL → PASS after this change** — the rewrite swaps from condition (a) reference (broken) to condition (b) reference (resolves via clone) |

**Net status:** this is the cleanest audit target in the current sprint. Single-line pointer fix; everything else is canonical.

---

## Companion files dropped with this handoff

- `AGENTS_updated.md` (at this repo's root) — the proposed file. Identical to current `AGENTS.md` except for the Working Style pointer rewrite.

---

## Recommended apply workflow for the handoff maintainer

1. **Review the diff** between `AGENTS.md` (current) and `AGENTS_updated.md` (proposed). Should be one change: the Working Style paragraph's pointer-and-refresh sentence.
2. **Verify the clone path resolves locally**: `ls Make-AI-Agents/knowledge/behavioral_discipline.md`. (It does as of 2026-05-13.)
3. **If approved**: `mv AGENTS_updated.md AGENTS.md` (overwrites current). Or manually copy the new pointer text over the old one.
4. **Consider this handoff itself as the seed for `examples/`** — `chaz-clark/handoff` issue #2 proposes creating an `examples/` folder for real-world reference handoffs. This handoff (producer-delivers from Make-AI-Agents) is the second concrete example of the convention in use (alongside `canvas-toolbox/handoffs/2026-05-13_knowledge_retrofit_and_agent_audit.md`). The maintainer may want to copy or symlink-document this file into `examples/` when that folder is created. The doc itself stays in `handoffs/` as the canonical consumer record.
5. **Optionally close `chaz-clark/handoff` issue #5** (AgentCard / REPO_CARD.md) — partly addressed by the explicit clone-path declaration in the new pointer text. Not a complete close, but a partial step toward "tell consumers where things actually live."
6. **No push needed** if you want to keep this local-only for review; otherwise commit and push when ready.

---

## Lifecycle marker

- **Authored:** 2026-05-13 (Make-AI-Agents @ commit `ba6b98a`)
- **Delivered:** 2026-05-13 — dropped to `handoff/handoffs/2026-05-13_AGENTS_md_v3.6_upgrade.md` (canonical consumer location per CONVENTION v0.2 `deliver` lifecycle), with visibility copy at `handoff/AGENTS_updated.md`.
- **Applied:** 2026-05-13 — Working Style discipline pointer rewritten in `handoff/AGENTS.md` via the `_temp` working-copy workflow; visibility copy `AGENTS_updated.md` deleted from root.
- **Direction-rename note:** Field value updated from `producer-delivers` (issue #6 proposed wording) to `deliver` (CONVENTION v0.1 canonical short form) at apply time. This handoff was authored before v0.1 landed.
- **Status:** applied
- **Disposition:** kept as canonical consumer record. To be copied (or symlink-documented) into `examples/` once Sprint 4 lands (per issue #2; this handoff is named explicitly in #2 as one of the seed examples).

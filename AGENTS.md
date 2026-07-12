# handoff

A tool-agnostic convention for cross-repo design coordination via markdown files.

## Project Purpose

**This is**:
- A lightweight convention defining where cross-repo handoff docs live (`handoffs/` at the consumer repo root), how to name them, their lifecycle (author → drop → apply → archive), and a markdown structure readable by humans and agentic dev tools alike.
- Designed to be **cloned at the consumer repo root and gitignored**, so consumers pick up templates by `git pull` inside the clone — no subtree mechanics in host history.
- A pairing with the `agents.md` precedent: an open convention, not a tool.

**This is NOT**:
- A tool, daemon, CLI, or workflow engine. (A small validator/lister script may come later; the convention works without it.)
- A replacement for GitHub Issues — Issues are good for public, conversation-heavy work; handoffs are for private or solo cross-repo design coordination where context lives next to the code.
- AI-agent-specific — agents read these well, but the pattern works for any cross-repo workflow.

**Audience**: Maintainers of multiple related repos (humans, plus agentic dev tools such as Claude Code, Cursor, Aider, Antigravity) who need lightweight cross-repo coordination.

## Structure

```
handoff/
├── AGENTS.md             # this file — project context for any agentic tool
├── AGENTS-snippet.md     # paste-into-consumer-AGENTS.md snippet teaching the handoff convention to receiving agents
├── CONVENTION.md         # the formal spec (Direction inc. internal, Status inc. parked, metadata header, lifecycle, filename conventions, REPO_CARD, AGENTS-snippet, internal handoffs)
├── LICENSE               # CC0 1.0 Universal — public-domain dedication
├── README.md             # public pitch + quickstart for consumers
├── archive/              # origin / historical docs (currently: SEED-CONTEXT.md, kept as provenance after README superseded it)
├── examples/             # README pointing at real-world handoffs in this repo (currently: 1 deliver-direction example)
├── handoffs/             # canonical record of handoffs for this repo: the 2026-05-13 Make-AI-Agents deliver (applied) + the two internal parking lots (parkinglot.md, long-term-parking.md)
├── knowledge/            # local state for tooling (currently: agile-sprint.md for the gh_issues_agent skill workflow)
├── templates/            # 4 scaffolds: contract-change, feature-request, bug-handoff, design-proposal
├── .gitignore            # ignores Make-AI-Agents/ and gh-issues-agent/ (local-only tooling clones)
└── .gitattributes
```

## Working Style

This project follows the behavioral discipline defined in `Make-AI-Agents/knowledge/behavioral_discipline.md` — the discipline file lives inside the `Make-AI-Agents/` clone at this repo's root (gitignored per the clone+gitignore convention noted in Project-specific rules below). To refresh the discipline: `cd Make-AI-Agents && git pull`. For the full principles and override rules, see that file → "The Ten Principles". The four no-override principles (P-001 Read Before Claiming, P-003 Stop on Defect, P-007 Pull Don't Push, P-010 Respect Intent) apply unconditionally.

**Project-specific rules**:
- Keep convention docs tool-agnostic. No "when using Claude Code, do X" or "in Cursor only" language in `README.md`, `CONVENTION.md`, or templates. Tool-specific guidance belongs in tool-specific config files, not in this repo.
- `Make-AI-Agents/` and `gh-issues-agent/` are **local-only clones** of the repos that host the skills used here (`make_AGENTS` / `make_AGENTS_qc` and `gh_issues_agent`, respectively). Both are gitignored and must never be committed or pushed. To refresh: `git -C <dir> pull`. To re-clone fresh: remove the folder and run `git clone https://github.com/chaz-clark/<repo>.git` from this repo's root. (Avoid `git subtree` here — we don't want this content in tracked history; clone-and-gitignore is cleaner.)
- Singular vs plural matters: `handoff` is the repo and the convention name; `handoffs/` (plural) is the folder name inside consumer repos. Don't conflate them in docs.
- Consumer posture is **clone-and-pull only** — this repo is the canonical source. Consumers clone it at their repo root, gitignore the folder, and refresh via `git pull` inside the clone. They do not push back upstream.
- **GitHub Issue vs handoff doc — when to use which**:
  - **GitHub Issue**: a concrete bug, contract drift, or improvement in another **public** repo that any user of that repo could hit. Issues are discoverable, trackable in the GitHub UI, and visible to maintainers and other consumers. Use `gh issue create --repo <owner>/<repo>` from this working directory.
  - **Handoff doc** (`handoffs/HANDOFF_<topic>.md`): a **design coordination** request to another repo discovered while working locally — typically a request that needs cross-repo context the producer's maintainer wouldn't have without reading the consumer's design state. Authored in the consumer's `handoffs/` folder, dropped into the producer's root as `<CONSUMER>_HANDOFF_<topic>.md` when ready, deleted from the producer once applied.
  - **Decision heuristic**: if the producer's maintainer can act on it knowing only the producer's own state, it's an Issue. If they need the consumer's design context to act on it correctly, it's a handoff. Bug reports and contract violations are almost always Issues; "I need this field added to your schema because of how my repo uses it" is almost always a handoff.
  - Working examples (this repo): the `make_AGENTS` ↔ `make_AGENTS_qc` Rule 6 contract drift was a public bug → filed as [Make-AI-Agents#9](https://github.com/chaz-clark/Make-AI-Agents/issues/9). The `AGENTJ_HANDOFF_folder_io_update.md` cross-repo design coordination is the canonical handoff example, to land in `examples/`.

## Toyota Quality Loop

Every task must complete the quality loop: **Prevent → Detect → Verify**.

### 1. Genchi Gembutsu (現地現物) - Go and See

**Don't assume, verify with real data:**
- Test with REAL user data, not synthetic fixtures
- When uncertain about format, examine actual files
- Verify in real environment, don't trust docs alone
- Read actual code before claiming understanding

**Behavioral trigger**: When you say "probably" or "should" → STOP and verify

### 2. Jidoka (自働化) - Built-in Quality / Stop on Defect

**Build quality in, stop when defect detected:**
- Write tests WITH code, not after
- Red tests block progress - fix immediately, don't defer
- Validation runs automatically (not manual step)
- Can't merge/export with errors (blocked by design)

**Behavioral trigger**: When you want to say "we'll fix this later" → STOP and fix now

**Aligns with**: P-003 Stop on Defect

### 3. Poka-yoke (ポカヨケ) - Mistake-Proofing

**Design so mistakes can't happen:**
- Automate validation (no manual steps)
- Use pre-commit hooks to catch errors
- Type hints catch errors at write-time
- Block operations that would create defects

**Behavioral trigger**: When manual verification required → Design it out

---

**The Quality Loop in action:**

When you find a defect:
1. **Fix it** (Jidoka - stop and correct)
2. **Verify the fix** (Genchi Gembutsu - test with real data)
3. **Prevent recurrence** (Poka-yoke - add automated check)

## Active Context

_Last updated: 2026-05-22_

- Convention is feature-complete at v0.5: `CONVENTION.md` codifies the full schema (Direction `request`/`deliver`/`internal`, Status incl. `parked`, metadata header, Sensitivity, Companions, REPO_CARD producer surface, AGENTS-snippet receiving-agent prefix) plus v0.4 authoring guidance (`_temp`-as-working-copy apply, two-commit pattern) and v0.5 **internal handoffs** (self-directed parking lots — `handoffs/parkinglot.md` near-term + `handoffs/long-term-parking.md` someday, `Trigger:`-gated). Paste-ready `AGENTS-snippet.md` (now 7 rules) lives at root; `templates/` holds 4 scaffolds; `examples/` seeded with one real-world `deliver` example; `README.md` published; `LICENSE` is CC0 1.0. All 9 founding stories + 3 enhancement issues (#10, #11, #12) closed.
- `Make-AI-Agents/` and `gh-issues-agent/` are gitignored local clones (each has its own `.git`) so the `make_AGENTS` / `make_AGENTS_qc` and `gh_issues_agent` skills are available in this working directory. Refresh via `git pull` inside each folder; never committed or pushed from this repo.
- **Deferred work now lives in the parking lots** (v0.5 dogfood): the old `bootstrap_tooling.sh` parked-idea bullet moved to `handoffs/long-term-parking.md`; near-term ideas (e.g. a `handoff_status_check.sh` validator) live in `handoffs/parkinglot.md`. Review them at each sprint close (backlog refinement) and bump their `Last-refined:` date.
- **Next steps**: tag `v0.5` lands with this sprint. No active sprint work beyond that — convention is feature-complete. Dogfood is **already happening** in consumer repos (bugs surface as GitHub issues filed here via the `gh_issues_agent` skill running there — genchi genbutsu). `examples/` grows organically as new real handoffs land (see `examples/README.md` → "Adding new examples").
- **Open design questions** (from `archive/SEED-CONTEXT.md` → "Open questions"; ~~license choice~~ → resolved as CC0): whether examples use real repo names (lean: yes); whether to add a small ack/archive script. (The "require `AGENTS-snippet`/`AGENTS.md` reference in consumers?" question now sits in `handoffs/long-term-parking.md`, evidence-gated.)

## Domain Terms

| Term | Definition |
| --- | --- |
| handoff (noun) | A markdown file describing a request from one repo to another, with a defined lifecycle. The single artifact; no companion tooling required. |
| consumer repo | The repo that **authors** a handoff because it needs a change from another repo. Canonical copy of the handoff lives here, in `handoffs/`. |
| producer repo | The repo that **receives** a dropped handoff and applies the requested change. Holds a temporary prefixed copy at its root until applied, then deletes it. |
| author / drop / apply / archive | The four lifecycle stages of a handoff. *Author* in the consumer; *drop* a copy into the producer's root; *apply* the changes; *archive* by deleting the producer's copy (the consumer keeps the canonical record forever). |
| `handoffs/` (folder, plural) | The folder inside a **consumer** repo holding canonical handoff docs (e.g., `handoffs/HANDOFF_<topic>.md`). |
| `handoff/` (folder, singular) | The clone directory inside a consumer repo (created via `git clone https://github.com/chaz-clark/handoff.git` at the consumer's root, then added to the consumer's `.gitignore`). Read-only by convention; holds the spec and templates. |
| `HANDOFF_<topic>.md` | Naming pattern in a consumer's `handoffs/` folder. The `HANDOFF_` prefix makes intent obvious at a glance. |
| `<CONSUMER>_HANDOFF_<topic>.md` | Naming pattern when the file is **dropped into the producer** (e.g., `AGENTJ_HANDOFF_folder_io_update.md`). The capitalized consumer prefix signals "external request, originated elsewhere." |
| internal handoff | A handoff to a *future session* of the same repo (`Direction: internal`) — crosses a session boundary, not a repo boundary. Lives in the two parking-lot files. Solves idea-loss without working an idea prematurely. |
| `parkinglot.md` | Near-term internal parking lot: "good idea, busy now." Capacity-gated items likely to be picked up soon. `Status: parked`. |
| `long-term-parking.md` | Far-horizon internal parking lot: versions-ahead / evidence-gated / someday-maybe. Each item has a `Trigger:` (its Definition of Ready). NOT a roadmap — committing an item moves it out. |
| `Trigger:` | The condition that pulls a parked item back out (its Definition of Ready). Distinguishes a parking lot from an unordered TODO list — items are condition-gated. |

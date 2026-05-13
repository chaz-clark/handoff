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
├── AGENTS_snippet.md     # paste-into-consumer-AGENTS.md snippet teaching the handoff convention to receiving agents
├── CONVENTION.md         # the formal spec (Direction, Status, metadata header, lifecycle, filename conventions, REPO_CARD, AGENTS_snippet)
├── knowledge/            # local state for tooling (currently: agile_sprint.md for the gh_issues_agent skill workflow)
├── handoffs/             # canonical consumer record of handoffs received here (currently: the 2026-05-13 Make-AI-Agents AGENTS.md v3.6 upgrade — applied)
├── archive/              # historical/origin docs (currently holds the seed context note; will be superseded by README once authored)
├── .gitignore            # ignores Make-AI-Agents/ and gh-issues-agent/ (local-only tooling clones)
└── .gitattributes
```

Planned additions (per archive/SEED_CONTEXT.md → "What goes in this repo"):

```
handoff/
├── README.md             # public pitch + quickstart
├── templates/            # contract_change.md, feature_request.md, bug_handoff.md, design_proposal.md
├── examples/             # real-world reference handoff(s)
└── LICENSE               # MIT or CC0 — open question
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

## Active Context

_Last updated: 2026-05-13_

- Repo is past seed stage. `CONVENTION.md` v0.3 codifies the full schema (Direction, Status, metadata header, Sensitivity, Companions, REPO_CARD producer surface, AGENTS_snippet receiving-agent prefix); paste-ready `AGENTS_snippet.md` lives at root. `README.md`, `templates/`, and `examples/` remain to be authored.
- `Make-AI-Agents/` and `gh-issues-agent/` are gitignored local clones (each has its own `.git`) so the `make_AGENTS` / `make_AGENTS_qc` and `gh_issues_agent` skills are available in this working directory. Refresh via `git pull` inside each folder; never committed or pushed from this repo.
- **Parked idea**: a `scripts/bootstrap_tooling.sh` that idempotently installs the local tooling clones (`Make-AI-Agents`, `gh-issues-agent`) + gitignore entries in any consumer repo. Discussed during initial setup; not yet authored — revisit when expanding tooling support.
- **Next steps** (suggested order, from `archive/SEED_CONTEXT.md` → "Next steps"; `CONVENTION.md` v0.3 + `AGENTS_snippet.md` land in this commit):
  1. Author canonical `README.md` (public-facing pitch).
  2. Author `templates/` (3–4 scaffolds: contract_change, feature_request, bug_handoff, design_proposal) matching CONVENTION.md v0.3's required header + REPO_CARD format.
  3. Populate `examples/` ([#2](https://github.com/chaz-clark/handoff/issues/2)) with real-world handoffs annotated against the v0.3 schema (one example already on hand: `handoffs/2026-05-13_AGENTS_md_v3.6_upgrade.md`).
  4. Add a `LICENSE` (MIT or CC0 — see open questions).
  5. First tag (v0.3) to give clone consumers a stable point.
  6. Clone into AgentJ first (dogfood) — gitignored at AgentJ's root — then propagate to canvas_toolbox, course repos, and Make-AI-Agents itself.
- **Open design questions** (from `archive/SEED_CONTEXT.md` → "Open questions"): license choice (MIT vs CC0); whether the convention should *require* an `AGENTS.md` reference in consumer repos; whether examples use real repo names (lean: yes); whether to add a small ack/archive script.

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

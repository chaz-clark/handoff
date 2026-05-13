# handoff

A tool-agnostic convention for cross-repo design coordination via markdown files. No tool, no daemon, no workflow engine — a single artifact (a markdown doc with a defined lifecycle) and a few naming rules.

[![Spec](https://img.shields.io/badge/spec-CONVENTION.md_v0.3-blue)](CONVENTION.md) [![License: CC0](https://img.shields.io/badge/license-CC0_1.0-lightgrey)](LICENSE)

## What this is

A common shape for the documents you write when one repo needs another repo to do something. Each handoff doc is a self-contained markdown file with a metadata header (Direction, Status, Author, Origin-Commit, …), a body describing the work, and a defined lifecycle (`author → drop → apply → archive`).

The convention works across any agentic dev tool (Claude Code, Cursor, Aider, Antigravity) and any human maintainer — agents read the metadata header to recognize a handoff doc; humans read the body.

## Why this isn't a GitHub Issue

Issues are great for public, conversation-heavy work where anyone can hit the bug. **Handoffs are for cross-repo *design coordination* — the producer maintainer needs the consumer's design context to act correctly.** A "please add this field to your schema because of how my repo uses it" is a handoff; a "your function throws on empty inputs" is an Issue.

Both flows coexist. The full decision heuristic is in [`CONVENTION.md`](CONVENTION.md) and [`AGENTS.md`](AGENTS.md).

## Quickstart for a consumer repo

1. **Clone handoff at your repo's root and gitignore the folder** (handoff is a vendored-by-clone convention, not a subtree):

   ```bash
   git clone https://github.com/chaz-clark/handoff.git
   echo "handoff/" >> .gitignore
   ```

2. **Refresh later via** `git -C handoff pull`.

3. **Paste [`handoff/AGENTS_snippet.md`](AGENTS_snippet.md) into your repo's `AGENTS.md`** (Working Style section). This teaches any agent operating in your repo how to recognize handoff docs.

4. **Optionally publish a [`REPO_CARD.md`](CONVENTION.md#repo_cardmd-producer-capability-surface) at your repo root** if your repo is a producer (i.e., other repos send handoffs to you). The card declares what handoff types you accept, your current `Status` (accepting / freeze / archived), and where to drop handoffs.

5. **Author a handoff** — copy a scaffold from [`templates/`](templates/), fill in the metadata header, drop the file per the [filename conventions](CONVENTION.md#filename-conventions).

## Repo layout

| Path | Role |
|---|---|
| [`AGENTS.md`](AGENTS.md) | Project context for any agentic tool entering this repo |
| [`AGENTS_snippet.md`](AGENTS_snippet.md) | Paste-into-consumer-AGENTS.md snippet (6 recognition rules) |
| [`CONVENTION.md`](CONVENTION.md) | The formal spec (Direction, Status, metadata header, lifecycle, REPO_CARD, AGENTS_snippet) |
| [`templates/`](templates/) | 4 scaffolds: contract_change, feature_request, bug_handoff, design_proposal |
| [`examples/`](examples/) | Real-world handoffs in this repo annotated against the spec |
| [`handoffs/`](handoffs/) | Canonical record of handoffs received here (this repo eats its own dog food) |
| [`archive/`](archive/) | Origin / historical docs |

## How to contribute

This is a convention, not code. The most useful contributions are:

- **Use it.** Genchi genbutsu — adopt it in a real repo, file GitHub issues here when something pinches.
- **Propose schema extensions** via a GitHub issue. The `CONVENTION.md` schema is additive across versions.
- **Send a handoff at this repo** if you find a cross-repo coordination need that the convention should accommodate. Drop it in `handoffs/<YYYY-MM-DD>_<topic>.md`.

## Convention history

Authored 2026-04 → 2026-05-13. Schema is feature-complete relative to the founding issues #1–9. See [`knowledge/agile_sprint.md`](knowledge/agile_sprint.md) for the sprint log and [`archive/SEED_CONTEXT.md`](archive/SEED_CONTEXT.md) for the origin notes.

## License

[CC0 1.0 Universal](LICENSE) — public domain dedication. Paste, fork, adopt without attribution. (The whole point of a convention is frictionless adoption.)

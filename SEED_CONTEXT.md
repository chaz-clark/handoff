# `handoff` — Seed Context

This is a seed document for the `handoff` repo. Use it as the starting reference for the README, CONVENTION, and template work that follows. It captures the problem this repo is meant to solve, the design rationale, and the workflow it enables — written down before implementation so the foundation stays clear.

---

## What this repo is

`handoff` defines a lightweight, tool-agnostic convention for cross-repo design coordination. It specifies:

- Where handoff documents live in a consuming repo (`handoffs/` folder at the root)
- How to name them so their purpose is obvious at a glance
- A lifecycle: how a handoff is authored, delivered, executed, and archived
- A markdown structure that's readable by both humans and AI coding agents (Claude Code, Cursor, Aider, Antigravity, etc.)

It is meant to be **subtreed into other repos** so each consumer of the convention picks up the same structure, templates, and rules without duplication.

---

## What this repo is NOT

- Not a tool — there's no daemon, no CLI binary required (though one could be added later as a small validator/lister)
- Not a GitHub Issues replacement — Issues are good for public, conversation-heavy work. Handoffs are for private or solo cross-repo design coordination, where the design context lives next to the code
- Not specific to AI agents — agents happen to read these well, but the pattern works for any cross-repo workflow
- Not a workflow engine — it's a convention. No state machines, no automation, no required tooling

---

## Why this exists

### The problem

When you maintain multiple related repos (e.g. a spec format in one, a runner in another, multiple consumers of the spec elsewhere), changes flow back and forth between them. A consumer realizes the producer needs a new field. A bug fix in one repo needs a counterpart change in another. A design decision authored while working in repo A needs to be applied in repo B by a future session.

Existing options all have flaws:

| Approach | Problem |
|---|---|
| GitHub Issues | Couples to GitHub UI; not local; not readable by Claude in the consuming repo without API calls |
| ADRs (Architecture Decision Records) | Great for one repo, weird across repos |
| Slack threads / DMs / email | Lost to time, not git-tracked, not agent-readable |
| Mental notes | Forgotten between sessions |
| In-line code TODOs | Stay forever; rot; don't capture cross-repo intent |
| Just doing nothing | The most common answer, and the worst one |

### The insight

A handoff is fundamentally a **markdown file describing a request from one repo to another**, with a clear lifecycle. It can be:

- **Authored** in the consumer repo — where the design context lives
- **Dropped** into the producer repo when ready to execute — visible to whoever opens that repo next
- **Applied** by the producer (human or agent reading the dropped file)
- **Archived** in the consumer repo as design history

The file is the single artifact. Git tracks it. Claude reads it. Cursor reads it. Humans read it. No tool required.

---

## Why standalone (not part of Make-AI-Agents)

This pattern was originally going to live as a subdirectory inside Make-AI-Agents because Make-AI-Agents is the natural cross-repo authority for spec format. But:

1. **The pattern is broader than agents.** Microservices teams, monorepo authors, anyone coordinating design across repos benefits. Burying it in an AI-agents-specific repo narrows the audience.
2. **Independent versioning matters.** Improvements to the handoff convention shouldn't wait for Make-AI-Agents releases.
3. **Open-source signal.** A standalone public repo says "this is a convention, adopt it freely" — same way `agents.md` does for project context files.
4. **Subtreeable into anything.** A repo that doesn't use Make-AI-Agents at all (e.g. a pure Canvas course materials repo) can still adopt the handoff convention without pulling in spec templates it doesn't need.

The `agents.md` precedent is instructive: that convention works because it's its own thing, not part of Claude or Cursor or any one tool. `handoff` aims for the same posture.

---

## How it works (lifecycle)

```
┌────────────────────────────────────────────────────────────────────┐
│ 1. AUTHOR                                                          │
│    In the consumer repo (the one needing something), write a       │
│    handoff doc:                                                    │
│       <consumer_repo>/handoffs/HANDOFF_<topic>.md                  │
│    Includes: what's needed, why, where in the producer repo to     │
│    apply it, success criteria, optional template references.       │
└──────────────────────────────┬─────────────────────────────────────┘
                               │
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 2. DROP                                                            │
│    When ready to execute, copy the file into the producer repo's   │
│    root with a source-repo prefix:                                 │
│       <producer_repo>/<CONSUMER>_HANDOFF_<topic>.md                │
│    The prefix makes its origin obvious to whoever opens the        │
│    producer repo next (Claude Code, a collaborator, future you).   │
└──────────────────────────────┬─────────────────────────────────────┘
                               │
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 3. APPLY                                                           │
│    Whoever opens the producer repo (typically a Claude Code        │
│    session) sees the dropped HANDOFF file, reads the request,      │
│    and applies the changes per its checklist.                      │
└──────────────────────────────┬─────────────────────────────────────┘
                               │
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 4. ARCHIVE                                                         │
│    Once applied, the dropped copy in the producer is deleted       │
│    (the work is done, the change is in code now). The canonical    │
│    record stays in the consumer repo's handoffs/ folder forever    │
│    as design history.                                              │
└────────────────────────────────────────────────────────────────────┘
```

---

## Naming convention (proposed)

**In the consumer repo:**

```
handoffs/HANDOFF_<topic>.md
```

Examples:
- `handoffs/HANDOFF_make_ai_agents_folder_io.md`
- `handoffs/HANDOFF_canvas_toolbox_grading_endpoint.md`

The `HANDOFF_` prefix makes intent obvious. The `<topic>` should encode the target repo + concern when ambiguous.

**Dropped into the producer repo:**

```
<CONSUMER_REPO>_HANDOFF_<topic>.md
```

Examples:
- `AGENTJ_HANDOFF_folder_io_update.md` — dropped into Make-AI-Agents
- `CANVAS_TOOLBOX_HANDOFF_grading_endpoint.md` — dropped into Make-AI-Agents

The capitalized prefix signals "external request, originated elsewhere." When applied and deleted, the canonical version remains in the consumer repo's `handoffs/` folder.

---

## What a handoff document contains

This isn't strict yet — the templates will pin it down. But a good handoff has:

1. **Purpose** — one paragraph: what does the consumer need from the producer, and why?
2. **Cross-repo handoff note** — explicitly identifies source and target repos
3. **Specific changes** — section by section, what files/sections need to change in the producer
4. **Examples** — before/after, schema additions, code snippets
5. **Acceptance / checklist** — bullet list the producer can tick off as items are applied
6. **Coordination note** — any timing dependencies, related handoffs, or downstream consumers to inform

The existing real-world example (`AGENTJ_HANDOFF_folder_io_update.md`, currently in `Make-AI-Agents/`) follows this shape and can serve as the first reference.

---

## Adoption mechanism — git subtree

Consumer repos pull this repo in as a subtree at their root, so the convention docs and templates ride along:

```bash
# In a consuming repo:
git subtree add --prefix=handoff https://github.com/chaz-clark/handoff main --squash
```

After the subtree:

```
<consumer_repo>/
├── handoff/                    ← this repo, subtreed in (read-only by convention)
│   ├── README.md
│   ├── CONVENTION.md
│   ├── templates/
│   └── ...
├── handoffs/                   ← this repo's handoffs (authored locally)
│   ├── HANDOFF_make_ai_agents_folder_io.md
│   └── ...
├── AGENTS.md                   ← references handoff/ for the convention
└── ...
```

To pull updates later:

```bash
git subtree pull --prefix=handoff https://github.com/chaz-clark/handoff main --squash
```

---

## What goes in this repo (proposed structure)

```
handoff/
├── README.md                   ← this repo's pitch + quickstart
├── CONVENTION.md               ← the formal rules (the "spec" of the convention)
├── templates/
│   ├── contract_change.md      ← "I need this field added to your schema"
│   ├── feature_request.md      ← "I need this capability"
│   ├── bug_handoff.md          ← "I hit something you need to fix"
│   └── design_proposal.md      ← "Here's a change worth considering"
├── examples/
│   └── agentj_to_make_ai_agents_folder_io.md
│       ← the real-world handoff that motivated this repo
├── AGENTS_snippet.md           ← a snippet to drop into a consuming repo's
│                                  AGENTS.md teaching agents about the convention
└── LICENSE                     ← MIT or CC0 — wide-open since it's a convention
```

Optional later additions:
- `tools/list_handoffs.py` — a small Python script that lists open handoffs in a repo
- `tools/validate.py` — checks a handoff file conforms to the convention
- `tools/archive.py` — moves applied handoffs to an `handoffs/archive/` subfolder

These are nice-to-haves; the convention works without them.

---

## Why this could matter beyond Chaz's repos

Cross-repo coordination is unsolved in most ecosystems. Most teams use GitHub Issues (UI-coupled, not local, not agent-readable) or nothing at all (most common). A markdown convention that's:

- Tool-agnostic
- Agent-readable (Claude Code, Cursor, Aider all parse markdown well)
- Git-trackable
- Travels via subtree
- Single artifact per handoff
- Has clear lifecycle

…is the kind of small idea other developers can pick up easily. It's the right size for an open convention. It pairs naturally with `agents.md`.

If it gets adopted by even a few other open-source maintainers, it stops being "Chaz's pattern" and becomes a community standard. The standalone public repo is what makes that possible.

---

## Next steps (suggested, in order)

1. **Author the canonical README.md** — the public-facing pitch (what it is, why, quickstart)
2. **Author CONVENTION.md** — the formal spec of the rules (file naming, lifecycle, content structure)
3. **Author templates/** — three or four scaffolds for common handoff types
4. **Drop the real-world example** — copy `AGENTJ_HANDOFF_folder_io_update.md` from `Make-AI-Agents/` into `examples/` (it's already a working reference)
5. **Add a license** — MIT or CC0 fits an open convention
6. **First tag / release** — even v0.1 establishes a stable point for subtree consumers
7. **Subtree into AgentJ first** — eat your own dog food. Validate the workflow on one repo before propagating.
8. **Then propagate** — canvas_toolbox, course repos, Make-AI-Agents itself

---

## Open questions to resolve while building

- Pluralization: `handoff` (singular, the convention) vs `handoffs` (plural, the collection)? Recommendation: singular for repo and convention name, plural for the folder inside consumers.
- License: MIT (familiar, permissive) vs CC0 (true public domain, fits a convention better)?
- Should the convention require an `AGENTS.md` reference in the consumer? (Yes, light requirement — Claude needs to know the convention exists in this repo)
- Should examples be anonymized or use real repo names? (Real names — they're more useful and the projects are public anyway)
- Is there a mechanism to "ack" a handoff was applied without manual archive? (A small Python script could detect "if this file exists in the producer and matches a `handoffs/` file in the consumer, mark archived." Future work.)

---

## Origin of this seed

Authored 2026-04-28 in a session working on AgentJ. The convention was extracted from a concrete real-world handoff (`AGENTJ_HANDOFF_folder_io_update.md`) that was shaping up while AgentJ Sprint 3 was being designed. We noticed the pattern was generalizable, considered keeping it inside Make-AI-Agents, and decided standalone-public was the right shape because the pattern isn't agent-specific and deserves its own discoverability.

This file should be deleted (or moved into `docs/origins.md`) once `README.md` and `CONVENTION.md` exist as the canonical source of truth.

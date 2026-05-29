# long-term-parking.md

**File:** long-term-parking.md
**Direction:** internal
**Status:** parked
**Purpose:** Far-horizon parked ideas for the `handoff` repo — versions-ahead, evidence-gated, someday-maybe, pie-in-the-sky. Each item is gated by external conditions (a use case, adoption evidence, an upstream change), not by current capacity. NOT a roadmap and NOT next-work. See `CONVENTION.md` → Internal handoffs. Reviewed occasionally for "has the evidence arrived yet?"
**Last-refined:** 2026-05-28

---

### `scripts/bootstrap_tooling.sh` installer

**Trigger:** when a 3rd+ repo needs the `Make-AI-Agents` + `gh-issues-agent` clone-and-gitignore setup (i.e. the manual setup has been repeated enough to justify a script)
**Routes-to:** issue
**Added:** 2026-05-22
**Origin-Commit:** e3df2b6

An idempotent script that installs the local tooling clones (`Make-AI-Agents`, `gh-issues-agent`) plus the matching `.gitignore` entries in any consumer repo. Discussed during initial setup and lived as a "Parked idea" bullet in `AGENTS.md` Active Context; moved here to its proper home. Evidence-gated: not worth automating a two-clone setup until it's been done by hand across enough repos to prove the pattern is stable.

---

### `.handoff-card.json` automated-consumption tooling

**Trigger:** if an automated pipeline (A2A-style) ever needs to consume `REPO_CARD.md` as structured JSON rather than markdown
**Routes-to:** handoff
**Added:** 2026-05-22
**Origin-Commit:** e3df2b6

`CONVENTION.md` already documents `.handoff-card.json` as an optional JSON variant of `REPO_CARD.md`. Pie-in-the-sky: there is no automated consumer today. If one ever appears, this graduates into a design proposal for a generator/validator that keeps the `.md` and `.json` forms in sync. No action until a real automated consumer exists.

---

### Require `AGENTS_snippet.md` adoption in consumer repos?

**Trigger:** when there is real adoption feedback from multiple consumer repos (an open design question from `archive/SEED_CONTEXT.md`)
**Routes-to:** handoff
**Added:** 2026-05-22
**Origin-Commit:** e3df2b6

Open question: should the convention *require* a consumer repo to carry the `AGENTS_snippet.md` recognition rules (and/or an `AGENTS.md` reference) rather than treating it as recommended? Evidence-gated — this should be decided from how real consumers actually adopt the snippet, not in the abstract. Until there are several real consumers to learn from, requiring it is premature.

---

### Hermes Kanban v1 spec as multi-agent coordination reference

**Trigger:** when the handoff convention needs to address multi-*agent* coordination (orchestrator + specialist roster + cross-profile collaboration patterns) — not just multi-*repo* coordination, which the current convention covers
**Routes-to:** convention update
**Added:** 2026-05-28
**Origin-Commit:** dbf8bc0

Nous Research's `hermes-agent` repo ships a 33-page design spec at `docs/hermes-kanban-v1-spec.pdf` (April 2026, design only — no implementation). Three pieces are well-articulated prior art if the convention ever extends from cross-repo to cross-agent handoffs: (a) eight reusable collaboration patterns — Fan-out, Pipeline, Voting/Quorum, Long-running journal, Human-in-the-loop triage, `@mention` delegation, Thread-scoped workspace, Fleet farming; (b) a first-class **orchestrator-profile** pattern with the "disabled execution toolsets" anti-temptation mechanism, fixing the canonical failure mode *"my orchestrator does the work itself instead of routing it"*; (c) the three-plane separation (control / state / execution) with **"no in-process subagent swarms"** as a load-bearing invariant. Evidence-gated: no action until the convention faces a real multi-agent design question. Reference, not roadmap.

---

### `agentskills.io` as parallel open standard for portable markdown skill artifacts

**Trigger:** if cross-tool portability of handoff docs themselves ever becomes a concern (i.e. handoff docs need to be loadable into `agentskills.io`-compliant runtimes like Hermes / OpenClaw, not just read in dev tools)
**Routes-to:** convention update *or* decline
**Added:** 2026-05-28
**Origin-Commit:** dbf8bc0

The `agentskills.io` open standard defines a YAML-frontmatter format for portable markdown skills (used by Hermes; adopted by OpenClaw via migration). Make-AI-Agents specs are candidates for adopting it layered over their existing `.json` sidecar (`make_*.md` files). The handoff convention is **NOT a candidate by current design**: `CONVENTION.md` explicitly chose **bold-labeled metadata over YAML frontmatter** because bold labels render inline in GitHub/IDE previews without requiring a frontmatter parser. This entry exists to record that the parallel standard exists and was considered — so a future reader doesn't propose adopting it without rediscovering the rationale. Unless the deliberate "no parser required" decision is revisited under new pressure, no action.

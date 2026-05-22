# long-term-parking.md

**File:** long-term-parking.md
**Direction:** internal
**Status:** parked
**Purpose:** Far-horizon parked ideas for the `handoff` repo — versions-ahead, evidence-gated, someday-maybe, pie-in-the-sky. Each item is gated by external conditions (a use case, adoption evidence, an upstream change), not by current capacity. NOT a roadmap and NOT next-work. See `CONVENTION.md` → Internal handoffs. Reviewed occasionally for "has the evidence arrived yet?"
**Last-refined:** 2026-05-22

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

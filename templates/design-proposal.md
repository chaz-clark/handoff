<!--
Template — design-proposal handoff
Use when: you want to propose a design across repo boundaries, or surface a design
choice the producer needs to make where the rationale spans both repos.

Differs from feature-request in tone — this is "let's talk through it" rather than
"please ship this." Often the prelude to a feature-request or contract-change handoff.

Fill in the placeholders below, then DELETE this comment block.
See CONVENTION.md → Required Metadata Header for field semantics.
-->

**Date:** YYYY-MM-DD
**Author:** <your name or agent + originating repo>
**Direction:** request
**Status:** draft
**Origin:** <one-sentence: what design pass / question / friction surfaced this proposal>
**Origin-Commit:** <git-sha in your repo whose state motivates the proposal>
**Topic:** <kebab-case slug matching this filename's <topic> portion>
<!-- Optional:
**Sensitivity:** standard
**Companions:**
- <related handoff or issue> — <relationship>: <one-line note>
-->

---

# Design proposal: <one-line summary>

## The question

<What design decision is on the table? Frame as a question if you can — that invites discussion rather than just "please do this.">

## Context (from your repo's side)

<What in your repo motivates this. Cite the design state by commit, file, or doc.>

## Context (from the producer's side, as you understand it)

<Your read of what the producer already does, what constraints they live with, why their current design is the way it is. Be honest about what you might be wrong about — the producer can correct.>

## Options considered

| Option | Pro | Con |
|---|---|---|
| A. <approach> | <pro> | <con> |
| B. <approach> | <pro> | <con> |
| C. … | | |

## Lean

<Which option you'd lean toward, if you have one. Or "no strong lean" if you're genuinely asking.>

## What a follow-up would look like

<If this proposal converges, what's the next concrete step? Often this is a `contract-change` or `feature-request` handoff with specifics.>

## Open questions for the producer

- <question 1>
- <question 2>

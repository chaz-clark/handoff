<!--
Template — bug_handoff
Use when: a bug surfaces in the producer's repo BUT the reproduction or design
context lives in YOUR repo and won't fit in a GitHub issue body.

Most bugs should be filed as GitHub issues against the producer. Use this template
only when:
- The repro requires consumer-side state the producer's maintainer can't easily set up
- The "is this even a bug or expected behavior?" depends on your repo's design intent
- The fix touches a contract both repos depend on (see also: contract_change template)

Fill in the placeholders below, then DELETE this comment block.
See CONVENTION.md → Required Metadata Header for field semantics.
-->

**Date:** YYYY-MM-DD
**Author:** <your name or agent + originating repo>
**Direction:** request
**Status:** draft
**Origin:** <one-sentence: what task / sync / pipeline run uncovered the bug>
**Origin-Commit:** <git-sha in your repo at the time the bug was hit>
**Topic:** <kebab-case slug matching this filename's <topic> portion>
<!-- Optional:
**Sensitivity:** standard
**Companions:**
- <related handoff or issue> — <relationship>: <one-line note>
-->

---

# Bug handoff: <one-line summary>

## Observed behavior

<What happened. Concrete: file paths, error text, observable state. No paraphrasing.>

```
<error output, stack trace, or before/after state — verbatim>
```

## Expected behavior

<What you expected. Cite the producer's docs / contract / prior commit if it tells you the expected shape.>

## Repro (cross-repo)

<Steps that involve BOTH repos. If repro requires consumer-side state, name the files/commits the producer would need to look at in your repo to understand the setup.>

1. In consumer (`<your-repo>` @ `<sha>`): <step>
2. In producer (`<their-repo>` @ `<sha>`): <step>
3. Observed: <bad outcome>

## Why this is a handoff, not an Issue

<One sentence. Usually: "the repro requires X consumer-side state" or "whether it's a bug at all depends on Y consumer-design decision.">

## Suggested fix direction (optional)

<If you have a theory — share it. Otherwise leave this section out.>

## Acceptance check

<How both repos verify the fix worked.>

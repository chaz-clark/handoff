<!--
Template — feature_request handoff
Use when: your repo needs another repo to add a feature it doesn't currently have,
and the rationale lives in your repo's design (not the producer's bug tracker).

If the producer's maintainer could act on this knowing only their own repo's state,
file a GitHub issue against them instead. Use this template when they need the
consumer's design context to act correctly.

Fill in the placeholders below, then DELETE this comment block.
See CONVENTION.md → Required Metadata Header for field semantics.
-->

**Date:** YYYY-MM-DD
**Author:** <your name or agent + originating repo>
**Direction:** request
**Status:** draft
**Origin:** <one-sentence: what design pass / capability gap surfaced this request>
**Origin-Commit:** <git-sha in your repo whose state shows the gap>
**Topic:** <kebab-case slug matching this filename's <topic> portion>
<!-- Optional:
**Sensitivity:** standard
**Companions:**
- <related handoff or issue> — <relationship>: <one-line note>
-->

---

# Feature request: <one-line summary>

## What feature

<One paragraph: the capability you want the producer to add.>

## Why this lives in the producer's repo

<Why this can't just be solved on the consumer side. What does the producer have access to (data, abstraction, ownership boundary) that your repo doesn't?>

## How your repo would use it

<The downstream use case. Show the call site, the integration point, or the workflow that's blocked.>

```
<optional: pseudo-code or call-site sketch>
```

## Suggested shape

<If you have a strong opinion on API surface — say so. If you'd defer to the producer's design judgment — say that too.>

## What you've ruled out

<Approaches you considered and rejected, with brief reason. Saves the producer from re-walking the same paths.>

## Acceptance check

<How both repos verify the feature works once shipped.>

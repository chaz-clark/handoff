# Examples — Real-world handoffs annotated against CONVENTION.md

This folder catalogs concrete handoff docs in this repo that demonstrate the convention in use. Each entry is a **pointer** to the canonical doc (in `handoffs/`), with a one-paragraph note on which convention features it exercises.

Examples are linked, not copied — the canonical doc is the single source of truth. When the canonical's `Status` changes, the example automatically reflects it.

---

## `deliver` direction

### [`handoffs/2026-05-13_AGENTS-md-v3-6-upgrade.md`](../handoffs/2026-05-13_AGENTS-md-v3-6-upgrade.md)

**What:** Applied `deliver`-direction handoff from `Make-AI-Agents` to `handoff`. Make-AI-Agents authored a one-paragraph rewrite of this repo's `AGENTS.md` Working Style discipline pointer (from a broken local-path reference to the resolving clone-path reference) and delivered it via the canonical `deliver` lifecycle.

**Convention features it exercises:**

- `Direction: deliver` — producer-authored, consumer-canonical lifecycle.
- Full v0.2 metadata header including `Companions`.
- `## Lifecycle marker` section logging Authored → Delivered → Applied transitions with dates.
- Status transitions `delivered → applied` updated in place on the file.
- Visibility-copy-at-root pattern (`AGENTS_updated.md`) used during the `delivered → applying` window, deleted on apply.
- `Direction` value renamed at apply time (`producer-delivers` → `deliver`) when the v0.1 short form landed between authoring and applying — illustrates how the convention can evolve mid-handoff.

**What it teaches:** the `_temp`-as-working-copy apply pattern when a delivered handoff would overwrite in-flight local changes. The producer's visibility copy was generated from an earlier snapshot of this repo; a manual surgical merge via `AGENTS_temp.md` preserved the consumer's in-flight Structure tree addition. Straight `mv AGENTS_updated.md AGENTS.md` would have lost work. This is the canonical apply procedure when a delivered handoff competes with local in-flight changes.

---

## `request` direction

*No real-world `request`-direction example in this repo yet.* The `request` lifecycle is documented in [`CONVENTION.md` → Lifecycle by direction → `request` lifecycle](../CONVENTION.md#request-lifecycle); an example will be added here when the first real `request` handoff is authored from or to this repo.

---

## Adding new examples

When a new real handoff lands in `handoffs/` (or, for outgoing `request` handoffs, when one is dropped at another repo's root), add an entry here with the same shape:

- One-line title with a link to the canonical path.
- **What** — what the handoff is and which repos it touched.
- **Convention features it exercises** — bullet list mapping to `CONVENTION.md` sections.
- **What it teaches** — what a reader takes away.

Keep entries short. Linking, not copying — the canonical doc lives once, in `handoffs/`.

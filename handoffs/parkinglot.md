# parkinglot.md

**File:** parkinglot.md
**Direction:** internal
**Status:** parked
**Purpose:** Near-term parked ideas for the `handoff` repo — "good idea, not fully baked, busy now." Capacity-gated: these would likely be picked up soon if attention were free. NOT committed work — an item scheduled into a sprint leaves this file. See `CONVENTION.md` → Internal handoffs.
**Last-refined:** 2026-05-22

---

### `handoff_status_check.sh` validator

**Trigger:** next time tooling work is picked up, OR the first time `handoffs/` field-checking is done by hand more than once
**Routes-to:** issue
**Added:** 2026-05-22
**Origin-Commit:** e3df2b6

A small shell validator that confirms every doc in `handoffs/` has the required metadata fields and valid enum values for `Status`, `Direction`, `Sensitivity`, and Companion `<relationship>` tokens (already sketched in `CONVENTION.md` → Tooling implications). Deferred only because the spec work is the current focus and the `handoffs/` set is still small enough to eyeball. Good first script once tooling becomes the active workstream.

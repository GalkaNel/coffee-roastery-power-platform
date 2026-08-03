# Production Readiness

This document records the current environment adaptations, known limitations, licensing considerations and the path from portfolio solution to production deployment.

## Contents

- [Environment adaptations](#environment-adaptations)
- [Known limitations](#known-limitations--production-path)
- [Licensing considerations](#licensing-considerations)
## Environment adaptations

Documented as adaptations, not hacks — each with a production path.

| Constraint | Adaptation | Production path |
|---|---|---|
| Choice sets not resolvable in one canvas app (`Value()` conversion failed; `Text() = "label"` comparisons unreliable for writes) | **Status dictionary in OnStart**: capture real choice values positionally over `Choices('Table'.'Column')`; value order frozen as a contract; display via `Text()`, all comparisons/writes via dictionary variables | Delegable direct choice comparison, or a numeric status-code column |
| Date-only columns arrive in canvas as **DateTime at noon** — `= Today()` misses | `Date(Year(d), Month(d), Day(d)) = Today()` in canvas; `convertTimeZone` in flows — one disease, two cures on both sides of the data | Consistent TZ policy per column |
| Non-delegable predicates (label comparisons, date reconstruction, CountRows on full tables) | Accepted at demo scale; reference data snapshotted to collections at start; transactional data always queried live | Delegable predicates; indexed status columns |
| No Teams/Exchange licences in the developer tenant | In-app surfacing (worklist tails, result stamp) carries the safety load | Teams adaptive cards / email on the same triggers |
| Canvas has no server push | Timer-based polling on always-open screens | Power Apps push notifications; model-driven auto-refresh |

---

## Known limitations & production path

- **Mid-run failure idempotency** (see the table above) — the one real gap, with a stated fix.
- **Mixed read-comparison legacy** — early screens use label comparisons, later ones the choice dictionary. It works; consistency would be a refactor.
- **Status reversal** — a roasted batch can only be reopened via the model-driven form (admin path), deliberately not from the operator's tap.
- **Notification channels** — the system surfaces exceptions in the operator's worklist rather than pushing alerts; Teams/email would be added in production.

---
### Domain simplifications (deliberate, with a stated next iteration)

- **Shrinkage is stored per coffee, not per coffee × roast level.** In reality, weight loss depends on both the bean's moisture content *and* the roast degree — the same origin roasted dark loses more than roasted light. *Next iteration: move the shrinkage percentage onto the blend × roast-level combination — a data-model change, not an architectural one.*
- **Machine capacity is not modelled.** A production task of 35 kg of green is not one machine load — a small roastery's machine typically takes 5–15 kg at a time, so the task becomes several sequential loads. The system plans *what and how much*, not *how many loads*. *Next iteration: add machine capacity as reference data and split tasks into loads automatically, which also unlocks realistic time estimates for the roasting day.*
- **Terminology note:** a `Roast Batch` record is a **production task** (one coffee × roast level for the day), not a single physical machine load. Merging tasks saves a **changeover** — profile setup, weighing, dialling in — which is the production-planning value the automation delivers.

---

## Licensing considerations

Built on the **Power Apps Developer Plan** (Dataverse included, single-maker environment). A production deployment would need:

- **Per-app or per-user Power Apps licences** for cafe managers and the roaster (three cafes + one operator makes per-app licensing the cheaper path at this scale).
- **Power Automate** — the daily flow runs under Power Platform request limits; the scheduled run is one execution per day, well within a standard entitlement.
- **Dataverse capacity** — the data volume here (orders, batches, tasks) is trivial; base capacity suffices.
- Alternatives evaluated: **Power Pages** for external cafes (external-user licensing) vs. **guest access to a canvas app** — for three known cafes with named managers, canvas + per-app licensing is materially cheaper; Power Pages becomes the answer at 20+ cafes or anonymous ordering.

---


# Automation Design

This document explains how Power Automate converts submitted café orders into roasting batches and packaging tasks.

## Contents

- [Flow structure](#flow-structure)
- [Child flow](#child-flow-step-by-step)
- [Merge logic](#merge-logic--batch-level-deduplication)
- [Idempotency](#idempotency-layered-honestly)

## Flow structure

![Autom](Automation.png)


All logic lives in the child flow; the two parents are thin entry points — one implementation, multiple doors. A third door (e.g. an agent) could be added without touching logic.

### Child flow, step by step

1. **TodayNZ** — `convertTimeZone(utcNow(), 'UTC', 'New Zealand Standard Time')`. All date logic is local-calendar based.
2. **List Submitted orders** with `Production Date ≤ TodayNZ`. Orders submitted after the 16:00 cutoff carry the *next business day* (cutoff + weekend skip, set by the order app), so a same-day run naturally ignores them — an explicit business rule: *after 16:00 the drum is running; add-ons roll to the next day*.
3. **List Order Lines** with **Expand Query** to reach Blend and its shrinkage in a single call (no N+1 lookups).
4. **Select** an 11-field flat card per line; everything downstream works from this projection.
5. **Batch grouping** — distinct `blend|roastLevel` keys via the classic `union(x, x)` de-dup; per key: Filter Array → **xpath sum** of kg → *merge-or-create*.
6. **Packaging grouping** — distinct `blend|roast|grind|size` keys; per key: bag-count sum → **Find Batch** → create Packaging Task.
7. **Mark Orders In Production** — the idempotency mechanism: a repeated run finds zero Submitted orders and exits in ~0.5 s.
8. **Respond** — `"Roast groups processed: N, packaging tasks: M"` (counts distinct groups — honest wording whether batches were created or merged).

### Merge logic — batch-level deduplication

**Problem found in acceptance testing:** two build waves in one day (early close + scheduled run), or an unroasted leftover from a previous day, produced duplicate batches for the same blend+roast — two drum runs where one suffices. The core value of the automation was leaking.

**Two valid business rules collided:**
- *Never touch a batch that is already roasting* (physics — beans are in the drum).
- *One blend+roast should be one drum run* (the point of the automation).

**Resolution — the boundary is the physical status:**

```text
For each blend|roast key:
    Find Planned Batch = List rows (Roast Batches)
                         filter: blend AND roastLevel AND statuscode = Planned   ← no date filter
                         sort:   createdon DESC, row count 1
    IF found:
        Update row: Ordered Kg = existing + new
                    Green Kg   = (existing + new) ÷ (1 − shrinkage/100)
        → packaging tasks attach to this batch
    ELSE:
        Add row (create a new batch)
```

- **No date filter** — a Planned leftover from *any* previous day is a valid merge target; yesterday's tail absorbs today's demand.
- **Planned only** — Roasting/Done batches are invisible to the merge. Verified: setting a batch to Roasting forced creation of a separate batch.
- **`createdon desc`, row count 1** in the packaging lookup — tasks always land on the newest live batch of the key (just created or just merged). This one-line sort also fixed a real defect: with two same-key batches in a day, tasks previously attached to the wrong one.
- **Merge by Update, never delete-and-recreate** — recreation would cascade-delete existing Packaging Tasks (Parental).

![Cafe orders Flow merge](flow-merge2.png)

### Idempotency, layered honestly

| Layer | Mechanism | Coverage |
|---|---|---|
| Order level | Status flip to *In Production* as the final step | Any repeated **successful** run processes nothing twice (empty rerun ≈ 500 ms) |
| Batch level | Merge into Planned (above) | Two waves a day / leftover tails never duplicate a drum run |
| **Gap (documented)** | Mark Orders is the last step → a run failing **mid-way** leaves orders Submitted with artifacts already created; a rerun duplicates them | Probability low, detection immediate, cleanup cheap. **Production path:** artifact-level existence checks before each Add row, or a claim-first pattern (flip status first, compensate on failure) |


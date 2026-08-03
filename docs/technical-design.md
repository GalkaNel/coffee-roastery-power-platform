
## Architecture
![Architecture](docs/Architecture.png)


**Tool choice is per scenario, not per system.** Canvas where the scenario is a purpose-built flow with no standard-UI equivalent (the order matrix; the tap-driven packing checklist). Model-driven where the need is records management — grids, forms, charts, Excel export, all generated from metadata at near-zero cost. A hybrid is the mature answer, not a compromise.

## Data model

![DM](docs/DataModel.png)


| Decision | Rationale |
|---|---|
| Custom tables instead of Dynamics Product Catalog / Account | No pricing or billing in scope; sales entities would drag price lists and units into a production domain and obscure the business language ("Cafe", "Stock Order"). Documented revisit trigger: franchise billing. |
| **Alternate keys** on all reference tables | Deterministic upserts; duplicate prevention at platform level rather than duplicate-detection jobs. |
| **Composite alternate key + composite primary name** on Product SKU | One SKU per real-world combination; the primary name renders a complete human-readable line ("Ethiopia Yirgacheffe - Light - Filter - 500 g") everywhere for free. |
| **Parental** cascade Order→Lines, Batch→Tasks; **Restrict** Lines→SKU, Batch→Blend | Deleting a whole removes its parts; deleting a referenced catalog item is blocked. Product retirement = Deactivate, never delete. |
| Shrinkage % stored **per blend** | `green = ordered ÷ (1 − shrinkage/100)`. Shrinkage is a property of the bean, so it lives on the origin. |
| **No batch↔order relationship** | A batch aggregates lines from *many* orders (M:N by nature). Order-level traceability lives where the relationship is direct — the packing screen. A junction table was evaluated and rejected: no business question required it. |

---

## Key design decisions

1. **Custom domain tables over Dynamics sales entities** — with a documented revisit trigger (billing).
2. **Tool per scenario; hybrid by design** — canvas for purpose-built flows, model-driven for records management.
3. **Copilot Studio agent evaluated and rejected** — every candidate user's questions were already served by purpose-built UI (cafes have *My Orders*; the roaster has the dashboard), and external cafes had no viable channel. Adding an agent would duplicate surfaces. (The skill is demonstrated in a separate D365 Customer Service project, where an agent has a native home.)
4. **Merge boundary = physics** — Planned batches absorb new demand; Roasting/Done are untouchable.
5. **Exception surfacing in the worklist**, not in notifications or separate zones.
6. **Per-batch tap confirmation over a bulk "close the day"** — bulk closure by date was designed, then rejected: it would blindly close a second wave nobody had roasted yet. Atomic facts beat batch ceremonies.
7. **Business Process Flow evaluated and rejected** — the process is linear with no stage decisions; a BPF here is ceremony without value.
8. **Security in data, not in app filters** — app-level filtering is UX; row-level security is the boundary.
9. **Validation before action** across both apps — constraints → contracts → DisplayMode/Visible.
10. **KPIs framed as measurement capability**, never fabricated outcomes.

---

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

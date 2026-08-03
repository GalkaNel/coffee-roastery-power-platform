# Technical Design

This document describes the architecture, data model and key design decisions behind the Coffee Roastery Operations System.

## Contents

- [Architecture](#architecture)
- [Data model](#data-model)
- [Key design decisions](#key-design-decisions)

## Architecture
![Architecture](docs/Architecture.png)


**Tool choice is per scenario, not per system.** Canvas where the scenario is a purpose-built flow with no standard-UI equivalent (the order matrix; the tap-driven packing checklist). Model-driven where the need is records management — grids, forms, charts, Excel export, all generated from metadata at near-zero cost. A hybrid is the mature answer, not a compromise.

## Data model

![DM](docs/DataModel.png)


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


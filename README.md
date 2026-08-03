# Coffee Roastery Operations System

**Microsoft Power Platform · Dataverse · Canvas Apps · Model-driven App · Power Automate**

A Power Platform solution that converts stock orders from three cafés into a consolidated daily roasting and packing plan.

The system gives café managers one place to order stock and gives the roaster a single operational worklist—from green-bean calculations through to completed boxes.

> **Portfolio project:** designed and built end to end, including business analysis, Dataverse architecture, security, automation, applications and production-readiness assessment.

## At a glance

| Business context                                     | Solution                                                |
| ---------------------------------------------------- | ------------------------------------------------------- |
| Three cafés supplied by one roastery                 | One shared Dataverse data platform                      |
| Orders received through email, chat and spreadsheets | A dedicated ordering application for café managers      |
| Production quantities calculated manually            | Automated aggregation and green-bean calculations       |
| Late orders could create duplicate production setups | Planned work absorbs new demand until roasting begins   |
| Unfinished work could be overlooked                  | Incomplete tasks remain in the operator's worklist      |
| Packing relied on manual checks                      | An order cannot be completed until every line is packed |

## What I delivered

I designed and built the solution end to end:

* translated the operating process into requirements and business rules;
* designed the Dataverse schema, relationships and security model;
* built two Canvas Apps and one model-driven application;
* created scheduled and manually triggered Power Automate flows;
* implemented café-level data isolation using security roles and Owner Teams;
* tested operational exceptions, repeated flow runs and late-order scenarios;
* evaluated licensing, build-vs-buy options and the path to production.

## Demo

**1.5-minute walkthrough:** placing an order, building the production plan and packing completed café orders.

[Watch the video demonstration](https://youtu.be/m7cRZf3rfo8)
## Explore the project

* [Business case](docs/coffee-roastery-business-case.md) — commercial context, build-vs-buy analysis and cost comparison.
* [Technical design](docs/technical-design.md) — architecture, data model and design decisions.
* [Automation](docs/automation.md) — grouping, merge logic and idempotency.
* [Security](docs/security.md) — Dataverse roles, Owner Teams and café-level isolation.
* [Production readiness](docs/production-readiness.md) — limitations, environment adaptations and licensing.


## Table of contents

- [The business problem](#the-business-problem)
- [How the solution works](#how-the-solution-works)
- [Architecture](#architecture)
- [Applications](#applications)
- [Key design decisions](#key-design-decisions)
- [Repository contents](#repository-contents)

---
## The business problem

A small Auckland coffee company roasts coffee centrally for three company-owned cafés, with plans to grow to seven locations.

Before the solution:

* orders arrived through email, group chat and spreadsheets;
* the roaster manually combined demand from every café;
* ordered roasted weight had to be converted into the required green-bean weight;
* late additions could cause duplicate production setups;
* unfinished production work could disappear from the next day's plan;
* packing depended on manual checking.

Coffee production also runs on short planning cycles. The roastery cannot simply produce a month of stock in advance, so ordering, roasting and delivery must be coordinated frequently.

### Domain calculation

Coffee loses weight during roasting. To produce **30 kg of roasted coffee** at an expected **16% weight loss**, the system calculates:

```text
Required green weight = 30 ÷ (1 − 0.16) = 35.7 kg
```

The system performs this calculation automatically after consolidating demand from all cafés.

---


## How the solution works

### 1. A café places an order

A café manager selects products and quantities in the Cafe Order App. The café is derived from the signed-in user and cannot be selected manually.

### 2. Order intake is closed

At 16:00—or earlier through a manual action—the system collects submitted orders that are ready for production.

### 3. The production plan is generated

Power Automate groups demand by coffee and roast level, calculates the required green-bean weight and creates the corresponding grinding and packaging tasks.

### 4. The roaster completes the work

The Packing Station presents today's production alongside unfinished work from previous days. Completed roasting tasks are confirmed individually.

### 5. Café orders are packed

Each order is treated as one box. The operator checks every order line, and the box can only be completed when all items have been packed.

### Late-order handling

A late order is merged into an existing production task while that task is still **Planned**. Once roasting has started, the system creates separate work rather than modifying a physical batch already in progress.

---

## Architecture
![Architecture](docs/Architecture.png)

For the complete architecture, Dataverse data model and design rationale, see the [technical design](docs/technical-design.md).

**Tool choice is per scenario, not per system.** Canvas where the scenario is a purpose-built flow with no standard-UI equivalent (the order matrix; the tap-driven packing checklist). Model-driven where the need is records management — grids, forms, charts, Excel export, all generated from metadata at near-zero cost. A hybrid is the mature answer, not a compromise.

---

## Applications

**1.5-minute walkthrough:** placing an order, building the production plan and packing completed café orders.

[Watch the video demonstration](https://youtu.be/m7cRZf3rfo8)

### Cafe Order App

**For café managers:** place stock orders, review previous orders and track fulfilment status.

![Cafe managers order](docs/OrderSteps.png)

The application provides:

* a product matrix designed for quick ordering;
* automatic café identification based on the signed-in user;
* validation before an order can be submitted;
* production-date calculation based on the 16:00 cutoff and weekends;
* order history with live fulfilment status.

![Cafe order review](docs/Cafe_Order_3_Review_Screen.png)

### Packing Station

**For the roaster:** generate the production plan, complete roasting work and pack each café order.

![Cafe orders for Roastery](docs/RoasterApp.png)

The application provides:

* one worklist containing today's production and unfinished work from previous days;
* manual intake closure and production-plan generation;
* individual confirmation of completed roasting tasks;
* a guided packing checklist for each café order;
* automatic prevention of incomplete boxes being closed.

![Packing Station](docs/CoffeRP.png)

### Roastery Ops

**For operations and administration:** inspect records, maintain reference data and review production reporting.

![Roastery Ops](docs/modelDr.png)

The model-driven application provides:

* searchable and filterable operational records;
* production, packaging and overdue-work views;
* charts and Excel export;
* record forms and reference-data maintenance;
* an administrative path for reviewing exceptions and production history.

Implementation details and notable Power Fx patterns are documented in the [technical design](docs/technical-design.md).

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
Additional implementation details are documented in the [technical design](docs/technical-design.md). 


---
## Repository contents
```text
├── README.md
├── solution/
│   ├── CoffeeRoastery_*.zip
│   └── CoffeeRoastery_*_managed.zip
└── docs/
    ├── technical-design.md
    ├── automation.md
    ├── security.md
    ├── production-readiness.md
    └── coffee-roastery-business-case.md
```


**To deploy:** import the managed solution into a Dataverse environment, then configure connection references (Dataverse), assign the two security roles, and create Owner Teams per cafe.

---

*Built by Galina Nelyubova as a portfolio project. Fictional business, real engineering.*

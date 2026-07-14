# From Scattered Orders to a Production Plan
### How a custom Power Platform system replaced a coffee roastery's daily guesswork — for less than the price of one flat white a day

*A portfolio case study by Galina Nelyubova · Microsoft Power Platform Functional Consultant*

---

## The business

A small Auckland coffee company roasts its own beans and runs **three of its own cafes**. One roaster at the roastery, three cafe managers on the floor, one roasting machine, and a daily deadline. Everyone works for the same business — and everyone was working around the same process.

**And there's a growth plan:** the owners intend to open up to **seven cafes over the next five years**. The process is already awkward at three. It does not survive seven.

## Why this business runs on short cycles

Roasted coffee has a **peak-freshness window**. It is not "fresher is always better": in the first days after roasting the beans are still degassing CO₂ and brew unpredictably, most coffees hit their peak around **days 5–14**, and past roughly a month they oxidise and lose the complexity that specialty coffee is sold on. Espresso needs more rest than filter; darker roasts settle faster than lighter ones.

The operational consequence is unavoidable:

- **You cannot roast a month of stock in advance** — it would leave the window before it is poured.
- **You cannot roast tonight for tomorrow morning** — it hasn't rested.
- Therefore the roastery must work in **short, repeating cycles**, and the cafes must order **frequently and in small quantities** to keep every bag inside its window.

Frequent small orders across multiple sites is the price of serving good coffee. It is also, precisely, a **daily coordination load** — and that load was being carried by people, by hand.

## What that cost, before the system

Nothing here is anyone's fault. The team was competent and the coffee was good. But a process held together by attention rather than structure has predictable failure modes:

| Failure | What it cost the business |
|---|---|
| **No single source of truth.** Orders arrived by email, by messages in a group chat, and occasionally through a shared spreadsheet nobody kept current. | Orders read twice or not at all; no reliable answer to "what exactly are we roasting today?" |
| **Manual aggregation and green-bean math.** Every morning: total each coffee across all sites, then convert roasted demand into green-bean weight. | 30–45 minutes of a skilled roaster's time daily, and arithmetic errors landing on expensive green coffee. |
| **Late add-on orders.** A site remembers something after the plan is set. | Either a second production setup for a coffee already run — reset, re-profile, re-weigh — or a manual recalculation nobody fully trusts. |
| **No visibility of unfinished work.** A disrupted day (illness, equipment, a rush) leaves work half-done. | The shortfall is discovered downstream, when a cafe runs out — too late to roast, rest and deliver inside the freshness window. |

None of these are dramatic failures. They are the ordinary, cumulative cost of running a multi-site operation without standardisation — and they scale badly: **the coordination load grows with every site the company opens.**

### The other half of the math: shrinkage

Green coffee loses **11–24% of its weight** during roasting: moisture evaporates, chaff separates, gases off-gas. Lighter roasts lose less, darker roasts more; specialty roasters typically work in the **11–16%** band.

So the roaster cannot simply weigh out what the cafes ordered. To deliver 30 kg of roasted coffee at 16% loss, they must load **30 ÷ (1 − 0.16) = 35.7 kg of green beans**. This backwards calculation — *"how much green do I need for today's orders?"* — is a real, daily, manual task in small roasteries. It is exactly what this system automates.

*(Roast-logging software calculates weight loss too — but* ***after*** *the roast, from actual green-in / roasted-out weights, for quality control. Planning the green load* ***before*** *the roast, against a day's worth of orders from every site, is a different job.)*

---

## What the market offers

I evaluated what this business could buy off the shelf. List prices in **USD per month**, as published July 2026.

### Roastery-specific software

| Product | Price | Built for |
|---|---|---|
| **Artisan** | Free (open source) | Roast profiling only — temperature curves, rate-of-rise. No business functions. |
| **RoastLog Starter** | **$129** (3 users) | Roast profiling, cupping, green inventory |
| **RoastLog Growth** | **$249** (5 users) | + inventory reporting, sample management |
| **RoastLog Scale** | **$499** | + **role-based permissions**, multi-location |
| **Cropster** | from **$79** (promo, ≤250 kg/mo) → **~$216** regular, **plus volume charges above 250 kg** | Roast profiling, production planning, quality intelligence |

### General inventory / manufacturing platforms (incl. two Auckland companies)

| Product | Price | Built for |
|---|---|---|
| **Cin7** (Auckland, NZ) | from **~$299–349** | Multi-channel retail & wholesale ERP |
| **Unleashed** (Auckland, NZ) | from **~$259–399** | Inventory & distribution, light BOM |
| **Katana** | from **$299** | Cloud MRP; tiers driven by sales-order volume |

---

## The honest gap analysis

**This is not a "my app beats Cropster" story.** Cropster and RoastLog are excellent at what they were built for — roast *quality*: temperature curves, rate-of-rise, cupping scores, profile consistency. My system does none of that, and doesn't try to.

The question is different: **what does this business's daily operation actually need — and can it buy that?**

| The operational need | Off-the-shelf answer |
|---|---|
| **Each cafe orders for itself, sees its own history and status, and cannot touch another site's data** | ❌ **Not offered.** These platforms treat order intake as an internal, single-desk function. A multi-site ordering surface with per-site isolation isn't in the box. |
| **Role-based access** (a cafe manager must not see production; the roaster must not edit catalogs) | 💰 RoastLog: **only on Scale, $499/mo** |
| **A late order for the same coffee joins a production task that hasn't been run yet** — no second setup | ❌ Not a feature anywhere. It depends on *this* roastery's physical constraint. |
| **Unfinished work from a bad day resurfaces automatically, numbers intact** | ❌ Generic "overdue" reports exist; a worklist that carries yesterday's green-bean weight into today's plan does not. |
| **Pricing that doesn't punish growth** | 💰 Cropster charges by roast volume. Katana's tiers move with order count — many small orders are penalised. |

**The pattern: the market sells roast quality and inventory. This business's pain was multi-site order intake, production planning and packing.** That is the layer nobody was selling.

---

## The solution

A custom system on Microsoft Power Platform, built in the business's own language ("Cafe", "Stock Order", "Roast Batch"), covering exactly those four failures:

- **Cafe Order App** — each cafe manager orders from a tablet and sees live status and history. Identity comes from their login; each site's data is isolated **at the database level**, not by a screen filter. One surface, one source of truth — no chasing an order across an inbox, a chat and a spreadsheet.
- **Automatic production plan** — at 16:00 daily (or on one tap), the system aggregates every site's order, groups it by coffee + roast level into **production tasks**, applies each coffee's shrinkage to compute the green weight to load, and generates the grind & packaging plan. No morning arithmetic, ever.
- **Merge logic** — a late order for the same coffee and roast level **joins a production task that hasn't been run yet** instead of creating a second one. If the task has already started, the system respects physics and plans a separate one.
- **Guided packing** — one order = one box; "Box ready" unlocks only when every line is ticked. An incomplete box cannot ship to a cafe.
- **Nothing is forgotten** — unfinished production tasks from previous days appear at the top of the daily worklist with a date badge and their numbers intact.
- **Management view** — full history, audit trail, demand charts by origin and package size: which cafes drive which coffees, and what the roastery is actually producing.

### Why merging matters — the production-planning term is *changeover cost*

Each coffee × roast level is a distinct setup: the profile is loaded, the green is weighed, the machine is dialled in. Running the same coffee twice in a day because one cafe ordered late means paying that setup twice — time, energy, and the roaster's attention.

Grouping demand by coffee × roast level, and merging late demand into a task that hasn't started, is a textbook **setup-reduction** move. It is also the rule that pays back hardest as the business grows: **more cafes means more overlapping orders for the same coffees — and more setups avoided.** The value of this system rises with every site the company opens.

---

## The cost, today and at seven cafes

**Power Apps Premium: $20 USD/user/month** (unlimited apps, Dataverse, automation). A pay-as-you-go option (~$10/active user/app/month) suits users who open only one app — which is exactly the cafe managers' pattern.

| Scenario | Users | Monthly (USD) |
|---|---|---|
| **Today: 3 cafes + roaster** | 4 | **$80** (or ~$50 pay-as-you-go) |
| **Five-year plan: 7 cafes + roaster** | 8 | **$160** (or ~$90 pay-as-you-go) |

Today's $80 USD ≈ **NZD ~133/month**:

> ### NZD ~4.40 per day — for the entire business.
> **Less than one flat white.**

Against what the shelf offers a business that needs multi-user roles:

| Option | USD/month | ≈ NZD/day | In coffees |
|---|---|---|---|
| **This system (3 cafes)** | **$80** | ~$4.40 | **under one** |
| **This system (7 cafes)** | **$160** | ~$8.80 | about one and a half |
| RoastLog Scale (roles included) | $499 | ~$27 | about five |
| Cin7 / Unleashed (entry) | $299–399 | ~$16–22 | three to four |
| Cropster (regular + volume charges) | $216+ and rising with output | ~$12 and climbing | two and climbing |

**Note the growth curve.** Cropster's bill rises with roast volume; Katana's with order count. This system's rises only with *people* — and at the full seven-cafe build-out it still costs less than a third of the roastery tier that merely *includes* user roles, while doing what none of them do.

### Five-year total cost of ownership — the honest version

A custom system carries a build cost that a subscription does not. Taking the growth plan at face value (three cafes now, seven by year five, licences rising as sites open):

| | Build | 5 years of licences (indicative) | 5-year total |
|---|---|---|---|
| **This system** | one-off consultant engagement | ~NZD 11,000–13,000 | **build + ~12,000** |
| **RoastLog Scale** | NZD 0 | ~NZD 49,800 | **~49,800** |
| **Cin7 / Unleashed** | onboarding fees typical | ~NZD 34,800+ | **~34,800 + onboarding** |

Even with a professional build, the custom route stays well ahead over the horizon the owners are actually planning for — **and it is the only route that covers the process end to end.**

---

## Where a custom system is the wrong answer

A consultant who cannot say this shouldn't be trusted with the rest of the advice:

- **If the real problem is roast quality** — profile consistency, cupping, rate-of-rise curves — **buy Cropster or RoastLog.** I cannot build that, and I shouldn't.
- **If the business needs full accounting, tax and multi-channel e-commerce** — buy an ERP. Cin7 and Unleashed exist for a reason.
- **A custom system needs an owner.** A subscription comes with a vendor, a support desk and a roadmap; a custom build needs someone to maintain it. That dependency is real.
- **Below a certain scale, spreadsheets win.** One cafe and one roaster? A shared sheet is honest and free. The case for a system begins the moment coordination becomes the bottleneck — which, for this business, it already has.

**Custom becomes the right answer when the process is specific, the constraints are physical, and off-the-shelf options either don't cover the workflow or price the necessary features at the top tier.** That was exactly this case.

---

## Known simplifications and the roadmap

Stated plainly, because a portfolio that hides its edges teaches nothing.

**1. Shrinkage is stored per coffee, not per coffee × roast level.**
In reality, weight loss depends on the bean's moisture content *and* the roast degree — the same coffee roasted dark loses more than roasted light. The model carries one shrinkage figure per origin.
*Next iteration: move the shrinkage percentage onto the coffee × roast-level combination — a data-model change, not an architectural one.*

**2. Machine capacity is not modelled.**
A production task of 35 kg of green coffee is not one machine load: a small roastery's machine typically takes 5–15 kg at a time, so the task becomes several sequential loads. The system plans *what and how much*, not *how many loads*.
*Next iteration: add machine capacity as reference data and split each task into loads automatically — which also unlocks realistic time estimates for the roasting day.*

**3. Growth to seven cafes changes the licence count, not the architecture.**
This is worth being precise about, because it's the question an owner asks first. Cafe managers are **staff of the same company**, not external customers — so they belong in the company tenant with security roles and team ownership, exactly as they are today. Opening a fourth, fifth or seventh site means: create the cafe record, create its owner team, add the manager, assign a licence. **No redesign, no rebuild, no migration.** The data model, the automation, the security model and all three apps carry over untouched.

*The architecture would only change if the business model did* — if the company began supplying **independent** cafes as wholesale customers, those external users would call for **Power Pages** (contact-based external identity, table permissions instead of security roles, per-authenticated-user licensing) rather than tenant accounts. Even then, only the ordering surface would be rebuilt: the Dataverse core, the flows and the internal apps would carry over. Knowing *which* layer a business decision touches — and which it leaves alone — is the difference between a system that scales and a system that gets replaced.

---

## Postscript: a conversation with a real roaster

After the build was finished, I went and talked to someone who does this for a living — the owner of a small Auckland roastery-cafe. We talked for about fifteen minutes while he made my coffee — he showed me his machine, described his day and how he tracks it. It was the most useful quarter of an hour of the whole project.

**What my model leaves out.** His operation is deeper than mine in exactly the dimension I deliberately excluded: quality. Every day begins with calibration. Roast profiles are built from metrics, not memory. Yield is recorded per roast, because it genuinely varies from sack to sack — same origin, different lot, different moisture, different outcome. His machine holds **1 kg** per load; that limit is a design choice rather than a quality one, and he intends to scale up. All of it is tracked in Google Sheets.

**What it confirmed.** A production task is not a machine load — it is a sequence of them. My model already carries this as a stated simplification; standing next to a 1 kg drum turned an abstraction into something physical.

**What it clarified about scope.** This system automates the **coordination** layer: demand across sites, consolidation, the green-bean calculation, the production plan, the packing, and the tracking of unfinished work. His pain isn't there — he runs one site, and coordination is a conversation with himself. His depth is in **quality**: profiles, calibration, yield analysis. Different problems, different systems.

### What I would actually recommend to him

Not "buy nothing". Not "buy everything". **Understand what the current setup actually rests on — and automate when that foundation stops being safe.**

Everything he records by hand is modellable: roast profiles as records with their metrics, yields per batch, calibration history, green-stock traceability from sack to cup — plus the analytics his sheets can't give him today (*which lots actually yield best at this profile?*). In the Microsoft stack this is a small system, not a large one: a Dataverse model, a couple of interfaces, automated reporting.

**And the entry cost is genuinely low.** His operation is two people — himself and one staff member. Two Power Apps licences (per-user, or pay-as-you-go for someone who opens a single app) puts a purpose-built, cloud-hosted, backed-up system in the same monthly range as a couple of streaming subscriptions. *(Licences are per person who uses an app — there is no single "owner licence" covering a team; verify current rates before quoting.)*

**So why isn't the answer "yes, today"?**

Because his process works. But it is worth being precise about *how* it works: the automation there is **semi-manual, and it rests on a person** — not on a structure. It runs because someone knows where everything is, why it's there, and what the numbers mean. That knowledge is real expertise, and it is doing real work. It is also completely undocumented, unshareable, and one sick day away from being unavailable.

That is not a criticism. At one site, with one roaster, holding the process in your head is *efficient* — a system would add ceremony to decisions a competent person makes instantly. The spreadsheet isn't the foundation. **He is.**

**What changes that is growth — and he named it himself.** A second site. Another roaster on the floor. Wholesale accounts. At that point the business begins to outgrow what one person can hold in their head, and the failure mode shifts: it is no longer *"the spreadsheet is clumsy"*, it is *"the knowledge is no longer shared, no longer checkable, and no longer survives an absence."* Numbers stop reconciling. Nobody is sure which sheet is current. Traceability breaks precisely when a customer asks for it.

**That is the moment to automate — and the reason is worth saying out loud:**

> **Automation isn't there to replace the person who holds the process. It is there to make what they know reliable — and legible to everyone else.**

Before that point, a system is an expense. After it, its *absence* is one — paid in errors, rework and forgotten commitments rather than in invoices, which is exactly why it goes unnoticed until it hurts.

### Why this matters for a consultant, not just for him

A good implementation partner doesn't win by selling everything to everyone. They win by being right about *timing*, and by being there when the client reaches it.

That conversation wasn't a lost sale. It was a **qualified opportunity with a named trigger**: the owner now knows his process is fully modellable, that the entry cost is modest, and that the moment his business grows past one head, the answer already exists. He also knows I told him the truth when the truth was "not yet" — which is the only reason he would believe me when it becomes "now".

**What I took away.** Fifteen minutes with an owner genuinely inspired by his craft taught me more about the domain than any documentation had — and it teaches judgement about timing. The question is never *"can this be automated?"*; the answer is always yes, and the ceiling on customisation is infinite. The question is *"what is this process currently resting on — and how long will that hold?"*

---

## What this project demonstrates

- **Requirements gathering from a real physical constraint**, not from a feature list: the merge boundary — *once a production task has started, nothing can be added to it* — came from how roasting physically works, and it became the most valuable rule in the system.
- **Fit-gap analysis with a documented "buy" recommendation** where buying is the right call.
- **Domain literacy**: the green-bean calculation, the 11–24% weight-loss range, the difference between *logging* a roast and *planning* one — plus honesty about what the model simplifies.
- **Design decisions with reasoning and revisit triggers** — including two capabilities evaluated and deliberately rejected (a chatbot and a process flow) because adding them would have made the system worse.
- **Security designed in data, not in screens** — site isolation survives even if an app link leaks.
- **An architecture with a known growth path** — and the ability to say precisely which layer a growth decision touches, and which it doesn't.
- **Field research over assumption** — and the judgement to time an automation decision by what the process currently rests on, rather than by what could technically be built.

---

*Prices are publicly listed rates as of July 2026 (USD), converted at an indicative rate — verify current pricing and exchange rates before relying on these figures. Weight-loss ranges are drawn from published industry sources (Barista Hustle, Royal Coffee, Rob Hoos Coffee Consulting). This is a portfolio case study built on a fictional business: the engineering, the decisions and the trade-offs are real.*

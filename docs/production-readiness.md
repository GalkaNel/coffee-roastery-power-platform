# Production Readiness

This document separates the current portfolio implementation from the work required for a reliable production deployment.

The solution is functional as a demonstration environment, but several design choices were shaped by developer-tenant constraints and small data volumes. Those choices are documented here together with their production path.

## Contents

* [Current environment](#current-environment)
* [Environment adaptations](#environment-adaptations)
* [Known limitations](#known-limitations)
* [Domain simplifications](#domain-simplifications)
* [Production hardening](#production-hardening)
* [Licensing considerations](#licensing-considerations)

## Current environment

The solution was built in a Power Apps Developer Plan environment with Dataverse.

The current implementation assumes:

* three company-owned cafés;
* named café managers rather than anonymous or public users;
* one primary roastery operator;
* relatively small transactional data volumes;
* no Teams or Exchange integration in the development tenant;
* portfolio demonstration rather than live commercial operation.

These assumptions explain several implementation choices but do not change the overall architecture.

## Environment adaptations

The following adaptations were made deliberately and each has a defined production path.

| Constraint                                                                                                                 | Current adaptation                                                                                                                                                           | Production path                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Choice values were not resolved reliably in one Canvas App. Label comparisons also proved unsafe for write operations.     | A status dictionary is initialised in `OnStart` using the actual values returned by `Choices()`. Displays use labels, while comparisons and writes use stored choice values. | Standardise direct choice comparisons and remove positional assumptions. A numeric status-code column could also be introduced where reporting or delegation requires it. |
| Date-only Dataverse columns arrive in Canvas Apps as DateTime values. Direct comparison with `Today()` can therefore fail. | Canvas formulas reconstruct the date using `Date(Year(d), Month(d), Day(d))`. Power Automate converts UTC to New Zealand time before applying business-date rules.           | Define and document a consistent timezone policy for every date and datetime column.                                                                                      |
| Some predicates are non-delegable, including reconstructed dates, label comparisons and full-table row counts.             | Reference data is loaded into collections at application start. Transactional records are queried live and the approach is accepted at demonstration scale.                  | Replace remaining non-delegable predicates, use indexed fields and test against expected production volumes.                                                              |
| The developer tenant does not include Teams or Exchange licences.                                                          | Exceptions remain visible inside the operator's normal worklist and flow results are shown inside the application.                                                           | Add Teams adaptive cards or email notifications using the same operational triggers.                                                                                      |
| Canvas Apps do not receive server-push updates.                                                                            | Always-open operational screens use timer-based polling.                                                                                                                     | Evaluate push notifications, shorter targeted refreshes or model-driven auto-refresh depending on the final operating environment.                                        |

## Known limitations

### Mid-run failure recovery

Orders are changed to `In Production` only after production artifacts have been created.

If a flow fails partway through, some Roast Batches or Packaging Tasks may exist while the source orders remain `Submitted`. A rerun could therefore create duplicates.

The current implementation documents this risk rather than claiming full transactional idempotency.

A production version should use artifact-level existence checks, claim-first processing or a processing-run identifier.

### Mixed status-comparison patterns

Earlier screens use some label-based status comparisons, while later screens use the status dictionary.

Both approaches currently work, but production code should use one consistent pattern throughout.

### Limited status reversal

A completed roasting task cannot be reopened through the operator's primary tap-based interface.

Reversal is available only through the model-driven administrative path. This reduces accidental changes but should be supported by a documented correction procedure and audit history.

### In-application notifications only

The current solution surfaces overdue and unfinished work inside the normal operational worklist.

It does not yet send Teams, email or mobile push notifications.

## Domain simplifications

### Shrinkage is stored per coffee

The current model stores one expected shrinkage percentage per coffee or blend.

In real production, weight loss also depends on roast level. The same coffee roasted dark generally loses more weight than when roasted light.

**Next iteration:** move shrinkage to the coffee-and-roast-level combination.

This is a data-model refinement rather than an architectural redesign.

### Machine capacity is not modelled

A production task represents the total required quantity for one coffee and roast level.

It does not split that quantity into individual physical machine loads.

For example, a task requiring 35 kg of green coffee may require several sequential roaster loads depending on machine capacity.

**Next iteration:** store machine capacity as reference data and automatically split production tasks into loads.

This would also support more realistic daily time estimates.

### `Roast Batch` means production task

In the current Dataverse schema, a `Roast Batch` record represents consolidated production demand for one coffee and roast level.

It does not necessarily represent one physical drum load.

The term is retained in the solution, but `Production Task` would be clearer in a future schema revision.

## Production hardening

Before live deployment, the following work should be completed:

### Automation reliability

* prevent concurrent scheduled and manual runs from processing the same orders;
* introduce run-level traceability;
* add duplicate detection before creating production artifacts;
* define recovery and compensation behaviour for failed runs;
* add structured error handling and operator-visible failure messages.

### Application quality

* standardise status comparisons across all screens;
* replace remaining non-delegable formulas;
* test with realistic production data volumes;
* test tablet, desktop and mobile layouts;
* document the correction path for mistakenly completed work.

### Security and governance

* validate every security role using end-user accounts;
* document café-team onboarding and offboarding;
* enable auditing for important status changes;
* use separate development, test and production environments;
* introduce environment variables and managed deployment practices;
* review data-retention and backup requirements.

### Operational readiness

* agree who owns reference data;
* define support and incident procedures;
* establish monitoring for failed flows;
* document daily cutoff exceptions;
* train café managers and roastery staff;
* confirm the meaning of production statuses with real operators.

## Licensing considerations

The portfolio solution was built using the Power Apps Developer Plan.

A production deployment would require appropriate licensing for Dataverse-backed applications and automation.

### Likely small-scale option

For three cafés and one roastery operator, per-app licensing may be more economical than broad per-user licensing, depending on the applications each person needs to access.

The final choice should be confirmed against current Microsoft licensing terms before deployment.

### Power Automate

The scheduled automation runs once per business day, with occasional manual execution.

The volume is small, but production planning should still consider:

* connector entitlements;
* Power Platform request limits;
* flow ownership;
* service accounts;
* premium connector requirements;
* support when the owning user leaves the organisation.

### Dataverse capacity

Order, production and packaging records are relatively small.

Base Dataverse capacity is likely to be sufficient at the initial scale, but actual usage should be monitored as the number of cafés and historical records grows.

### External-user alternatives

The current design assumes named internal or guest users accessing a Canvas App.

Power Pages may become more appropriate if the company expands to many independently operated cafés, requires anonymous ordering or needs a public-facing portal.

The decision should be based on user identity, scale, security and current licensing costs rather than a fixed café-count threshold.

> Licensing changes frequently. All production estimates must be validated against Microsoft's current licensing documentation before purchase.

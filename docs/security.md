# Security Model

This document explains how the solution restricts café data, assigns application access and applies least-privilege permissions in Dataverse.

## Security principles

The design uses two independent layers:

1. **Dataverse security** controls which records and operations a user can access.
2. **Application sharing** controls which application interfaces are available to each role.

Application access narrows the visible surface, but it is not treated as the security boundary.

## Café-level data isolation

Each café has a dedicated Dataverse **Owner Team**.

Stock Orders and Order Lines are owned by the corresponding café team. Café managers inherit access through their team membership and therefore see only records belonging to their own café.

The café is derived from the signed-in user and is never selected manually in the application.

Row-level isolation was tested under café-manager accounts. A System Administrator can see all records by design, so security validation must always be performed using the intended end-user role rather than the maker account.

## Security roles

| Role        | Dataverse access                                                                                                                                                                                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Cafe User` | Create, read, update and delete Stock Orders and Order Lines at User-level scope with team inheritance; read access to product and café reference data; no access to production tables.                                                                                                |
| `Roaster`   | Organization-level create, read and update access to Roast Batches and Packaging Tasks; no delete access to production history; read access to Stock Orders; update access to Order Lines for packing confirmation; limited update access to Stock Orders for completing a packed box. |

Delete access is deliberately withheld from the `Roaster` role because production history should not be removed through normal operations.

## Application access

* **Cafe Order App** is shared with users assigned the `Cafe User` role.
* **Packing Station** is shared with the roaster.
* **Roastery Ops** is shared only with the `Roaster` role.

Even if an application link is shared accidentally, Dataverse permissions continue to restrict the underlying records.

## Least privilege

Permissions are tied to specific user actions.

Whenever the interface introduces a new write operation, the corresponding security role must be reviewed and retested under an end-user account.

This keeps least privilege as an ongoing design constraint rather than a one-time configuration task.

## Production considerations

A production deployment should also include:

* named team owners and a documented process for café-manager onboarding;
* periodic review of team membership and security-role assignments;
* auditing for status changes and other operationally important updates;
* separate maker, test and production environments;
* formal testing of every role after solution import.

Related environment and deployment considerations are documented in [Production readiness](production-readiness.md).

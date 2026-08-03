## Security model
This document describes how café-level data isolation and application access are enforced in Dataverse.



Two independent layers:

**1. Data layer — security roles + Owner Teams**

| Role | Access |
|---|---|
| `Cafe User` | CRUD on Stock Order / Order Line at **User (Basic)** level with team inheritance; Read on catalog; **no access** to production tables. Each cafe's orders are owned by that cafe's **Owner Team** → managers see only their own cafe (verified by test). |
| `Roaster` | Create/Read/Write **Organization** on Roast Batch & Packaging Task (**no Delete** — production history is never deleted); Read on orders; Read+Write on Order Lines (packing ticks); **Write on Stock Order** scoped to the process need (closing a box) — Create/Delete deliberately not granted. |

**2. Application layer — app-to-role assignment.** Roastery Ops is shared only to `Roaster`. App access narrows the surface but is **not** the security boundary — row-level security holds even if an app link leaks.

**Cafe identity is derived from login, never selected by the user.**

*Least privilege as a living contract: every new writing gesture in the UI triggers a role review, tested under the end user's account — not the maker's.*
Row-level isolation is enforced by security roles and Owner Teams, not by app-side filters — the history gallery deliberately queries the table without a cafe filter. Verified under end-user accounts: a cafe manager sees only their own cafe's orders. (A System Administrator sees all rows by design — security must always be tested under the end user, never under the maker.)

---
kind: ard
key: BOOK-1
title: Back-in-stock alerts — ARD
scope: prd
prd: BOOK-1
epic: null
area: null
status: draft
grounded_repos:
  - bookstore @ /workspace/bookstore
components:
  - id: bookstore:alerts
  - id: bookstore:storage
  - id: bookstore:books
  - id: bookstore:clients
  - id: bookstore:web
  - id: bookstore:ingest
  - id: bookstore:k8s
    kind: deploy
inherits: null
derived_from: specifications/PRD-BOOK-1-back-in-stock-alerts/prd.md
---

# Back-in-stock alerts — ARD

## Context

The PRD lets a client subscribe to a book that has no available copies and be alerted inside the store within one minute of the book coming back into stock ([AC#6]); every subscriber is alerted, a subscription ends when its alert is delivered, alerts stay listed for 30 days, a book removed from the catalogue ends its subscriptions with a withdrawal alert ([AC#14]), and the synthetic client traffic must show subscribing, cancelling and alerting in every five-minute window ([AC#16]). The one-minute limit must hold with up to 1,000 clients waiting on one book, and the feature must not slow the store's existing journeys for clients who never use it ([SMC#1]). This ARD fixes where that state lives, how a stock change reaches the clients waiting on it, how a client is identified, and the interfaces between the services involved.

## Grounding findings (architecture as-is)

Grounded on `bookstore` at `master` (`dbb9f2a`, committed 2026-10-07).

- **Stock is owned by the storage service alone.** The only code that raises a book's quantity is `StorageController.ingestBook()`, reached from `POST ""` and `POST /ingest-book`; it adds to an existing row or inserts a new one — `storage/src/main/java/com/dynatrace/storage/controller/StorageController.java:74` (add at `:84`, insert at `:82`). `PUT /{id}` saves an arbitrary quantity — `StorageController.java:131`. Rows are removed by `DELETE /{id}` and by `DELETE /delete-all`, a truncate — `StorageController.java:135`, `:141`.
- **Nothing compares a quantity before and after a write.** The entity holds `isbn` (unique), `quantity`, `created_at` and `updated_at` and no stock state — `storage/src/main/java/com/dynatrace/storage/model/Storage.java:8`; the repository is a plain JPA repository with no listeners or events — `storage/src/main/java/com/dynatrace/storage/repository/StorageRepository.java:13`.
- **Orders also restock.** A failed or cancelled order returns stock through storage's `/ingest-book` — `orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:295`, `:316`.
- **Storage already answers stock queries by ISBN** — `GET /api/v1/storage/findByISBN`, `StorageController.java:51`.
- **The catalogue's removals.** `DELETE /{id}` deletes by id only, with no ISBN lookup and no notice to any other service — `books/src/main/java/com/dynatrace/books/controller/BookController.java:96`; `DELETE /delete-all` truncates — `BookController.java:104`; unpublishing is a separate, non-removing toggle — `BookController.java:112`. Books are found by ISBN through `GET /find` — `BookController.java:52`.
- **There is no client identity.** No service has sign-in, session or token handling; clients are plain records keyed by a unique email with no credential fields — `clients/src/main/java/com/dynatrace/clients/model/Client.java:10` — found through `GET /find?email=` — `clients/src/main/java/com/dynatrace/clients/controller/ClientController.java:50` — and every service allows every origin — `storage/src/main/java/com/dynatrace/storage/config/WebConfig.java:10`. Other services name a client by email and a book by ISBN, with no foreign key across services — `carts/src/main/java/com/dynatrace/carts/model/Cart.java:25`, `:28`.
- **Services call each other only over synchronous HTTP.** One `RestTemplate`-backed `*Repository` class per remote service, its URL injected from properties — `storage/src/main/java/com/dynatrace/storage/repository/BookRepository.java:15`, `storage/src/main/resources/application-dev.properties:18`. No message broker, server push or scheduler exists anywhere in the repository; the only asynchronous construct is ingest's thread pool for its own HTTP calls — `ingest/src/main/java/com/dynatrace/ingest/config/AsyncConfiguration.java:13`.
- **The web store is a set of per-entity pages with no per-client view.** Routes are per-entity CRUD — `web/src/app/app-routing.module.ts:38`; each page calls one service over HTTP through its own Angular service — `web/src/app/books/book.service.ts:11` — against a base URL per service — `web/src/environments/environment.ts:8`.
- **Synthetic traffic is one bulk loop started through the API.** `POST /api/v1/ingest` starts a thread that repeats `generate()` with no delay between iterations — `ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:226` — and every iteration first wipes orders, carts and storage, and optionally clients and books, through their `delete-all` endpoints — `IngestController.java:256`. Ingest can already sell and return stock — `ingest/src/main/java/com/dynatrace/ingest/repository/StorageRepository.java:32`. Generated books take ISBNs counting down from `9999999999999` — `ingest/src/main/java/com/dynatrace/ingest/model/Book.java:31`, `:54` — and generated clients take faker emails that must match one pattern — `ingest/src/main/java/com/dynatrace/ingest/model/Client.java:12`.
- **Bulk resets have three kinds of caller.** Books' and clients' `DELETE /delete-all` are called by ingest's bulk loop through its shared repository method — `ingest/src/main/java/com/dynatrace/ingest/repository/IngestRepository.java:16` — by the web store's *Delete All* actions — `web/src/app/books/book.service.ts:42`, reached from the button at `web/src/app/books/book-list/book-list.component.html:35`, and `web/src/app/clients/client.service.ts:34` — and by the ingest Deployment's init container, which resets every service before ingest starts — `k8s/ingest.yaml:125`, `:126`.
- **Deployment.** Each service runs on container port 8080 — `k8s/ingest.yaml:30` — and the browser reaches it through an ingress path named after the service, such as `/api/ingest/` and `/api/storage/` — `k8s/ingress.yaml:16`, `:44`.

## Architecture decisions

### [AD#1]: Subscriptions and alerts live in a new alerts service

**Binds:** `bookstore:alerts`, and every component that reads or changes a subscription or an alert.
**Prevents:** subscription and alert state spreading across the services that own clients or stock, and any service reading another's database.
**Rule:** Subscriptions and alerts are created, read, changed and deleted only by `bookstore:alerts`, in a PostgreSQL database no other service connects to; every other component reaches them through alerts' REST API only, and alerts refers to a client by email and to a book by ISBN, holding no foreign key into any other service.
**Alternatives:**
- A new alerts service on MySQL — no advantage over PostgreSQL, which the stock-side services (books, storage, orders) already use.
- Extend `bookstore:clients` — puts the restock fan-out and the store-wide unread-count polling on the service every journey already depends on.
- Extend `bookstore:storage` — couples every stock write with alert fan-out and subscription reads in the one service that owns stock.

### [AD#2]: A client is identified by a caller-supplied email, checked on subscribe

**Binds:** `bookstore:alerts`, `bookstore:web`, `bookstore:ingest`.
**Prevents:** a partial authentication scheme appearing in one service while every other service stays open, and the unread-count polling of [AD#9] turning into load on `bookstore:clients`.
**Rule:** Every request to alerts' client-facing API ([AD#8]) names its client by an `email` query parameter, and alerts performs no authentication; only subscribe checks the email, accepting it when `bookstore:clients` `GET /api/v1/clients/find?email=` returns a client, while every other operation trusts the email as given — a read for an email alerts holds nothing for returns empty lists and a count of 0. The web store sends the email of the *current client* the user has selected in the store.
**Alternatives:**
- Add client sign-in in this PRD — a new cross-service capability the PRD does not scope.
- A token issued by alerts alone — no identity provider exists to issue it from, and no other service would honour it.
- Check the email on every request, with a cache — the same polling load on `bookstore:clients` at every cache expiry, for no gain on reads that already return only what that email subscribed to.

### [AD#3]: Out of stock means no storage row or a quantity of zero

**Binds:** `bookstore:storage`, `bookstore:alerts`.
**Prevents:** storage and alerts disagreeing on when a book is out of stock or has come back.
**Rule:** An ISBN is out of stock when storage holds no row for it or its row's quantity is 0; a *restock* is any committed storage write — `POST ""`, `POST /ingest-book`, `PUT /{id}`, a new row included — that leaves the ISBN with a quantity of 1 or more from out of stock, and deleting or truncating rows is never a restock.
**Alternatives:**
- A quantity of zero only — a book with no storage row could then never be subscribed to, though it has no available copies.

### [AD#4]: Storage tells alerts when a book becomes available

**Binds:** `bookstore:storage` (consumer), `bookstore:alerts` (producer).
**Prevents:** a restock reaching subscribers late, or a failing alerts service failing or slowing a stock write.
**Rule:** After committing a restock ([AD#3]), storage calls alerts `POST /api/v1/alerts/available?isbn=<the ISBN>` with no body, once, and never retries; alerts, in one transaction completed within 10 seconds of the call even with 1,000 pending subscriptions, records a back-in-stock alert for every pending subscription to that ISBN and ends those subscriptions, and answers `204 No Content` whether or not any subscription existed. A repeated call alerts nobody again, since the subscriptions it would alert have ended. Storage never fails, rolls back or delays its own response because of this call; [AD#5] recovers a call that was lost or failed. No ordering is required between calls for different ISBNs.
**Alternatives:**
- Alerts polls storage only — every restock waits for the next sweep, and the sweep load grows with the number of waiting ISBNs.
- A message broker between storage and alerts — a new infrastructure dependency in every deployment, where nothing in the repository uses one.
- Call alerts inside storage's transaction — a slow or failing alerts service would then fail or slow the restock.

### [AD#5]: Alerts sweeps pending subscriptions at least every 30 seconds

**Binds:** `bookstore:alerts` (consumer); `bookstore:storage` and `bookstore:books` (producers of existing endpoints).
**Prevents:** a lost [AD#4] or [AD#6] call leaving clients waiting for a book that has come back or has been removed, and a transient error ending subscriptions with a false withdrawal.
**Rule:** At least every 30 seconds, alerts checks each ISBN that has a pending subscription: where storage `GET /api/v1/storage/findByISBN` reports a quantity of 1 or more it acts exactly as on an [AD#4] call, and where books `GET /api/v1/books/find` answers not-found it acts exactly as on an [AD#6] call, in either case recording the alerts within 10 seconds; a missing storage row is out of stock ([AD#3]), and any other answer from either service — an error, a timeout, a busy response — leaves the subscription pending until the next sweep. It relies on both endpoints as they are today.
**Alternatives:**
- No sweep — a single failed call then loses every alert it carried.
- Sweep every storage row rather than only the pending ISBNs — load grows with the catalogue instead of with what clients wait for.

### [AD#6]: Books tells alerts when a book is removed

**Binds:** `bookstore:books` (consumer), `bookstore:alerts` (producer).
**Prevents:** subscriptions waiting forever on a book that can no longer come back.
**Rule:** Books' single delete (`DELETE /api/v1/books/{id}`) reads the book's ISBN, deletes it, and then calls alerts `POST /api/v1/alerts/withdrawn?isbn=<the ISBN>` with no body, once, never retrying; alerts ends every pending subscription to it with a withdrawal alert, carrying the book title it stored when the subscription was made, and answers `204 No Content` whether or not any existed, so a repeat ends nothing new. A failed call never fails the delete; [AD#5] recovers it. Books' bulk `delete-all` and unpublishing (`/vend`) never call this endpoint.
**Alternatives:**
- Also send withdrawal alerts on `delete-all` — the synthetic traffic's reset would then alert every waiting client on each iteration.
- Treat unpublishing as removal — unpublishing is reversible and leaves the book in the catalogue.

### [AD#7]: A bulk catalogue reset resets alerts, silently

**Binds:** `bookstore:books` (consumer), `bookstore:alerts` (producer).
**Prevents:** subscriptions and alerts outliving the books a reset removed, a reset flooding clients with alerts, and a late repeat of a reset deleting subscriptions made after it.
**Rule:** Books' `DELETE /api/v1/books/delete-all` calls alerts `DELETE /api/v1/alerts/delete-all` exactly once per reset, after its own delete, and never retries it; alerts removes every subscription and alert that is not reserved ([AD#11]) without recording any alert, and answers `204 No Content`. A failed call never fails the reset; subscriptions it left behind are withdrawn by [AD#5] once books reports their book not-found.
**Alternatives:**
- Leave alerts untouched on a reset — subscriptions to removed books then stay pending until [AD#5] withdraws them, alerting their clients.
- Have only the synthetic traffic reset alerts — a reset run by hand would leave the same orphans.
- Let books retry the call — a retry arriving after new subscriptions were made would delete them.

### [AD#8]: Alerts publishes one client-facing API

**Binds:** `bookstore:alerts` (producer), `bookstore:web` and `bookstore:ingest` (consumers).
**Prevents:** the web store and the synthetic traffic relying on different behaviour for the same journey.
**Rule:** Under `/api/v1/alerts`, every operation names its client by the `email` query parameter ([AD#2]):
- `POST /subscriptions?email=&isbn=` — subscribe. `201 Created` with the new subscription; `200 OK` with the existing one where the client already waits on that ISBN; `409 Conflict` where storage reports the ISBN in stock ([AD#3]); `404 Not Found` where books knows no such ISBN or clients no such email; `503 Service Unavailable` where storage, books or clients does not answer, and no subscription is created.
- `GET /subscriptions?email=` — `200 OK` with the client's pending subscriptions.
- `DELETE /subscriptions/{id}?email=` — cancel. `204 No Content`, also for a subscription already cancelled or ended; `404 Not Found` where the id is not one of that email's subscriptions.
- `GET ?email=` — `200 OK` with the client's alerts, newest first.
- `GET /unread-count?email=` — `200 OK` with a JSON object whose `count` field is the number of unread alerts.
- `POST /read?email=` — mark every alert of the client read. `204 No Content`, idempotent.
- `DELETE /{id}?email=` — dismiss. `204 No Content`, also for an alert already dismissed; `404 Not Found` where the id is not one of that email's alerts.

A subscription carries `id`, `email`, `isbn`, `title` and `createdAt`; an alert carries `id`, `type` (`BACK_IN_STOCK` or `WITHDRAWN`), `isbn`, `title`, `occurredAt` (when the book came back or was removed), `createdAt` and `read`. Lists and counts never include an alert older than 30 days or one the client dismissed, and an email alerts holds nothing for reads as empty lists and a count of 0.
**Alternatives:** none weighed — an interface row; its shape follows the repository's existing REST convention of query-parameter lookups (`ClientController.java:50`, `BookController.java:112`).

### [AD#9]: The one-minute limit is split between sweep, record and poll

**Binds:** `bookstore:web`, `bookstore:alerts`.
**Prevents:** an alert recorded within its limit staying invisible past one minute on an open page.
**Rule:** While any store page is open with a current client selected, the web store asks alerts for that client's unread-alert count at least every 20 seconds, and fetches the alerts list when the client opens it; with [AD#5]'s sweep at most 30 seconds and recording within 10 seconds ([AD#4], [AD#5]), a restock or removal is visible within 60 seconds when its call was lost and within 30 seconds when it arrived, meeting the limit of [AC#6] and [AC#14].
**Alternatives:**
- Server-sent events from alerts — the repository's first streaming endpoint, plus ingress configuration, for a count that changes rarely.
- Refresh only on navigation — an idle page could show the count late by any amount.
- A longer recording budget with a shorter poll — twice the polling load from open pages.

### [AD#10]: The synthetic back-in-stock journey runs on its own schedule

**Binds:** `bookstore:ingest`.
**Prevents:** the demo cadence of [AC#16] depending on the bulk loop being started, or on its speed.
**Rule:** Ingest runs a back-in-stock journey on a schedule of at most five minutes, started with the ingest service itself and independent of the bulk loop; each cycle uses reserved books and clients ([AD#11]), sells a reserved book to out of stock, subscribes several reserved clients to it, cancels one subscription, and restocks it, through the same public interfaces any client uses ([AD#8]) and storage's existing sell and restock calls.
**Alternatives:**
- Fold the journey into the bulk loop — its cadence would then depend on the loop being started through the API and on its speed, and the loop's own reset would erase the journey's state between steps.

### [AD#11]: No bulk reset removes the journey's reserved data

**Binds:** `bookstore:ingest` (owner of the definition), `bookstore:books`, `bookstore:clients`, `bookstore:alerts`.
**Prevents:** a reset landing mid-cycle and costing [AC#16] a five-minute window, and the services disagreeing on which data is reserved.
**Rule:** A *reserved book* is one whose ISBN starts with `00000000`; a *reserved client* is one whose email ends with `@bis-journey.invalid`; a *reserved subscription or alert* is one whose ISBN or email is reserved. Ingest owns this definition and creates its journey data only inside it. Books' and clients' `DELETE /delete-all` and alerts' `DELETE /api/v1/alerts/delete-all` remove every row except reserved ones, which they leave in place; storage's `delete-all` is unchanged and may remove reserved rows, since a missing row is out of stock ([AD#3]).
**Alternatives:**
- A reserved flag on each record — a schema change in books, clients and alerts for what a literal prefix and domain already express.
- A configured list of reserved ISBNs and emails — every service then reads shared configuration that the bulk loop can drift from.
- The journey rebuilds its data every cycle — a reset that lands mid-cycle still costs that window.

## Cross-repo / component approach

All work is in the one repository, `bookstore`.

| Capability | Lands in | Decisions |
|---|---|---|
| Subscribe, cancel, list pending subscriptions | `bookstore:alerts`, `bookstore:web`, `bookstore:ingest` | [AD#1], [AD#2], [AD#8] |
| Detect a restock and alert the waiting clients | `bookstore:storage`, `bookstore:alerts` | [AD#3], [AD#4], [AD#5] |
| Alerts list, unread count, dismissal, 30-day retention | `bookstore:alerts`, `bookstore:web` | [AD#8], [AD#9] |
| Withdraw subscriptions when a book is removed | `bookstore:books`, `bookstore:alerts` | [AD#5], [AD#6] |
| Reset alerts with the catalogue | `bookstore:books`, `bookstore:alerts` | [AD#7] |
| Keep the journey's reserved data through bulk resets | `bookstore:ingest`, `bookstore:books`, `bookstore:clients`, `bookstore:alerts` | [AD#11] |
| Identify the current client | `bookstore:web`, `bookstore:alerts`, `bookstore:clients` (existing endpoint) | [AD#2] |
| Synthetic back-in-stock journey | `bookstore:ingest`, `bookstore:alerts`, `bookstore:storage` (existing endpoints) | [AD#10] |
| Deploy the new service | `bookstore:k8s` — rides along on `bookstore:alerts` | — |

## Contracts

| AD | Producer | Consumers | Kind | Status | Artifact |
|---|---|---|---|---|---|
| [AD#4] | bookstore:alerts | bookstore:storage | REST | new | — |
| [AD#5] | bookstore:storage | bookstore:alerts | REST | exists | — |
| [AD#5] | bookstore:books | bookstore:alerts | REST | exists | — |
| [AD#6] | bookstore:alerts | bookstore:books | REST | new | — |
| [AD#7] | bookstore:alerts | bookstore:books | REST | new | — |
| [AD#8] | bookstore:alerts | bookstore:web, bookstore:ingest | REST | new | — |
| [AD#2] | bookstore:clients | bookstore:alerts | REST | exists | — |
| [AD#10] | bookstore:storage | bookstore:ingest | REST | exists | — |
| [AD#11] | bookstore:books | bookstore:ingest, bookstore:web | REST | changed | — |
| [AD#11] | bookstore:clients | bookstore:ingest, bookstore:web | REST | changed | — |
| [AD#11] | bookstore:ingest | bookstore:books, bookstore:clients, bookstore:alerts | shared schema | new | — |

### Schema ownership

- `bookstore:alerts` owns the subscription and alert shapes and their database; no other component persists them ([AD#1], [AD#8]).
- `bookstore:storage` owns stock — the quantity per ISBN and the meaning of out of stock ([AD#3]).
- `bookstore:books` owns the catalogue and the ISBN as a book's identity.
- `bookstore:clients` owns the client record and the email as a client's identity.
- `bookstore:ingest` owns the definition of reserved data ([AD#11]); books, clients and alerts apply it as stated there.

### Versioning and compatibility

Every new endpoint is under `/api/v1/alerts`. Within v1 a change is additive only — a new optional field or a new endpoint; a change that removes or alters a field or a status code takes `/api/v2`, with v1 kept until storage, books, web and ingest have moved. **The meaning of two existing v1 endpoints changes**: books' and clients' `DELETE /delete-all` now keep reserved rows ([AD#11]), and books' also resets alerts ([AD#7]). The path and the status code are unchanged, so no caller breaks, but every caller now gets the new behaviour: ingest's bulk loop, the web store's *Delete All* actions on books and clients, and the ingest init container's reset at startup (Grounding findings). A hand reset from the web store therefore leaves the journey's reserved books and clients in place and clears every non-reserved subscription and alert; the web Epic states this in its *Delete All* behaviour rather than discovering it. The reserved-data definition changes only through a new decision superseding [AD#11], landed in ingest, books, clients and alerts together. The existing endpoints alerts relies on (`storage` `findByISBN`, `books` `find`, `clients` `find`) and the storage calls ingest uses keep the fields alerts and ingest read today — a book's existence by ISBN, a stock quantity, a client's existence by email.

### Landing order

1. `bookstore:alerts`
2. `bookstore:storage`
3. `bookstore:books`
4. `bookstore:clients`
5. `bookstore:web`
6. `bookstore:ingest`

## Stack & invariants

- Spring Boot 3.5.3 for every service, the new alerts service included — `build.gradle:3`. The web store is Angular — `web/package.json:16`.
- Inter-service calls follow the repository's convention: one `RestTemplate`-backed `*Repository` class per remote service, its URL from `application-*.properties` with a `DT_<SERVICE>_SERVER` default — `storage/src/main/java/com/dynatrace/storage/repository/BookRepository.java:15`, `storage/src/main/resources/application-dev.properties:18`.
- Every service controller extends the shared base controller — `common/src/main/java/com/dynatrace/controller/SecurityController.java:14`, `books/src/main/java/com/dynatrace/books/controller/BookController.java:22` — and alerts' controllers do too.
- Each service runs on container port 8080 behind its own Kubernetes service port and an ingress path named after the service — `k8s/ingest.yaml:30`, `k8s/ingress.yaml:16`; alerts takes the next free service port and `/api/alerts/`.
- Each service owns its database; none connects to another's ([AD#1]).

## Edge cases & risks

- **Fan-out at scale.** One restock may alert 1,000 waiting clients, and [AD#4] and [AD#5] give the recording 10 seconds of the one-minute limit; the fan-out must stay inside those 10 seconds at that size.
- **Restock and sell-out race.** A restock may sell out again before an alerted client looks; the alert makes no claim about current stock ([AC#7]).
- **Orders restock too.** A cancelled or failed order that returns the last copies alerts waiting clients ([AD#3], `OrderController.java:295`, `:316`), as the PRD intends.
- **No authentication.** Any caller can act as any client by naming its email ([AD#2]); this matches every other service today and is accepted for this PRD.
- **Deleted and re-created clients.** A client removed from `bookstore:clients` keeps its subscriptions and alerts in alerts, referenced by email only ([AD#1]): its pending subscriptions end only on a restock, a withdrawal or a reset, and its alerts age out after 30 days. A client later re-created with the same email — the bulk loop re-creates clients on every iteration — sees the earlier client's subscriptions and alerts, since alerts knows a client only by email.
- **The bulk loop's storage wipes.** The bulk loop truncates storage on every iteration (`IngestController.java:256`), so every ISBN briefly goes out of stock and the regenerated rows restock it; waiting clients of non-reserved books are alerted on such a cycle, which is correct under [AD#3].
- **Sweep load.** [AD#5] calls storage and books once per ISBN with a pending subscription every 30 seconds; the load grows with the number of distinct ISBNs clients wait for.
- **Polling load.** Every open store page with a current client polls alerts every 20 seconds ([AD#9]); because only subscribe checks the email with clients ([AD#2]), that load stays on alerts and does not reach `bookstore:clients`, which carts and orders already call on the store's existing journeys.

## Open questions

- **The PRD assumes signed-in clients, which the store does not have.** The PRD's assumption that "the store can already tell which signed-in client is viewing a page" is false: no service has sign-in, session or token handling (`ClientController.java:50`, `WebConfig.java:10`). [AD#2] adopts a current client selected by email in the store. The PRD's [AC#1] wording "signed-in client" and "a visitor who is not signed in as a client", its [AC#8] reliance on knowing the client viewing a page, and the assumption itself should be revised with `/product-workflows:update-prd` to say *the current client selected in the store*.

- **A bulk catalogue reset departs from two acceptance criteria.** [AD#7] has books' `delete-all` clear every non-reserved subscription and alert without sending any alert, and the web store's *Delete All* makes that reachable by hand as well as by the synthetic traffic (Grounding findings). That departs from [AC#14], under which every client waiting on a removed book receives a withdrawal alert, and from [AC#12], under which a delivered alert stays listed for 30 days; the PRD makes no exception for a bulk reset. [AC#12] and [AC#14] should be revised with `/product-workflows:update-prd` to exclude a bulk catalogue reset, or [AD#7] revisited in a refine of this ARD.

## Deferred

Left to `/dev-workflows:design` for the Epics this PRD splits into:

- The 30-day retention mechanism in alerts — a scheduled purge or an on-read filter ([AD#8]).
- How each `delete-all` excludes reserved rows under [AD#11]'s definition, and the journey's choice of reserved ISBNs and emails within it.
- Alerts' schema and indexes, and how [AD#4]'s fan-out meets 10 seconds for 1,000 waiting clients.
- The timeout and threading of storage's and books' once-only calls ([AD#4], [AD#6], [AD#7]).
- Where the current-client picker, the alerts list and the unread count sit in the web store ([AD#2], [AD#9]).
- Alerts' Kubernetes manifests, database deployment and service port.

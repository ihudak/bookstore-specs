---
kind: epic
key: BOOK-1-01
target: bookstore:alerts
---

# Subscriptions and alerts service

## Goal
Every client waiting on an out-of-stock book is recorded once and receives a back-in-stock or withdrawal alert within the one-minute limit, kept for 30 days, so the store can tell clients a book is back instead of their having to keep checking.

## Business value
Today the interest a client shows in an out-of-stock book is lost the moment they leave its page; holding that interest and turning a restock into an alert for every waiting client is what recovers the sales the PRD targets, and it gives the demo its first journey in which one stock change reaches many waiting clients.

## Scope

### In scope
- A new `alerts` Spring Boot 3.5.3 module and service that alone creates, reads, changes and deletes subscriptions and alerts, in its own PostgreSQL database `dt_books_alerts` that no other service connects to, referring to a client by email and to a book by ISBN with no foreign key into any other service ([AD#1]).
- The client-facing API under `/api/v1/alerts` exactly as [AD#8] states it, each operation naming its client by the `email` query parameter with no authentication ([AD#2]): `POST /subscriptions?email=&isbn=` (subscribe), `GET /subscriptions?email=` (pending subscriptions), `DELETE /subscriptions/{id}?email=` (cancel), `GET ?email=` (alerts, newest first), `GET /unread-count?email=`, `POST /read?email=` (mark every alert read) and `DELETE /{id}?email=` (dismiss); a subscription carries `id`, `email`, `isbn`, `title` and `createdAt`, and an alert carries `id`, `type` (`BACK_IN_STOCK` or `WITHDRAWN`), `isbn`, `title`, `occurredAt`, `createdAt` and `read`.
- Subscribe's checks: the email against clients' existing `GET /api/v1/clients/find?email=` ([AD#2]), the ISBN and the book's title against books' existing `GET /api/v1/books/find`, and stock against storage's existing `GET /api/v1/storage/findByISBN`, where no storage row or a quantity of 0 is out of stock ([AD#3]).
- `POST /api/v1/alerts/available?isbn=` for storage's restock notice ([AD#4]) and `POST /api/v1/alerts/withdrawn?isbn=` for books' removal notice ([AD#6]).
- `DELETE /api/v1/alerts/delete-all` for books' bulk catalogue reset, removing every subscription and alert that is not reserved under [AD#11] and recording no alert ([AD#7]).
- A sweep, at least every 30 seconds, of each ISBN with a pending subscription against storage's `findByISBN` and books' `find`, recovering a restock or removal notice that was lost or failed ([AD#5]).
- Keeping each alert in lists and counts for 30 days from when it was recorded, and leaving out dismissed and older alerts ([AD#8]); whether by a scheduled purge or an on-read filter is left to design, as are the schema, the indexes and how the fan-out meets 10 seconds for 1,000 waiting clients.
- The scaffolding every service shares: the `settings.gradle` include, `build.gradle`, `Dockerfile`, `push_docker.sh`, `application-{dev,test,stage,prod}.properties` with `http.service.storage`, `http.service.books` and `http.service.clients` on `DT_<SERVICE>_SERVER` defaults, the per-service `WebConfig` CORS, controllers that extend the shared `SecurityController` with alerts' own config and version endpoints, and `dt_books_alerts` added to `database/pg/init/01-create-db.sql`.
- Also touches: bookstore:k8s — `k8s/alerts.yaml` (Deployment and Service on the next free service port), the `/api/alerts/` path in `k8s/ingress.yaml` ahead of the web catch-all, `DT_ALERTS_SERVER` in `k8s/configmap.yaml`, `alerts_agent` in `k8s/config_agents.yaml`, and alerts' entries in `restart.sh`, `delete.sh`, `build_docker_all.sh` and, where it lists services, `preset_deployment.sh` — each exists only to deploy and configure alerts; the port and the manifests' detail are left to design.

### Out of scope
- Detecting a restock in storage and calling `/available` — [[BOOK-1-02]].
- Calling `/withdrawn` and `/delete-all` from books, and keeping reserved books through a reset — [[BOOK-1-03]].
- The web store's current client, subscribe action, alerts list, pending-subscriptions list and unread count — [[BOOK-1-05]].
- The synthetic back-in-stock journey and the choice of its reserved ISBNs and emails — [[BOOK-1-06]].
- Email, SMS or any other notification sent outside the store.
- Holding or reserving a restocked copy for an alerted client, or alerting only as many clients as there are copies.
- Subscriptions that lapse on a timer, or that stay active after their alert and fire again on a later restock.
- Withdrawal alerts on a bulk catalogue reset alerts is told of, and treating unpublishing a book as removing it.
- Marking an alert unread again, or restoring a dismissed alert.
- Checking that a caller is the client whose email it names, and removing a deleted client's subscriptions and alerts.
- Any number of available copies in a subscription or an alert.

## Acceptance criteria
- Given a client email and an ISBN, when subscribe is called, then alerts records one pending subscription of that email to that ISBN, carrying the book's title, and answers `201 Created` where clients' `find` returns the email, books knows the ISBN and storage reports it out of stock; answers `200 OK` with the existing subscription and records nothing new where the email already waits on that ISBN; and records nothing, answering `409 Conflict` where storage reports the ISBN in stock, `404 Not Found` where books knows no such ISBN or clients no such email, and `503 Service Unavailable` where storage, books or clients does not answer.
- Given pending subscriptions to an ISBN, when `POST /api/v1/alerts/available?isbn=` is called, then alerts records, in one transaction completed within 10 seconds even with 1,000 pending subscriptions, a `BACK_IN_STOCK` alert for every pending subscription to that ISBN and ends each of them, answering `204 No Content` whether or not any existed; a repeated call alerts nobody again, and a cancelled or already-ended subscription is never alerted.
- Given pending subscriptions to an ISBN, when `POST /api/v1/alerts/withdrawn?isbn=` is called, then alerts records, in one transaction completed within 10 seconds even with 1,000 pending subscriptions, a `WITHDRAWN` alert carrying the book title it stored when the subscription was made for every pending subscription to that ISBN and ends each of them, answering `204 No Content` whether or not any existed, so a repeated call ends nothing new.
- Given a pending subscription whose restock or removal notice never arrived, when at most 30 seconds pass, then alerts' sweep acts exactly as on `/available` where storage's `findByISBN` reports a quantity of 1 or more, and exactly as on `/withdrawn` where books' `find` answers not-found, recording the alerts within 10 seconds; a missing storage row, a quantity of 0, a book that is unpublished but still found, and any other answer — an error, a timeout, a busy answer — leave the subscription pending, and no subscription ever ends because of the time since it was made.
- Given an email, when its pending subscriptions, alerts or unread count are read, then alerts returns only that email's own — alerts newest first, leaving out any the email dismissed or that were recorded more than 30 days ago — and empty lists and a `count` of 0 for an email it holds nothing for; `POST /read` marks every alert of that email read and answers `204 No Content`, however often it is repeated.
- Given an email, when it cancels a subscription or dismisses an alert by id, then alerts ends that subscription or leaves that alert out of every later read and answers `204 No Content`, also where it was already ended or dismissed, and answers `404 Not Found` and changes nothing where the id is not one of that email's.
- Given subscriptions and alerts of reserved and other books and clients, when `DELETE /api/v1/alerts/delete-all` is called, then alerts removes every subscription and alert whose ISBN does not start with `00000000` and whose email does not end with `@bis-journey.invalid`, keeps the rest — another client's subscription to a reserved book included — records no alert, and answers `204 No Content`.
- Given the Kubernetes manifests applied with `restart.sh`, when the browser calls `/api/alerts/` through the ingress or a pod calls the address in `DT_ALERTS_SERVER`, then alerts answers, and every ingress path that existed before routes as it did, with the web catch-all still last.

## Independent Test
[[BOOK-1-01]] is verifiable standalone by deploying alerts beside storage, books and clients as they run today and calling its API directly — every subscribe outcome, `/available` and `/withdrawn` each recording 1,000 alerts within 10 seconds, the sweep alerting within 40 seconds on a restock made in storage with no notice, the reads, cancel, dismiss, and `delete-all` against fixtures built to [AD#11]'s reserved definition — and delivers the store's record of who waits for which book, and their alerts, without any not-yet-built Epic.

## Dependencies
- Storage's existing `GET /api/v1/storage/findByISBN` ([AD#5]) — exists, used as it runs; subscribe and the sweep need a quantity per ISBN, where no row means out of stock ([AD#3]).
- Books' existing `GET /api/v1/books/find` ([AD#5]) — exists, used as it runs; subscribe needs the book's existence and title, and the sweep reads not-found as removal; it returns a book whether or not it is published, which keeps an unpublished book's subscriptions pending ([AD#6]).
- Clients' existing `GET /api/v1/clients/find?email=` ([AD#2]) — exists, used as it runs; subscribe needs a client's existence by email.
- PostgreSQL, as books, storage, orders and ingest use it — `dt_books_alerts` is added to `database/pg/init/01-create-db.sql`, which runs only when a volume is first initialised, so an existing deployment needs the database created separately; how is left to design.
- [[BOOK-1-06]] — owns [AD#11]'s reserved-data definition (an ISBN starting with `00000000`, an email ending with `@bis-journey.invalid`); this Epic needs only the definition as the ARD fixes it, not BOOK-1-06's code, so it does not wait for it, and its Independent Test uses fixtures built to the definition in place of the journey's data.
- No earlier Epic: alerts lands first, and before [[BOOK-1-02]] and [[BOOK-1-03]] land, the sweep alone delivers restock and removal alerts.

## Contract
- Produces: [AD#4] — `POST /api/v1/alerts/available?isbn=`: alert and end every pending subscription to the ISBN in one transaction within 10 seconds, `204` whether or not any existed.
- Produces: [AD#6] — `POST /api/v1/alerts/withdrawn?isbn=`: end every pending subscription to the ISBN with a withdrawal alert, `204` whether or not any existed.
- Produces: [AD#7] — `DELETE /api/v1/alerts/delete-all`: remove every non-reserved subscription and alert without recording an alert, `204`.
- Produces: [AD#8] — the client-facing API under `/api/v1/alerts`: subscribe, pending subscriptions, cancel, alerts, unread count, mark read and dismiss.
- Consumes: [AD#5] — storage's `GET /api/v1/storage/findByISBN` — exists — used as it runs.
- Consumes: [AD#5] — books' `GET /api/v1/books/find` — exists — used as it runs.
- Consumes: [AD#2] — clients' `GET /api/v1/clients/find?email=` — exists — used as it runs.
- Consumes: [AD#11] — the reserved-data definition; the Independent Test runs against fixtures built to it, standing in for the journey's data.

## Covers
- [US#1], [US#2], [US#3], [US#4], [US#5]
- [AC#1], [AC#3], [AC#4], [AC#6], [AC#7], [AC#8], [AC#9], [AC#10], [AC#11], [AC#12], [AC#13], [AC#14], [AC#15], [AC#17]
- [SM#1], [SM#2]
- [U01], [U01/AC01], [U01/AC04], [U01/AC05], [U01/AC07]
- [U02], [U02/AC01], [U02/AC02], [U02/AC04], [U02/AC07]
- [U04], [U04/AC02], [U04/AC03], [U04/AC04], [U04/AC05], [U04/AC08]
- [U05], [U05/AC01], [U05/AC02], [U05/AC03], [U05/AC04], [U05/AC06]
- [U06], [U06/AC01], [U06/AC02], [U06/AC03], [U06/AC07], [U06/AC08], [U06/AC09]
- [U07], [U07/AC01], [U07/AC02], [U07/AC04]
- [U08/AC04]

## Suggested stories
- Scaffold the alerts module, its database and its Kubernetes deployment: Gradle include, build file, Dockerfile, properties for every profile, `SecurityController`-based controllers with config and version endpoints, `dt_books_alerts`, manifest, ingress path, configmap and agents entries, and the build, apply and delete scripts.
- Persist subscriptions and alerts by email and ISBN, with the dismissed and 30-day rules applied to every list and count.
- Subscribe, with the clients, books and storage checks and every [AD#8] outcome.
- Pending-subscriptions list, cancel, alerts list, unread count, mark read and dismiss.
- `/available`: the transactional fan-out, with a load test showing 1,000 alerts recorded within 10 seconds.
- `/withdrawn` and `delete-all`, the latter keeping reserved subscriptions and alerts; `/withdrawn` with a load test showing 1,000 pending subscriptions ended with their alerts recorded within 10 seconds.
- The 30-second sweep against storage and books, leaving subscriptions pending on any answer other than a quantity of 1 or more or a not-found book.

## References
- Parent PRD: [[BOOK-1]]
- [Source: ard.md#Architecture decisions] — [AD#1] to [AD#9] and [AD#11]; [Source: ard.md#Contracts] — interface rows and landing order
- [Source: specification.md#U01], [Source: specification.md#U02], [Source: specification.md#U04], [Source: specification.md#U05], [Source: specification.md#U06], [Source: specification.md#U07] — the test cases these criteria condense
- [Source: bookstore/settings.gradle#include] — one include per module; there is no alerts module
- [Source: bookstore/database/pg/init/01-create-db.sql#create database] — four databases at lines 1–4, no `dt_books_alerts`
- [Source: bookstore/common/src/main/java/com/dynatrace/controller/SecurityController.java#SecurityController] — base class every controller extends, requiring `getConfigRepository()`
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/repository/BookRepository.java#BookRepository] — the RestTemplate client convention to copy for the storage, books and clients clients
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/controller/StorageController.java#getStorageItemByISBN] — `findByISBN`, line 51
- [Source: bookstore/books/src/main/java/com/dynatrace/books/controller/BookController.java#getBookByIsbn] — `find`, line 52
- [Source: bookstore/clients/src/main/java/com/dynatrace/clients/controller/ClientController.java#getClientByEmail] — `find?email=`, line 50
- [Source: bookstore/k8s/storage.yaml#Deployment] — manifest template, including the agent key and `DT_PG_DBNAME`
- [Source: bookstore/k8s/ingress.yaml#paths] — one path per service, web catch-all last
- [Source: bookstore/k8s/configmap.yaml#DT_*_SERVER] — service addresses at lines 11–20, no `DT_ALERTS_SERVER`
- [Source: bookstore/k8s/config_agents.yaml#storage_agent] — per-service agent keys

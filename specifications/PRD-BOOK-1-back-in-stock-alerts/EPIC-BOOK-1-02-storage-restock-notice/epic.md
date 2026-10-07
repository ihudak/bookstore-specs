---
kind: epic
key: BOOK-1-02
target: bookstore:storage
---

# Restock notice from storage

## Goal
Every client waiting on a book is alerted within seconds of its copies going from none to one or more — whether from new stock, a corrected quantity or a returned order — without any stock write becoming slower or less reliable.

## Business value
The restock is the moment a waiting client can buy, and an alert that arrives while copies remain is the sale the PRD sets out to recover; telling alerts at that moment rather than at its next sweep spends little of the one-minute limit, while the stock writes every order depends on stay as fast and reliable as today.

## Scope

### In scope
- Reading the ISBN's quantity before each stock write that can raise it: `POST ""` and `POST /ingest-book`, which both run through `StorageController.ingestBook` and which also serves orders' returns of stock on a failed payment or a cancelled order, including the case where no row exists yet; and `PUT /{id}`, which today saves the request body without reading the stored row.
- After such a write commits and leaves the ISBN with a quantity of 1 or more from out of stock — no row or a quantity of 0 ([AD#3]) — calling alerts' `POST /api/v1/alerts/available?isbn=<the ISBN>` once, with no body, and never retrying it ([AD#4]).
- A RestTemplate-backed `AlertsRepository` following storage's `BookRepository` convention, with `http.service.alerts=http://${DT_ALERTS_SERVER:localhost:<alerts' dev port>}/api/v1/alerts` in each `application-*.properties`.
- Isolating the call so that it never fails, rolls back or delays the stock write's own response ([AD#4]); its timeout and threading are left to design.

### Out of scope
- Calling alerts on a sell, a `DELETE /{id}`, a `delete-all` or any write that leaves the quantity at 0, none of which is a restock ([AD#3]).
- Calling alerts when a book that already has copies gains more.
- Retrying a failed or lost call; alerts' sweep recovers it ([AD#5]).
- Any change to `findByISBN`, `sell-book` or `delete-all`; storage's `delete-all` stays as it is under [AD#11] and may remove the journey's stock rows.
- Any change in `orders`, whose returns of stock already reach `ingestBook`.
- Holding or reserving restocked copies for alerted clients.

## Acceptance criteria
- Given an ISBN that is out of stock — no storage row or a quantity of 0 — when a `POST ""`, `POST /ingest-book` (an order's return of stock included) or `PUT /{id}` commits and leaves it with 1 or more copies, then storage calls `POST /api/v1/alerts/available?isbn=<that ISBN>` exactly once, with no body, after the commit, whatever the number of copies; and on any other committed write — copies added to an ISBN that already has some, a write that leaves 0, a sell, a `DELETE /{id}` or a `delete-all` — storage makes no call to alerts.
- Given alerts does not answer, answers with an error or takes 30 seconds to answer, when a restock commits, then the write returns the same response and leaves the same quantity as it does without this feature, with no added delay, and storage does not call alerts again for it.
- (guard) Every storage endpoint — `findByISBN`, `sell-book` and `ingest-book`, which alerts and ingest read, included — keeps its path, status codes, response fields and stored result as today, and under the same synthetic load and store configuration the 95th-percentile response time of the browse, cart and order journeys of clients holding no subscription stays within 10% of a baseline run without this feature.

## Independent Test
[[BOOK-1-02]] is verifiable standalone by running storage against a stub of alerts' `POST /api/v1/alerts/available` ([AD#4]) that records every call — restocking an out-of-stock ISBN through `POST ""`, `POST /ingest-book` and `PUT /{id}` records one call each, while adding copies to an in-stock ISBN, selling and deleting record none, and with the stub stopped or answering after 30 seconds every write completes as before with no added delay — and delivers the immediate restock signal without any not-yet-built Epic.

## Dependencies
- [[BOOK-1-01]] — produces `POST /api/v1/alerts/available` ([AD#4]) and adds `DT_ALERTS_SERVER` to `k8s/configmap.yaml`, which storage's Deployment already takes whole through `envFrom`; until it lands, the Independent Test runs against a stub and local development uses the `localhost` default.
- [AD#3] — what out of stock and a restock mean; this Epic applies the definition as stated.
- Orders' existing returns of stock through `POST /ingest-book`, on a failed payment and on a cancelled order — exist, used as they run.

## Contract
- Consumes: [AD#4] — alerts' `POST /api/v1/alerts/available?isbn=`; the Independent Test runs against a stub of it that records each call.

## Covers
- [US#2]
- [AC#5], [AC#6]
- [SM#1], [SMC#1]
- [U01/AC11]
- [U02], [U02/AC01], [U02/AC03], [U02/AC12]

## Suggested stories
- Read the previous quantity on every path that can raise it — `ingestBook` for a new row and for an existing one, and `PUT /{id}` — and decide from [AD#3] whether the committed write is a restock.
- Add `AlertsRepository` and its property for every profile, and make the call once, after the commit, isolated from the write's response.
- Tests: one call per restock path; none for an in-stock increase, a write to 0, a sell or a delete; writes unchanged with alerts stopped or slow.
- Compare the browse, cart and order journeys' 95th-percentile response times under the same synthetic load with a baseline run.

## References
- Parent PRD: [[BOOK-1]]
- [Source: ard.md#Architecture decisions] — [AD#3], [AD#4], [AD#5]
- [Source: specification.md#U02] — AC01, AC03 and AC12 and their test cases
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/controller/StorageController.java#ingestBook] — line 74, shared by `POST ""` and `POST /ingest-book`: a new row is saved at line 82, an existing row's quantity is added at line 84 and saved at line 86
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/controller/StorageController.java#updateStorageById] — `PUT /{id}` saves the body as-is at line 131
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/controller/StorageController.java#sellBooksFromStorage] — the only path that takes stock to zero
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/repository/StorageRepository.java#findByIsbn] — plain JPA repository, no listener or event
- [Source: bookstore/orders/src/main/java/com/dynatrace/orders/repository/StorageRepository.java#returnBook] — orders restock through `/ingest-book`
- [Source: bookstore/orders/src/main/java/com/dynatrace/orders/controller/OrderController.java#returnToStorage] — restock callers at lines 295 and 316
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/repository/BookRepository.java#BookRepository] — the RestTemplate client convention to copy
- [Source: bookstore/k8s/storage.yaml#Deployment] — takes `bookstore-configmap` through `envFrom`

---
kind: epic
key: BOOK-1-03
target: bookstore:books
---

# Removal and reset notices from books

## Goal
Clients waiting on a book that leaves the catalogue are told it is no longer offered, and a bulk catalogue reset neither floods clients with alerts nor removes the books the demo's back-in-stock journey runs on.

## Business value
A client left waiting on a book the store no longer sells is a client the store quietly lets down; telling them lets them look elsewhere in the store. Keeping the journey's books through the resets the demo runs on every pass of its bulk loop is what lets the five-minute cadence of [AC#16] hold at any moment.

## Scope

### In scope
- `DELETE /api/v1/books/{id}`: reading the book's ISBN before deleting it, where today it deletes by id alone, and after the delete calling alerts' `POST /api/v1/alerts/withdrawn?isbn=<the ISBN>` once, with no body, never retrying ([AD#6]).
- `DELETE /api/v1/books/delete-all`: removing every book except reserved ones — those whose ISBN starts with `00000000` ([AD#11]) — in place of today's truncate, then calling alerts' `DELETE /api/v1/alerts/delete-all` exactly once, never retrying ([AD#7]); how the delete leaves reserved rows in place is left to design.
- Isolating both calls so that neither fails, rolls back or delays the delete or the reset; their timeout and threading are left to design.
- Books' first outbound client: a RestTemplate-backed `AlertsRepository` following storage's `BookRepository` convention, with `http.service.alerts=http://${DT_ALERTS_SERVER:localhost:<alerts' dev port>}/api/v1/alerts` in each `application-*.properties`.

### Out of scope
- Withdrawal alerts on a bulk reset ([AD#7]), and any call to alerts from unpublishing or publishing through `/vend`, which is not removal ([AD#6]).
- Retrying either call; alerts' sweep withdraws what a lost call left ([AD#5]).
- Any change to `GET /find`, which alerts' subscribe and sweep rely on as it is today.
- Hiding the journey's books from the store's lists.
- Changing any caller of `delete-all` — ingest's bulk loop, the books list's Delete All in the web store and the start-up reset in `k8s/ingest.yaml` all take the new behaviour unchanged.

## Acceptance criteria
- Given a book in the catalogue, when `DELETE /api/v1/books/{id}` removes it, then books calls `POST /api/v1/alerts/withdrawn?isbn=<its ISBN>` exactly once, with no body, after the delete; a delete of an id the catalogue does not hold answers as it does today, and it and unpublishing or publishing a book through `/vend` call nothing.
- Given a catalogue holding books whose ISBN starts with `00000000` and others, when `DELETE /api/v1/books/delete-all` runs, then every book whose ISBN starts with `00000000` remains and every other book is removed — one whose ISBN starts with seven zeros and then another digit included — with the same path and status code as today, and books then calls `DELETE /api/v1/alerts/delete-all` exactly once.
- Given alerts does not answer, answers with an error or takes 30 seconds to answer, when a single delete or a `delete-all` runs, then it completes with the same response and result as when alerts answers, with no added delay, and books does not call alerts again for it.
- (guard) `GET /find` keeps returning a book by ISBN whether or not it is published, and not-found once it is removed, every other books endpoint keeps its path, status codes and responses, and under the same synthetic load and store configuration the 95th-percentile response time of the browse, cart and order journeys of clients holding no subscription stays within 10% of a baseline run without this feature.

## Independent Test
[[BOOK-1-03]] is verifiable standalone by running books against a stub of alerts' `POST /withdrawn` and `DELETE /delete-all` ([AD#6], [AD#7]) that records each call, with the catalogue seeded from fixtures built to [AD#11]'s reserved-ISBN definition — deleting one book records one withdrawn call for its ISBN, `delete-all` keeps exactly the reserved books and records one reset call, unpublishing records none, and with the stub stopped or slow every delete completes as before with no added delay — and delivers the withdrawal and silent-reset signals without any not-yet-built Epic.

## Dependencies
- [[BOOK-1-01]] — produces `POST /api/v1/alerts/withdrawn` ([AD#6]) and `DELETE /api/v1/alerts/delete-all` ([AD#7]), and adds `DT_ALERTS_SERVER` to `k8s/configmap.yaml`, which books' Deployment takes whole through `envFrom`; until it lands, the Independent Test runs against a stub and local development uses the `localhost` default.
- [[BOOK-1-06]] — owns [AD#11]'s reserved-data definition; this Epic needs only the definition as the ARD fixes it (an ISBN starting with `00000000`), not BOOK-1-06's code, so it does not wait for it, and its Independent Test uses fixtures built to the definition in place of the journey's books.
- The callers of `delete-all`, which take its new meaning with no change of their own: ingest's bulk loop (`IngestController.clearData` through `IngestRepository.deleteAll`), the books list's Delete All in the web store, and the start-up reset in `k8s/ingest.yaml`.

## Contract
- Produces: [AD#11] — books' `DELETE /api/v1/books/delete-all` keeps every book whose ISBN starts with `00000000`, with the same path and status code.
- Consumes: [AD#6] — alerts' `POST /api/v1/alerts/withdrawn?isbn=`; the Independent Test runs against a stub of it that records each call.
- Consumes: [AD#7] — alerts' `DELETE /api/v1/alerts/delete-all`; the Independent Test runs against a stub of it that records each call.
- Consumes: [AD#11] — the reserved-data definition; the Independent Test runs against fixtures built to it, standing in for the journey's books.

## Covers
- [US#5], [US#6]
- [AC#5], [AC#14], [AC#16]
- [SM#1], [SMC#1]
- [U01/AC11]
- [U06], [U06/AC01], [U06/AC03], [U06/AC09], [U06/AC10]
- [U07], [U07/AC01], [U07/AC03]
- [U08], [U08/AC05]

## Suggested stories
- Single delete: load the book to read its ISBN, delete it, then notify alerts once, isolated from the delete's response.
- Reserved-aware `delete-all` in place of the truncate, followed by one isolated reset call to alerts.
- Add `AlertsRepository` and its property for every profile — books' first outbound HTTP client.
- Tests: one withdrawn call per single delete; reserved books kept and the seven-zeros look-alike removed; one reset call per `delete-all`; no call from `/vend`; deletes unchanged with alerts stopped or slow.

## References
- Parent PRD: [[BOOK-1]]
- [Source: ard.md#Architecture decisions] — [AD#5], [AD#6], [AD#7], [AD#11]; [Source: ard.md#Versioning and compatibility] — the changed meaning of `delete-all` and its callers
- [Source: specification.md#U06] — AC09 and AC10; [Source: specification.md#U07] — AC01 and AC03; [Source: specification.md#U08] — AC05
- [Source: bookstore/books/src/main/java/com/dynatrace/books/controller/BookController.java#deleteBookById] — deletes by id only at line 100, with no ISBN lookup and no outbound call
- [Source: bookstore/books/src/main/java/com/dynatrace/books/controller/BookController.java#deleteAllBooks] — truncates at line 108
- [Source: bookstore/books/src/main/java/com/dynatrace/books/controller/BookController.java#vendBookByIsbn] — unpublishing toggle at line 112
- [Source: bookstore/books/src/main/java/com/dynatrace/books/controller/BookController.java#getBookByIsbn] — `find` at line 52
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/repository/IngestRepository.java#deleteAll] — the bulk loop's `delete-all` caller
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java#clearData] — wipes storage, clients and books at lines 256–272
- [Source: bookstore/k8s/ingest.yaml#populate-configs] — start-up reset, books at line 125
- [Source: bookstore/web/src/app/books/book-list/book-list.component.html#Delete All] — the web store's Delete All button at line 35
- [Source: bookstore/storage/src/main/java/com/dynatrace/storage/repository/BookRepository.java#BookRepository] — the RestTemplate client convention to copy

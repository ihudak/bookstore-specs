---
kind: epic
key: BOOK-1-06
target: bookstore:ingest
---

# Synthetic back-in-stock journey

## Goal
From the moment the ingest service starts, its synthetic clients subscribe to an out-of-stock book, cancel, and are alerted when it comes back in every five-minute window, so a demo started at any moment shows the back-in-stock journey working.

## Business value
The demo operator can show one stock change reaching many waiting clients without starting the bulk loop or waiting for a real client to subscribe, and the journey's steady traffic keeps the new alerts path exercised under load like the rest of the store.

## Scope

### In scope
- The reserved-data definition, which ingest owns ([AD#11]): a reserved book is one whose ISBN starts with `00000000`, and a reserved client one whose email ends with `@bis-journey.invalid`. The journey creates its books and clients only inside it, through books' and clients' existing create calls, creating any it needs that is missing; how many there are, and which ISBNs and emails, is left to design.
- A back-in-stock journey on its own schedule of at most five minutes, started with the ingest service itself and independent of the bulk loop ([AD#10]); nothing in ingest is scheduled today, so the scheduling is new.
- Each cycle: sell a reserved book to out of stock through storage's existing sell call, subscribe several reserved clients to it and cancel one of those subscriptions through alerts' client-facing API ([AD#8]), then restock it through storage's existing `ingest-book` call ([AD#10]).
- A RestTemplate-backed `AlertsRepository` with `http.service.alerts=http://${DT_ALERTS_SERVER:localhost:<alerts' dev port>}/api/v1/alerts` in each `application-*.properties`.

### Out of scope
- Keeping reserved rows through books', clients' and alerts' `delete-all` — [[BOOK-1-03]], [[BOOK-1-04]] and [[BOOK-1-01]].
- Changing storage's `delete-all`, which may remove the journey's stock rows, since a missing row is out of stock ([AD#3]).
- Changing the bulk loop's generation, cadence or resets, or the start-up reset in `k8s/ingest.yaml`, which keeps the journey's books and clients once books and clients do.
- Recording the alert, which alerts does for synthetic clients under the same rules as for any other ([AD#4], [AD#5]).
- Hiding the journey's books and clients from the store's lists.

## Acceptance criteria
- Given the ingest service is running, when its activity is watched for 30 minutes, then every 5-minute window holds at least one journey subscription, one cancellation and one delivered back-in-stock alert, whether the bulk loop is not started or runs continuously and resets books, clients and stock on every pass.
- Given the ingest service starts or restarts, then the journey begins within 5 minutes without anyone starting the bulk loop, and starting, stopping or clearing the bulk loop (`POST` or `DELETE /api/v1/ingest`) neither starts nor stops the journey.
- Given a cycle starts, then it sells its reserved book to zero through storage's sell call, taking a book with no storage row as already out of stock, then subscribes several reserved clients to it and cancels one of those subscriptions through alerts' `/api/v1/alerts` API, then restocks it through storage's `ingest-book` call, reaching alerts and storage through no other interface.
- Given a cycle's book has available copies when the journey would subscribe — given one by hand, say — then the journey subscribes no client to it until it has none, and takes alerts' `409 Conflict` as no subscription made.
- Given the journey creates or uses a book or a client, then its ISBN starts with `00000000` or its email ends with `@bis-journey.invalid`, any the journey needs that is missing is created again inside that definition before it is used, and the bulk loop creates no book or client inside it.
- (guard) With the journey running beside it, the bulk loop's start, stop, generation and resets behave as they did before this feature.

## Independent Test
[[BOOK-1-06]] is verifiable standalone by running ingest — first with the bulk loop stopped, then with it running — against a stub of alerts' [AD#8] API that records each subscribe and cancel, stubs of books' and clients' [AD#11] `delete-all` that keep reserved rows, and storage's sell and `ingest-book` calls as they run: every 5-minute window over 30 minutes holds a sell to zero, several reserved clients' subscriptions, one cancel and a restock, the sequence on which alerts records a back-in-stock alert. It delivers the always-on demo journey without any not-yet-built Epic.

## Dependencies
- [[BOOK-1-01]] — produces alerts' client-facing API ([AD#8]) the journey subscribes and cancels through, records the back-in-stock alert on the journey's restock — through its own sweep ([AD#5]) whether or not storage's restock notice from [[BOOK-1-02]] has landed — and adds `DT_ALERTS_SERVER` to `k8s/configmap.yaml`, which ingest's Deployment takes whole through `envFrom`; until it lands, the Independent Test runs against a stub.
- [[BOOK-1-03]] — produces books' `delete-all` that keeps the journey's books ([AD#11]); until it lands, the Independent Test runs against a stub.
- [[BOOK-1-04]] — produces clients' `delete-all` that keeps the journey's clients ([AD#11]); until it lands, the Independent Test runs against a stub.
- Storage's existing `POST /sell-book` and `POST /ingest-book` ([AD#10]), and books' and clients' existing create calls — exist, used as they run.

## Contract
- Produces: [AD#11] — the reserved-data definition (shared schema): a reserved book's ISBN starts with `00000000` and a reserved client's email ends with `@bis-journey.invalid`; ingest creates its journey data only inside it.
- Consumes: [AD#8] — alerts' client-facing API (subscribe and cancel); the Independent Test runs against a stub of it.
- Consumes: [AD#10] — storage's `POST /sell-book` and `POST /ingest-book` — exists — used as it runs.
- Consumes: [AD#11] — books' `delete-all` keeping reserved books; the Independent Test runs against a stub of it.
- Consumes: [AD#11] — clients' `delete-all` keeping reserved clients; the Independent Test runs against a stub of it.

## Covers
- [US#6]
- [AC#16], [AC#17]
- [U08], [U08/AC01], [U08/AC02], [U08/AC03], [U08/AC04]

## Suggested stories
- Reserved-data definition and the journey's books and clients, created inside it wherever missing.
- Add `AlertsRepository` and its property for every profile.
- Schedule the journey to start with the service, separate from the bulk loop's thread and its start and stop.
- The cycle: sell to zero, subscribe several reserved clients, cancel one, restock — handling a missing stock row and a `409`.
- 30-minute cadence check, with the bulk loop stopped and with it running.

## References
- Parent PRD: [[BOOK-1]]
- [Source: ard.md#Architecture decisions] — [AD#8], [AD#10], [AD#11]; [Source: ard.md#Edge cases & risks] — the bulk loop truncates storage on every iteration
- [Source: specification.md#U08] — AC01 to AC04 and their test cases
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java#generateData] — `POST /api/v1/ingest` starts or stops the bulk loop, clearing data first at line 53
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java#BookstoreDataGenerator] — one thread looping `while (isWorking)`, with no scheduler
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java#clearData] — wipes storage always, and clients and books when asked
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/config/AsyncConfiguration.java#asyncExecutor] — `@EnableAsync` only; nothing uses `@Scheduled`
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/repository/StorageRepository.java#buyBook] — sells through `/sell-book` and restocks through `/ingest-book`
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/model/Book.java#generate] — generated ISBNs count down from `9999999999999`
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/model/Client.java#generate] — generated clients take faker emails
- [Source: bookstore/k8s/ingest.yaml#populate-configs] — the start-up reset at lines 121–127, then `POST /api/v1/ingest` with `continuousLoad`

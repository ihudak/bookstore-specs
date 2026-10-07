---
kind: epic
key: BOOK-1-04
target: bookstore:clients
---

# Clients reset keeps the journey's clients

## Goal
A bulk reset of the clients never removes the clients the demo's back-in-stock journey runs on, so a demo started at any moment still shows the journey working.

## Business value
The demo resets the clients on every pass of its bulk loop and at start-up; if those resets wiped the journey's clients, the journey would lose the five-minute windows [AC#16] promises at exactly the moments a demo is most likely to be running.

## Scope

### In scope
- `DELETE /api/v1/clients/delete-all`: removing every client except those whose email ends with `@bis-journey.invalid` ([AD#11]) in place of today's truncate, with the same path and status code; how the delete leaves reserved rows in place is left to design.

### Out of scope
- Any call from clients to alerts: alerts is not told of a clients reset, and a deleted client's subscriptions and alerts stay in alerts, found by email.
- Any change to `GET /find?email=`, which alerts' subscribe relies on as it is today ([AD#2]).
- Sign-in, or any check that a caller is the client it names.
- Hiding the journey's clients from the store's lists.
- Changing any caller of `delete-all` — ingest's bulk loop, the Clients list's Delete All in the web store and the start-up reset in `k8s/ingest.yaml` all take the new behaviour unchanged.

## Acceptance criteria
- Given clients whose email ends with `@bis-journey.invalid` and others, when `DELETE /api/v1/clients/delete-all` runs, then every client whose email ends with `@bis-journey.invalid` remains and every other client is removed — one whose email only contains the domain, such as one ending with `@bis-journey.invalid.example`, included.
- (guard) `delete-all` keeps its path and status code for each of its callers — the bulk loop, the web store's Delete All and the start-up reset.
- (guard) `GET /find?email=` keeps returning the client with that email and not-found where there is none, every other clients endpoint keeps its path, status codes and responses, and clients still calls no other service.

## Independent Test
[[BOOK-1-04]] is verifiable standalone by seeding the clients service with fixtures built to [AD#11]'s reserved-email definition beside ordinary and look-alike clients and calling `delete-all` — exactly the reserved clients remain, the response is as today, and `find` answers as before — and delivers a clients reset the journey survives without any not-yet-built Epic.

## Dependencies
- [[BOOK-1-06]] — owns [AD#11]'s reserved-data definition; this Epic needs only the definition as the ARD fixes it (an email ending with `@bis-journey.invalid`), not BOOK-1-06's code, so it does not wait for it, and its Independent Test uses fixtures built to the definition in place of the journey's clients.
- The callers of `delete-all`, which take its new meaning with no change of their own: ingest's bulk loop (`IngestController.clearData` through `IngestRepository.deleteAll`), the Clients list's Delete All in the web store, and the start-up reset in `k8s/ingest.yaml`.
- No other Epic: clients calls no other service, and nothing here needs alerts or books.

## Contract
- Produces: [AD#11] — clients' `DELETE /api/v1/clients/delete-all` keeps every client whose email ends with `@bis-journey.invalid`, with the same path and status code.
- Consumes: [AD#11] — the reserved-data definition; the Independent Test runs against fixtures built to it, standing in for the journey's clients.

## Covers
- [US#6]
- [AC#16]
- [U08], [U08/AC06]

## Suggested stories
- Reserved-aware `delete-all` in place of `ClientRepository.truncateTable`.
- Tests: reserved clients kept, the look-alike domain removed, the response unchanged, `find?email=` unchanged.

## References
- Parent PRD: [[BOOK-1]]
- [Source: ard.md#Architecture decisions] — [AD#2], [AD#11]; [Source: ard.md#Versioning and compatibility] — the changed meaning of `delete-all` and its callers
- [Source: specification.md#U08] — AC06 and its test cases
- [Source: bookstore/clients/src/main/java/com/dynatrace/clients/controller/ClientController.java#deleteAllClients] — truncates at lines 100–103
- [Source: bookstore/clients/src/main/java/com/dynatrace/clients/repository/ClientRepository.java#truncateTable] — the truncate query
- [Source: bookstore/clients/src/main/java/com/dynatrace/clients/controller/ClientController.java#getClientByEmail] — `find?email=` at line 50
- [Source: bookstore/ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java#clearData] — the bulk loop wipes clients through `delete-all`
- [Source: bookstore/k8s/ingest.yaml#populate-configs] — start-up reset, clients at line 126
- [Source: bookstore/web/src/app/clients/client-list/client-list.component.html#Delete All] — the web store's Delete All button at line 29

---
kind: epic
key: BOOK-1-05
target: bookstore:web
---

# Back-in-stock alerts in the web store

## Goal
A client chosen as the current client in the store can subscribe from an out-of-stock book's page, sees an unread-alert count on every page within a minute of the book coming back or being withdrawn, and can list, cancel and dismiss their subscriptions and alerts, so they no longer have to keep checking whether a book is back.

## Business value
This is where the client meets the feature: the alert's link back to the book's page is the path from a restock to the order [SM#2] counts, and the current client stands in for the sign-in the store does not have ([AD#2]), so the journey works in the store as it is.

## Scope

### In scope
- A current client, made current with a make-current-client action in the Clients list and on a client's details page, remembered in the browser across reloads, new tabs and restarts until it is changed or cleared, with its email shown on every page and a way to clear it ([AD#2]).
- Ways into the alerts list and the pending-subscriptions list on every page while a current client is selected, and a prompt to choose one from the Clients pages in place of either list while none is.
- On the book's page: the book's stock read through storage's existing `findByISBN`, a subscribe action on a book with no available copies ([AD#3]), the subscribed state with a cancel action, the reason a subscription was not made, and a notice when subscriptions are unavailable.
- An unread-alert count on every page, asked of the alerts service at least every 20 seconds while a page is open with a current client ([AD#9]) and cleared by opening the alerts list.
- The alerts list, back-in-stock and withdrawal alerts alike and newest first, with dismissal; and the pending-subscriptions list, with cancellation; each with its unavailable, could-not-be-done and current-client-changed notices.
- An alerts Angular service on the per-service pattern, an alerts base URL in `environment.ts` (the ingress path `/api/alerts`), `environment.development.ts` and `environment.staging.ts`, and the new components declared in the `NgModule` and routed in `app-routing.module.ts`; where the picker, the count and the two lists sit in the pages is left to design.
- The Delete All behaviour on the books and Clients lists under [AD#11] and [AD#7]: a reset keeps the synthetic journey's books or clients, which the reloaded list still shows, and a books reset also clears every subscription and alert except those whose book or client is the demo's own (the journey's). Both confirmations are reworded to say so: on the books list, "Delete all books except the demo's own books? This also clears every subscription and alert except those for the demo's own books and clients."; on the Clients list, "Delete all clients except the demo's own clients?".

### Out of scope
- Sign-in, or any check that the person choosing a current client is that client.
- Showing how many copies are available, on the book's page or in an alert.
- Marking an alert unread again, or restoring a dismissed alert.
- Server push of alerts; the count is polled ([AD#9]).
- Hiding the synthetic journey's books and clients from the store's lists.
- Email, SMS or any other notification sent outside the store.
- Any change to what browsing books, using the cart or placing orders do.

## Acceptance criteria
- Given the Clients list or a client's details page, when someone selects the make-current-client action for a client, then that client becomes the current client, identified by exactly its email, and stays current across page reloads, new tabs and restarts of the same browser — though not in another browser — until a different client is made current or the current client is cleared, which leaves none selected; a client made current in place of another sees only its own unread-alert count, alerts and pending subscriptions, with none of the previous client's left on screen.
- While a current client is selected, every page shows its email, a way into the alerts list and a way into the pending-subscriptions list; while none is selected, no page shows a current client's email, a way into either list, an unread-alert count or a subscribe action, and opening either list by its address shows a prompt to choose a current client from the Clients pages in place of the list.
- Given a current client on a book's page, then the page shows a subscribe action where the book has no available copies — no stock held for it or a stock of zero — and the client holds no pending subscription to it, shows in its place that the client is subscribed, with a cancel action, where the client holds one, shows neither on a book with one or more available copies, and shows that subscriptions are unavailable right now in place of both where it cannot read the book's stock or, for a book with no copies, the client's subscriptions; when the client subscribes, the page shows the subscribed state where the alerts service answers `201` or `200`, and otherwise that no subscription was made and why — copies are available now (`409`), the book or the current client no longer exists (`404`), or the store could not check and the client may try again (`503` or no answer).
- While a page is open with a current client selected, the web store asks the alerts service for that client's unread-alert count at least every 20 seconds and shows the count on every page while it is one or more, so that an alert, back-in-stock alert or withdrawal alert alike, is counted within one minute of the restock or removal that caused it; it shows no count at zero, with no current client or while the alerts service does not answer, and opening the alerts list marks every alert read and clears the count.
- When the current client opens the alerts list, then the web store shows the alerts the alerts service returns, newest first: each back-in-stock alert with the book's title, the date and time the store recorded the book as back in stock and a link to the book's page, and each withdrawal alert, listed and ordered the same way, with the title, the date and time the store recorded that the book is no longer offered and a statement that the book is no longer offered, with no link — neither with any number of copies; where the alerts service does not answer, it shows in place of the list that the alerts list is unavailable right now and the client may try again later.
- When the current client opens the pending-subscriptions list, then the web store shows every subscription the alerts service returns as pending, each with the book's title linked to the book's page and the date it was made, so that one ended by its alert, a withdrawal, a cancel or a bulk reset is no longer listed; where the alerts service does not answer, it shows in place of the list that the pending-subscriptions list is unavailable right now and the client may try again later.
- When the current client cancels a pending subscription, from the pending-subscriptions list or the book's page, or dismisses an alert, then the web store shows it gone — also where it had already ended or been dismissed, with no error — and a dismissed alert stays out of the list and the count; where the alerts service does not answer, it shows that the cancel or dismissal could not be done and keeps the subscription shown as pending or the alert listed; and where the alerts service rejects it as not the current client's (`404`), it shows the current client's list with a notice that the current client has changed.
- (guard) Browsing books, using the cart and placing orders behave as they did before this feature — with no current client, and with a current client while the alerts service does not answer — apart from the current-client display, the subscribe action and subscribed state, the unread-alert count, the ways into the two lists and the notices that subscriptions or alerts are unavailable; and Delete All on the books and Clients lists still calls the same endpoint and reloads the list, which then shows the synthetic journey's books or clients that the reset keeps ([AD#11]), while the subscriptions and alerts a books reset cleared are gone from the current client's count and lists at their next read ([AD#7]).

## Independent Test
[[BOOK-1-05]] is verifiable standalone by running the web store against a stub of alerts' [AD#8] API that serves canned subscriptions, alerts and counts, each answer switchable to `404`, `409`, `503` or no answer, and against stubs of books' and clients' [AD#11] `delete-all` that keep reserved fixtures, with storage's `findByISBN` and the Clients pages used as they run — a tester chooses a current client and walks every criterion above in a browser — and delivers the whole client-facing journey without any not-yet-built Epic.

## Dependencies
- [[BOOK-1-01]] — produces the client-facing API under `/api/v1/alerts` ([AD#8]) and the `/api/alerts/` ingress path that the production alerts base URL uses; until it lands, the Independent Test runs against a stub.
- [[BOOK-1-03]] — produces books' `delete-all` that keeps the journey's books and resets alerts ([AD#11], [AD#7]), which the books list's Delete All calls; until it lands, the Independent Test runs against a stub.
- [[BOOK-1-04]] — produces clients' `delete-all` that keeps the journey's clients ([AD#11]), which the Clients list's Delete All calls; until it lands, the Independent Test runs against a stub.
- Storage's existing `GET /api/v1/storage/findByISBN`, through `StorageService.getStorageByISBN` — exists, used as it runs; a book with no storage row has no available copies ([AD#3]), and how the page reads that answer is left to design, since the code scan did not check it.
- The existing Clients list and client details pages — exist, extended with the make-current-client action.

## Contract
- Consumes: [AD#8] — alerts' client-facing API (subscribe, pending subscriptions, cancel, alerts, unread count, mark read, dismiss); the Independent Test runs against a stub of it.
- Consumes: [AD#11] — books' `delete-all` keeping the journey's books; the Independent Test runs against a stub of it.
- Consumes: [AD#11] — clients' `delete-all` keeping the journey's clients; the Independent Test runs against a stub of it.

## Covers
- [US#1], [US#2], [US#3], [US#4], [US#5]
- [AC#1], [AC#2], [AC#5], [AC#7], [AC#8], [AC#10], [AC#11], [AC#12], [AC#13], [AC#14], [AC#15]
- [SM#1], [SM#2], [SMC#1]
- [U01], [U01/AC01], [U01/AC02], [U01/AC03], [U01/AC06], [U01/AC08], [U01/AC09], [U01/AC10]
- [U02], [U02/AC05], [U02/AC06], [U02/AC07], [U02/AC08], [U02/AC09], [U02/AC10], [U02/AC11]
- [U03], [U03/AC01], [U03/AC02], [U03/AC03], [U03/AC04], [U03/AC05], [U03/AC06], [U03/AC07], [U03/AC08]
- [U04], [U04/AC01], [U04/AC02], [U04/AC04], [U04/AC05], [U04/AC06], [U04/AC07], [U04/AC08]
- [U05], [U05/AC01], [U05/AC02], [U05/AC03], [U05/AC04], [U05/AC05], [U05/AC06]
- [U06], [U06/AC04], [U06/AC05], [U06/AC06], [U06/AC07], [U06/AC08]

## Suggested stories
- Current-client state kept in the browser, the make-current-client action on the Clients list and details page, and the current client's email with a clear action on every page.
- Alerts Angular service, its base URL in the three environment files, and the routes and module declarations for the new pages.
- Unread-alert count in the global navbar with 20-second polling, the ways into both lists, and the no-current-client prompt.
- Book page: stock read, subscribe action, subscribed state with cancel, and the not-made and unavailable notices.
- Alerts list with mark-read on open, dismissal and its notices.
- Pending-subscriptions list with cancellation and its notices.
- Delete All behaviour on the books and Clients lists, with the reworded confirmations.
- Regression pass of browse, cart and order with no current client and with alerts down.

## References
- Parent PRD: [[BOOK-1]]
- [Source: ard.md#Architecture decisions] — [AD#2], [AD#3], [AD#7], [AD#8], [AD#9], [AD#11]; [Source: ard.md#Versioning and compatibility] — the web Epic states the hand-reset behaviour in its Delete All behaviour
- [Source: specification.md#U01], [Source: specification.md#U02], [Source: specification.md#U03], [Source: specification.md#U04], [Source: specification.md#U05], [Source: specification.md#U06] — the test cases these criteria condense
- [Source: _glossary.md#Current client] — replaces the PRD's "signed-in client"
- [Source: bookstore/web/src/app/app.component.html#navbar] — one global navbar around the router outlet, shared by every page; static links, no client selector
- [Source: bookstore/web/src/app/app-routing.module.ts#routes] — flat route table, `book-details/:isbn` at line 44
- [Source: bookstore/web/src/app/books/book-details/book-details.component.ts#ngOnInit] — loads the book by ISBN and reads no stock
- [Source: bookstore/web/src/app/storage/storage.service.ts#getStorageByISBN] — the per-service Angular service pattern and the stock read
- [Source: bookstore/web/src/environments/environment.ts#environment] — per-service base URLs, no alerts entry
- [Source: bookstore/web/src/app/app.module.ts#declarations] — components declared in a classic `NgModule`
- [Source: bookstore/web/src/app/books/book-list/book-list.component.ts#deleteAllBooks] — confirms, calls `delete-all` and reloads the list
- [Source: bookstore/web/src/app/clients/client-list/client-list.component.ts#deleteAllClients] — confirms, calls `delete-all`

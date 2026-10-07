---
kind: prd
key: BOOK-1
title: Back-in-stock alerts
summary: Clients subscribe to an out-of-stock book and are alerted inside the store when it comes back into stock.
status: draft
sources:
  - provenance: prompt
derived_from: specifications/PRD-BOOK-1-back-in-stock-alerts/idea.md
---

# Back-in-stock alerts

## Problem

A client who finds that a book they want is out of stock has no way to learn when it comes back other than returning and checking again. Most do not keep checking, so the interest they showed is lost, and a restocked copy that a waiting client would have bought sits unsold or goes to whoever happens to look first. BookStore is also run continuously under synthetic client traffic for demonstrations, and none of the journeys that traffic drives today is set off by a book coming back into stock, so the demo has nothing to show for this situation either.

## Goal

A client can subscribe to a book that is out of stock and is alerted inside the store within about a minute of the book coming back, so they no longer have to keep checking. The store recovers sales lost today when a book is unavailable at the moment a client wants it, and the journey runs continuously under synthetic traffic so a demo started at any moment shows it working.

## Target audience

- **Client (primary)** — a registered BookStore client, signed in, who looks at a book while it has no available copies and wants it once it returns.
- **Demo operator (secondary)** — the person running a BookStore demonstration, who needs the back-in-stock journey to be happening continuously under synthetic traffic so it can be shown at any moment.

## User Stories

Terms used throughout: a *subscription* is a client's standing request to be told when one book is back in stock; an *alert* is a notice the client receives in the store — a *back-in-stock alert* when a subscribed book comes back, a *withdrawal alert* when it is removed from the catalogue.

### [US#1]: Subscribe to an out-of-stock book

As a client, I want to ask to be told when an out-of-stock book is available again, so that I don't have to keep checking back.

### [US#2]: Be alerted when the book is back

As a client, I want to be alerted in the store when a book I subscribed to comes back into stock, so that I can buy it before it sells out again.

### [US#3]: Manage my pending subscriptions

As a client, I want to see and cancel the subscriptions I am still waiting on, so that I only hear about books I still want.

### [US#4]: Manage my delivered alerts

As a client, I want my alerts kept for 30 days and to dismiss the ones I have dealt with, so that I can come back to an alert I have not acted on yet and see only the ones I still need.

### [US#5]: Be told when a book I wait for is withdrawn

As a client, I want to be told when a book I am waiting for will no longer be offered, so that I stop waiting for something that cannot come back.

### [US#6]: See the journey running in the demo

As a demo operator, I want the synthetic client traffic to subscribe, cancel and receive alerts continuously, so that a demo started at any moment shows the back-in-stock journey working.

## Acceptance Criteria

**[US#1] Subscribe to an out-of-stock book**

- [AC#1] On the page of a book with no available copies, a signed-in client is offered a way to subscribe to that book; no subscription is offered on a book with one or more available copies, or to a visitor who is not signed in as a client.
- [AC#2] After subscribing, the book's page shows the client that they are subscribed.
- [AC#3] Subscribing to a book the client already holds a pending subscription for leaves the client with exactly one pending subscription to that book.
- [AC#4] A pending subscription stays pending for as long as its book is out of stock and still offered in the catalogue; it never lapses on a timer.
- [AC#5] (guard) Apart from the subscription offer on the page of a book with no available copies, the unread-alert count, and the ways into the alerts list and the pending-subscriptions list, browsing books, using the cart and placing orders behave as they did before this feature.

**[US#2] Be alerted when the book is back**

- [AC#6] When a book's available copies go from zero to one or more, and only then, every client holding a pending subscription to that book receives a back-in-stock alert in their alerts list within one minute, whatever the number of copies that came back; no copy is held or reserved for any of them. This one-minute limit is the pass/fail limit in acceptance testing; [SM#1] is the production target.
- [AC#7] A back-in-stock alert names the book, states when it came back into stock, and links to the book's page; the alert itself makes no statement about how many copies are available now.
- [AC#8] While a client has at least one unread alert, a count of unread alerts is visible on every page of the store; opening the alerts list clears the count.
- [AC#9] Delivering a back-in-stock alert ends the subscription it fulfils: a later restock of the same book produces no further alert from it.

**[US#3] Manage my pending subscriptions**

- [AC#10] A client can see a list of every subscription they hold that is still pending, each naming its book.
- [AC#11] A client can cancel any pending subscription from that list or from the book's page, and a cancelled subscription produces no alert on any later restock.

**[US#4] Manage my delivered alerts**

- [AC#12] A delivered alert stays in the client's alerts list for 30 days from when it arrived, and leaves the list after that.
- [AC#13] A client can dismiss any alert before its 30 days are up; a dismissed alert leaves the list and is not counted as unread.

**[US#5] Be told when a book I wait for is withdrawn**

- [AC#14] When a book is removed from the catalogue, every pending subscription to it ends, and within one minute each client who held one receives a withdrawal alert that names the book and says it is no longer offered.
- [AC#15] A withdrawal alert is listed, counted as unread, retained and dismissed like any other alert, and offers no link to the removed book.

**[US#6] See the journey running in the demo**

- [AC#16] While synthetic client traffic runs, at least one subscription, one cancellation and one delivered back-in-stock alert occur in every 5-minute window.
- [AC#17] Synthetic clients subscribe, cancel and are alerted under the same rules as any other client — they subscribe only to out-of-stock books and receive a back-in-stock alert only when a subscribed book comes back into stock.

## Scope

**In scope**

- Subscribing to a book that has no available copies, by a signed-in client.
- Alerting every subscribed client, inside the store, when the book's available copies go from zero to one or more.
- An alerts list and an unread-alert count shown throughout the store.
- Back-in-stock alerts that name the book, say when it came back, and link to its page.
- One-shot subscriptions that end when their back-in-stock alert is delivered.
- Listing and cancelling pending subscriptions, from the list or from the book's page.
- Keeping delivered alerts for 30 days, and dismissing them earlier.
- Ending pending subscriptions to a book removed from the catalogue, with a withdrawal alert to each waiting client.
- Synthetic client traffic that subscribes, cancels and receives alerts continuously.
- Synthetic traffic that takes books out of stock and restocks them often enough to meet [AC#16].

**Out of scope**

- Email, SMS or any other notification sent outside the store.
- Holding or reserving a restocked copy for an alerted client.
- Alerting only as many clients as there are copies.
- Pending subscriptions that lapse on their own after a period.
- Subscriptions that stay active after their alert and fire again on later restocks.
- Alerts on increases in copies of a book that was already in stock.
- Subscriptions by visitors who are not signed in as a client.
- Showing the current number of available copies inside an alert.

## Success Metrics

- [SM#1] At least 95% of alerts — back-in-stock and withdrawal alike — are visible in the client's alerts list within one minute of the restock or removal that caused them, measured over all clients, synthetic included. This is the production target; [AC#6] and [AC#14] are the acceptance-test limits.
- [SM#2] At least 5% of clients who receive a back-in-stock alert order that book within 7 days of the alert, measured over clients that are not synthetic traffic only. Adding the book to the cart within 7 days is reported beside it as an early indicator, with no target.
- [SMC#1] (counter-metric) For clients who hold no subscription, the 95th-percentile response time of the browse, cart and order journeys, measured over one week, stays within 10% of the same measure for the week before release — [SM#1] is not to be bought by slowing the rest of the store.

## Non-functional requirements

- **Timeliness at scale.** The one-minute limits ([AC#6], [AC#14]) hold with up to 1,000 clients waiting on a single book.
- **No cost to the rest of the store.** The feature must not make the store's existing journeys slower for clients who never use it ([AC#5], [SMC#1]).

## Assumptions & open questions

- **Assumption:** No external demand evidence exists; the idea is internal and arrived as an inline prompt.
- **Assumption:** The store can already tell which signed-in client is viewing a page, which [AC#1] and [AC#8] rely on.
- No open questions.

## Why now / differentiation

BookStore is exercised continuously by synthetic client traffic to demonstrate a busy, many-service store. Today none of the journeys that traffic drives is set off by stock changing, so a whole class of behaviour — one event reaching many waiting clients — is absent from the demo. Back-in-stock alerts add that journey while also addressing a real client frustration.

## Documentation impact

None. BookStore has no end-user documentation today (the documentation root holds no pages), so there is nothing to update.

## References / linked issues

- Idea brief: [idea.md](idea.md)

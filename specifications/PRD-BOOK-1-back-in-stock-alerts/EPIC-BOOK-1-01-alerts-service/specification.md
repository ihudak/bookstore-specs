---
kind: specification
key: BOOK-1-01
---

# Specification

- **Feature name**: Back-in-stock alerts — subscriptions and alerts service
- **Version**: 1
- **Created**: 2026-10-08
- **Author**: ivan.gudak
- **Published**: no
- **Open questions**: 4

## Problem statement

A client who finds a book with no available copies has nowhere to leave their interest in it: the store keeps no record of who is waiting for which book, so nothing can tell them when it comes back or when it is removed from the catalogue, and they must keep returning to check. Most stop checking, and the sale a waiting client would have made is lost. Every other part of the back-in-stock feature — the store noticing a restock or a removal, the client's pages and the synthetic journey — has nothing to act on until that record exists, together with an alert for every waiting client, kept long enough for them to see it. The record has to keep up when many clients wait on one book, and stay right when word of a stock change or a removal does not arrive, because a missed or repeated alert is exactly what a client would notice. Solving it recovers sales lost while a book is unavailable, and lets the rest of the feature be built and tested against it.

## Scope

**In scope**

- Recording a client's subscription to a book with no available copies, published or not — the client named by email, matched ignoring letter case, the book by ISBN — once the clients service knows the email, the catalogue knows the book and it has no available copies, keeping one pending subscription per client and book ([AD#1], [AD#2], [AD#3], [AD#8]).
- Answering a subscription that was not made with its reason — the book has available copies, the book or the client is unknown, or the store could not check — and refusing any operation whose email or ISBN is missing or blank ([AD#8]).
- Recording a back-in-stock alert for every pending subscription to a book when told the book is available, and ending those subscriptions, within 10 seconds for up to 1,000 waiting clients ([AD#4]).
- Recording a withdrawal alert, carrying the book title stored when the subscription was made, for every pending subscription to a book when told the book was removed, and ending those subscriptions, within 10 seconds for up to 1,000 waiting clients ([AD#6]).
- Checking every book with a pending subscription against the catalogue and then its stock at least every 30 seconds, whatever the number of books waited on, and acting as on a notice where the book is gone or has available copies again, so the alerts of a lost notice are recorded within 40 seconds ([AD#5]).
- Listing a client's pending subscriptions, and cancelling one ([AD#8]).
- Listing a client's alerts newest first, counting the unread ones, marking them all read and dismissing one, each alert kept for 30 days from when it was recorded ([AD#8]).
- Removing every subscription and alert other than the reserved ones when told of a bulk catalogue reset, recording no alert ([AD#7], [AD#11]).
- Running the alerts service in the store's deployment, under the monitoring agent the store selects for it, reachable from the browser through the store's address and from the other services — on an existing deployment whose database predates this feature too, without resetting any other service's data — and removing it with the rest of the store.
- Answering the version and configuration operations every BookStore service offers.

**Out of scope**

- Detecting a restock in storage and telling the alerts service — [[BOOK-1-02]].
- Telling the alerts service of a single removal or a bulk catalogue reset, and keeping reserved books through a reset — [[BOOK-1-03]].
- The web store's current client, subscribe action, alerts list, pending-subscriptions list and unread count — [[BOOK-1-05]].
- The synthetic back-in-stock journey and its choice of reserved books and clients — [[BOOK-1-06]].
- Withdrawal alerts on a bulk catalogue reset the alerts service is told of, and treating unpublishing a book as removing it.
- Checking that a caller is the client whose email it names, and removing a deleted client's subscriptions and alerts.
- Subscriptions that lapse on a timer, or that stay active after their alert and fire again on a later restock.
- Marking an alert unread again, or restoring a dismissed alert.
- Holding or reserving a restocked copy for an alerted client, alerting only as many clients as there are copies, or any number of available copies in a subscription or an alert.
- Capping or paging the alerts list.
- Email, SMS or any other notification sent outside the store.

### Open questions

- [ ] What does each operation of the alerts service answer while it cannot read or write its own records — for example 503 Service Unavailable, changing nothing — and does an available or withdrawn notice answered that way count as lost, for the alerts service's own check to recover? It puts the notice's own answer in [U02] AC08 and [U03] AC07 in doubt.
- [ ] ARD deviation: [AD#8] — this specification takes an alert's occurredAt to be when the alerts service learned of the return or removal, up to 30 seconds after it on the alerts service's own check ([U02] AC09, [U03] AC08), where [AD#8] says when the book came back or was removed; and it answers cancel and dismiss with 404 once a subscription ended, or an alert was recorded, more than 30 days ago ([U04] AC07, [U05] AC04), where [AD#8] answers 204 for an ended or dismissed one with no window — because no notice carries a time, and ended records kept forever would grow without bound — flag: architect. Settled by refining [AD#8] with `/product-workflows:create-ard BOOK-1`, or by changing those four criteria.

## User stories

### [U01]: Subscribe to a book with no available copies

As a client, I want my wish for a book with no available copies recorded once, so that I am told when it comes back instead of checking again.

#### [AC01]: Record the subscription

When subscribe is called with an email the clients service knows and the ISBN of a book the books service knows and the storage service reports with no row or a quantity of 0, the alerts service shall answer 201 Created with a newly recorded pending subscription of that email to that ISBN, carrying the book's title ([AD#2], [AD#3], [AD#8]).

##### Test cases

**[TC01]: A book with a quantity of 0 — Happy path:**
- *Preconditions:* Client C exists in the clients service; book B, titled "The Long Wait", is in the books service and the storage service holds B with a quantity of 0; C holds no subscription.
- *Steps:* 1. Call subscribe with C's email and B's ISBN. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 201 Created with a newly recorded pending subscription of C's email to B's ISBN carrying the title "The Long Wait", and C's pending subscriptions list it.

**[TC02]: A book the storage service holds no row for — Negative / boundary:**
- *Preconditions:* Client C exists in the clients service; book B is in the books service and the storage service holds no row for B; C holds no subscription.
- *Steps:* 1. Call subscribe with C's email and B's ISBN. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 201 Created with a newly recorded pending subscription of C's email to B's ISBN, and C's pending subscriptions list it.

**[TC03]: A second client subscribes to the same book — Negative / boundary:**
- *Preconditions:* Clients C1 and C2 exist; book B has a quantity of 0; C1 holds a pending subscription to B and C2 holds none.
- *Steps:* 1. Call subscribe with C2's email and B's ISBN. 2. Read C2's pending subscriptions.
- *Expected result:* Subscribe answers 201 Created with a newly recorded pending subscription of C2's email to B's ISBN, and C2's pending subscriptions list it.

**[TC04]: Subscribe again after the subscription ended — State / lifecycle:**
- *Preconditions:* Client C's subscription S to book B was cancelled; B has a quantity of 0.
- *Steps:* 1. Call subscribe with C's email and B's ISBN. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 201 Created with a newly recorded pending subscription of C's email to B's ISBN, whose id is not S's, and C's pending subscriptions list it.

#### [AC02]: Keep one pending subscription per client and book

When subscribe is called with an email that, ignoring letter case, already holds a pending subscription to that ISBN, the alerts service shall answer 200 OK with that subscription, recording nothing new and calling none of the clients, books or storage services ([AD#8]).

##### Test cases

**[TC01]: Subscribe twice — Happy path:**
- *Preconditions:* Client C holds pending subscription S to book B, which has a quantity of 0.
- *Steps:* 1. Call subscribe with C's email and B's ISBN. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 200 OK with S, recording nothing new; C's pending subscriptions list exactly one subscription to B.

**[TC02]: The same email in other letter case — Negative / boundary:**
- *Preconditions:* Client C, with the email c.one@example.com, holds pending subscription S to book B.
- *Steps:* 1. Call subscribe with C.One@Example.com and B's ISBN. 2. Read the pending subscriptions of c.one@example.com.
- *Expected result:* Subscribe answers 200 OK with S, recording nothing new; exactly one pending subscription to B is listed.

**[TC03]: The other services are down — Negative / boundary:**
- *Preconditions:* Client C holds pending subscription S to book B; the clients, books and storage services are stopped.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 200 OK with S, recording nothing new and calling none of the clients, books or storage services.

**[TC04]: Two subscribe calls at the same moment — State / lifecycle:**
- *Preconditions:* Client C exists and holds no subscription; book B has a quantity of 0.
- *Steps:* 1. Send two subscribe calls with C's email and B's ISBN at the same moment. 2. Read C's pending subscriptions.
- *Expected result:* One call answers 201 Created and the other 200 OK with that same subscription; C's pending subscriptions list exactly one subscription to B.

#### [AC03]: Never lapse

While a subscribed book has no available copies and is still in the catalogue, the alerts service shall keep its subscription pending, whatever the time since the subscription was made ([AD#5]).

##### Test cases

**[TC01]: A subscription made 31 days ago — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B made 31 days ago; B is in the books service with a quantity of 0.
- *Steps:* 1. Wait 40 seconds. 2. Read C's pending subscriptions.
- *Expected result:* The alerts service keeps C's subscription to B pending, and it is listed.

**[TC02]: A year-old subscription to a book with no storage row — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B made 365 days ago; B is in the books service and the storage service holds no row for it.
- *Steps:* 1. Wait 40 seconds. 2. Read C's pending subscriptions.
- *Expected result:* The alerts service keeps C's subscription to B pending.

#### [AC04]: Accept an unpublished book

When subscribe is called for a book that is unpublished and has no available copies, the alerts service shall answer 201 Created with a newly recorded pending subscription, exactly as for a published book ([AD#6], [AD#8]).

##### Test cases

**[TC01]: An unpublished book with a quantity of 0 — Happy path:**
- *Preconditions:* Client C exists; book B is in the books service, unpublished, with a quantity of 0; C holds no subscription.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 201 Created with a newly recorded pending subscription of C to B, exactly as for a published book.

**[TC02]: An unpublished book with no storage row — Negative / boundary:**
- *Preconditions:* Client C exists; book B is in the books service, unpublished, and the storage service holds no row for it.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 201 Created with a newly recorded pending subscription of C to B, exactly as for a published book.

#### [AC05]: Match an email ignoring letter case

The alerts service shall treat two emails that differ only in letter case as the same client in subscribe, in every read, in mark-read, in cancel and in dismiss ([AD#2]).

##### Test cases

**[TC01]: Read pending subscriptions in other letter case — Happy path:**
- *Preconditions:* A pending subscription to book B was made with the email First.Last@Example.com.
- *Steps:* 1. Read the pending subscriptions of first.last@example.com.
- *Expected result:* The alerts service treats both emails as the same client and lists the subscription to B.

**[TC02]: Cancel and dismiss in other letter case — Negative / boundary:**
- *Preconditions:* The email first.last@example.com holds pending subscription S and alert X.
- *Steps:* 1. Cancel S with FIRST.LAST@EXAMPLE.COM. 2. Dismiss X with First.Last@example.com. 3. Read the pending subscriptions and alerts of first.last@example.com.
- *Expected result:* The alerts service treats the emails as the same client: both calls answer 204 No Content, S is no longer pending and X is no longer listed.

**[TC03]: An email that differs by more than letter case — Security / privacy:**
- *Preconditions:* The email first.last@example.com holds a pending subscription to book B and an alert; firstlast@example.com holds none.
- *Steps:* 1. Read the pending subscriptions, alerts and unread count of firstlast@example.com.
- *Expected result:* The alerts service treats firstlast@example.com as a different client: an empty list of subscriptions, an empty list of alerts and a count of 0.

**[TC04]: Mark-read in other letter case — Negative / boundary:**
- *Preconditions:* Two unread alerts were recorded for a subscription made with the email First.Last@Example.com.
- *Steps:* 1. Call mark-read for first.last@example.com. 2. Read the unread count of First.Last@Example.com.
- *Expected result:* The alerts service treats both emails as the same client: mark-read answers 204 No Content and the count is 0.

#### [AC06]: Keep a plus sign in an email

When subscribe is called with the email of a known client that contains a plus sign, the alerts service shall answer 201 Created with a subscription whose email is the one given, plus sign included ([AD#2], [AD#8]).

##### Test cases

**[TC01]: An email with a plus sign — Happy path:**
- *Preconditions:* The clients service holds a client with the email first.last+books@example.com and no client with the email first.last@example.com; book B has a quantity of 0.
- *Steps:* 1. Call subscribe with first.last+books@example.com, encoded for a query string, and B's ISBN.
- *Expected result:* Subscribe answers 201 Created with a subscription whose email is first.last+books@example.com, plus sign included.

**[TC02]: Read back under the email with a plus sign — Negative / boundary:**
- *Preconditions:* The clients service holds a client with the email First.Last+Books@Example.com; book B has a quantity of 0; that email holds no subscription.
- *Steps:* 1. Call subscribe with First.Last+Books@Example.com, encoded for a query string, and B's ISBN. 2. Read the pending subscriptions of first.last+books@example.com, encoded for a query string.
- *Expected result:* Subscribe answers 201 Created with a subscription whose email is First.Last+Books@Example.com, plus sign included, and the read lists it.

#### [AC07]: Refuse a book with available copies

If the storage service reports one or more available copies of the ISBN, then the alerts service shall answer subscribe with 409 Conflict, recording no subscription ([AD#3], [AD#8]).

##### Test cases

**[TC01]: A book with 2 copies — Happy path:**
- *Preconditions:* Client C exists and holds no subscription; book B has a quantity of 2.
- *Steps:* 1. Call subscribe with C's email and B's ISBN. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 409 Conflict, recording no subscription; C's pending subscriptions are an empty list.

**[TC02]: A book with exactly 1 copy — Negative / boundary:**
- *Preconditions:* Client C exists and holds no subscription; book B has a quantity of 1.
- *Steps:* 1. Call subscribe with C's email and B's ISBN. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 409 Conflict, recording no subscription.

#### [AC08]: Refuse an unknown book or client

If the clients service answers that it knows no client with the email, or the books service that it knows no book with the ISBN, then the alerts service shall answer subscribe with 404 Not Found, recording no subscription ([AD#2], [AD#8]).

##### Test cases

**[TC01]: An ISBN the catalogue does not know — Happy path:**
- *Preconditions:* Client C exists and holds no subscription; the books service holds no book with the ISBN 9789999999991.
- *Steps:* 1. Call subscribe with C's email and 9789999999991. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 404 Not Found, recording no subscription.

**[TC02]: An email the clients service does not know — Negative / boundary:**
- *Preconditions:* The clients service holds no client with the email nobody@example.com; book B has a quantity of 0.
- *Steps:* 1. Call subscribe with nobody@example.com and B's ISBN. 2. Read the pending subscriptions of nobody@example.com.
- *Expected result:* Subscribe answers 404 Not Found, recording no subscription.

**[TC03]: A book removed from the catalogue — Negative / boundary:**
- *Preconditions:* Client C exists; book B had a quantity of 0 and has since been deleted from the books service.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 404 Not Found, recording no subscription.

#### [AC09]: Refuse when the store cannot check

If the clients, books or storage service does not answer — no reply within 3 seconds, a refused connection, or any reply other than its found or not-found answer — then the alerts service shall answer subscribe with 503 Service Unavailable within 10 seconds of the request, recording no subscription ([AD#8]).

##### Test cases

**[TC01]: The storage service is stopped — Happy path:**
- *Preconditions:* Client C exists and holds no subscription; book B is in the books service; the storage service is stopped.
- *Steps:* 1. Call subscribe with C's email and B's ISBN and time the answer. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 503 Service Unavailable within 10 seconds of the request, recording no subscription.

**[TC02]: The books service holds every request for 5 seconds — Negative / boundary:**
- *Preconditions:* Client C exists; book B has a quantity of 0; the books service holds every request for 5 seconds before replying.
- *Steps:* 1. Call subscribe with C's email and B's ISBN and time the answer. 2. Read C's pending subscriptions.
- *Expected result:* Subscribe answers 503 Service Unavailable within 10 seconds of the request, recording no subscription.

**[TC03]: The clients service replies with an error — Negative / boundary:**
- *Preconditions:* Client C exists; book B has a quantity of 0; the clients service replies 500 to every request.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 503 Service Unavailable, recording no subscription.

**[TC04]: The storage service replies busy — Negative / boundary:**
- *Preconditions:* Client C exists; book B is in the books service; the storage service replies 429 to every request.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 503 Service Unavailable, recording no subscription.

**[TC05]: All three services hold every request for 5 seconds — Negative / boundary:**
- *Preconditions:* Client C exists; book B has a quantity of 0; the clients, books and storage services each hold every request for 5 seconds before replying.
- *Steps:* 1. Call subscribe with C's email and B's ISBN and time the answer.
- *Expected result:* Subscribe answers 503 Service Unavailable within 10 seconds of the request, recording no subscription.

#### [AC10]: Decide one outcome in a fixed order

When more than one subscribe outcome applies, the alerts service shall answer with the first that applies in this order: a missing or blank email or ISBN (400), an existing pending subscription (200), the client check (404 or 503), the book check (404 or 503), the stock check (409 or 503) ([AD#8]).

##### Test cases

**[TC01]: An existing subscription while the stock cannot be checked — Happy path:**
- *Preconditions:* Client C holds pending subscription S to book B, which is in the books service; the storage service is stopped.
- *Steps:* 1. Call subscribe with C's email and B's ISBN.
- *Expected result:* Subscribe answers 200 OK with S, the existing pending subscription coming before the stock check's 503.

**[TC02]: An unknown client while the storage service is stopped — Negative / boundary:**
- *Preconditions:* The clients service holds no client with the email nobody@example.com; book B is in the books service; the storage service is stopped.
- *Steps:* 1. Call subscribe with nobody@example.com and B's ISBN.
- *Expected result:* Subscribe answers 404 Not Found, the client check coming before the stock check.

**[TC03]: An unknown book while the storage service is stopped — Negative / boundary:**
- *Preconditions:* Client C exists; the books service holds no book with the ISBN 9789999999991; the storage service is stopped.
- *Steps:* 1. Call subscribe with C's email and 9789999999991.
- *Expected result:* Subscribe answers 404 Not Found, the book check coming before the stock check.

**[TC04]: The clients service stopped and an unknown book — Negative / boundary:**
- *Preconditions:* The clients service is stopped; the books service holds no book with the ISBN 9789999999991.
- *Steps:* 1. Call subscribe with any email and 9789999999991.
- *Expected result:* Subscribe answers 503 Service Unavailable, the client check coming before the book check.

**[TC05]: A blank ISBN while the clients service is stopped — Negative / boundary:**
- *Preconditions:* Client C exists; the clients service is stopped.
- *Steps:* 1. Call subscribe with C's email and a blank ISBN.
- *Expected result:* Subscribe answers 400 Bad Request, the missing or blank email or ISBN coming before the client check.

#### [AC11]: Refuse a missing or blank email or ISBN

If any operation of the alerts service is called with its email or ISBN missing or blank, then the alerts service shall answer 400 Bad Request, changing nothing ([AD#8]).

##### Test cases

**[TC01]: Subscribe with a blank email — Happy path:**
- *Preconditions:* Client C exists and holds no subscription; book B has a quantity of 0.
- *Steps:* 1. Call subscribe with an empty email and B's ISBN. 2. Call subscribe with C's email and an empty ISBN. 3. Read C's pending subscriptions.
- *Expected result:* Both calls answer 400 Bad Request, changing nothing: C's pending subscriptions are an empty list.

**[TC02]: Read alerts with no email — Negative / boundary:**
- *Preconditions:* Client C has an alert.
- *Steps:* 1. Call the read of alerts with no email parameter.
- *Expected result:* The alerts service answers 400 Bad Request.

**[TC03]: An available notice with a blank ISBN — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that an ISBN is available, with the ISBN blank. 2. Read C's pending subscriptions and alerts.
- *Expected result:* The alerts service answers 400 Bad Request, changing nothing: C's subscription to B is still pending and C has no new alert.

**[TC04]: Cancel with a blank email — Negative / boundary:**
- *Preconditions:* Client C holds pending subscription S.
- *Steps:* 1. Cancel S with a blank email. 2. Read C's pending subscriptions.
- *Expected result:* The alerts service answers 400 Bad Request, changing nothing: S is still pending.

---

### [U02]: Be alerted when the book is back

As a client, I want an alert recorded for me within a minute of a book I wait for coming back into stock, ending my wait, so that I can buy it before it sells out again.

#### [AC01]: Alert every subscriber on a notice

When the alerts service is told that an ISBN is available, the alerts service shall end every pending subscription to it with a back-in-stock alert, answering 204 No Content whether or not any existed ([AD#4]).

##### Test cases

**[TC01]: Three subscribers — Happy path:**
- *Preconditions:* Clients C1, C2 and C3 each hold a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Read each client's alerts and pending subscriptions.
- *Expected result:* The notice answers 204 No Content; each of C1, C2 and C3 has a back-in-stock alert for B and no pending subscription to B.

**[TC02]: A book nobody waits for — Negative / boundary:**
- *Preconditions:* No pending subscription to book B exists; client C holds a pending subscription to book B2.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Read C's alerts and pending subscriptions.
- *Expected result:* The notice answers 204 No Content; no alert is recorded, and C's subscription to B2 is still pending.

**[TC03]: Only the named book's subscriptions end — Negative / boundary:**
- *Preconditions:* Client C holds pending subscriptions to books B1 and B2.
- *Steps:* 1. Tell the alerts service that B1's ISBN is available. 2. Read C's alerts and pending subscriptions.
- *Expected result:* The notice answers 204 No Content; C's subscription to B1 has ended with a back-in-stock alert, and C's subscription to B2 is still pending.

#### [AC02]: Alert 1,000 subscribers within 10 seconds

While up to 1,000 pending subscriptions to one ISBN exist, the alerts service shall record every one of their back-in-stock alerts within 10 seconds of being told the ISBN is available ([AD#4]).

##### Test cases

**[TC01]: 1,000 subscribers — Happy path:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. 10 seconds after the notice, count the back-in-stock alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 back-in-stock alerts within 10 seconds of the notice.

**[TC02]: 1,000 subscribers beside another book's 1,000 — Negative / boundary:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B, and 1,000 others each hold one to book B2.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. 10 seconds after the notice, count the back-in-stock alerts for B and the pending subscriptions to B2.
- *Expected result:* The alerts service has recorded every one of the 1,000 back-in-stock alerts for B within 10 seconds, and all 1,000 subscriptions to B2 are still pending.

#### [AC03]: Recover a lost notice

While the books service answers that it knows a book that has pending subscriptions, and the alerts service is not told the book is available, the alerts service shall record their back-in-stock alerts within 40 seconds of the first moment at which the storage service holds one or more available copies of it and answers the alerts service's calls ([AD#5]).

##### Test cases

**[TC01]: A restock with no notice — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Set B's quantity to 1 in the storage service, sending no notice. 2. 40 seconds after the change, read C's alerts.
- *Expected result:* The alerts service has recorded C's back-in-stock alert for B within 40 seconds of the storage service first reporting one or more available copies.

**[TC02]: First stock for a book with no storage row — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, for which the storage service holds no row.
- *Steps:* 1. Create a storage row for B with a quantity of 2, sending no notice. 2. 40 seconds after the change, read C's alerts.
- *Expected result:* The alerts service has recorded C's back-in-stock alert for B within 40 seconds.

**[TC03]: 1,000 subscribers with no notice — Negative / boundary:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Set B's quantity to 1 in the storage service, sending no notice. 2. 40 seconds after the change, count the back-in-stock alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 back-in-stock alerts within 40 seconds.

**[TC04]: An unpublished book restocked with no notice — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which is unpublished in the books service and has a quantity of 0.
- *Steps:* 1. Set B's quantity to 1 in the storage service, sending no notice. 2. 40 seconds after the change, read C's alerts.
- *Expected result:* The alerts service has recorded C's back-in-stock alert for B within 40 seconds, the books service still knowing the book.

#### [AC04]: Recover 100 books at once

While 100 distinct books have pending subscriptions, the alerts service shall record, with no notice, the back-in-stock alerts of every one of them that returns to stock within 40 seconds of the first moment at which the storage service holds one or more available copies of it and answers the alerts service's calls ([AD#5]).

##### Test cases

**[TC01]: 100 books restocked together — Happy path:**
- *Preconditions:* 100 distinct books each have one pending subscription and a quantity of 0.
- *Steps:* 1. Set every one of the 100 quantities to 1 in the storage service at once, sending no notice. 2. 40 seconds after the change, count the back-in-stock alerts.
- *Expected result:* The alerts service has recorded all 100 back-in-stock alerts within 40 seconds, with no notice.

**[TC02]: One of 100 books restocked — Negative / boundary:**
- *Preconditions:* 100 distinct books each have one pending subscription and a quantity of 0.
- *Steps:* 1. Set the quantity of the book subscribed to last to 1, sending no notice. 2. 40 seconds after the change, read the alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded that book's back-in-stock alert within 40 seconds, with no notice, and the other 99 subscriptions are still pending.

#### [AC05]: Alert nothing while there are no copies

If, on the alerts service's own check, the storage service reports no row or a quantity of 0 for a book, or does not answer, then the alerts service shall record no back-in-stock alert for that book ([AD#3], [AD#5]).

##### Test cases

**[TC01]: A quantity of 0 — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Wait 40 seconds. 2. Read C's pending subscriptions and alerts.
- *Expected result:* The alerts service has recorded no back-in-stock alert for B, and C's subscription to B is still pending.

**[TC02]: The storage row is deleted — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Delete B's storage row. 2. Wait 40 seconds. 3. Read C's pending subscriptions and alerts.
- *Expected result:* The alerts service has recorded no back-in-stock alert for B, and C's subscription to B is still pending.

**[TC03]: The storage service is stopped — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B; the storage service is stopped.
- *Steps:* 1. Wait 40 seconds. 2. Read C's pending subscriptions and alerts.
- *Expected result:* The alerts service has recorded no back-in-stock alert for B, and C's subscription to B is still pending.

**[TC04]: The storage service holds every request for 5 seconds — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Make the storage service hold every request for 5 seconds before replying. 2. Set B's quantity to 3 in the storage service, sending no notice. 3. Wait 40 seconds. 4. Read C's pending subscriptions and alerts.
- *Expected result:* The alerts service has recorded no back-in-stock alert for B, and C's subscription to B is still pending.

#### [AC06]: Alert each subscription once

When the alerts service learns more than once that an ISBN is available — by repeated notices, or by a notice and its own check — the alerts service shall record at most one alert for each subscription ([AD#4], [AD#5]).

##### Test cases

**[TC01]: Two notices in a row — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Tell it again. 3. Read C's alerts.
- *Expected result:* The alerts service has recorded exactly one alert for C's subscription.

**[TC02]: A notice and the alerts service's own check — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Set B's quantity to 1 in the storage service. 2. Tell the alerts service that B's ISBN is available. 3. Wait 40 seconds. 4. Read C's alerts.
- *Expected result:* The alerts service has recorded exactly one alert for C's subscription.

**[TC03]: Two notices at the same moment for 1,000 subscribers — State / lifecycle:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B.
- *Steps:* 1. Send two notices that B's ISBN is available at the same moment. 2. 10 seconds later, count the alerts for B.
- *Expected result:* The alerts service has recorded exactly 1,000 alerts, one for each subscription.

#### [AC07]: Never alert an ended subscription

If a subscription was cancelled or has already ended, then the alerts service shall record no alert for it on any later return of its book to stock ([AD#4], [AD#5]).

##### Test cases

**[TC01]: A cancelled subscription — Happy path:**
- *Preconditions:* Client C cancelled a subscription to book B; client D holds a pending subscription to B.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Read C's and D's alerts.
- *Expected result:* The alerts service has recorded no alert for C's cancelled subscription; D has a back-in-stock alert for B.

**[TC02]: A second return of the same book — Negative / boundary:**
- *Preconditions:* Client C's subscription to book B ended with a back-in-stock alert; B's quantity has since gone back to 0.
- *Steps:* 1. Set B's quantity to 1 and tell the alerts service that B's ISBN is available. 2. Read C's alerts.
- *Expected result:* The alerts service has recorded no alert for C's ended subscription; C has exactly one back-in-stock alert for B.

**[TC03]: A book re-created under the same ISBN — Negative / boundary:**
- *Preconditions:* Client C's subscription to book B ended with a withdrawal alert when B was removed.
- *Steps:* 1. Add a book with B's ISBN to the books service again, with a quantity of 1. 2. Tell the alerts service that B's ISBN is available. 3. Wait 40 seconds. 4. Read C's alerts.
- *Expected result:* The alerts service has recorded no alert for C's ended subscription; C has only the withdrawal alert for B.

**[TC04]: A new subscription after the alert — State / lifecycle:**
- *Preconditions:* Client C's subscription S1 to book B ended with a back-in-stock alert; B's quantity has since gone back to 0.
- *Steps:* 1. Call subscribe with C's email and B's ISBN, recording subscription S2. 2. Set B's quantity to 1 and tell the alerts service that B's ISBN is available. 3. Read C's alerts.
- *Expected result:* C has two back-in-stock alerts for B, one from S1 and one from S2; the alerts service recorded no further alert for the ended S1.

#### [AC08]: Record all or none

If the alerts service cannot complete recording the back-in-stock alerts for an ISBN, then the alerts service shall keep every pending subscription to that ISBN pending, with none of their alerts recorded ([AD#4]).

##### Test cases

**[TC01]: The database is unavailable when the notice arrives — Happy path:**
- *Preconditions:* 10 clients each hold a pending subscription to book B; the alerts service's database stops answering.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Restore the database before the alerts service's next check could run, with B still at a quantity of 0 in the storage service. 3. Read the 10 clients' pending subscriptions and alerts.
- *Expected result:* The alerts service keeps all 10 subscriptions to B pending, with none of their alerts recorded.

**[TC02]: The database connection is lost part-way through — Negative / boundary:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B, which has a quantity of 0 in the storage service.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Cut the alerts service's database connection after recording has started and before it finishes. 3. Restore the connection and read the pending subscriptions to B and the alerts for B.
- *Expected result:* The alerts service keeps all 1,000 subscriptions to B pending, with none of their alerts recorded.

#### [AC09]: Fill the back-in-stock alert

The alerts service shall record each back-in-stock alert with the type BACK_IN_STOCK, the ISBN, the book title stored when the subscription was made, the time it learned the book was available as occurredAt, the time it recorded the alert as createdAt, and read as false ([AD#8]).

##### Test cases

**[TC01]: An alert from a notice — Happy path:**
- *Preconditions:* Client C subscribed to book B while its title was "The Long Wait".
- *Steps:* 1. Tell the alerts service at time T that B's ISBN is available. 2. Read C's alerts.
- *Expected result:* The back-in-stock alert has the type BACK_IN_STOCK, B's ISBN, the title "The Long Wait", an occurredAt of T, a createdAt no later than 10 seconds after T, and read false.

**[TC02]: The title changed after subscribing — Negative / boundary:**
- *Preconditions:* Client C subscribed to book B while its title was "The Long Wait"; B's title in the books service has since become "The Longer Wait".
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Read C's alerts.
- *Expected result:* The back-in-stock alert carries the title stored when the subscription was made, "The Long Wait".

**[TC03]: An alert from the alerts service's own check — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. At time T0, set B's quantity to 1 in the storage service, sending no notice. 2. 40 seconds later, read C's alerts.
- *Expected result:* The back-in-stock alert's occurredAt, the time the alerts service learned B was available, is between T0 and 30 seconds after T0; its createdAt is no later than 10 seconds after its occurredAt; read is false.

#### [AC10]: Alert synthetic clients by the same rules

The alerts service shall alert the subscriptions of reserved clients and reserved books under exactly the rules it applies to every other subscription ([AD#4], [AD#5], [AD#11]).

##### Test cases

**[TC01]: A reserved client waiting on a reserved book — Happy path:**
- *Preconditions:* The client j1@bis-journey.invalid holds a pending subscription to the book with the ISBN 0000000012345, which has a quantity of 0.
- *Steps:* 1. Tell the alerts service that 0000000012345 is available. 2. Read the alerts of j1@bis-journey.invalid.
- *Expected result:* The alerts service has recorded a back-in-stock alert, exactly as for any other subscription.

**[TC02]: A reserved client's cancelled subscription — Negative / boundary:**
- *Preconditions:* The client j1@bis-journey.invalid cancelled a subscription to the book with the ISBN 0000000012345.
- *Steps:* 1. Set that book's quantity to 1 and tell the alerts service that 0000000012345 is available. 2. Read the alerts of j1@bis-journey.invalid.
- *Expected result:* The alerts service has recorded no alert for the cancelled subscription, exactly as for any other subscription.

---

### [U03]: Be told when a book I wait for is removed

As a client, I want an alert recorded for me when a book I wait for is removed from the catalogue, ending my wait, so that I stop waiting for a book that cannot come back.

#### [AC01]: Withdraw on a notice

When the alerts service is told that an ISBN was removed, the alerts service shall end every pending subscription to it with a withdrawal alert, answering 204 No Content whether or not any existed ([AD#6]).

##### Test cases

**[TC01]: Two waiting clients — Happy path:**
- *Preconditions:* Clients C1 and C2 each hold a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. Read each client's alerts and pending subscriptions.
- *Expected result:* The notice answers 204 No Content; C1 and C2 each have a withdrawal alert for B and no pending subscription to B.

**[TC02]: A removed book nobody waits for — Negative / boundary:**
- *Preconditions:* No pending subscription to book B exists; client C holds a pending subscription to book B2.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. Read C's alerts and pending subscriptions.
- *Expected result:* The notice answers 204 No Content; no alert is recorded, and C's subscription to B2 is still pending.

**[TC03]: A repeated removal notice — State / lifecycle:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. Tell it again. 3. Read C's alerts.
- *Expected result:* Both notices answer 204 No Content; C has exactly one withdrawal alert for B.

#### [AC02]: Withdraw 1,000 subscriptions within 10 seconds

While up to 1,000 pending subscriptions to one ISBN exist, the alerts service shall record every one of their withdrawal alerts within 10 seconds of being told the ISBN was removed ([AD#6]).

##### Test cases

**[TC01]: 1,000 waiting clients — Happy path:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. 10 seconds after the notice, count the withdrawal alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 withdrawal alerts within 10 seconds of the notice.

**[TC02]: 1,000 waiting clients beside another book's 1,000 — Negative / boundary:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B, and 1,000 others each hold one to book B2.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. 10 seconds after the notice, count the withdrawal alerts for B and the pending subscriptions to B2.
- *Expected result:* The alerts service has recorded every one of the 1,000 withdrawal alerts for B within 10 seconds, and all 1,000 subscriptions to B2 are still pending.

#### [AC03]: Recover a lost removal notice

While a book has pending subscriptions and the alerts service is not told it was removed, the alerts service shall record their withdrawal alerts within 40 seconds of the first moment at which the books service holds no book with the ISBN and answers the alerts service's calls, whatever the storage service holds for it ([AD#5], [AD#7]).

##### Test cases

**[TC01]: A single removal with no notice — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Delete B from the books service, sending no notice. 2. 40 seconds after the deletion, read C's alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded C's withdrawal alert for B within 40 seconds, and C's subscription to B is no longer pending.

**[TC02]: The leftovers of a missed reset — Negative / boundary:**
- *Preconditions:* Client C, whose email is not reserved, holds a pending subscription to book B, whose ISBN is not reserved.
- *Steps:* 1. Reset the books service's catalogue in bulk, sending the alerts service no reset. 2. 40 seconds after the reset, read C's alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded C's withdrawal alert for B within 40 seconds, as for a single removal, and C's subscription to B is no longer pending.

**[TC03]: The storage service still holds copies — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Make the books service hold every request for 5 seconds before replying. 2. Set B's storage quantity to 3, sending no notice. 3. Delete B from the books service, sending no notice. 4. Restore the books service's normal replies. 5. 40 seconds after the restore, read C's alerts.
- *Expected result:* The alerts service has recorded C's withdrawal alert for B within 40 seconds of the books service holding no book with B's ISBN and answering again, whatever the storage service holds for it, and no back-in-stock alert.

#### [AC04]: Recover 100 removed books at once

While 100 distinct books have pending subscriptions, the alerts service shall record, with no notice, the withdrawal alerts of every one of them removed from the catalogue within 40 seconds of the first moment at which the books service holds no book with its ISBN and answers the alerts service's calls ([AD#5]).

##### Test cases

**[TC01]: 100 books removed together — Happy path:**
- *Preconditions:* 100 distinct books each have one pending subscription and a quantity of 0.
- *Steps:* 1. Delete all 100 books from the books service at once, sending no notice. 2. 40 seconds after the deletions, count the withdrawal alerts.
- *Expected result:* The alerts service has recorded all 100 withdrawal alerts within 40 seconds, with no notice.

**[TC02]: One of 100 books removed — Negative / boundary:**
- *Preconditions:* 100 distinct books each have one pending subscription and a quantity of 0.
- *Steps:* 1. Delete the book subscribed to last from the books service, sending no notice. 2. 40 seconds after the deletion, read the alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded that book's withdrawal alert within 40 seconds, with no notice, and the other 99 subscriptions are still pending.

#### [AC05]: Withdraw nothing for an unpublished book

If, on the alerts service's own check, the books service finds a subscribed book unpublished, then the alerts service shall record no withdrawal alert for it ([AD#6]).

##### Test cases

**[TC01]: Unpublishing a book — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0.
- *Steps:* 1. Unpublish B in the books service. 2. Wait 40 seconds. 3. Read C's alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded no withdrawal alert for B, and C's subscription to B is still pending.

**[TC02]: Unpublished, published again and restocked — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a quantity of 0 and has been unpublished for 40 seconds.
- *Steps:* 1. Publish B again. 2. Set B's quantity to 1 and tell the alerts service that B's ISBN is available. 3. Read C's alerts.
- *Expected result:* C has a back-in-stock alert for B, and the alerts service recorded no withdrawal alert for B through the unpublishing.

#### [AC06]: Alert nothing while the catalogue cannot answer

If the books service does not answer about a subscribed book, then the alerts service shall record no alert for that book's subscriptions on that check ([AD#5]).

##### Test cases

**[TC01]: The books service is stopped — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B; the books service is stopped.
- *Steps:* 1. Wait 40 seconds. 2. Read C's alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded no alert for B, and C's subscription to B is still pending.

**[TC02]: The books service replies with an error — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B; the books service replies 500 to every request.
- *Steps:* 1. Wait 40 seconds. 2. Read C's alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded no alert for B, and C's subscription to B is still pending.

**[TC03]: The books service holds every request for 5 seconds — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Make the books service hold every request for 5 seconds before replying. 2. Delete B from the books service. 3. Wait 40 seconds. 4. Read C's alerts and pending subscriptions.
- *Expected result:* The alerts service has recorded no alert for B, and C's subscription to B is still pending.

#### [AC07]: Record all or none

If the alerts service cannot complete recording the withdrawal alerts for an ISBN, then the alerts service shall keep every pending subscription to that ISBN pending, with none of their alerts recorded ([AD#6]).

##### Test cases

**[TC01]: The database is unavailable when the notice arrives — Happy path:**
- *Preconditions:* 10 clients each hold a pending subscription to book B, which is still in the books service with a quantity of 0 in the storage service; the alerts service's database stops answering.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. Restore the database before the alerts service's next check could run. 3. Read the 10 clients' pending subscriptions and alerts.
- *Expected result:* The alerts service keeps all 10 subscriptions to B pending, with none of their alerts recorded.

**[TC02]: The database connection is lost part-way through — Negative / boundary:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B, which is still in the books service with a quantity of 0 in the storage service.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. Cut the alerts service's database connection after recording has started and before it finishes. 3. Restore the connection and read the pending subscriptions to B and the alerts for B.
- *Expected result:* The alerts service keeps all 1,000 subscriptions to B pending, with none of their alerts recorded.

#### [AC08]: Fill the withdrawal alert

The alerts service shall record each withdrawal alert with the type WITHDRAWN, the ISBN, the book title stored when the subscription was made, the time it learned the book was removed as occurredAt, the time it recorded the alert as createdAt, and read as false ([AD#6], [AD#8]).

##### Test cases

**[TC01]: An alert from a notice — Happy path:**
- *Preconditions:* Client C subscribed to book B while its title was "The Last Copy".
- *Steps:* 1. Tell the alerts service at time T that B's ISBN was removed. 2. Read C's alerts.
- *Expected result:* The withdrawal alert has the type WITHDRAWN, B's ISBN, the title "The Last Copy", an occurredAt of T, a createdAt no later than 10 seconds after T, and read false.

**[TC02]: The title once the book is gone — Negative / boundary:**
- *Preconditions:* Client C subscribed to book B while its title was "The Last Copy"; B has since been deleted from the books service.
- *Steps:* 1. Wait 40 seconds. 2. Read C's alerts.
- *Expected result:* The withdrawal alert carries the title stored when the subscription was made, "The Last Copy", and the type WITHDRAWN.

---

### [U04]: Read, count and dismiss my alerts

As a client, I want my alerts listed newest first for 30 days, counted while unread, all marked read at once and dismissed one at a time, so that I see only the alerts I still need.

#### [AC01]: List the email's alerts newest first

When an email's alerts are read, the alerts service shall return only that email's alerts, newest first by occurredAt and then createdAt, each with its id, type, ISBN, title, occurredAt, createdAt and read ([AD#8]).

##### Test cases

**[TC01]: Two alerts — Happy path:**
- *Preconditions:* Client C has alert X1 with an occurredAt of 09:00 and alert X2 with an occurredAt of 10:00.
- *Steps:* 1. Read C's alerts.
- *Expected result:* The alerts service returns X2 then X1, each with its id, type, ISBN, title, occurredAt, createdAt and read.

**[TC02]: The same occurredAt — Negative / boundary:**
- *Preconditions:* Client C has alerts X1 and X2 with the same occurredAt; X2 was recorded 2 seconds after X1.
- *Steps:* 1. Read C's alerts.
- *Expected result:* The alerts service returns X2 before X1, newest first by createdAt.

**[TC03]: A withdrawal alert among back-in-stock alerts — Negative / boundary:**
- *Preconditions:* Client C has a back-in-stock alert with an occurredAt of 09:00, a withdrawal alert with an occurredAt of 10:00 and a back-in-stock alert with an occurredAt of 11:00.
- *Steps:* 1. Read C's alerts.
- *Expected result:* The alerts service returns the 11:00, 10:00 and 09:00 alerts in that order, whatever their type.

**[TC04]: Another email's alerts are not returned — Security / privacy:**
- *Preconditions:* Clients C1 and C2 each have two alerts.
- *Steps:* 1. Read C1's alerts.
- *Expected result:* The alerts service returns only C1's two alerts.

#### [AC02]: Leave out dismissed and old alerts

While an alert is dismissed or was recorded more than 30 days ago, the alerts service shall leave it out of the email's alerts and unread count ([AD#8]).

##### Test cases

**[TC01]: An unread alert recorded 30 days and 1 minute ago — Happy path:**
- *Preconditions:* Client C has one unread alert, recorded 30 days and 1 minute ago.
- *Steps:* 1. Read C's alerts and unread count.
- *Expected result:* The alerts service leaves the alert out: an empty list and a count of 0.

**[TC02]: Either side of 30 days — Negative / boundary:**
- *Preconditions:* Client C has unread alert X1, recorded 30 days less 1 minute ago, and unread alert X2, recorded 30 days and 1 minute ago.
- *Steps:* 1. Read C's alerts and unread count.
- *Expected result:* The alerts service returns X1 and leaves out X2; the count is 1.

**[TC03]: A dismissed unread alert — Negative / boundary:**
- *Preconditions:* Client C has unread alerts X1 and X2; C has dismissed X1.
- *Steps:* 1. Read C's alerts and unread count.
- *Expected result:* The alerts service leaves X1 out: it returns X2 only, and the count is 1.

#### [AC03]: Count unread alerts

When an email's unread count is read, the alerts service shall answer with a count field equal to the number of that email's listed alerts that are unread ([AD#8]).

##### Test cases

**[TC01]: Two of three unread — Happy path:**
- *Preconditions:* Client C has three listed alerts, two of them unread.
- *Steps:* 1. Read C's unread count.
- *Expected result:* The alerts service answers with a count field of 2.

**[TC02]: Unread alerts that are not listed — Negative / boundary:**
- *Preconditions:* Client C has one unread listed alert, one unread dismissed alert and one unread alert recorded 31 days ago.
- *Steps:* 1. Read C's unread count.
- *Expected result:* The alerts service answers with a count field of 1.

**[TC03]: Another email's unread alerts are not counted — Security / privacy:**
- *Preconditions:* Client C1 has one unread alert; client C2 has three.
- *Steps:* 1. Read C1's unread count.
- *Expected result:* The alerts service answers with a count field of 1.

#### [AC04]: Mark every alert read

When mark-read is called for an email, the alerts service shall mark every alert of that email read, answering 204 No Content however often it is repeated ([AD#8]).

##### Test cases

**[TC01]: Three unread alerts — Happy path:**
- *Preconditions:* Client C has three unread alerts.
- *Steps:* 1. Call mark-read for C. 2. Read C's alerts and unread count.
- *Expected result:* Mark-read answers 204 No Content; every alert of C is listed with read true, and the count is 0.

**[TC02]: Mark-read repeated — Negative / boundary:**
- *Preconditions:* Client C has two alerts, both read.
- *Steps:* 1. Call mark-read for C twice. 2. Read C's alerts.
- *Expected result:* Both calls answer 204 No Content; both alerts are still listed with read true.

**[TC03]: An alert recorded after the list was read — Negative / boundary:**
- *Preconditions:* Client C has one unread alert X1 and has fetched their alerts list; alert X2 is then recorded for C.
- *Steps:* 1. Call mark-read for C. 2. Read C's alerts.
- *Expected result:* The alerts service marks every alert of C read, X2 included.

**[TC04]: Another email's alerts stay unread — Security / privacy:**
- *Preconditions:* Clients C1 and C2 each have two unread alerts.
- *Steps:* 1. Call mark-read for C1. 2. Read C2's unread count.
- *Expected result:* Mark-read answers 204 No Content and marks only C1's alerts read; C2's count is 2.

#### [AC05]: Dismiss an alert

When an email dismisses by id one of its alerts recorded within the last 30 days, the alerts service shall leave that alert out of every later read and unread count, answering 204 No Content ([AD#8]).

##### Test cases

**[TC01]: Dismiss one of two — Happy path:**
- *Preconditions:* Client C has unread alerts X1 and X2.
- *Steps:* 1. Dismiss X1 for C. 2. Read C's alerts and unread count.
- *Expected result:* Dismiss answers 204 No Content; the alerts service returns X2 only, and the count is 1.

**[TC02]: Dismiss the only alert — Negative / boundary:**
- *Preconditions:* Client C has exactly one alert, X, unread.
- *Steps:* 1. Dismiss X for C. 2. Read C's alerts and unread count.
- *Expected result:* Dismiss answers 204 No Content; the alerts service returns an empty list and a count of 0.

#### [AC06]: Dismiss a dismissed alert without error

When an email dismisses one of its alerts that was recorded within the last 30 days and is already dismissed, the alerts service shall answer 204 No Content, changing nothing ([AD#8]).

##### Test cases

**[TC01]: Dismiss again — Happy path:**
- *Preconditions:* Client C has dismissed alert X1 and still has alert X2.
- *Steps:* 1. Dismiss X1 for C again. 2. Read C's alerts.
- *Expected result:* Dismiss answers 204 No Content, changing nothing: X2 is still listed and X1 is not.

**[TC02]: Two dismiss calls at the same moment — State / lifecycle:**
- *Preconditions:* Client C has alert X.
- *Steps:* 1. Send two dismiss calls for X at the same moment. 2. Read C's alerts.
- *Expected result:* Both calls answer 204 No Content, and X is not listed.

**[TC03]: A dismissed alert recorded 30 days less 1 minute ago — Negative / boundary:**
- *Preconditions:* Client C dismissed alert X, which was recorded 30 days less 1 minute ago.
- *Steps:* 1. Dismiss X for C again.
- *Expected result:* Dismiss answers 204 No Content, changing nothing.

#### [AC07]: Refuse an alert that is not the email's

If dismiss names an id that is not one of that email's alerts, or one recorded more than 30 days ago, then the alerts service shall answer 404 Not Found, changing nothing ([AD#8]).

##### Test cases

**[TC01]: An id that does not exist — Happy path:**
- *Preconditions:* Client C has alert X; no alert has the id 999999.
- *Steps:* 1. Dismiss the id 999999 for C. 2. Read C's alerts.
- *Expected result:* Dismiss answers 404 Not Found, changing nothing: X is still listed.

**[TC02]: Another email's alert — Security / privacy:**
- *Preconditions:* Client C2 has alert X; client C1 has none.
- *Steps:* 1. Dismiss X for C1. 2. Read C2's alerts.
- *Expected result:* Dismiss answers 404 Not Found, changing nothing: X is still listed for C2.

**[TC03]: An alert recorded 30 days and 1 minute ago — Negative / boundary:**
- *Preconditions:* Client C has alert X, recorded 30 days and 1 minute ago.
- *Steps:* 1. Dismiss X for C.
- *Expected result:* Dismiss answers 404 Not Found.

#### [AC08]: Answer empty for an email with no alert

When the alerts or the unread count of an email the alerts service holds no alert for are read, the alerts service shall return an empty list and a count of 0 ([AD#2], [AD#8]).

##### Test cases

**[TC01]: An email never used — Happy path:**
- *Preconditions:* The alerts service holds no alert for new.client@example.com.
- *Steps:* 1. Read the alerts and the unread count of new.client@example.com.
- *Expected result:* The alerts service returns an empty list and a count of 0.

**[TC02]: An email the clients service does not know — Negative / boundary:**
- *Preconditions:* The clients service holds no client with the email nobody@example.com, and the alerts service holds no alert for it.
- *Steps:* 1. Read the alerts and the unread count of nobody@example.com.
- *Expected result:* The alerts service returns an empty list and a count of 0.

#### Open questions

- [ ] How fast must the unread count and the reads of alerts and pending subscriptions answer, and for how many open store pages polling every 20 seconds ([AD#9]) — for example a 95th percentile under a stated number of milliseconds at a stated number of polling clients? No criterion bounds it, and [AD#9]'s 60-second budget leaves the read itself no time; it puts [U04] AC01, AC03 and [U05] AC01 in doubt under load.

---

### [U05]: See and cancel my pending subscriptions

As a client, I want to see every subscription I still wait on and cancel any of them, so that I am alerted only about books I still want.

#### [AC01]: List the email's pending subscriptions

When an email's pending subscriptions are read, the alerts service shall return only that email's pending subscriptions, newest first by createdAt, each with its id, the email as given at subscribe, ISBN, title and createdAt ([AD#8]).

##### Test cases

**[TC01]: Two pending subscriptions — Happy path:**
- *Preconditions:* Client C subscribed as First.Last@Example.com to book B1 on 2026-10-01 and to book B2 on 2026-10-05.
- *Steps:* 1. Read the pending subscriptions of first.last@example.com.
- *Expected result:* The alerts service returns the subscription to B2 and then the one to B1, newest first, each with its id, the email First.Last@Example.com, its ISBN, its title and its createdAt of 2026-10-05 and 2026-10-01.

**[TC02]: Another email's subscriptions are not returned — Security / privacy:**
- *Preconditions:* Clients C1 and C2 each hold a pending subscription to a different book.
- *Steps:* 1. Read C1's pending subscriptions.
- *Expected result:* The alerts service returns only C1's subscription.

**[TC03]: Only pending subscriptions are returned — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B1, and C's subscription to book B2 ended with a back-in-stock alert.
- *Steps:* 1. Read C's pending subscriptions.
- *Expected result:* The alerts service returns only the pending subscription to B1.

#### [AC02]: Cancel a subscription

When an email cancels one of its pending subscriptions by id, the alerts service shall end that subscription, answering 204 No Content ([AD#8]).

##### Test cases

**[TC01]: Cancel a pending subscription — Happy path:**
- *Preconditions:* Client C holds pending subscription S to book B.
- *Steps:* 1. Cancel S for C. 2. Read C's pending subscriptions.
- *Expected result:* Cancel answers 204 No Content; the alerts service has ended S, and it is no longer listed.

**[TC02]: Cancel one of two — Negative / boundary:**
- *Preconditions:* Client C holds pending subscriptions S1 to book B1 and S2 to book B2.
- *Steps:* 1. Cancel S1 for C. 2. Read C's pending subscriptions.
- *Expected result:* Cancel answers 204 No Content; the alerts service has ended S1 only, and S2 is still listed.

#### [AC03]: Cancel an ended subscription without error

When an email cancels one of its subscriptions that ended — by cancellation, a back-in-stock alert or a withdrawal alert — within the last 30 days, the alerts service shall answer 204 No Content, changing nothing ([AD#8]).

##### Test cases

**[TC01]: Cancel twice — Happy path:**
- *Preconditions:* Client C cancelled subscription S a minute ago.
- *Steps:* 1. Cancel S for C again.
- *Expected result:* Cancel answers 204 No Content, changing nothing.

**[TC02]: Cancel after the alert ended it — Negative / boundary:**
- *Preconditions:* Client C's subscription S to book B ended with a back-in-stock alert X.
- *Steps:* 1. Cancel S for C. 2. Read C's alerts.
- *Expected result:* Cancel answers 204 No Content, changing nothing: X is still listed.

**[TC03]: Cancel a subscription that ended 30 days less 1 minute ago — Negative / boundary:**
- *Preconditions:* Client C's subscription S ended with a withdrawal alert 30 days less 1 minute ago.
- *Steps:* 1. Cancel S for C.
- *Expected result:* Cancel answers 204 No Content, changing nothing.

#### [AC04]: Refuse a subscription that is not the email's

If cancel names an id that is not one of that email's subscriptions, or one that ended more than 30 days ago, then the alerts service shall answer 404 Not Found, changing nothing ([AD#8]).

##### Test cases

**[TC01]: An id that does not exist — Happy path:**
- *Preconditions:* Client C holds pending subscription S; no subscription has the id 999999.
- *Steps:* 1. Cancel the id 999999 for C. 2. Read C's pending subscriptions.
- *Expected result:* Cancel answers 404 Not Found, changing nothing: S is still listed.

**[TC02]: Another email's subscription — Security / privacy:**
- *Preconditions:* Client C2 holds pending subscription S; client C1 holds none.
- *Steps:* 1. Cancel S for C1. 2. Read C2's pending subscriptions.
- *Expected result:* Cancel answers 404 Not Found, changing nothing: S is still pending for C2.

**[TC03]: A subscription that ended 30 days and 1 minute ago — Negative / boundary:**
- *Preconditions:* Client C's subscription S was cancelled 30 days and 1 minute ago.
- *Steps:* 1. Cancel S for C.
- *Expected result:* Cancel answers 404 Not Found.

#### [AC05]: Drop ended subscriptions from the list

When a subscription ends — by cancellation, a back-in-stock alert or a withdrawal alert — the alerts service shall leave it out of every later read of pending subscriptions ([AD#4], [AD#6], [AD#8]).

##### Test cases

**[TC01]: Ended by a back-in-stock alert — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN is available. 2. Read C's pending subscriptions.
- *Expected result:* The alerts service leaves the subscription to B out of the pending subscriptions.

**[TC02]: Ended by a withdrawal alert — State / lifecycle:**
- *Preconditions:* Client C holds a pending subscription to book B.
- *Steps:* 1. Tell the alerts service that B's ISBN was removed. 2. Read C's pending subscriptions.
- *Expected result:* The alerts service leaves the subscription to B out of the pending subscriptions.

**[TC03]: A subscription that did not end stays listed — Negative / boundary:**
- *Preconditions:* Client C holds pending subscriptions to books B1 and B2.
- *Steps:* 1. Tell the alerts service that B1's ISBN is available. 2. Read C's pending subscriptions.
- *Expected result:* The alerts service leaves the subscription to B1 out and still lists the subscription to B2.

#### [AC06]: Answer empty for an email with no subscription

When the pending subscriptions of an email the alerts service holds none for are read, the alerts service shall return an empty list ([AD#2], [AD#8]).

##### Test cases

**[TC01]: An email never used — Happy path:**
- *Preconditions:* The alerts service holds no subscription for new.client@example.com.
- *Steps:* 1. Read the pending subscriptions of new.client@example.com.
- *Expected result:* The alerts service returns an empty list.

**[TC02]: An email the clients service does not know — Negative / boundary:**
- *Preconditions:* The clients service holds no client with the email nobody@example.com, and the alerts service holds no subscription for it.
- *Steps:* 1. Read the pending subscriptions of nobody@example.com.
- *Expected result:* The alerts service returns an empty list.

---

### [U06]: Clear subscriptions and alerts on a catalogue reset

As a demo operator, I want a bulk catalogue reset to clear every subscription and alert except the synthetic journey's own, without alerting anyone, so that resetting the store neither floods clients with alerts nor breaks the running journey.

#### [AC01]: Remove what is not reserved

When the reset is called, the alerts service shall remove every subscription and alert — pending or ended, read, unread or dismissed — whose ISBN does not start with 00000000 and whose email does not end with @bis-journey.invalid, answering 204 No Content ([AD#7], [AD#11]).

##### Test cases

**[TC01]: A client's whole record — Happy path:**
- *Preconditions:* Client c@example.com holds a pending subscription S1 to book B1 and a cancelled subscription S2 to book B2, and has three alerts about other books, one read and one dismissed; no ISBN involved starts with 00000000.
- *Steps:* 1. Call the reset. 2. Read c@example.com's pending subscriptions, alerts and unread count. 3. Cancel S2 for c@example.com.
- *Expected result:* The reset answers 204 No Content; the alerts service has removed every one of the subscriptions and alerts: empty lists, a count of 0, and cancel answers 404 Not Found.

**[TC02]: An ISBN that starts with seven zeros — Negative / boundary:**
- *Preconditions:* Client c@example.com holds a pending subscription to the book with the ISBN 0000000123456.
- *Steps:* 1. Call the reset. 2. Read c@example.com's pending subscriptions.
- *Expected result:* The alerts service has removed the subscription: an empty list.

**[TC03]: An email that only contains the reserved domain — Negative / boundary:**
- *Preconditions:* The email j1@bis-journey.invalid.example holds a pending subscription to book B, whose ISBN does not start with 00000000.
- *Steps:* 1. Call the reset. 2. Read the pending subscriptions of j1@bis-journey.invalid.example.
- *Expected result:* The alerts service has removed the subscription: an empty list.

**[TC04]: The reserved domain written in capitals — Negative / boundary:**
- *Preconditions:* A pending subscription to book B, whose ISBN does not start with 00000000, was made with the email J1@BIS-JOURNEY.INVALID.
- *Steps:* 1. Call the reset. 2. Read the pending subscriptions of J1@BIS-JOURNEY.INVALID.
- *Expected result:* The alerts service has removed the subscription, its email not ending with @bis-journey.invalid as written: an empty list.

#### [AC02]: Keep what is reserved

When the reset is called, the alerts service shall keep every subscription and alert whose ISBN starts with 00000000 or whose email ends with @bis-journey.invalid — another client's subscription to a reserved book included ([AD#11]).

##### Test cases

**[TC01]: The journey's own client and book — Happy path:**
- *Preconditions:* The client j1@bis-journey.invalid holds a pending subscription to the book with the ISBN 0000000012345 and has one alert.
- *Steps:* 1. Call the reset. 2. Read the pending subscriptions and alerts of j1@bis-journey.invalid.
- *Expected result:* The alerts service has kept the subscription and the alert.

**[TC02]: Another client's subscription to a reserved book — Negative / boundary:**
- *Preconditions:* Client c@example.com holds a pending subscription to the book with the ISBN 0000000012345.
- *Steps:* 1. Call the reset. 2. Read c@example.com's pending subscriptions.
- *Expected result:* The alerts service has kept the subscription, because its book is reserved.

**[TC03]: A reserved client's subscription to another book — Negative / boundary:**
- *Preconditions:* The client j1@bis-journey.invalid holds a pending subscription to book B, whose ISBN does not start with 00000000.
- *Steps:* 1. Call the reset. 2. Read the pending subscriptions of j1@bis-journey.invalid.
- *Expected result:* The alerts service has kept the subscription, because its client is reserved.

#### [AC03]: Record no alert on a reset

When the reset is called, the alerts service shall record no alert ([AD#7]).

##### Test cases

**[TC01]: Ten waiting clients — Happy path:**
- *Preconditions:* Ten clients, none reserved, each hold a pending subscription to a different book, none reserved; none has an alert.
- *Steps:* 1. Call the reset. 2. Wait 40 seconds. 3. Read every one of the ten clients' alerts.
- *Expected result:* The alerts service has recorded no alert.

**[TC02]: A kept subscription is not alerted by the reset — Negative / boundary:**
- *Preconditions:* The client j1@bis-journey.invalid holds a pending subscription to the book with the ISBN 0000000012345, which has a quantity of 0 and stays in the books service.
- *Steps:* 1. Call the reset. 2. Wait 40 seconds. 3. Read the alerts of j1@bis-journey.invalid.
- *Expected result:* The alerts service has recorded no alert, and the subscription is still pending.

---

### [U07]: Run the alerts service with the store

As a demo operator, I want the alerts service deployed with the rest of the store, on a fresh or an existing deployment, reachable from the browser and from the other services, so that every other part of the feature can call it without my resetting the store's data.

#### [AC01]: Answer through the store's addresses

When the store is deployed with its deployment scripts, the alerts service shall answer requests the browser sends to /api/alerts/ through the ingress and requests another service sends to the address in DT_ALERTS_SERVER.

##### Test cases

**[TC01]: From the browser through the ingress — Happy path:**
- *Preconditions:* The store has been deployed with its deployment scripts, the alerts service included.
- *Steps:* 1. From a browser on the operator's machine, read the alerts of new.client@example.com through the store's address under /api/alerts/.
- *Expected result:* The alerts service answers with an empty list.

**[TC02]: From another service inside the cluster — Happy path:**
- *Preconditions:* The store has been deployed with its deployment scripts, the alerts service included.
- *Steps:* 1. From inside the storage service's pod, read the unread count of new.client@example.com at the address in DT_ALERTS_SERVER.
- *Expected result:* The alerts service answers with a count field of 0.

**[TC03]: A request under /api/alerts/ with no email — Negative / boundary:**
- *Preconditions:* The store has been deployed with its deployment scripts, the alerts service included.
- *Steps:* 1. From a browser on the operator's machine, read alerts through the store's address under /api/alerts/ with no email parameter.
- *Expected result:* The alerts service, not the web store's catch-all page, answers the request, with 400 Bad Request.

#### [AC02]: Leave every existing route unchanged

While the alerts service is deployed, the ingress shall route every path that existed before this feature as it did before, with the web store's catch-all route still last.

##### Test cases

**[TC01]: Every service's existing path — Happy path:**
- *Preconditions:* The store has been deployed with the alerts service included.
- *Steps:* 1. Through the store's address, call the version operation under each of /api/clients/, /api/books/, /api/carts/, /api/storage/, /api/orders/, /api/payments/, /api/dynapay/, /api/ratings/ and /api/ingest/.
- *Expected result:* The ingress routes each call to the same service as before this feature, and each answers its version.

**[TC02]: An address no service path matches — Negative / boundary:**
- *Preconditions:* The store has been deployed with the alerts service included.
- *Steps:* 1. Through the store's address, open the web store's books page and its orders page.
- *Expected result:* The ingress routes both to the web store through its catch-all route, which is still last.

#### [AC03]: Start on an existing deployment

When the alerts service is deployed onto a deployment whose database was initialised before this feature, the alerts service shall answer its operations, with no other service's data reset.

##### Test cases

**[TC01]: An existing deployment keeping its database — Happy path:**
- *Preconditions:* A deployment of the store from before this feature runs with books, storage, orders and ingest data in its PostgreSQL database and clients in its MySQL database; the number of books, storage rows, orders and clients has been recorded.
- *Steps:* 1. Deploy this release with `restart.sh -nodb`. 2. Read the alerts of new.client@example.com. 3. Count the books, storage rows, orders and clients again.
- *Expected result:* The alerts service answers its operations with an empty list, and every recorded count is unchanged — no other service's data reset.

**[TC02]: A fresh deployment — Negative / boundary:**
- *Preconditions:* No deployment of the store exists; its database volumes are new.
- *Steps:* 1. Deploy this release with its deployment scripts. 2. Read the alerts of new.client@example.com.
- *Expected result:* The alerts service answers its operations with an empty list.

#### [AC04]: Keep data across a restart

When the alerts service restarts, or is redeployed keeping its database, the alerts service shall keep every subscription and alert it held, each in the state it was in ([AD#1]).

##### Test cases

**[TC01]: A restart of the alerts service — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, a cancelled subscription to book B2, an unread alert X1 and a dismissed alert X2.
- *Steps:* 1. Restart the alerts service. 2. Read C's pending subscriptions, alerts and unread count. 3. Cancel the subscription to B2 for C.
- *Expected result:* The alerts service has kept every subscription and alert in the state it was in: the subscription to B is listed, X1 is listed unread, X2 is not listed, the count is 1, and cancel answers 204 No Content.

**[TC02]: A redeploy keeping the database — State / lifecycle:**
- *Preconditions:* Client C holds a pending subscription to book B and has one unread alert.
- *Steps:* 1. Redeploy the store with `restart.sh -nodb`. 2. Read C's pending subscriptions and unread count.
- *Expected result:* The alerts service has kept the subscription and the alert in the state they were in: the subscription is listed and the count is 1.

**[TC03]: A restart while 1,000 subscriptions wait — Negative / boundary:**
- *Preconditions:* 1,000 distinct clients each hold a pending subscription to book B.
- *Steps:* 1. Restart the alerts service. 2. Count the pending subscriptions to B.
- *Expected result:* The alerts service has kept all 1,000 subscriptions pending.

#### [AC05]: Remove the alerts service with the store

When the store is removed with its deletion scripts, the deletion scripts shall remove the alerts service's deployment and service and, where they also remove the store's databases, its data.

##### Test cases

**[TC01]: Remove the store's services — Happy path:**
- *Preconditions:* The store has been deployed with the alerts service included.
- *Steps:* 1. Run `delete.sh`. 2. List the deployments and services in the store's namespace. 3. From a pod still in the cluster, call the address that was in DT_ALERTS_SERVER.
- *Expected result:* The deletion scripts have removed the alerts service's deployment and service: neither is listed, and the address does not answer.

**[TC02]: Remove everything, then deploy again — Negative / boundary:**
- *Preconditions:* The store has been deployed with the alerts service included; client C holds a pending subscription to book B.
- *Steps:* 1. Run `delete.sh -all`. 2. Deploy the store again with its deployment scripts. 3. Read C's pending subscriptions.
- *Expected result:* The deletion scripts removed the alerts service's data with the store's databases: the alerts service answers with an empty list.

#### [AC06]: Run under the agent the store selects

Where the store's agent configuration selects oneAgent, otelAgent or none for the alerts service, the alerts service shall run with that agent, or with none, exactly as every other BookStore service does.

##### Test cases

**[TC01]: An agent selected — Happy path:**
- *Preconditions:* The store's agent configuration selects oneAgent for the alerts service and for the storage service.
- *Steps:* 1. Deploy the store with its deployment scripts. 2. Call the version operation of the alerts service and of the storage service.
- *Expected result:* The alerts service runs with oneAgent: its version reports that agent exactly as the storage service's version reports its own.

**[TC02]: No agent selected — Negative / boundary:**
- *Preconditions:* The store's agent configuration selects none for the alerts service.
- *Steps:* 1. Deploy the store with its deployment scripts. 2. Call the alerts service's version operation.
- *Expected result:* The alerts service runs with no agent: its version reports none as its agent.

#### [AC07]: Answer the version and configuration operations

The alerts service shall answer the version and configuration operations every BookStore service offers, its version naming the alerts service.

##### Test cases

**[TC01]: The version operation — Happy path:**
- *Preconditions:* The alerts service is deployed.
- *Steps:* 1. Call the alerts service's version operation.
- *Expected result:* The alerts service answers with its version, naming the alerts service.

**[TC02]: The configuration operations on a fresh database — Negative / boundary:**
- *Preconditions:* The alerts service is deployed on a fresh database.
- *Steps:* 1. Call the alerts service's operation that lists its configuration entries.
- *Expected result:* The alerts service answers the configuration operation with an empty list of entries, as every BookStore service does on a fresh database.

#### Open questions

- [ ] Does the alerts service apply the configuration entries set through its configuration operations as every other BookStore service does, and do the time limits of [U01] AC09, [U02] AC02–AC04 and [U03] AC02–AC04 then hold only while no such entry is turned on — or is their effect on the alerts service out of scope? It puts [U07] AC07 in doubt.

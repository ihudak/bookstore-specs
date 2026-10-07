---
kind: specification
key: BOOK-1
---

# Specification

- **Feature name**: Back-in-stock alerts
- **Version**: 1
- **Created**: 2026-10-07
- **Author**: ivan.gudak
- **Published**: no
- **Open questions**: 0

## Problem statement

A client who looks at a book while no copies are available has no way to learn when it comes back except returning and checking again. Most stop checking, so the interest they showed is lost, and a returned copy a waiting client would have bought sits unsold or goes to whoever happens to look first. BookStore also runs continuously under synthetic client traffic for demonstrations, and none of the journeys that traffic drives today starts from a book coming back into stock or leaving the catalogue, so a demo cannot show one stock change reaching many waiting clients. Solving it recovers sales lost while a book is unavailable and gives the demo a journey it can show at any moment.

## Scope

**In scope**

- Subscribing the current client, from a book's page, to a book that has no available copies.
- Alerting every client subscribed to a book, inside the store, within one minute of the book's available copies going from none to one or more — whatever caused it: new stock, a corrected quantity or a returned order — and ending each subscription that alert fulfils.
- An unread-alert count shown on every page while the current client has one or more unread alerts, and an alerts list.
- Listing the current client's pending subscriptions, and cancelling one from that list or from the book's page.
- Keeping each alert listed for 30 days, and dismissing it earlier.
- Ending every pending subscription to a book removed from the catalogue on its own, with a withdrawal alert to each waiting client within one minute.
- Choosing the current client from the Clients pages, showing it on every page and clearing it; the choice is remembered in the browser.
- Telling the current client when a subscription could not be made — the book has copies again, the book or the client no longer exists, or the store could not check — and making none.
- Telling the current client when subscriptions or alerts are unavailable, and when a cancel or a dismissal could not be done.
- A bulk catalogue reset the alerts service is told of clearing every subscription and alert other than the synthetic journey's own — those whose book or client is the journey's — without sending any alert; the subscriptions a reset the alerts service missed leaves behind are withdrawn with withdrawal alerts, as for a single removal.
- Synthetic traffic that continuously takes its own books out of stock, subscribes, cancels and restocks them, so every 5-minute window shows a subscription, a cancellation and a delivered back-in-stock alert, with its books and clients kept through every bulk reset.

**Out of scope**

- Withdrawal alerts on a bulk catalogue reset the alerts service is told of.
- Treating unpublishing a book as removing it from the catalogue; its pending subscriptions stay pending.
- Marking an alert unread again.
- Restoring a dismissed alert.
- Sign-in, or any check that the person choosing a current client is that client.
- Email, SMS or any other notification sent outside the store.
- Holding or reserving a restocked copy for an alerted client, or alerting only as many clients as there are copies.
- Pending subscriptions that lapse on their own, and subscriptions that stay active after their alert.
- Alerts when a book that already has available copies gains more.
- Showing how many copies are available, in an alert or on the book's page.
- Removing a deleted client's subscriptions and alerts; a client re-created with the same email sees them.
- Hiding the synthetic journey's books and clients from the store's lists.

## User stories

### [U01]: Subscribe to an out-of-stock book

As a client, I want to subscribe to a book that has no available copies, so that I am told when it is back instead of checking again.

#### [AC01]: Record the subscription

When the current client subscribes from the page of a book with no available copies, the alerts service shall record one pending subscription of that client to that book ([AD#8]).

##### Test cases

**[TC01]: Subscribe to a book with a stock of zero — Happy path:**
- *Preconditions:* Client C exists and is the current client; book B is in the catalogue with a stock of zero; C holds no subscription.
- *Steps:* 1. Open B's page. 2. Select the subscribe action. 3. Open the pending-subscriptions list.
- *Expected result:* The alerts service has recorded one pending subscription of C to B, and C's pending-subscriptions list shows it.

**[TC02]: Subscribe to a book the store holds no stock for — Negative / boundary:**
- *Preconditions:* Client C is the current client; book B is in the catalogue and the store has never held stock for it.
- *Steps:* 1. Open B's page. 2. Select the subscribe action. 3. Open the pending-subscriptions list.
- *Expected result:* The alerts service has recorded one pending subscription of C to B.

**[TC03]: The subscription belongs to the current client only — Security / privacy:**
- *Preconditions:* Clients C1 and C2 exist; C1 is the current client; book B has no available copies; neither client holds a subscription.
- *Steps:* 1. Open B's page and select the subscribe action. 2. Make C2 the current client. 3. Open the pending-subscriptions list.
- *Expected result:* The alerts service has recorded the pending subscription for C1 only; C2's pending-subscriptions list shows no subscription to B.

**[TC04]: Subscribe as an email with a plus sign and capitals — Negative / boundary:**
- *Preconditions:* Client C's email is First.Last+Books@Example.com and C is the current client; book B has no available copies; C holds no subscription.
- *Steps:* 1. Open B's page. 2. Select the subscribe action. 3. Open the pending-subscriptions list.
- *Expected result:* The alerts service has recorded one pending subscription of C to B, and C's pending-subscriptions list shows it.

#### [AC02]: Offer the subscription

While a current client is selected, the web store shall show a subscribe action on the page of every book that has no available copies — no stock held for it, or a stock of zero — and to which the current client holds no pending subscription ([AD#3], [AD#8]).

##### Test cases

**[TC01]: Offer on a book with a stock of zero — Happy path:**
- *Preconditions:* Client C is the current client; book B has a stock of zero; C holds no subscription to B.
- *Steps:* 1. Open B's page.
- *Expected result:* The web store shows a subscribe action on B's page.

**[TC02]: Offer once the last copy is sold — Negative / boundary:**
- *Preconditions:* Client C is the current client; book B has exactly one available copy.
- *Steps:* 1. Sell B's last copy. 2. Open B's page.
- *Expected result:* The web store shows a subscribe action on B's page, which now has no available copies.

#### [AC03]: Show the subscribed state

While the current client holds a pending subscription to a book, the web store shall show on that book's page that the client is subscribed, in place of the subscribe action ([AD#8]).

##### Test cases

**[TC01]: Subscribed state replaces the action — Happy path:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies.
- *Steps:* 1. Open B's page.
- *Expected result:* B's page shows that C is subscribed, in place of the subscribe action.

**[TC02]: Another client's subscription does not show as subscribed — Negative / boundary:**
- *Preconditions:* Client C2 holds a pending subscription to book B, which has no available copies; client C1 is the current client and holds none.
- *Steps:* 1. Open B's page.
- *Expected result:* B's page shows the subscribe action and does not show that the client is subscribed.

**[TC03]: Subscribed state ends with the subscription — State / lifecycle:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies.
- *Steps:* 1. Open the pending-subscriptions list and cancel the subscription to B. 2. Open B's page.
- *Expected result:* B's page no longer shows that C is subscribed, because C no longer holds a pending subscription to B; it shows the subscribe action.

#### [AC04]: Keep one pending subscription per book

When the current client subscribes to a book they already hold a pending subscription to, the alerts service shall leave exactly one pending subscription of that client to that book ([AD#8]).

##### Test cases

**[TC01]: Subscribing twice from two tabs — Happy path:**
- *Preconditions:* Client C is the current client; book B has no available copies; B's page is open in two tabs, both showing the subscribe action.
- *Steps:* 1. Select the subscribe action in the first tab. 2. Select the subscribe action in the second tab. 3. Open the pending-subscriptions list.
- *Expected result:* The alerts service holds exactly one pending subscription of C to B.

**[TC02]: Two clients keep one subscription each — Negative / boundary:**
- *Preconditions:* Clients C1 and C2 exist; book B has no available copies; neither holds a subscription.
- *Steps:* 1. Make C1 the current client and subscribe to B twice. 2. Make C2 the current client and subscribe to B. 3. Check each client's pending-subscriptions list.
- *Expected result:* C1 holds exactly one pending subscription to B and C2 holds exactly one pending subscription to B.

#### [AC05]: Never lapse

While a subscribed book has no available copies and is still in the catalogue, the alerts service shall not end the subscription because of the time elapsed since it was made.

##### Test cases

**[TC01]: A month-old subscription is still pending — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B made 31 days ago; B is still in the catalogue with no available copies.
- *Steps:* 1. Make C the current client. 2. Open the pending-subscriptions list.
- *Expected result:* The alerts service keeps C's subscription to B pending, and the list shows it.

**[TC02]: A stock write that leaves zero copies does not end it — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Set B's stock to zero again. 2. Wait 60 seconds. 3. Open C's pending-subscriptions list.
- *Expected result:* The alerts service keeps C's subscription to B pending.

**[TC03]: A restart of the alerts service does not end it — State / lifecycle:**
- *Preconditions:* Client C holds a pending subscription to book B, which has no available copies.
- *Steps:* 1. Restart the alerts service. 2. Open C's pending-subscriptions list.
- *Expected result:* The alerts service keeps C's subscription to B pending.

#### [AC06]: No offer otherwise

If a book has one or more available copies, or no current client is selected, then the web store shall show no subscribe action on the book's page ([AD#3]).

##### Test cases

**[TC01]: No offer on a book with copies — Happy path:**
- *Preconditions:* Client C is the current client; book B has 3 available copies.
- *Steps:* 1. Open B's page.
- *Expected result:* The web store shows no subscribe action on B's page.

**[TC02]: No offer on a book with exactly one copy — Negative / boundary:**
- *Preconditions:* Client C is the current client; book B has exactly one available copy.
- *Steps:* 1. Open B's page.
- *Expected result:* The web store shows no subscribe action on B's page.

**[TC03]: No offer with no current client — Negative / boundary:**
- *Preconditions:* No current client is selected; book B has no available copies.
- *Steps:* 1. Open B's page.
- *Expected result:* The web store shows no subscribe action on B's page.

#### [AC07]: Record nothing on a failed subscription

If a subscription is requested for a book that has available copies again, for a book or a current client that no longer exists, or while the store cannot check the book's stock, the book or the client, then the alerts service shall record no subscription ([AD#8]).

##### Test cases

**[TC01]: The book has copies again — Happy path:**
- *Preconditions:* Client C is the current client; B's page was opened while book B had no available copies and still shows the subscribe action; 2 copies of B have since been received.
- *Steps:* 1. Select the subscribe action on the open page. 2. Open the pending-subscriptions list.
- *Expected result:* The alerts service records no subscription of C to B.

**[TC02]: The book has been removed — Negative / boundary:**
- *Preconditions:* Client C is the current client; B's page shows the subscribe action; book B has since been removed from the catalogue.
- *Steps:* 1. Select the subscribe action. 2. Open the pending-subscriptions list.
- *Expected result:* The alerts service records no subscription of C to B.

**[TC03]: The current client no longer exists — Negative / boundary:**
- *Preconditions:* Client C is the current client and has since been deleted from the store; book B has no available copies.
- *Steps:* 1. Open B's page and select the subscribe action. 2. Open the pending-subscriptions list.
- *Expected result:* The alerts service records no subscription for C's email.

**[TC04]: The store cannot check the stock — Negative / boundary:**
- *Preconditions:* Client C is the current client; B's page was opened while the storage service answered and shows the subscribe action for book B, which has no available copies; the storage service has since stopped answering.
- *Steps:* 1. Select the subscribe action on the open page. 2. Restore the storage service and open the pending-subscriptions list.
- *Expected result:* The alerts service records no subscription of C to B.

#### [AC08]: Say why a subscription was not made

If a subscription is not made, then the web store shall show on the book's page that no subscription was made and which reason applied: the book has copies available now, the book or the current client no longer exists, or the store could not check and the client may try again ([AD#8]).

##### Test cases

**[TC01]: Copies are available now — Happy path:**
- *Preconditions:* Client C is the current client; B's page shows the subscribe action; 2 copies of book B have since been received.
- *Steps:* 1. Select the subscribe action.
- *Expected result:* B's page shows that no subscription was made because the book has copies available now.

**[TC02]: The store could not check — Negative / boundary:**
- *Preconditions:* Client C is the current client; B's page was opened while the storage service answered and shows the subscribe action for book B, which has no available copies; the storage service has since stopped answering.
- *Steps:* 1. Select the subscribe action on the open page.
- *Expected result:* B's page shows that no subscription was made because the store could not check, and that the client may try again.

**[TC03]: The book is no longer offered — Negative / boundary:**
- *Preconditions:* Client C is the current client; B's page shows the subscribe action; book B has since been removed from the catalogue.
- *Steps:* 1. Select the subscribe action.
- *Expected result:* B's page shows that no subscription was made because the book or the current client no longer exists.

**[TC04]: The current client no longer exists — Negative / boundary:**
- *Preconditions:* Client C is the current client and has since been deleted from the store; book B has no available copies.
- *Steps:* 1. Open B's page and select the subscribe action.
- *Expected result:* B's page shows that no subscription was made because the book or the current client no longer exists.

#### [AC09]: Say when subscriptions are unavailable

If the web store cannot read a book's stock, or cannot read the current client's subscription to a book with no available copies, then the web store shall show on the book's page that subscriptions are unavailable right now, in place of the subscribe action and the subscribed state ([AD#8]).

##### Test cases

**[TC01]: Stock cannot be read — Happy path:**
- *Preconditions:* Client C is the current client; book B has no available copies; the storage service does not answer.
- *Steps:* 1. Open B's page.
- *Expected result:* B's page shows that subscriptions are unavailable right now, in place of the subscribe action and the subscribed state.

**[TC02]: The subscription cannot be read — Negative / boundary:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies; the alerts service does not answer.
- *Steps:* 1. Open B's page.
- *Expected result:* B's page shows that subscriptions are unavailable right now, in place of the subscribe action and the subscribed state.

**[TC03]: A book with copies needs no subscription read — Negative / boundary:**
- *Preconditions:* Client C is the current client; book B has 2 available copies; the alerts service does not answer.
- *Steps:* 1. Open B's page.
- *Expected result:* B's page shows no notice that subscriptions are unavailable, since B has available copies and no subscription needs to be read.

#### [AC10]: Leave existing journeys unchanged

The web store shall keep browsing books, using the cart and placing orders behaving as before this feature, apart from the subscribe action, the subscribed state, the current-client display, the unread-alert count, the ways into the alerts list and the pending-subscriptions list, and the notices that subscriptions or alerts are unavailable.

##### Test cases

**[TC01]: Browse, cart and order with no current client — Happy path:**
- *Preconditions:* No current client is selected; book B has available copies; client C exists. The comparison uses the same catalogue data, leaving out the synthetic journey's books.
- *Steps:* 1. Open the books list and B's page. 2. Add B to a cart for C. 3. Place and pay the order.
- *Expected result:* Each page, action and result is the same as the same steps on the release before this feature.

**[TC02]: Browse, cart and order while the alerts service is down — Negative / boundary:**
- *Preconditions:* Client C is the current client; the alerts service does not answer; book B has available copies. The comparison uses the same catalogue data, leaving out the synthetic journey's books.
- *Steps:* 1. Open the books list and B's page. 2. Add B to a cart for C. 3. Place and pay the order.
- *Expected result:* Each page, action and result is the same as on the release before this feature, apart from the current-client display, the absent unread-alert count and the ways into the alerts list and the pending-subscriptions list.

#### [AC11]: Do not slow the store

While the same synthetic load runs against the same store configuration, the books, carts, orders and storage services shall keep the 95th-percentile response time of the browse, cart and order journeys of clients holding no subscription within 10% of a baseline run without this feature.

##### Test cases

**[TC01]: Same-load comparison — Happy path:**
- *Preconditions:* A 1-hour baseline run of the synthetic load on the release before this feature has recorded the 95th-percentile response time of the browse, cart and order journeys; this feature is deployed with the same store configuration.
- *Steps:* 1. Run the same synthetic load for 1 hour. 2. Compute the 95th-percentile response time of the browse, cart and order journeys of clients holding no subscription. 3. Compare each with the baseline.
- *Expected result:* Each 95th-percentile response time is within 10% of the baseline.

**[TC02]: Comparison with 1,000 subscribers and the journey running — Negative / boundary:**
- *Preconditions:* A 1-hour baseline run of the synthetic load on the release before this feature has recorded the 95th-percentile response time of the browse, cart and order journeys; this feature is deployed with the same store configuration; 1,000 clients hold a pending subscription to one book; the back-in-stock journey is running.
- *Steps:* 1. Run the same synthetic load for 1 hour. 2. Compute the 95th-percentile response time of the browse, cart and order journeys of clients holding no subscription. 3. Compare each with the baseline.
- *Expected result:* Each 95th-percentile response time is within 10% of the baseline.

---

### [U02]: Be alerted when the book is back

As a client, I want an alert in the store, and a count of my unread alerts on every page, when a book I subscribed to comes back into stock, so that I can buy it before it sells out again.

#### [AC01]: Alert every subscriber

When a book's available copies go from none to one or more, the alerts service shall record a back-in-stock alert for every client holding a pending subscription to it within one minute, whatever the number of copies and whatever caused the change ([AD#3], [AD#4], [AD#5]).

##### Test cases

**[TC01]: New stock is received — Happy path:**
- *Preconditions:* Clients C1, C2 and C3 each hold a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Receive 1 copy of B into stock. 2. After 60 seconds, open each client's alerts list.
- *Expected result:* The alerts service has recorded a back-in-stock alert for B for each of C1, C2 and C3 within one minute.

**[TC02]: The notice to the alerts service is lost — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a stock of zero; the storage service's notice to the alerts service is blocked.
- *Steps:* 1. Receive 1 copy of B into stock. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded a back-in-stock alert for B for C within one minute.

**[TC03]: A corrected quantity — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Correct B's stock to 5. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded a back-in-stock alert for B for C within one minute.

**[TC04]: A returned order — Happy path:**
- *Preconditions:* Book B's last copy was sold in a completed order; client C holds a pending subscription to B.
- *Steps:* 1. Cancel the order so that its copy returns to stock. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded a back-in-stock alert for B for C within one minute.

**[TC05]: First stock for a book never stocked — Negative / boundary:**
- *Preconditions:* Book B is in the catalogue and the store has never held stock for it; client C holds a pending subscription to B.
- *Steps:* 1. Receive 2 copies of B into stock. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded a back-in-stock alert for B for C within one minute.

**[TC06]: Removing the stock record is not a return to stock — Negative / boundary:**
- *Preconditions:* Book B has a stock of zero; client C holds a pending subscription to B.
- *Steps:* 1. Delete B's stock record. 2. After 60 seconds, open C's alerts list and pending-subscriptions list.
- *Expected result:* The alerts service has recorded no back-in-stock alert for B, since B's available copies did not go from none to one or more, and C's subscription is still pending.

#### [AC02]: Alert 1,000 subscribers in time

While up to 1,000 clients hold a pending subscription to one book, the alerts service shall record every one of their back-in-stock alerts within one minute of the book's available copies going from none to one or more ([AD#4], [AD#5]).

##### Test cases

**[TC01]: 1,000 subscribers — Happy path:**
- *Preconditions:* 1,000 clients each hold a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Receive 1 copy of B into stock. 2. After 60 seconds, count the back-in-stock alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 back-in-stock alerts within one minute.

**[TC02]: 1,000 subscribers with the notice lost — Negative / boundary:**
- *Preconditions:* 1,000 clients each hold a pending subscription to book B, which has a stock of zero; the storage service's notice to the alerts service is blocked.
- *Steps:* 1. Receive 1 copy of B into stock. 2. After 60 seconds, count the back-in-stock alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 back-in-stock alerts within one minute.

#### [AC03]: Alert only on a return from none

If a book that already has one or more available copies gains more, then the alerts service shall record no back-in-stock alert ([AD#3]).

##### Test cases

**[TC01]: More copies of a book in stock — Happy path:**
- *Preconditions:* Book B has 1 available copy; client C has one back-in-stock alert for B, from B's return to stock.
- *Steps:* 1. Receive 5 more copies of B. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded no further back-in-stock alert; C's list still shows one alert for B.

**[TC02]: The smallest increase — Negative / boundary:**
- *Preconditions:* Book B has exactly 1 available copy; client C has one back-in-stock alert for B, from B's return to stock.
- *Steps:* 1. Receive 1 more copy of B. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded no further back-in-stock alert.

#### [AC04]: End the fulfilled subscription

When the alerts service records a back-in-stock alert for a subscription, the alerts service shall end that subscription, so that a later return of the same book to stock produces no further alert from it ([AD#4], [AD#5]).

##### Test cases

**[TC01]: A second return produces no second alert — Happy path:**
- *Preconditions:* Client C held a pending subscription to book B and received a back-in-stock alert when B returned to stock.
- *Steps:* 1. Sell every copy of B. 2. Receive 1 copy of B. 3. After 60 seconds, open C's alerts list.
- *Expected result:* C's alerts list shows exactly one back-in-stock alert for B; the alerts service ended the subscription with the first.

**[TC02]: A new subscription after the alert alerts again — Negative / boundary:**
- *Preconditions:* Client C received a back-in-stock alert for book B; every copy of B has since been sold.
- *Steps:* 1. Subscribe C to B again from B's page. 2. Receive 1 copy of B. 3. After 60 seconds, open C's alerts list.
- *Expected result:* C's alerts list shows two back-in-stock alerts for B, one from each subscription; the ended subscription produced no further alert.

#### [AC05]: Show the alert

The web store shall show each back-in-stock alert in the current client's alerts list, newest first, with the book's title, the date and time the store recorded the book's return to stock and a link to the book's page, and with no number of available copies ([AD#8]).

##### Test cases

**[TC01]: The alert's fields — Happy path:**
- *Preconditions:* Client C is the current client and has a back-in-stock alert for book B, whose return to stock the store recorded on 2026-10-07 at 10:05.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The alert shows B's title, 2026-10-07 10:05 as when the store recorded B's return to stock and a link to B's page, and shows no number of available copies.

**[TC02]: A large return shows no count and the link opens the book — Negative / boundary:**
- *Preconditions:* Client C is the current client and has a back-in-stock alert for book B, which came back with 50 copies.
- *Steps:* 1. Open the alerts list. 2. Select the alert's link.
- *Expected result:* The alert shows no number of available copies, and the link opens B's page.

**[TC03]: Newest first — Happy path:**
- *Preconditions:* Client C is the current client and has back-in-stock alerts for book B1, whose return to stock the store recorded at 09:00, and book B2, whose return it recorded at 10:00.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The alerts list shows the alert for B2 above the alert for B1.

#### [AC06]: Show the unread count

While the current client has one or more unread alerts, the web store shall show the number of unread alerts on every page ([AD#8]).

##### Test cases

**[TC01]: The count on every page — Happy path:**
- *Preconditions:* Client C is the current client and has 2 unread alerts.
- *Steps:* 1. Open the books list, a book's page, the Clients list and the Orders list in turn.
- *Expected result:* Every page shows the number of unread alerts as 2.

**[TC02]: A count of one — Negative / boundary:**
- *Preconditions:* Client C is the current client and has exactly 1 unread alert.
- *Steps:* 1. Open the books list and a book's page.
- *Expected result:* Both pages show the number of unread alerts as 1.

#### [AC07]: Update the count on an open page

While a store page stays open with a current client selected, the web store shall include an alert in the unread-alert count within one minute of the return to stock or the removal that caused it ([AD#9]).

##### Test cases

**[TC01]: A return to stock on an open page — Happy path:**
- *Preconditions:* Client C is the current client with no unread alert and holds a pending subscription to book B, which has a stock of zero; the books list is open.
- *Steps:* 1. Receive 1 copy of B without reloading the open page. 2. Watch the open page for 60 seconds.
- *Expected result:* The unread-alert count on the open page includes the new alert within one minute of B's return to stock.

**[TC02]: The notice is lost while the page is open — Negative / boundary:**
- *Preconditions:* Client C is the current client with no unread alert and holds a pending subscription to book B, which has a stock of zero; the books list is open; the storage service's notice to the alerts service is blocked.
- *Steps:* 1. Receive 1 copy of B without reloading the open page. 2. Watch the open page for 60 seconds.
- *Expected result:* The unread-alert count on the open page includes the new alert within one minute of B's return to stock.

**[TC03]: A removal on an open page — Happy path:**
- *Preconditions:* Client C is the current client with no unread alert and holds a pending subscription to book B; the books list is open.
- *Steps:* 1. Remove B from the catalogue on its own without reloading the open page. 2. Watch the open page for 60 seconds.
- *Expected result:* The unread-alert count on the open page includes the withdrawal alert within one minute of B's removal.

#### [AC08]: Clear the count on opening the list

When the current client opens the alerts list, the web store shall clear the unread-alert count ([AD#8]).

##### Test cases

**[TC01]: Opening the list clears the count — Happy path:**
- *Preconditions:* Client C is the current client and has 3 unread alerts.
- *Steps:* 1. Open the alerts list. 2. Open the books list.
- *Expected result:* The web store shows no unread-alert count; the count was cleared when the alerts list was opened.

**[TC02]: A later alert counts again — Negative / boundary:**
- *Preconditions:* Client C is the current client, has opened the alerts list and has no unread alert; C holds a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Receive 1 copy of B. 2. After 60 seconds, open the books list.
- *Expected result:* The unread-alert count shows 1.

#### [AC09]: Show no count at zero

While the current client has no unread alert, or no current client is selected, the web store shall show no unread-alert count ([AD#8]).

##### Test cases

**[TC01]: Every alert read — Happy path:**
- *Preconditions:* Client C is the current client; all of C's alerts are read.
- *Steps:* 1. Open the books list and a book's page.
- *Expected result:* The web store shows no unread-alert count on either page.

**[TC02]: No current client — Negative / boundary:**
- *Preconditions:* Client C has 2 unread alerts; no current client is selected.
- *Steps:* 1. Open the books list and a book's page.
- *Expected result:* The web store shows no unread-alert count on either page.

#### [AC10]: Hide the count when alerts are unavailable

If the alerts service does not answer, then the web store shall show no unread-alert count ([AD#8]).

##### Test cases

**[TC01]: Alerts unavailable when a page opens — Happy path:**
- *Preconditions:* Client C is the current client and has 2 unread alerts; the alerts service does not answer.
- *Steps:* 1. Open the books list.
- *Expected result:* The web store shows no unread-alert count.

**[TC02]: Alerts stop answering while a page is open — Negative / boundary:**
- *Preconditions:* Client C is the current client and has 2 unread alerts; the books list is open and shows a count of 2.
- *Steps:* 1. Stop the alerts service. 2. Watch the open page for 60 seconds.
- *Expected result:* The web store shows no unread-alert count once the alerts service does not answer.

#### [AC11]: Say when the alerts list is unavailable

If the alerts service does not answer when the web store reads the current client's alerts list, then the web store shall show that the alerts list is unavailable right now and that the client may try again later, in place of the alerts list ([AD#8]).

##### Test cases

**[TC01]: Open the alerts list while alerts are unavailable — Happy path:**
- *Preconditions:* Client C is the current client; the alerts service does not answer.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store shows that the alerts list is unavailable right now and that the client may try again later, in place of the alerts list.

**[TC02]: Alerts stop answering after the list was shown — Negative / boundary:**
- *Preconditions:* Client C is the current client and has 2 alerts; the alerts list is open and shows them.
- *Steps:* 1. Stop the alerts service. 2. Reload the alerts list.
- *Expected result:* The web store shows that the alerts list is unavailable right now and that the client may try again later, in place of the alerts list.

#### [AC12]: Complete the stock change regardless

If the alerts service does not answer when a book returns to stock, then the storage service shall complete the stock change exactly as it does without this feature ([AD#4]).

##### Test cases

**[TC01]: The alerts service is down — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a stock of zero; the alerts service is stopped.
- *Steps:* 1. Receive 2 copies of B into stock. 2. Read B's stock.
- *Expected result:* The storage service completes the stock change exactly as it does without this feature: the request succeeds and B's stock is 2.

**[TC02]: The alerts service answers slowly — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a stock of zero; the alerts service takes 30 seconds to answer.
- *Steps:* 1. Receive 2 copies of B into stock and time the request.
- *Expected result:* The storage service completes the stock change exactly as it does without this feature, with no added delay; B's stock is 2.

---

### [U03]: Choose the current client

As a client using the store, I want to make myself the current client from the Clients pages and have the browser remember it, so that every page shows my alerts and offers me subscriptions without my choosing again.

#### [AC01]: Make a client current

When someone selects the make-current-client action for a client in the Clients list or on that client's details page, the web store shall make that client the current client.

##### Test cases

**[TC01]: From the Clients list — Happy path:**
- *Preconditions:* Clients C1 and C2 exist; no current client is selected.
- *Steps:* 1. Open the Clients list. 2. Select the make-current-client action for C1.
- *Expected result:* The web store makes C1 the current client.

**[TC02]: From the client's details page — Happy path:**
- *Preconditions:* Client C exists; no current client is selected.
- *Steps:* 1. Open C's details page. 2. Select the make-current-client action.
- *Expected result:* The web store makes C the current client.

**[TC03]: An email with a plus sign and capitals — Negative / boundary:**
- *Preconditions:* Client C's email is First.Last+Books@Example.com; no current client is selected.
- *Steps:* 1. Open the Clients list. 2. Select the make-current-client action for C.
- *Expected result:* The web store makes C the current client, identified by exactly that email.

#### [AC02]: Show the current client

While a current client is selected, the web store shall show the current client's email on every page.

##### Test cases

**[TC01]: The email on every page — Happy path:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Open the books list, a book's page, the Carts list, the Storage list and the Orders list in turn.
- *Expected result:* Every page shows C's email as the current client.

**[TC02]: No email when no client is current — Negative / boundary:**
- *Preconditions:* No current client is selected.
- *Steps:* 1. Open the books list and a book's page.
- *Expected result:* Neither page shows a current client's email.

#### [AC03]: Remember the current client

While a current client is selected, the web store shall keep that client current across page reloads, new tabs and restarts of the same browser until it is changed or cleared.

##### Test cases

**[TC01]: Across a reload — Happy path:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Reload the page.
- *Expected result:* The web store keeps C current.

**[TC02]: In a new tab — State / lifecycle:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Open the store in a new tab of the same browser.
- *Expected result:* The web store keeps C current in the new tab.

**[TC03]: Across a browser restart — State / lifecycle:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Close every window of the browser. 2. Start the browser and open the store.
- *Expected result:* The web store keeps C current.

**[TC04]: Not in another browser — Negative / boundary:**
- *Preconditions:* Client C is the current client in one browser.
- *Steps:* 1. Open the store in a different browser.
- *Expected result:* No current client is selected in the different browser; C stays current in the first.

#### [AC04]: Clear the current client

When someone clears the current client, the web store shall leave no current client selected.

##### Test cases

**[TC01]: Clear the current client — Happy path:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Clear the current client. 2. Open the books list.
- *Expected result:* The web store leaves no current client selected and shows no current client's email.

**[TC02]: Cleared in every tab — Negative / boundary:**
- *Preconditions:* Client C is the current client; the store is open in two tabs.
- *Steps:* 1. Clear the current client in the first tab. 2. Reload the second tab.
- *Expected result:* The web store leaves no current client selected in the second tab.

#### [AC05]: Switch to another client

When a different client is made the current client, the web store shall show that client's unread-alert count, alerts and pending subscriptions in place of the previous client's ([AD#2], [AD#8]).

##### Test cases

**[TC01]: Switch between clients — Happy path:**
- *Preconditions:* Client C1 has 2 unread alerts and 1 pending subscription; client C2 has 1 unread alert and 3 pending subscriptions; C1 is the current client.
- *Steps:* 1. Make C2 the current client. 2. Read the unread-alert count, then open the alerts list and the pending-subscriptions list.
- *Expected result:* The web store shows C2's count of 1, C2's alerts and C2's 3 pending subscriptions in place of C1's.

**[TC02]: None of the previous client's data remains — Security / privacy:**
- *Preconditions:* Client C1 has 2 unread alerts and 1 pending subscription; client C2 has 1 unread alert and 3 pending subscriptions; C1 is the current client.
- *Steps:* 1. Make C2 the current client. 2. Open the alerts list and the pending-subscriptions list.
- *Expected result:* No alert or pending subscription of C1 appears.

**[TC03]: Switch to a client with nothing — Negative / boundary:**
- *Preconditions:* Client C1 has unread alerts and pending subscriptions; client C2 has none; C1 is the current client.
- *Steps:* 1. Make C2 the current client. 2. Open the alerts list and the pending-subscriptions list.
- *Expected result:* The web store shows no unread-alert count and two empty lists, in place of C1's.

#### [AC06]: Prompt when no client is current

While no current client is selected, the web store shall show, in place of the alerts list and the pending-subscriptions list, a prompt to choose a current client from the Clients pages.

##### Test cases

**[TC01]: The alerts list with no current client — Happy path:**
- *Preconditions:* No current client is selected.
- *Steps:* 1. Open the alerts list by its address.
- *Expected result:* The web store shows a prompt to choose a current client from the Clients pages, in place of the alerts list.

**[TC02]: A saved link to the pending list with no current client — Negative / boundary:**
- *Preconditions:* No current client is selected; a link to the pending-subscriptions list was saved earlier.
- *Steps:* 1. Open the saved link.
- *Expected result:* The web store shows a prompt to choose a current client from the Clients pages, in place of the pending-subscriptions list.

#### [AC07]: Offer the ways into both lists

While a current client is selected, the web store shall show a way into the alerts list and a way into the pending-subscriptions list on every page.

##### Test cases

**[TC01]: Ways in on every page — Happy path:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Open the books list, a book's page, the Clients list and the Orders list in turn.
- *Expected result:* Every page shows a way into the alerts list and a way into the pending-subscriptions list.

**[TC02]: Ways in for a client with nothing yet — Negative / boundary:**
- *Preconditions:* Client C is the current client and has no alert and no subscription.
- *Steps:* 1. Open the books list.
- *Expected result:* The page shows a way into the alerts list and a way into the pending-subscriptions list.

#### [AC08]: Hide the ways in when no client is current

While no current client is selected, the web store shall show no way into the alerts list or the pending-subscriptions list.

##### Test cases

**[TC01]: No ways in with no current client — Happy path:**
- *Preconditions:* No current client is selected.
- *Steps:* 1. Open the books list, a book's page and the Clients list in turn.
- *Expected result:* No page shows a way into the alerts list or the pending-subscriptions list.

**[TC02]: No ways in after clearing — Negative / boundary:**
- *Preconditions:* Client C is the current client.
- *Steps:* 1. Clear the current client. 2. Open the books list.
- *Expected result:* The page shows no way into the alerts list or the pending-subscriptions list.

---

### [U04]: See and cancel my pending subscriptions

As a client, I want to see the subscriptions I am still waiting on and cancel any of them, so that I am alerted only about books I still want.

#### [AC01]: List pending subscriptions

The web store shall list every pending subscription of the current client, each with the book's title, linked to the book's page, and the date the subscription was made ([AD#8]).

##### Test cases

**[TC01]: List two subscriptions — Happy path:**
- *Preconditions:* Client C is the current client and holds pending subscriptions to book B1, made on 2026-10-01, and book B2, made on 2026-10-05.
- *Steps:* 1. Open the pending-subscriptions list. 2. Select B1's title.
- *Expected result:* The list shows B1 with 2026-10-01 and B2 with 2026-10-05, each title linked to its book's page; selecting B1's title opens B1's page.

**[TC02]: No pending subscription — Negative / boundary:**
- *Preconditions:* Client C is the current client and holds no pending subscription.
- *Steps:* 1. Open the pending-subscriptions list.
- *Expected result:* The list shows no subscription.

**[TC03]: Another client's subscriptions are not listed — Security / privacy:**
- *Preconditions:* Clients C1 and C2 each hold a pending subscription to a different book; C1 is the current client.
- *Steps:* 1. Open the pending-subscriptions list.
- *Expected result:* The list shows only C1's subscription.

#### [AC02]: Cancel a subscription

When the current client cancels a pending subscription from the pending-subscriptions list or from the book's page, the alerts service shall end that subscription ([AD#8]).

##### Test cases

**[TC01]: Cancel from the list — Happy path:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies.
- *Steps:* 1. Open the pending-subscriptions list. 2. Cancel the subscription to B. 3. Open B's page.
- *Expected result:* The alerts service has ended the subscription: the list no longer shows it and B's page shows the subscribe action.

**[TC02]: Cancel from the book's page — Happy path:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies.
- *Steps:* 1. Open B's page. 2. Cancel the subscription. 3. Open the pending-subscriptions list.
- *Expected result:* The alerts service has ended the subscription, and the list no longer shows it.

**[TC03]: Cancel one of two — Negative / boundary:**
- *Preconditions:* Client C is the current client and holds pending subscriptions to books B1 and B2.
- *Steps:* 1. Cancel the subscription to B1 from the pending-subscriptions list.
- *Expected result:* The alerts service has ended the subscription to B1 only; the subscription to B2 is still pending.

#### [AC03]: No alert after cancelling

When the book of a cancelled subscription later returns to stock, the alerts service shall record no alert from that subscription ([AD#4], [AD#5]).

##### Test cases

**[TC01]: Return after cancelling — Happy path:**
- *Preconditions:* Client C cancelled a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Receive 1 copy of B. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded no alert for C from the cancelled subscription.

**[TC02]: Another subscriber is still alerted — Negative / boundary:**
- *Preconditions:* Client C cancelled a subscription to book B; client D still holds a pending subscription to B, which has a stock of zero.
- *Steps:* 1. Receive 1 copy of B. 2. After 60 seconds, open each client's alerts list.
- *Expected result:* The alerts service has recorded no alert for C from the cancelled subscription, and has recorded D's back-in-stock alert.

#### [AC04]: Drop ended subscriptions from the list

When a subscription ends — by its alert, a withdrawal or a cancellation — or is removed by a bulk reset, the web store shall no longer list it as pending ([AD#4], [AD#6], [AD#7], [AD#8]).

##### Test cases

**[TC01]: Ended by its alert — Happy path:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has a stock of zero.
- *Steps:* 1. Receive 1 copy of B. 2. After 60 seconds, open the pending-subscriptions list.
- *Expected result:* The web store no longer lists the subscription to B as pending.

**[TC02]: Ended by a withdrawal — State / lifecycle:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, open the pending-subscriptions list.
- *Expected result:* The web store no longer lists the subscription to B as pending.

**[TC03]: Ended by a bulk reset — State / lifecycle:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B; neither is part of the synthetic journey.
- *Steps:* 1. Reset the books catalogue in bulk. 2. Open the pending-subscriptions list.
- *Expected result:* The web store no longer lists the subscription to B as pending.

**[TC04]: A subscription that did not end stays listed — Negative / boundary:**
- *Preconditions:* Client C is the current client and holds pending subscriptions to books B1 and B2, both with a stock of zero.
- *Steps:* 1. Receive 1 copy of B1. 2. After 60 seconds, open the pending-subscriptions list.
- *Expected result:* The web store no longer lists the subscription to B1 as pending and still lists the subscription to B2.

#### [AC05]: Cancel an ended subscription without error

If the current client cancels a subscription that has already ended, then the web store shall show it as no longer pending, with no error ([AD#8]).

##### Test cases

**[TC01]: Cancel twice from two tabs — Happy path:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B; the pending-subscriptions list is open in two tabs.
- *Steps:* 1. Cancel the subscription to B in the first tab. 2. Cancel it in the second tab.
- *Expected result:* The second tab shows the subscription as no longer pending, with no error.

**[TC02]: Cancel after the alert ended it — Negative / boundary:**
- *Preconditions:* Client C is the current client; the pending-subscriptions list is open and shows a subscription to book B; B has since returned to stock and C has been alerted.
- *Steps:* 1. Cancel the subscription to B from the open list.
- *Expected result:* The web store shows the subscription as no longer pending, with no error.

#### [AC06]: Say when the pending list is unavailable

If the alerts service does not answer when the web store reads the current client's pending-subscriptions list, then the web store shall show that the pending-subscriptions list is unavailable right now and that the client may try again later, in place of the pending-subscriptions list ([AD#8]).

##### Test cases

**[TC01]: Open the pending list while alerts are unavailable — Happy path:**
- *Preconditions:* Client C is the current client; the alerts service does not answer.
- *Steps:* 1. Open the pending-subscriptions list.
- *Expected result:* The web store shows that the pending-subscriptions list is unavailable right now and that the client may try again later, in place of the pending-subscriptions list.

**[TC02]: Alerts stop answering after the list was shown — Negative / boundary:**
- *Preconditions:* Client C is the current client and holds 2 pending subscriptions; the pending-subscriptions list is open and shows them.
- *Steps:* 1. Stop the alerts service. 2. Reload the pending-subscriptions list.
- *Expected result:* The web store shows that the pending-subscriptions list is unavailable right now and that the client may try again later, in place of the pending-subscriptions list.

#### [AC07]: Say when a cancel could not be done

If the alerts service does not answer a cancel, then the web store shall show that the cancel could not be done, with the subscription still shown as pending ([AD#8]).

##### Test cases

**[TC01]: Cancel from the list with no answer — Happy path:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies; the pending-subscriptions list is open; the alerts service has since stopped answering.
- *Steps:* 1. Cancel the subscription to B.
- *Expected result:* The web store shows that the cancel could not be done, with the subscription to B still shown as pending.

**[TC02]: Cancel from the book's page with no answer — Negative / boundary:**
- *Preconditions:* Client C is the current client and holds a pending subscription to book B, which has no available copies; B's page is open and shows that C is subscribed; the alerts service has since stopped answering.
- *Steps:* 1. Cancel the subscription on B's page.
- *Expected result:* The web store shows that the cancel could not be done, with B's page still showing that C is subscribed.

#### [AC08]: Follow a changed current client on cancel

If a cancel is rejected because the subscription is not the current client's, then the web store shall show the current client's pending subscriptions, with a notice that the current client has changed ([AD#8]).

##### Test cases

**[TC01]: The current client changed in another tab — Happy path:**
- *Preconditions:* Client C1 holds a pending subscription to book B1 and client C2 holds one to book B2; C1 is the current client and the pending-subscriptions list is open in a first tab, showing B1.
- *Steps:* 1. Make C2 the current client in a second tab. 2. In the first tab, cancel the subscription to B1.
- *Expected result:* The first tab shows C2's pending subscriptions, listing B2, with a notice that the current client has changed.

**[TC02]: The other client's subscription is untouched — Security / privacy:**
- *Preconditions:* Client C1 holds a pending subscription to book B1 and client C2 holds one to book B2; C1 is the current client and the pending-subscriptions list is open in a first tab, showing B1; C2 has then been made the current client in a second tab.
- *Steps:* 1. In the first tab, cancel the subscription to B1. 2. Make C1 the current client and open the pending-subscriptions list.
- *Expected result:* The first tab shows C2's pending subscriptions, with a notice that the current client has changed, and C1's subscription to B1 is still listed as pending once C1 is current again.

**[TC03]: The new current client has no subscriptions — Negative / boundary:**
- *Preconditions:* Client C1 holds a pending subscription to book B1 and client C2 holds none; C1 is the current client and the pending-subscriptions list is open in a first tab.
- *Steps:* 1. Make C2 the current client in a second tab. 2. In the first tab, cancel the subscription to B1.
- *Expected result:* The first tab shows C2's pending subscriptions, an empty list, with a notice that the current client has changed.

---

### [U05]: Keep and dismiss my alerts

As a client, I want my alerts kept for 30 days and to dismiss the ones I have dealt with, so that I can come back to an alert I have not acted on and see only those I still need.

#### [AC01]: Keep alerts for 30 days

Unless the current client dismisses it or a bulk catalogue reset removes it, the web store shall list each alert in the current client's alerts list for 30 days from when it was recorded ([AD#8]).

##### Test cases

**[TC01]: An alert just inside 30 days — Happy path:**
- *Preconditions:* Client C is the current client and has an alert recorded 30 days minus 1 minute ago.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store lists the alert.

**[TC02]: A read alert stays listed — Negative / boundary:**
- *Preconditions:* Client C is the current client and has an alert recorded 1 minute ago, which C has already read.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store lists the alert.

#### [AC02]: Age alerts out

While an alert has been recorded for more than 30 days, the web store shall leave it out of the alerts list and the unread-alert count ([AD#8]).

##### Test cases

**[TC01]: An unread alert just past 30 days — Happy path:**
- *Preconditions:* Client C is the current client and has one unread alert, recorded 30 days plus 1 minute ago.
- *Steps:* 1. Open the books list. 2. Open the alerts list.
- *Expected result:* The web store leaves the alert out of the unread-alert count and out of the alerts list.

**[TC02]: Either side of the boundary — Negative / boundary:**
- *Preconditions:* Client C is the current client and has an alert recorded 30 days minus 1 minute ago and another recorded 30 days plus 1 minute ago.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store lists the first alert and leaves out the second.

#### [AC03]: Dismiss an alert

When the current client dismisses an alert, the web store shall leave it out of the alerts list and the unread-alert count from then on ([AD#8]).

##### Test cases

**[TC01]: Dismiss one alert — Happy path:**
- *Preconditions:* Client C is the current client and has 2 alerts.
- *Steps:* 1. Open the alerts list. 2. Dismiss one alert. 3. Reload the alerts list.
- *Expected result:* The web store leaves the dismissed alert out of the alerts list from then on, and still lists the other.

**[TC02]: Dismiss the only alert — Negative / boundary:**
- *Preconditions:* Client C is the current client and has exactly 1 alert.
- *Steps:* 1. Open the alerts list. 2. Dismiss the alert. 3. Open the books list.
- *Expected result:* The alerts list is empty and the web store shows no unread-alert count.

#### [AC04]: Dismiss a dismissed alert without error

If the current client dismisses an alert that is already dismissed, then the web store shall show it as gone, with no error ([AD#8]).

##### Test cases

**[TC01]: Dismiss from two tabs — Happy path:**
- *Preconditions:* Client C is the current client; the alerts list is open in two tabs and shows alert X in both.
- *Steps:* 1. Dismiss X in the first tab. 2. Dismiss X in the second tab.
- *Expected result:* The second tab shows X as gone, with no error.

**[TC02]: Dismiss twice in quick succession — Negative / boundary:**
- *Preconditions:* Client C is the current client; the alerts list shows alert X.
- *Steps:* 1. Select dismiss on X twice in quick succession.
- *Expected result:* The web store shows X as gone, with no error.

#### [AC05]: Say when a dismissal could not be done

If the alerts service does not answer a dismissal, then the web store shall show that the dismissal could not be done, with the alert still listed ([AD#8]).

##### Test cases

**[TC01]: Dismiss with no answer — Happy path:**
- *Preconditions:* Client C is the current client and has 2 alerts; the alerts list is open; the alerts service has since stopped answering.
- *Steps:* 1. Dismiss one alert.
- *Expected result:* The web store shows that the dismissal could not be done, with the alert still listed.

**[TC02]: Dismiss the only alert with no answer — Negative / boundary:**
- *Preconditions:* Client C is the current client and has exactly 1 alert; the alerts list is open; the alerts service has since stopped answering.
- *Steps:* 1. Dismiss the alert.
- *Expected result:* The web store shows that the dismissal could not be done, with the alert still listed.

#### [AC06]: Follow a changed current client on dismissal

If a dismissal is rejected because the alert is not the current client's, then the web store shall show the current client's alerts, with a notice that the current client has changed ([AD#8]).

##### Test cases

**[TC01]: The current client changed in another tab — Happy path:**
- *Preconditions:* Client C1 has alert X1 and client C2 has alert X2; C1 is the current client and the alerts list is open in a first tab, showing X1.
- *Steps:* 1. Make C2 the current client in a second tab. 2. In the first tab, dismiss X1.
- *Expected result:* The first tab shows C2's alerts, listing X2, with a notice that the current client has changed.

**[TC02]: The other client's alert is untouched — Security / privacy:**
- *Preconditions:* Client C1 has alert X1 and client C2 has alert X2; C1 is the current client and the alerts list is open in a first tab, showing X1; C2 has then been made the current client in a second tab.
- *Steps:* 1. In the first tab, dismiss X1. 2. Make C1 the current client and open the alerts list.
- *Expected result:* The first tab shows C2's alerts, with a notice that the current client has changed, and X1 is still listed once C1 is current again.

**[TC03]: The new current client has no alerts — Negative / boundary:**
- *Preconditions:* Client C1 has alert X1 and client C2 has none; C1 is the current client and the alerts list is open in a first tab.
- *Steps:* 1. Make C2 the current client in a second tab. 2. In the first tab, dismiss X1.
- *Expected result:* The first tab shows C2's alerts, an empty list, with a notice that the current client has changed.

---

### [U06]: Be told when a book I wait for is withdrawn

As a client, I want an alert when a book I am waiting for is removed from the catalogue, so that I stop waiting for a book that cannot come back.

#### [AC01]: Send the withdrawal alert

When a single book is removed from the catalogue, the alerts service shall record a withdrawal alert, within one minute, for every client holding a pending subscription to it ([AD#5], [AD#6]).

##### Test cases

**[TC01]: Every waiting client is told — Happy path:**
- *Preconditions:* Clients C1 and C2 each hold a pending subscription to book B.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, open each client's alerts list.
- *Expected result:* The alerts service has recorded a withdrawal alert for B for C1 and C2 within one minute.

**[TC02]: The notice to the alerts service is lost — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B; the books service's notice to the alerts service is blocked.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded a withdrawal alert for B for C within one minute.

**[TC03]: A removed book nobody waits for — Negative / boundary:**
- *Preconditions:* Book B has no pending subscription; client C holds a pending subscription to a different book.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded no withdrawal alert.

#### [AC02]: Alert 1,000 waiting clients in time

While up to 1,000 clients hold a pending subscription to one book, the alerts service shall record every one of their withdrawal alerts within one minute of the book's removal from the catalogue on its own ([AD#5], [AD#6]).

##### Test cases

**[TC01]: 1,000 waiting clients — Happy path:**
- *Preconditions:* 1,000 clients each hold a pending subscription to book B.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, count the withdrawal alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 withdrawal alerts within one minute.

**[TC02]: 1,000 waiting clients with the notice lost — Negative / boundary:**
- *Preconditions:* 1,000 clients each hold a pending subscription to book B; the books service's notice to the alerts service is blocked.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, count the withdrawal alerts for B.
- *Expected result:* The alerts service has recorded every one of the 1,000 withdrawal alerts within one minute.

#### [AC03]: End the withdrawn book's subscriptions

When a single book is removed from the catalogue, the alerts service shall end every pending subscription to it ([AD#5], [AD#6]).

##### Test cases

**[TC01]: Subscriptions end on removal — Happy path:**
- *Preconditions:* Clients C1 and C2 each hold a pending subscription to book B.
- *Steps:* 1. Remove B from the catalogue on its own. 2. After 60 seconds, open each client's pending-subscriptions list.
- *Expected result:* The alerts service has ended every pending subscription to B; neither list shows one.

**[TC02]: A book re-created with the same ISBN does not revive them — Negative / boundary:**
- *Preconditions:* Client C held a pending subscription to book B, which has been removed from the catalogue, and C has received the withdrawal alert.
- *Steps:* 1. Add a book with B's ISBN to the catalogue again. 2. Receive 1 copy of it. 3. After 60 seconds, open C's alerts list.
- *Expected result:* The alerts service has recorded no back-in-stock alert for C from the ended subscription.

#### [AC04]: Show the withdrawal alert

The web store shall show each withdrawal alert with the book's title, the date and time the store recorded the book's removal and a statement that the book is no longer offered, and with no link to the book ([AD#6], [AD#8]).

##### Test cases

**[TC01]: The withdrawal alert's fields — Happy path:**
- *Preconditions:* Client C is the current client and has a withdrawal alert for book B, whose removal the store recorded on 2026-10-07 at 11:20.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The alert shows B's title, 2026-10-07 11:20 as when the store recorded B's removal and that B is no longer offered, and shows no link to B.

**[TC02]: The title shows after the book is gone — Negative / boundary:**
- *Preconditions:* Client C is the current client and has a withdrawal alert for book B; B no longer exists in the catalogue.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The alert shows B's title as it was when C subscribed.

#### [AC05]: List it like any alert

The web store shall list a withdrawal alert in the current client's alerts list exactly as it does a back-in-stock alert ([AD#8]).

##### Test cases

**[TC01]: A withdrawal alert is listed — Happy path:**
- *Preconditions:* Client C is the current client and has one withdrawal alert.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store lists the withdrawal alert exactly as it does a back-in-stock alert.

**[TC02]: Listed newest first among back-in-stock alerts — Negative / boundary:**
- *Preconditions:* Client C is the current client and has a back-in-stock alert recorded at 09:00 and a withdrawal alert recorded at 10:00.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store lists the withdrawal alert above the back-in-stock alert, exactly as it orders back-in-stock alerts.

#### [AC06]: Count it like any alert

The web store shall count an unread withdrawal alert in the unread-alert count exactly as it does a back-in-stock alert ([AD#8]).

##### Test cases

**[TC01]: One unread withdrawal alert — Happy path:**
- *Preconditions:* Client C is the current client and has one unread withdrawal alert and no other unread alert.
- *Steps:* 1. Open the books list.
- *Expected result:* The unread-alert count shows 1.

**[TC02]: Counted together with a back-in-stock alert — Negative / boundary:**
- *Preconditions:* Client C is the current client and has one unread withdrawal alert and one unread back-in-stock alert.
- *Steps:* 1. Open the books list.
- *Expected result:* The unread-alert count shows 2.

#### [AC07]: Keep it like any alert

Unless the current client dismisses it or a bulk catalogue reset removes it, the web store shall keep a withdrawal alert listed for 30 days from when it was recorded, exactly as it does a back-in-stock alert ([AD#8]).

##### Test cases

**[TC01]: Just inside 30 days — Happy path:**
- *Preconditions:* Client C is the current client and has a withdrawal alert recorded 30 days minus 1 minute ago.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store lists the withdrawal alert.

**[TC02]: Just past 30 days — Negative / boundary:**
- *Preconditions:* Client C is the current client and has a withdrawal alert recorded 30 days plus 1 minute ago.
- *Steps:* 1. Open the alerts list.
- *Expected result:* The web store leaves the withdrawal alert out of the list, exactly as a back-in-stock alert.

#### [AC08]: Dismiss it like any alert

When the current client dismisses a withdrawal alert, the web store shall leave it out of the alerts list and the unread-alert count from then on, exactly as it does a back-in-stock alert ([AD#8]).

##### Test cases

**[TC01]: Dismiss a withdrawal alert — Happy path:**
- *Preconditions:* Client C is the current client and has one withdrawal alert and one back-in-stock alert.
- *Steps:* 1. Open the alerts list. 2. Dismiss the withdrawal alert. 3. Reload the alerts list.
- *Expected result:* The web store leaves the withdrawal alert out of the alerts list from then on and still lists the back-in-stock alert.

**[TC02]: Dismiss an unread withdrawal alert — Negative / boundary:**
- *Preconditions:* Client C is the current client and has one unread withdrawal alert, which the open alerts list shows; it arrived after the list was opened.
- *Steps:* 1. Dismiss the withdrawal alert. 2. Open the books list.
- *Expected result:* The web store leaves the withdrawal alert out of the alerts list and the unread-alert count.

#### [AC09]: Ignore unpublishing

If a book is unpublished, then the alerts service shall keep its pending subscriptions pending, with no withdrawal alert ([AD#6]).

##### Test cases

**[TC01]: Unpublishing keeps the subscription — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B, which has no available copies.
- *Steps:* 1. Unpublish B. 2. After 60 seconds, open C's alerts list and pending-subscriptions list.
- *Expected result:* The alerts service keeps C's subscription to B pending, with no withdrawal alert.

**[TC02]: Republished and restocked — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B, which has a stock of zero; B has been unpublished.
- *Steps:* 1. Publish B again. 2. Receive 1 copy of B. 3. After 60 seconds, open C's alerts list.
- *Expected result:* C has a back-in-stock alert for B and no withdrawal alert: the alerts service kept the subscription pending through the unpublishing.

#### [AC10]: Complete the removal regardless

If the alerts service does not answer when a book is removed, then the books service shall complete the removal exactly as it does without this feature ([AD#6]).

##### Test cases

**[TC01]: The alerts service is down — Happy path:**
- *Preconditions:* Client C holds a pending subscription to book B; the alerts service is stopped.
- *Steps:* 1. Remove B from the catalogue on its own. 2. Look B up in the catalogue.
- *Expected result:* The books service completes the removal exactly as it does without this feature: the request succeeds and B is no longer found.

**[TC02]: The alerts service answers slowly — Negative / boundary:**
- *Preconditions:* Client C holds a pending subscription to book B; the alerts service takes 30 seconds to answer.
- *Steps:* 1. Remove B from the catalogue on its own and time the request.
- *Expected result:* The books service completes the removal exactly as it does without this feature, with no added delay.

---

### [U07]: Reset the catalogue without alerting clients

As a demo operator, I want a bulk catalogue reset to clear every client's subscriptions and alerts, other than the synthetic journey's own, and to send no alert for a reset the alerts service is told of, so that resetting the store does not flood clients with alerts about books that were wiped.

#### [AC01]: Clear subscriptions and alerts

When the alerts service is told of a bulk catalogue reset, the alerts service shall remove every subscription and alert other than the synthetic journey's own — those whose book or client is the journey's ([AD#7], [AD#11]).

##### Test cases

**[TC01]: Reset from the API — Happy path:**
- *Preconditions:* Client C, outside the synthetic journey, holds pending subscriptions to books B1 and B2, both outside the journey, and has 3 alerts about books outside the journey.
- *Steps:* 1. Reset the books catalogue in bulk. 2. Read C's subscriptions and alerts from the alerts service.
- *Expected result:* The alerts service has removed every one of C's subscriptions and alerts.

**[TC02]: The journey's own are kept — Negative / boundary:**
- *Preconditions:* A journey client holds a pending subscription to a journey book and has an alert; client C, outside the journey, holds a pending subscription.
- *Steps:* 1. Reset the books catalogue in bulk. 2. Read both clients' subscriptions and alerts from the alerts service.
- *Expected result:* The alerts service has removed C's subscription and kept the journey client's subscription and alert.

**[TC03]: Reset from the web store's Delete All — Happy path:**
- *Preconditions:* Client C, outside the synthetic journey, holds a pending subscription to a book outside the journey and has an alert about a book outside the journey.
- *Steps:* 1. Select Delete All on the web store's books list. 2. Read C's subscriptions and alerts from the alerts service.
- *Expected result:* The alerts service has removed C's subscription and alert.

**[TC04]: A subscription to a journey book is the journey's own — Negative / boundary:**
- *Preconditions:* Client C, outside the synthetic journey, holds a pending subscription to a journey book.
- *Steps:* 1. Reset the books catalogue in bulk. 2. Read C's subscriptions from the alerts service.
- *Expected result:* The alerts service has kept C's subscription, because its book is the journey's.

#### [AC02]: Send no alert

When the alerts service is told of a bulk catalogue reset, the alerts service shall record no alert for it ([AD#7]).

##### Test cases

**[TC01]: Ten waiting clients get nothing — Happy path:**
- *Preconditions:* Ten clients outside the synthetic journey each hold a pending subscription to a different book outside the journey; none has an alert.
- *Steps:* 1. Reset the books catalogue in bulk. 2. After 60 seconds, read every client's alerts.
- *Expected result:* The alerts service has recorded no alert for the reset.

**[TC02]: Nothing arrives after the next sweep either — Negative / boundary:**
- *Preconditions:* Client C, outside the journey, holds a pending subscription to book B, outside the journey.
- *Steps:* 1. Reset the books catalogue in bulk. 2. Wait 2 minutes. 3. Read C's alerts.
- *Expected result:* The alerts service has recorded no alert for the reset.

#### [AC03]: Complete the reset regardless

If the alerts service does not answer when the books catalogue is reset in bulk, then the books service shall complete the reset as it does when the alerts service answers, with no added delay ([AD#7], [AD#11]).

##### Test cases

**[TC01]: The alerts service is down — Happy path:**
- *Preconditions:* The catalogue holds books; the alerts service is stopped.
- *Steps:* 1. Reset the books catalogue in bulk. 2. List the books.
- *Expected result:* The books service completes the reset as it does when the alerts service answers: the request succeeds and only the journey's own books remain.

**[TC02]: The alerts service answers slowly — Negative / boundary:**
- *Preconditions:* The catalogue holds journey books and other books; the alerts service takes 30 seconds to answer.
- *Steps:* 1. Reset the books catalogue in bulk and time the request.
- *Expected result:* The books service completes the reset as it does when the alerts service answers, with no added delay.

#### [AC04]: Withdraw what a missed reset left

If the alerts service missed a bulk reset, then the alerts service shall end each subscription it left behind to a book no longer in the catalogue with a withdrawal alert, as for a single removal ([AD#5], [AD#7]).

##### Test cases

**[TC01]: Leftovers are withdrawn — Happy path:**
- *Preconditions:* Client C, outside the journey, holds a pending subscription to book B, outside the journey; the books service's reset notice to the alerts service is blocked.
- *Steps:* 1. Reset the books catalogue in bulk. 2. After 60 seconds, read C's subscriptions and alerts.
- *Expected result:* The alerts service has ended C's subscription to B with a withdrawal alert, as for a single removal.

**[TC02]: A leftover whose book is back is not withdrawn — Negative / boundary:**
- *Preconditions:* Client C, outside the journey, holds a pending subscription to book B, outside the journey; the books service's reset notice to the alerts service is blocked, and so is the alerts service's lookup of books by ISBN.
- *Steps:* 1. Reset the books catalogue in bulk. 2. Add a book with B's ISBN to the catalogue again. 3. Restore the alerts service's lookup of books by ISBN. 4. After 60 seconds, read C's subscriptions and alerts.
- *Expected result:* The alerts service keeps C's subscription pending, with no withdrawal alert, because its book is in the catalogue again.

---

### [U08]: See the journey running in the demo

As a demo operator, I want synthetic client traffic to subscribe, cancel and receive back-in-stock alerts continuously, and keep doing so through every bulk reset, so that a demo started at any moment shows the journey working.

#### [AC01]: Keep the cadence

While the ingest service runs, the ingest service shall produce at least one subscription, one cancellation and one delivered back-in-stock alert in every 5-minute window ([AD#10]).

##### Test cases

**[TC01]: Six windows without the bulk loop — Happy path:**
- *Preconditions:* The ingest service is running; the bulk data loop is not started.
- *Steps:* 1. Watch the alerts service's records for 30 minutes. 2. Split them into six 5-minute windows.
- *Expected result:* Every window holds at least one subscription, one cancellation and one delivered back-in-stock alert produced by the ingest service.

**[TC02]: Six windows with the bulk loop resetting — Negative / boundary:**
- *Preconditions:* The ingest service is running and the bulk data loop runs continuously, resetting books, clients and stock on every pass.
- *Steps:* 1. Watch the alerts service's records for 30 minutes. 2. Split them into six 5-minute windows.
- *Expected result:* Every window holds at least one subscription, one cancellation and one delivered back-in-stock alert produced by the ingest service.

#### [AC02]: Start with the ingest service

When the ingest service starts, the ingest service shall begin the back-in-stock journey without the bulk data loop being started ([AD#10]).

##### Test cases

**[TC01]: A fresh start — Happy path:**
- *Preconditions:* The ingest service has just been deployed; nobody has started the bulk data loop.
- *Steps:* 1. Start the ingest service. 2. Watch the alerts service's records for 5 minutes.
- *Expected result:* The ingest service has begun the back-in-stock journey: a journey subscription appears within the 5 minutes, with the bulk data loop still not started.

**[TC02]: A restart — Negative / boundary:**
- *Preconditions:* The ingest service is running the journey.
- *Steps:* 1. Restart the ingest service. 2. Watch the alerts service's records for 5 minutes.
- *Expected result:* The ingest service has begun the back-in-stock journey again within the 5 minutes, without the bulk data loop being started.

#### [AC03]: Subscribe only to books with no copies

The ingest service shall subscribe synthetic clients only to books with no available copies ([AD#8]).

##### Test cases

**[TC01]: Every synthetic subscription is to a book with no copies — Happy path:**
- *Preconditions:* The ingest service is running the journey.
- *Steps:* 1. Watch for 15 minutes, recording each synthetic subscription and its book's stock at that moment.
- *Expected result:* The ingest service subscribed synthetic clients only to books with no available copies.

**[TC02]: A journey book given a copy by hand — Negative / boundary:**
- *Preconditions:* The ingest service is running the journey; one journey book has been given 1 copy by hand.
- *Steps:* 1. Watch for 15 minutes, recording each synthetic subscription to that book and its stock at that moment.
- *Expected result:* The ingest service subscribed synthetic clients to that book only while it had no available copies.

#### [AC04]: Alert synthetic clients by the same rules

The alerts service shall alert synthetic clients under the same rules as every other client ([AD#4], [AD#5]).

##### Test cases

**[TC01]: A synthetic alert follows a return from none — Happy path:**
- *Preconditions:* The ingest service is running the journey.
- *Steps:* 1. Watch for 15 minutes, recording each synthetic back-in-stock alert and its book's stock history.
- *Expected result:* Every back-in-stock alert to a synthetic client follows its book's available copies going from none to one or more, under the same rules as every other client.

**[TC02]: A cancelled synthetic subscription gets nothing — Negative / boundary:**
- *Preconditions:* The ingest service is running the journey.
- *Steps:* 1. Watch for 15 minutes, recording each cancelled synthetic subscription and the alerts that follow.
- *Expected result:* No cancelled synthetic subscription produces an alert, under the same rules as every other client.

#### [AC05]: Keep the journey's books

When the books catalogue is reset in bulk, the books service shall keep the synthetic journey's own books ([AD#11]).

##### Test cases

**[TC01]: Reset from the API — Happy path:**
- *Preconditions:* The catalogue holds journey books and other books.
- *Steps:* 1. Reset the books catalogue in bulk. 2. List the books.
- *Expected result:* The books service has kept every journey book and removed every other book.

**[TC02]: Reset from the web store's Delete All — Happy path:**
- *Preconditions:* The catalogue holds journey books and other books.
- *Steps:* 1. Select Delete All on the web store's books list. 2. List the books.
- *Expected result:* The books service has kept every journey book and removed every other book.

**[TC03]: A book whose ISBN starts with seven zeros — Negative / boundary:**
- *Preconditions:* The catalogue holds a journey book and a book whose ISBN starts with seven zeros followed by another digit.
- *Steps:* 1. Reset the books catalogue in bulk. 2. List the books.
- *Expected result:* The books service has kept the journey book and removed the book whose ISBN starts with seven zeros.

#### [AC06]: Keep the journey's clients

When the clients are reset in bulk, the clients service shall keep the synthetic journey's own clients ([AD#11]).

##### Test cases

**[TC01]: Reset from the API — Happy path:**
- *Preconditions:* The store holds journey clients and other clients.
- *Steps:* 1. Reset the clients in bulk. 2. List the clients.
- *Expected result:* The clients service has kept every journey client and removed every other client.

**[TC02]: Reset from the web store's Delete All — Happy path:**
- *Preconditions:* The store holds journey clients and other clients.
- *Steps:* 1. Select Delete All on the web store's Clients list. 2. List the clients.
- *Expected result:* The clients service has kept every journey client and removed every other client.

**[TC03]: An email that only contains the journey's domain — Negative / boundary:**
- *Preconditions:* The store holds a journey client and a client whose email ends in @bis-journey.invalid.example.
- *Steps:* 1. Reset the clients in bulk. 2. List the clients.
- *Expected result:* The clients service has kept the journey client and removed the client whose email ends in @bis-journey.invalid.example.

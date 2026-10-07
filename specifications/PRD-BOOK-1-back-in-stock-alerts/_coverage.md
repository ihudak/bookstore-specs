# Requirement coverage — BOOK-1

_source: prd + PRD-level spec_
**Roll-up: READY — 99/99 requirements covered (100%), 0 gaps**

| Req  | Type      | Text (short) | Covered by                           | Status |
|------|-----------|--------------|--------------------------------------|--------|
| [US#1] | story | Subscribe to an out-of-stock book | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [US#2] | story | Be alerted when the book is back | BOOK-1-01 (new); BOOK-1-02 (new); BOOK-1-05 (new) | ✅ |
| [US#3] | story | Manage my pending subscriptions | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [US#4] | story | Manage my delivered alerts | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [US#5] | story | Be told when a book I wait for is withdrawn | BOOK-1-01 (new); BOOK-1-03 (new); BOOK-1-05 (new) | ✅ |
| [US#6] | story | See the journey running in the demo | BOOK-1-03 (new); BOOK-1-04 (new); BOOK-1-06 (new) | ✅ |
| [AC#1] | criterion | Subscribe offered only on a book with no copies, to a client (current client per [AD#2]) | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#2] | criterion | Book page shows the client is subscribed | BOOK-1-05 (new) | ✅ |
| [AC#3] | criterion | Exactly one pending subscription per client and book | BOOK-1-01 (new) | ✅ |
| [AC#4] | criterion | A pending subscription never lapses on a timer | BOOK-1-01 (new) | ✅ |
| [AC#5] | criterion | (guard) Browse, cart and order unchanged | BOOK-1-02 (new); BOOK-1-03 (new); BOOK-1-05 (new) | ✅ |
| [AC#6] | criterion | Every subscriber alerted within one minute of none to one or more, only then | BOOK-1-01 (new); BOOK-1-02 (new) | ✅ |
| [AC#7] | criterion | Alert names the book, when it came back, links to it, no copy count | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#8] | criterion | Unread count on every page; opening the list clears it | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#9] | criterion | Delivering the alert ends the subscription | BOOK-1-01 (new) | ✅ |
| [AC#10] | criterion | List of pending subscriptions, each naming its book | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#11] | criterion | Cancel from the list or the book's page; no later alert | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#12] | criterion | Alert kept 30 days, then leaves the list (bulk-reset exception per [AD#7]) | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#13] | criterion | Dismiss before 30 days; not counted as unread | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#14] | criterion | Removal ends subscriptions with a withdrawal alert within one minute (bulk-reset exception per [AD#7]) | BOOK-1-01 (new); BOOK-1-03 (new); BOOK-1-05 (new) | ✅ |
| [AC#15] | criterion | Withdrawal alert handled like any alert, no link | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [AC#16] | criterion | Subscription, cancel and delivered alert in every 5-minute window | BOOK-1-03 (new); BOOK-1-04 (new); BOOK-1-06 (new) | ✅ |
| [AC#17] | criterion | Synthetic clients under the same rules | BOOK-1-01 (new); BOOK-1-06 (new) | ✅ |
| [SM#1] | metric | 95% of alerts visible within one minute | BOOK-1-01 (new); BOOK-1-02 (new); BOOK-1-03 (new); BOOK-1-05 (new) | ✅ |
| [SM#2] | metric | 5% of alerted clients order within 7 days (measured from alerts' records and orders; no Epic builds the report) | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [SMC#1] | metric | (counter) p95 of browse, cart and order within 10% | BOOK-1-02 (new); BOOK-1-03 (new); BOOK-1-05 (new) | ✅ |
| [U01] | spec-story | Subscribe to an out-of-stock book | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U01/AC01] | spec-criterion | Record the subscription | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U01/AC02] | spec-criterion | Offer the subscription | BOOK-1-05 (new) | ✅ |
| [U01/AC03] | spec-criterion | Show the subscribed state | BOOK-1-05 (new) | ✅ |
| [U01/AC04] | spec-criterion | Keep one pending subscription per book | BOOK-1-01 (new) | ✅ |
| [U01/AC05] | spec-criterion | Never lapse | BOOK-1-01 (new) | ✅ |
| [U01/AC06] | spec-criterion | No offer otherwise | BOOK-1-05 (new) | ✅ |
| [U01/AC07] | spec-criterion | Record nothing on a failed subscription | BOOK-1-01 (new) | ✅ |
| [U01/AC08] | spec-criterion | Say why a subscription was not made | BOOK-1-05 (new) | ✅ |
| [U01/AC09] | spec-criterion | Say when subscriptions are unavailable | BOOK-1-05 (new) | ✅ |
| [U01/AC10] | spec-criterion | Leave existing journeys unchanged | BOOK-1-05 (new) | ✅ |
| [U01/AC11] | spec-criterion | Do not slow the store | BOOK-1-02 (new); BOOK-1-03 (new) | ✅ |
| [U02] | spec-story | Be alerted when the book is back | BOOK-1-01 (new); BOOK-1-02 (new); BOOK-1-05 (new) | ✅ |
| [U02/AC01] | spec-criterion | Alert every subscriber | BOOK-1-01 (new); BOOK-1-02 (new) | ✅ |
| [U02/AC02] | spec-criterion | Alert 1,000 subscribers in time | BOOK-1-01 (new) | ✅ |
| [U02/AC03] | spec-criterion | Alert only on a return from none | BOOK-1-02 (new) | ✅ |
| [U02/AC04] | spec-criterion | End the fulfilled subscription | BOOK-1-01 (new) | ✅ |
| [U02/AC05] | spec-criterion | Show the alert | BOOK-1-05 (new) | ✅ |
| [U02/AC06] | spec-criterion | Show the unread count | BOOK-1-05 (new) | ✅ |
| [U02/AC07] | spec-criterion | Update the count on an open page | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U02/AC08] | spec-criterion | Clear the count on opening the list | BOOK-1-05 (new) | ✅ |
| [U02/AC09] | spec-criterion | Show no count at zero | BOOK-1-05 (new) | ✅ |
| [U02/AC10] | spec-criterion | Hide the count when alerts are unavailable | BOOK-1-05 (new) | ✅ |
| [U02/AC11] | spec-criterion | Say when the alerts list is unavailable | BOOK-1-05 (new) | ✅ |
| [U02/AC12] | spec-criterion | Complete the stock change regardless | BOOK-1-02 (new) | ✅ |
| [U03] | spec-story | Choose the current client | BOOK-1-05 (new) | ✅ |
| [U03/AC01] | spec-criterion | Make a client current | BOOK-1-05 (new) | ✅ |
| [U03/AC02] | spec-criterion | Show the current client | BOOK-1-05 (new) | ✅ |
| [U03/AC03] | spec-criterion | Remember the current client | BOOK-1-05 (new) | ✅ |
| [U03/AC04] | spec-criterion | Clear the current client | BOOK-1-05 (new) | ✅ |
| [U03/AC05] | spec-criterion | Switch to another client | BOOK-1-05 (new) | ✅ |
| [U03/AC06] | spec-criterion | Prompt when no client is current | BOOK-1-05 (new) | ✅ |
| [U03/AC07] | spec-criterion | Offer the ways into both lists | BOOK-1-05 (new) | ✅ |
| [U03/AC08] | spec-criterion | Hide the ways in when no client is current | BOOK-1-05 (new) | ✅ |
| [U04] | spec-story | See and cancel my pending subscriptions | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U04/AC01] | spec-criterion | List pending subscriptions | BOOK-1-05 (new) | ✅ |
| [U04/AC02] | spec-criterion | Cancel a subscription | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U04/AC03] | spec-criterion | No alert after cancelling | BOOK-1-01 (new) | ✅ |
| [U04/AC04] | spec-criterion | Drop ended subscriptions from the list | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U04/AC05] | spec-criterion | Cancel an ended subscription without error | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U04/AC06] | spec-criterion | Say when the pending list is unavailable | BOOK-1-05 (new) | ✅ |
| [U04/AC07] | spec-criterion | Say when a cancel could not be done | BOOK-1-05 (new) | ✅ |
| [U04/AC08] | spec-criterion | Follow a changed current client on cancel | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U05] | spec-story | Keep and dismiss my alerts | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U05/AC01] | spec-criterion | Keep alerts for 30 days | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U05/AC02] | spec-criterion | Age alerts out | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U05/AC03] | spec-criterion | Dismiss an alert | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U05/AC04] | spec-criterion | Dismiss a dismissed alert without error | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U05/AC05] | spec-criterion | Say when a dismissal could not be done | BOOK-1-05 (new) | ✅ |
| [U05/AC06] | spec-criterion | Follow a changed current client on dismissal | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U06] | spec-story | Be told when a book I wait for is withdrawn | BOOK-1-01 (new); BOOK-1-03 (new); BOOK-1-05 (new) | ✅ |
| [U06/AC01] | spec-criterion | Send the withdrawal alert | BOOK-1-01 (new); BOOK-1-03 (new) | ✅ |
| [U06/AC02] | spec-criterion | Alert 1,000 waiting clients in time | BOOK-1-01 (new) | ✅ |
| [U06/AC03] | spec-criterion | End the withdrawn book's subscriptions | BOOK-1-01 (new); BOOK-1-03 (new) | ✅ |
| [U06/AC04] | spec-criterion | Show the withdrawal alert | BOOK-1-05 (new) | ✅ |
| [U06/AC05] | spec-criterion | List it like any alert | BOOK-1-05 (new) | ✅ |
| [U06/AC06] | spec-criterion | Count it like any alert | BOOK-1-05 (new) | ✅ |
| [U06/AC07] | spec-criterion | Keep it like any alert | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U06/AC08] | spec-criterion | Dismiss it like any alert | BOOK-1-01 (new); BOOK-1-05 (new) | ✅ |
| [U06/AC09] | spec-criterion | Ignore unpublishing | BOOK-1-01 (new); BOOK-1-03 (new) | ✅ |
| [U06/AC10] | spec-criterion | Complete the removal regardless | BOOK-1-03 (new) | ✅ |
| [U07] | spec-story | Reset the catalogue without alerting clients | BOOK-1-01 (new); BOOK-1-03 (new) | ✅ |
| [U07/AC01] | spec-criterion | Clear subscriptions and alerts | BOOK-1-01 (new); BOOK-1-03 (new) | ✅ |
| [U07/AC02] | spec-criterion | Send no alert | BOOK-1-01 (new) | ✅ |
| [U07/AC03] | spec-criterion | Complete the reset regardless | BOOK-1-03 (new) | ✅ |
| [U07/AC04] | spec-criterion | Withdraw what a missed reset left | BOOK-1-01 (new) | ✅ |
| [U08] | spec-story | See the journey running in the demo | BOOK-1-03 (new); BOOK-1-04 (new); BOOK-1-06 (new) | ✅ |
| [U08/AC01] | spec-criterion | Keep the cadence | BOOK-1-06 (new) | ✅ |
| [U08/AC02] | spec-criterion | Start with the ingest service | BOOK-1-06 (new) | ✅ |
| [U08/AC03] | spec-criterion | Subscribe only to books with no copies | BOOK-1-06 (new) | ✅ |
| [U08/AC04] | spec-criterion | Alert synthetic clients by the same rules | BOOK-1-01 (new); BOOK-1-06 (new) | ✅ |
| [U08/AC05] | spec-criterion | Keep the journey's books | BOOK-1-03 (new) | ✅ |
| [U08/AC06] | spec-criterion | Keep the journey's clients | BOOK-1-04 (new) | ✅ |

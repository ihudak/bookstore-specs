# Glossary — BOOK-1-01 specification

Terms the PRD-level glossary (`../_glossary.md`) defines — *current client*, *available copies*, *removed from the catalogue*, *bulk catalogue reset*, *the synthetic journey's own data* — keep their meaning here. This file adds the terms this Epic's specification uses for the alerts service's own interface ([AD#8], [AD#4], [AD#6], [AD#7]).

- **Subscribe** — `POST /api/v1/alerts/subscriptions?email=&isbn=` ([AD#8]).
- **Read of pending subscriptions** — `GET /api/v1/alerts/subscriptions?email=` ([AD#8]).
- **Cancel** — `DELETE /api/v1/alerts/subscriptions/{id}?email=` ([AD#8]).
- **Read of alerts** — `GET /api/v1/alerts?email=` ([AD#8]).
- **Unread count** — `GET /api/v1/alerts/unread-count?email=`, answering a JSON object with a `count` field ([AD#8]).
- **Mark-read** — `POST /api/v1/alerts/read?email=` ([AD#8]).
- **Dismiss** — `DELETE /api/v1/alerts/{id}?email=` ([AD#8]).
- **Told that an ISBN is available (the available notice)** — `POST /api/v1/alerts/available?isbn=` ([AD#4]).
- **Told that an ISBN was removed (the withdrawn notice)** — `POST /api/v1/alerts/withdrawn?isbn=` ([AD#6]).
- **The reset** — `DELETE /api/v1/alerts/delete-all` ([AD#7]).
- **The alerts service's own check (the sweep)** — the check, at least every 30 seconds and whatever the number of books waited on, of every book with a pending subscription ([AD#5]): the books service first — not-found withdraws the book's subscriptions, no answer leaves them for the next check — and, where the books service knows the book, published or not, the storage service, where one or more copies alerts them as back in stock.
- **Ended subscription** — a subscription that is no longer pending because it was cancelled or because a back-in-stock or withdrawal alert ended it; a bulk reset removes a subscription rather than ending it.
- **Does not answer** — for a call from the alerts service to the clients, books or storage service: no reply within 3 seconds, a refused connection, or any reply other than that service's found or not-found answer (an error or a busy reply included).
- **Reserved** — an ISBN starting with `00000000`, or an email ending with `@bis-journey.invalid` exactly as written; a subscription or alert is reserved when its ISBN or its email is ([AD#11]). Unlike matching a client, this test does not ignore letter case.
- **occurredAt / createdAt** — on an alert, when the alerts service learned of the return to stock or the removal (a notice's arrival, or its own check's observation), and when it recorded the alert.

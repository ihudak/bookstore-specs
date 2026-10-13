# Design glossary: BOOK-2-01-01

Terms the design settles beyond the specification's `_glossary.md` (this folder) and the PRD-level `../_glossary.md`.

- **Purchase check** — the one call the ratings service makes to orders, `GET /api/v1/orders/findByEmail?email=<EMAIL>`, before saving a give or a change, with the e-mail exactly as the give or change carries it (or the stored e-mail for a change that carries none) ([AD#2]). It is a read with no side effect.
- **Qualifying order** — as the specification's glossary defines it: an order in orders' `findByEmail` answer whose `isbn` equals the rating's ISBN and whose `completed` is `true`.
- **Orders unavailable** — every purchase-check outcome that is not an answer about the e-mail: a timeout, a connection failure, a status other than `200` or orders' unknown-customer `404`, or a `200` whose body the ratings service cannot read. The give or change is answered `503` and nothing is saved.
- **Unknown-customer 404** — the `404` orders answers from `findByEmail` because `verifyClient` found no client for the e-mail; it is a buyers-only refusal, not orders unavailable.
- **Misrouted 404** — a `404` at the configured orders URL that does not come from orders' `findByEmail` handler: a wrong host or path in ratings' configuration, or a route that does not exist. Orders' unknown-customer `404` and a no-route `404` carry the same Spring default error body save its `path`, so telling them apart is a design decision (`ard.md`, Deferred).
- **Refusal reason** — the reason a buyers-only refusal is logged with, one of: the e-mail is not a customer; the customer holds no order for the book; the customer's order for the book is not completed ([U01] [AC11]). It reaches the log only, never the answer.

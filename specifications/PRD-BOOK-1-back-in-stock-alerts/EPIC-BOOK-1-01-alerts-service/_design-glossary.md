# Design glossary — BOOK-1-01

Terms this design adds on top of the Epic's `_glossary.md` and the PRD's `../_glossary.md`, whose meanings are kept unchanged.

- **email_key** — `email.toLowerCase(Locale.ROOT)`; the column every lookup, uniqueness rule and ownership check matches on. The `email` column keeps the address exactly as given at subscribe, and the reserved-email test of [AD#11] reads `email`, never `email_key`.
- **Subscription status** — `PENDING` (waiting), `CANCELLED` (ended by cancel), `ALERTED` (ended by a back-in-stock alert), `WITHDRAWN` (ended by a withdrawal alert). *Ended* means any status but `PENDING`; `ended_at` is set exactly when the status leaves `PENDING`.
- **Notice** — a call to `/available` or `/withdrawn`; it carries only an ISBN, so its `occurredAt` is the instant alerts received it.
- **Notice recorder** — the in-process operation that ends every pending subscription to one ISBN with one alert type at one instant and records one alert per ended subscription, all or none, in one transaction. Its three callers are the `/available` endpoint, the `/withdrawn` endpoint and the sweep.
- **Sweep pass** — one run of the alerts service's own check over every ISBN that has a pending subscription at the pass's start.
- **Answerable window** — 30 days: an alert is listed, counted and dismissible while `created_at` is within the last 30 days; an ended subscription answers cancel with 204 while `ended_at` is within the last 30 days.

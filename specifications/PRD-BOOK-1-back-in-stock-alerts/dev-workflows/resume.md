# Resume — BOOK-1 / BOOK-1-01 (pe)

- **Last completed:** /product-workflows:specify BOOK-1-01 — command complete (2026-10-08T09:00:28Z)
- **Artifact:** specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01-alerts-service/specification.md (with _session.md, _glossary.md) — on branch spec/BOOK-1-01-alerts-service, PR #6 open
- **Next step:** merge PR #6 (https://github.com/ihudak/bookstore-specs/pull/6), then /dev-workflows:design BOOK-1-01 (Dev); in breadth, /product-workflows:specify BOOK-1-02 for the next Epic in landing order
- **Suggested session name:** BOOK-1-back-in-stock-alerts-pe
- **Carry-forward decisions:**
  - The BOOK-1-01 spec carries 4 open questions for a product or architect ruling: the status every operation answers while alerts' own database is down; an ARD deviation on [AD#8] (occurredAt = when alerts learned; ended subscriptions and alerts answerable for 30 days, then 404) — flag: architect; a response-time limit for the polled reads; whether alerts' configuration entries change its behaviour and time limits.
  - The spec's re-review verdict (PASS WITH RECOMMENDATIONS) was taken before the edit-only recommendations were applied; those edits are listed in its _session.md decision 25.
  - Settled in the BOOK-1-01 spec and binding on its consumers' Epics: emails matched ignoring letter case and `+` carried percent-encoded; subscribe answers 503 when clients, books or storage gives no reply within 3 s, an error or busy; the sweep asks books first. The reserved-email rule stays [AD#11]'s literal suffix.
  - Still open from /epics: alerts' dev and Kubernetes ports (BOOK-1-02, -03, -05 and -06 say `<alerts' dev port>`); whether alerts joins ingest's config/version fan-out; [SM#1]'s measurement source; requests naming the client current at send time.
  - ARD omissions still open (no exists rows for storage→web, clients→web, ingest→books/clients create; ingest's [AD#11] landing order) — /product-workflows:create-ard BOOK-1 refine pending; PRD wording and [AC#12]/[AC#14] bulk-reset exception — /product-workflows:update-prd BOOK-1 pending (follow-ups.md).

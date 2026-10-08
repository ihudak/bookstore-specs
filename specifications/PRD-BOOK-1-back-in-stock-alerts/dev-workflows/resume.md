# Resume — BOOK-1 / BOOK-1-01 (dev)

- **Last completed:** /dev-workflows:design BOOK-1-01 — command complete (2026-10-08T11:26:05Z)
- **Artifact:** specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01-alerts-service/design.md (with the amended specification.md, _design-session.md, _design-glossary.md) — on branch design/BOOK-1-01-alerts-service, PR #7 open
- **Next step:** fix the deferred re-review MAJOR in design.md § Migration (the namespace-wide `kubectl rollout restart deployment -n bookstore` must become `deployment/storage` only), merge PR #7 (https://github.com/ihudak/bookstore-specs/pull/7), then /dev-workflows:implement BOOK-1-01 (Dev); in breadth, /product-workflows:specify BOOK-1-02 for the next Epic in landing order
- **Suggested session name:** BOOK-1-back-in-stock-alerts-dev
- **Carry-forward decisions:**
  - Re-review findings deferred (cap spent), listed in PR #7's body and the session log decision 23: the rollout-restart MAJOR; MINORs — fixedRate catch-up bursts (use a Trigger), assert `NoAnswer.timedOut` in the adapter test, `socketTimeout=5` on the test container URL plus an in-flight-statement test, explicit transactions for every write, sweep scale limit restated (≈ 480 at P ≈ 30 s) with an [AD#5] capacity note for the architect, rollback signal blind to sweep load at launch; NITs — `deployment/alerts` in redeploy.cmd, [U02] AC05 TC04 citation.
  - Alerts' ports are settled for the sibling Epics: Kubernetes service `alerts-svc:91` (`DT_ALERTS_SERVER`), debug 5010, nodePorts 30010 / 32010; dev `localhost:8091`.
  - The BOOK-1-01 spec now carries 3 open questions: the [AD#8] deviation (architect), the proposed [U04] read-latency criterion [ER3] (PM), and the sweep's scale bound [ER12] (PM).
  - Alerts is not added to ingest's `serviceId: all` config fan-out or its version polling — a follow-up for ingest's Epic (BOOK-1-06).
  - ARD omissions and the PRD wording / [AC#12]/[AC#14] bulk-reset exception from earlier runs are still pending (follow-ups.md).

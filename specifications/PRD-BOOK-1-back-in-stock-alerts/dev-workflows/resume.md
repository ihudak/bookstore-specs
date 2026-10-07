# Resume — BOOK-1 (pe)

- **Last completed:** /product-workflows:specify BOOK-1 — command complete (2026-10-07T20:02:24Z)
- **Artifact:** specifications/PRD-BOOK-1-back-in-stock-alerts/specification.md (PR #4, branch spec/BOOK-1-back-in-stock-alerts)
- **Next step:** /product-workflows:epics BOOK-1 (PE) — once the pull request above is merged
- **Suggested session name:** BOOK-1-back-in-stock-alerts-pe
- **Carry-forward decisions:**
  - The spec's last edits were unreviewed: the manual fix notes for the re-review's two BLOCKERs.
  - The re-review left 2 MAJOR findings open:
    - The one-minute promise has no exception for a lost notice whose state reverts before the next sweep.
    - A subscribe the alerts service does not answer is not covered.
  - It also left 13 MINOR and 5 NIT findings open. Four are product-input gaps: observability of the alerts service and the journey, a bound on how many books can be waited on, stopping the journey, and the link of an alert whose book was later removed.
  - The ARD's Contracts table lacks `exists` rows for storage→web (the book page reads stock) and clients→web (the Clients pages).
  - The PRD still says "signed-in client", and still has no bulk-reset exception to [AC#12]/[AC#14]. Both corrections are pending `/product-workflows:update-prd`.

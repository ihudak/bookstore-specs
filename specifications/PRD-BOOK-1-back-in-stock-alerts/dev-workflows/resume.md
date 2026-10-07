# Resume — BOOK-1 (pe)

- **Last completed:** /product-workflows:epics BOOK-1 — command complete (2026-10-07T22:54:11Z)
- **Artifact:** specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01…06-*/epic.md and _coverage.md (uncommitted — /epics never commits its drafts)
- **Next step:** commit the Epic drafts and _coverage.md on a branch and merge a PR to main, then /product-workflows:specify BOOK-1-01 (PE), and the other Epics in landing order (02 storage, 03 books, 04 clients, 05 web, 06 ingest)
- **Suggested session name:** BOOK-1-back-in-stock-alerts-pe
- **Carry-forward decisions:**
  - epic-reviewer left 12 MINOR and 7 NIT findings open, recorded only in the /epics final report. The ones needing a decision are: alerts' dev and Kubernetes ports (BOOK-1-02, -03, -05 and -06 still say `<alerts' dev port>`); whether alerts joins ingest's config/version fan-out; [SM#1]'s measurement source; subscribe's outcome on an error answer from storage, books or clients; requests naming the client current at send time; emails with `+` sent percent-encoded.
  - BOOK-1-05's Delete All captions are the user's decision: books "Delete all books except the demo's own books? This also clears every subscription and alert except those for the demo's own books and clients."; clients "Delete all clients except the demo's own clients?".
  - ARD omissions are still open: no `exists` contract rows for storage→web, clients→web or ingest→books/clients create, and the landing order puts ingest (the [AD#11] producer) after its consumers. A /create-ard BOOK-1 refine is pending.
  - The PRD still says "signed-in client" and has no bulk-reset exception to [AC#12]/[AC#14]. /update-prd BOOK-1 is pending (follow-ups.md).

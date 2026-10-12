# Resume — BOOK-2-01 (pa)

- **Last completed:** /product-workflows:create-ard BOOK-2-01 — command complete (2026-10-12T21:37:30Z)
- **Artifact:** ard.md (PR #21, branch ard/BOOK-2-01-verified-ratings)
- **Next step:** /product-workflows:specify BOOK-2-01 (PE), once PR #21 is merged
- **Suggested session name:** BOOK-2-01-verified-ratings-pa
- **Carry-forward decisions:** ard.md was simplified by hand after the run, to keep the slice to two small Epics (ratings, web), and re-reviewed by ard-reviewer: [AD#2] makes one orders call (`findByEmail`, the e-mail as typed) with no `findByISBN` fallback and no lowercasing; within ratings two e-mails that differ only in capitals are one customer, as its MySQL collation compares them; [AD#5] keeps the per-rating `DELETE` exactly as today (`200`), and the removed `delete-all` falls through to it (`400`); a test environment's `ratings` table is emptied by an operator before the deploy, with no migration tool or start-up check. Also deferred to /specify: the web's wording when ratings answers 503 or a summary request fails.

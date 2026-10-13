# Resume — BOOK-2-01 (pe)

- **Last completed:** /product-workflows:epics BOOK-2-01 — command complete (2026-10-13T11:41:01Z), handed off as specs PR #24 on `epics/BOOK-2-01-verified-ratings`
- **Artifact:** EPIC-BOOK-2-01-01-ratings-service/epic.md, EPIC-BOOK-2-01-02-storefront-ratings/epic.md, _coverage.md (specs PR #24, open)
- **Next step:** merge specs PR #24, then /product-workflows:specify BOOK-2-01-01 and /product-workflows:specify BOOK-2-01-02 (PE), optionally after /product-workflows:create-ard <EPIC> (PA)
- **Suggested session name:** BOOK-2-01-verified-ratings-pe
- **Carry-forward decisions:**
  - Both Epics are kept whole over the 5-criteria / 4-story limit, in the operator's words: "It's a small change, and that's my decision to keep one Epic per module". The ratings Epic has 17 criteria and 6 stories, and the storefront Epic has 20 criteria and 4 stories. bookstore:orders has no Epic.
  - A `400` from ratings carries its reason word for word in the error body's `message` field, the field the storefront's forms already read. Today's code sends no reason there.
  - The last epic-reviewer verdict, the re-review's PASS WITH RECOMMENDATIONS, covers the current text. Ten MINOR and nine NIT findings, all fixable by editing, were listed in PR #24's body. Two MINOR were then fixed by hand, unreviewed: BOOK-2-01-01's comment criterion now reads "a give that carries a comment", so it no longer contradicts the spaces-only comment rule, and BOOK-2-01-02's refusal criterion names the text for each answer (`403`, `503`, `400`, anything else). BOOK-2-01-02's scope also now says that only the read model drops the comment and both forms still send it. Eight MINOR and nine NIT stay open.
  - An uncommitted edit in the bookstore repo changes CLAUDE.md's Angular version from 18 to 21. web/package.json pins @angular/core ^21.1.5.

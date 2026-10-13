# Resume — BOOK-2-01 / BOOK-2-01-01 (dev)

- **Last completed:** /dev-workflows:design BOOK-2-01-01 — command complete (2026-10-13T15:05:08Z), handed off as specs PR #27 on `design/BOOK-2-01-01-ratings-service`
- **Artifact:** EPIC-BOOK-2-01-01-ratings-service/design.md (+ amended specification.md; specs PR #27, open)
- **Next step:** merge specs PR #27, then /dev-workflows:implement BOOK-2-01-01 (Dev). /dev-workflows:design BOOK-2-01-02 (Dev) can run now — its specification (PR #26) is merged.
- **Suggested session name:** BOOK-2-01-verified-ratings-dev
- **Carry-forward decisions:**
  - design-reviewer's last verdict, PASS WITH RECOMMENDATIONS on the re-review, came before three applied edits (`RatingErrorBodiesIT`, an ISBN accepted as a 13-digit JSON integer, the timeout test's isolation); the current text is unreviewed. No finding is left open.
  - specification.md now carries 2 open questions for the PM ([ER1] misrouted orders `404` → `503`; [ER2] only a JSON integer is a whole number of stars); the design proceeds on both and they do not block /implement.
  - For /dev-workflows:design BOOK-2-01-02: only ratings' `400`/`403`/`503` write refusals (and a blank-`isbn` summary `400`) carry `message`; a `PUT` `404` carries Boot's default body with no `message` key. A write waits at most ~4 s on orders (~8 s on the first unconfirmed `404`), inside the storefront's 10 s limit.
  - Release: operators empty the `ratings` table and confirm `uk_ratings_email_isbn` (and `idx_ratings_isbn`) exist before the rules are relied on in an environment; ratings lands before web.

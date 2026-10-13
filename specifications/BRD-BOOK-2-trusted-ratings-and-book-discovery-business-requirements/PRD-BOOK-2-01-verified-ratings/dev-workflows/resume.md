# Resume — BOOK-2-01 / BOOK-2-01-01 (dev)

- **Last completed:** `/dev-workflows:implement BOOK-2-01-01` finished on 2026-10-13T18:05:53Z.
  - Code: PR ihudak/bookstore#13 on `feat/BOOK-2-01-01-buyers-only-ratings-one-per-book`, commit `0a857b5`, pushed.
  - Spec note: specs PR #29 on `spec/BOOK-2-01-01-ratings-service`.
- **Artifact:** `EPIC-BOOK-2-01-01-ratings-service/implementation.md`, plus `follow-ups.md`. `specification.md` gained the [ER8] note, which is in PR #29.
- **Next step:**
  1. `/dev-workflows:conformance BOOK-2-01-01`, once PR #29 is merged, while code PR #13 is still open.
  2. `/dev-workflows:implement BOOK-2-01-02`, the storefront (Dev). Its design (PR #28) is merged.
- **Suggested session name:** BOOK-2-01-verified-ratings-dev
- **Carry-forward decisions:**
  - [U02] [AC05] is deliberately not implemented, at the user's request, as a demo gap.
    - On a change, a blank or absent comment clears the stored comment.
    - It is recorded as an [AD#4] deviation in code PR #13 and as the open [ER8] note, so the spec has 1 open question.
  - Because no answer returns the comment, every storefront change made without retyping the comment clears it.
  - The review was PASS WITH RECOMMENDATIONS. Not applied:
    - a MAJOR rollback-runbook gap (a rollback would expose comments; a follow-up is recorded);
    - a MINOR untested rethrow branch;
    - 2 NITs.
  - Tests: 360/360 green. The verdict came before a one-line test-assertion fix.
  - Landing order: ratings before web in every environment. The release steps are to empty `ratings` and confirm `uk_ratings_email_isbn` exists before the web deploys.

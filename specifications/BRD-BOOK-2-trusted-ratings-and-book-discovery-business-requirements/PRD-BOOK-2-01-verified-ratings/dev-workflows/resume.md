# Resume — BOOK-2-01 / BOOK-2-01-01 (dev)

- **Last completed:** `/dev-workflows:conformance BOOK-2-01-01` (Epic run, Dev) finished on 2026-10-13T18:39:11Z.
  - Code checked: `bookstore` at `feat/BOOK-2-01-01-buyers-only-ratings-one-per-book` @ `7410896` (code PR ihudak/bookstore#13, still open and green).
  - Record: specs PR #30 on `conformance/BOOK-2-01-01-ratings-service`.
- **Artifact:** `EPIC-BOOK-2-01-01-ratings-service/_conformance.md` (in PR #30), plus 6 new lines in `follow-ups.md`.
- **Verdicts:** 77 traced — 71 satisfied · 4 partial · 2 contradicts · 0 missing · 0 not-checkable · 0 unverified.
- **Next step:**
  1. Merge specs PR #30.
  2. `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`. It reads back the 6 fix-code follow-ups ([DD#1]–[DD#6]), all one fix in `Rating.applyChange`. Then re-run `/dev-workflows:conformance BOOK-2-01-01`. PR #13 is still open, so decide first whether the fix lands on its branch or as a follow-up PR after PR #13 merges.
  3. `/dev-workflows:implement BOOK-2-01-02`, the storefront (Dev). Ratings lands before web.
  4. Once every Epic has merged: the PE's `/dev-workflows:conformance BOOK-2-01`.
- **Suggested session name:** BOOK-2-01-verified-ratings-dev
- **Carry-forward decisions:**
  - [U02] [AC05] was left unimplemented on purpose, as the demo gap: a blank comment on a change clears the stored comment.
    - Conformance recorded it as [DD#1]–[DD#6]: [U02]/[AC03], [U02]/[AC05], two `epic.md` bullets, the `epic.md` Contract [AD#4] line and the ARD [AD#4] Rule.
    - Each one is `fix-code`, `proposed`, with the rationale "deliberate demo gap, see [ER8]" (added by hand after the run).
    - The spec's [ER8] note stays open until the fix lands or [AC05] is withdrawn.
  - The re-check dropped the tracer's story-level [U02] `partial`, because the story's "Done when" omits the comment rule.
  - The earlier follow-up is still open: amend `design.md`'s rollback step so that rolling back cannot expose stored comments.
  - Landing order: ratings before web in every environment.

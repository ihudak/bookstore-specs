# /specify session: BOOK-2-01-02

- **Command:** /product-workflows:specify BOOK-2-01-02
- **Started:** 2026-10-13
- **Route:** BRD route through the slice — Epic `BOOK-2-01-02` (target `bookstore:web`) in slice `BOOK-2-01`, parent `BOOK-2`; per-Epic specification
- **Feature folder:** `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/EPIC-BOOK-2-01-02-storefront-ratings/`
- **prd_dir:** `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/`
- **Gates:** PRD `pass` (row A); decision register `pass` (row A); Epic `pass` (row A); ARD `found` (PRD-level `ard.md`, on `main`; no Epic-level ARD)
- **Config:** repo refresh = fetch + pull default branch; repos base = `/workspace`; docs grounding OFF (no markdown under `/workspace/docs`); architecture grounding OFF (`ARCHITECTURE_REPO_PATH` unset); team decisions OFF (no knowledge base yet)
- **Model routing:** MODERATE; authoring `claude-opus-5-5`; detection `sonnet` (family-only harness); review `opus` (spec-reviewer frontmatter pin)

## Inputs read

- `epic.md` — 20 criteria, 4 suggested stories, `kept_whole`; Contract: consumes [AD#4]–[AD#7]; Covers lists the storefront-side PRD-level spec rows
- `prd.md` — live requirements [US#1]–[US#5], [US#7]; [US#6], [AC#22], [AC#25]–[AC#27] withdrawn
- `ard.md` — [AD#1]–[AD#7], none superseded; components `bookstore:ratings`, `bookstore:web`, `bookstore:orders`; landing order orders → ratings → web
- `specification.md` (PRD level) — the storefront-side criteria this Epic carries; its `_session.md` and `_glossary.md` for the texts and decisions already settled
- `decisions.md` — implementation-altitude input: [VD#2] (decided); product and architecture records read for context; no open, reopened or [AS#n] gaps
- `grounding/code-grounding.md` — [CG#1]–[CG#12], each with a verifier outcome; derivation matrix (EXISTS / DERIVED rows)
- `grounding/design-grounding.md` — no [DG#n]; the BRD's book-page wireframe (parent `brd/source/assets/book-page-wireframe.svg`) puts the rating line below the book's details, unlabelled
- `spec-seed.md` — absent in the slice and in the parent
- `dev-workflows/resume.md` — what this Epic consumes from the merged ratings specification (BOOK-2-01-01): the `403` text, the `400` reasons, a `PUT` needing its body id, a give's id disregarded, the `503` rules
- `/epics` PR #24's open findings for this Epic (5 MINOR, 6 NIT), to settle here

## Code scan (Phase 4)

`bookstore` at `master` (fresh pull; 1 local commit ahead of `origin/master`, a CLAUDE.md edit), scanned on `web/`. Status OK. Findings the spec rests on:

- Rating form (`create-rating`): heading "Create Rating"; the stars a free `type="number"` input with no min, max or step; Submit always enabled; no note; on error `alert(error.error.message)` whatever the status; entries kept on error; success opens `/ratings`.
- Update form (`update-rating`): a copy of the create template — heading "Create Rating", e-mail and ISBN editable, the comment pre-filled from the loaded rating; loads by id and PUTs the whole rating, id included, to `/ratings/{id}`; a load failure alerts and leaves the form shown; success opens `/ratings`.
- Ratings list: columns email, ISBN, Ratings, Comment, LastUpdated, Actions; row actions Update, Delete, Details; list-wide New Rating and Delete All; loads with no error handler.
- Rating details: ISBN, e-mail, Rating, Comment, Updated At, Created At; no error handler.
- `RatingService`: list, by id, by e-mail, by ISBN, create, update, `deleteRating`, `deleteAllRatings`; no summary call. `Rating` model `id: number = 0`.
- Book page: labelled fields ISBN, Title, Author, Price, Lang, Vend; loads the book only. Catalogue list: columns ISBN, Title, Author, Price, Lang, OnVend, Actions; loads the books only.
- Tests: Karma + Jasmine; 12 ratings and books specs, each a single "should create"; none provides HttpClient, so they likely fail today (not run).

## Decisions

1. Stories mirror the PRD-level specification's, storefront side only: [U01] rate with the rating form, [U02] change with the update form, [U03] the book page's rating line, [U04] the catalogue list's Rating column, [U05] only Quillmark sees the comment, [U06] no removal from the storefront (user chose the recommended option).
2. Each form shows its outcome text on the form itself, beside Submit, never as a browser alert; selecting Submit clears the previous text, so only the latest outcome shows (user chose the recommended option).
3. A `400` whose error body carries no readable reason (no `message`, an empty one, or a body that is not JSON) is "any other answer": the form shows "We can't save your rating right now. Please try again in a few minutes." (user chose the recommended option).
4. While the summary or the summaries are still loading, the book page's rating line and the catalogue list's Rating cells show nothing; never "Not yet rated" before a successful answer (user chose the recommended option).
5. Every ratings request the storefront makes — summary, summaries, give, change, loading a rating for the update form — gives up after a bounded time (value set by /design) and shows its failure text: "Rating unavailable", the cannot-save text, or the cannot-load text (user chose the recommended option).
6. Submit is held back while a give or change is in flight, until that Submit has its outcome (user chose the recommended option).
7. The update form shows "Leave this empty to keep any comment you gave before." at the comment field, with the comment note (user chose the recommended option).
8. The rating form shows the same text, so both forms read alike (user chose the recommended option).
9. The rating form sends the e-mail and the ISBN as typed; a field holding only spaces counts as empty and holds Submit back like an empty one (user chose the recommended option).
10. Stages 1–2 confirmed as played back, inferred items included: no format check of the e-mail or ISBN in the storefront; raters' e-mails stay visible; criteria and test cases name requests and statuses, the problem statement, scope and stories do not.
11. Stage 3 user stories confirmed: [U01]–[U06] as decision 1; [U06]'s role is the shopper reading ratings.
12. Stage 4 acceptance criteria confirmed as played back (45), inferred items included: the give's and the change's request contents ([U01]/[U02] [AC03]: the change carries the loaded id, e-mail and ISBN); Submit held until the update form has loaded the rating ([U02] [AC15]); a fresh summary request on every load and one summaries request per catalogue load ([U03]/[U04] [AC06]); the rating line below the book's details, unlabelled (BRD wireframe); "readable summary" defined in the glossary; the `/epics` review's storefront findings settled (one `404`-on-change criterion, list and details fields named, [AC#19] as its own criterion, the stored-give criterion worded as unchanged behaviour, freshness limited to the web's part, the update-form criterion not limited to a rating with a comment, the book-page criterion split, Update asserted once).
13. Stage 5 test-case approach confirmed: every criterion has a Happy path and a Negative / boundary case; the storefront is driven against a stubbed ratings service, its answers as preconditions; requests read from the stub; bounded-time tests with a stub that never answers; stubs that answer with a comment, as the old ratings service did; self-contained fixtures from the PRD-level specification.

## Stages

- Stage 1 Header + Problem statement — written (1,178 characters)
- Stage 2 Scope — written
- Stage 3 User stories — written
- Stage 4 Acceptance criteria — written
- Stage 5 Test cases — written
- Phase 5.5 pre-lint — clean after one mechanical fix (a doubled period after closing quotes, 7 lines); no placeholder, headings present, IDs contiguous, every criterion with a Happy path and a Negative / boundary case, open-questions count 0 = 0
- Overlap read before the whole-artifact gate: no pair found giving one situation two outcomes
- Whole-artifact gate confirmed: 6 stories, 45 ACs, 120 TCs, 0 open questions
- Phase 6 spec-reviewer (1st) — BLOCK: 3 BLOCKER, 8 MINOR, 4 NIT
- Fix cycle — decisions 14–20 applied in one pass, read against their overlaps; pre-lint clean; now 6 stories, 49 ACs, 132 TCs, 0 open questions
- Phase 6 spec-reviewer (re-review) — PASS WITH RECOMMENDATIONS: 7 MINOR, 3 NIT, every fix only an edit; cap spent
- Offer to apply them — user chose "Apply them" (decision 21); applied in one pass, read against their overlaps; pre-lint clean after the markup sample's `<b>` was written `<B>` (the lowercase tag matched pre-lint's placeholder pattern); now 6 stories, 51 ACs, 140 TCs, 0 open questions; no finding left open
- **Verdict version:** PASS WITH RECOMMENDATIONS (re-review) was reached before the ten edits of decision 21, which followed it; the current text has not been reviewed.
- Phase 7 — no `consumed_by` write: every grounding finding and register decision this specification draws on ([CG#4]–[CG#10], [VD#2]) was already `consumed_by: specification`; [CG#12] is not drawn on and stays `none`.

## Fix-cycle decisions

14. BLOCKERs (edit only): [U03] [AC01]/[AC03] and [U04] [AC01]/[AC02] apply only to a readable summary or summaries response; [U03] [AC05] and [U04] [AC05] end once the bounded time has passed.
15. MINOR — while the update form's load has no outcome, it shows only its heading "Update Rating", with no field, text or Submit ([U02] [AC15] rewritten) (user chose the recommended option).
16. MINOR — Submit is shown disabled while the e-mail, the ISBN or the star choice is missing; no new text. Applied as well to Submit while a give or change is in flight ([U01] [AC10], [U02] [AC11]), so a held Submit always looks the same (user chose the recommended option).
17. MINOR — the book page when the book itself cannot be loaded stays as today, its rating line included: Out of scope (user chose the recommended option).
18. MINOR — accessibility: the outcome text is announced by a screen reader when it appears, and the star choices can be chosen with the keyboard alone, on both forms; new [U01] [AC12], [AC13], [U02] [AC16], [AC17] (user chose the recommended option).
19. MINOR — the bounded time is at most 10 seconds, its exact value set by /design (user chose "At most 10 seconds", over the recommended "no ceiling"). The ratings service's own wait on orders should end well inside it, or its `503` reaches the form as the cannot-save text: a carry-forward for /dev-workflows:design BOOK-2-01-01.
20. The edit-only findings applied in this cycle (user chose the recommended option): the scope item names the five requests the bounded time covers, and an answer after it is disregarded (scope and glossary), with a late-answer test on each of [U01] [AC08], [U02] [AC09], [U03] [AC04], [U04] [AC04]; never-answered tests wait 10 seconds; [U04] [AC03] covers "Not yet rated" as well as an average; [U01] [AC06] [TC03] uses a reason a give can produce; [U05] [AC03] [TC01]'s read answer carries its e-mail and ISBN; a readable summary's average has at most one decimal place; the In-scope list follows the story order.

## Re-review decisions

21. The re-review's edit-only findings, all applied (user chose "Apply them"): a summaries response with one unreadable entry shows "Rating unavailable" on every row ([U04] [AC04] [TC06]); a late answer to the update form's load is disregarded ([U02] [AC14] [TC05]); [U04] [AC03] reworded to the storefront's part (two agreeing answers shown alike); the bounded time covers "each of these requests", and the Ratings list's and the rating details screen's loads stay as today (scope, glossary); the catalogue list when the books cannot be loaded stays as today, merged with decision 17's Out-of-scope line; Submit enabled once a give or change can be sent (new [U01] [AC14], [U02] [AC18]; [U01] [AC02] [TC01] now asserts a just-opened form's disabled Submit); a `400` reason holding markup shown as plain text ([U01] [AC06] [TC04], [U02] [AC06] [TC03]); a readable reason holds a character other than a space; "exactly" in three `400` expected results; "nothing in any book's Rating cell" in [U04] [AC05].

## Open questions

(none)

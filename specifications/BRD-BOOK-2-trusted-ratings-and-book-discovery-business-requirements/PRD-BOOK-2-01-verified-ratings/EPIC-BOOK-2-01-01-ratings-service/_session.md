# /specify session: BOOK-2-01-01

- **Command:** /product-workflows:specify BOOK-2-01-01
- **Started:** 2026-10-13
- **Route:** BRD route through the slice — Epic `BOOK-2-01-01` (target `bookstore:ratings`) in slice `BOOK-2-01`, parent `BOOK-2`; per-Epic specification
- **Feature folder:** `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/EPIC-BOOK-2-01-01-ratings-service/`
- **prd_dir:** `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/`
- **Gates:** PRD `pass` (row A); decision register `pass` (row A); Epic `pass` (row A); ARD `found` (PRD-level `ard.md`, on `main`; no Epic-level ARD)
- **Config:** repo refresh = fetch + pull default branch; repos base = `/workspace`; docs grounding OFF (no markdown under `/workspace/docs`); architecture grounding OFF (`ARCHITECTURE_REPO_PATH` unset); team decisions OFF (no knowledge base yet)
- **Model routing:** MODERATE; authoring `claude-opus-5-5`; detection `sonnet` (family-only harness); review `opus` (spec-reviewer frontmatter pin)

## Inputs read

- `epic.md` — 17 criteria, 6 suggested stories, `kept_whole`; Contract: produces [AD#4]–[AD#7], consumes [AD#2] (exists); Covers lists the ratings-side PRD-level spec rows
- `prd.md` — live requirements [US#1]–[US#5], [US#7]; [US#6], [AC#22], [AC#25]–[AC#27] withdrawn
- `ard.md` — [AD#1]–[AD#7]; components `bookstore:ratings`, `bookstore:web`, `bookstore:orders`; landing order orders → ratings → web
- `specification.md` (PRD level) — the ratings-side criteria this Epic carries; its `_session.md` and `_glossary.md` for the terms and decisions already settled
- `decisions.md` — implementation-altitude input: [VD#2] (decided); product and architecture records read for context; [CD#2], [CD#3] superseded; no open, reopened or [AS#n] gaps
- `grounding/code-grounding.md` — [CG#1]–[CG#12], each with a verifier outcome; derivation matrix (EXISTS / DERIVED rows)
- `grounding/design-grounding.md` — no [DG#n]
- `spec-seed.md` — absent in the slice and in the parent

## Code scan (Phase 4)

`bookstore` at `master` (fresh pull, not a read-only mount; 1 local commit ahead of `origin/master`), scanned on `ratings/`, with orders' `findByEmail` and the web's error handling read as the contract's other sides. Status OK. Findings the spec rests on:

- No orders client in ratings and no orders URL in any of its four profile files; orders' `findByEmail` exists and answers `404` for an unknown customer.
- Validation lives in the entity setters, which run while the request body is read; their reasons today: "Invalid email address", "ISBN must be a 13-digits value", "Invalid rating. Must be between 0 and 5. Got: n" (wrong range in the text). A missing `rating` stores 0; a missing `email` or `isbn` is not checked.
- `BadRequestException` is a `ResponseStatusException`, no `@ControllerAdvice` exists and `server.error.include-message` is unset, so a reason does not reach the error body's `message`, which the web forms read (`error.error.message`).
- `PUT` saves the request body whole: e-mail, ISBN and comment overwritten, Created At reset. Unknown id `404` "Rating not found"; body id ≠ path id `400` "bad rating id".
- `DELETE /{id}` deletes with no existence check; `delete-all` truncates. The load generator calls `delete-all` (`IngestRepository`, `k8s/ingest.yaml:124`).
- No aggregate query; the comment is serialised on every `Rating` response.
- Tests: one empty `contextLoads` on H2; no stubbing of remote calls anywhere.

## Decisions

1. Stories mirror the PRD-level specification's, ratings side only: [U01] rate, [U02] change, [U03] rating summaries, [U04] comments, [U05] single-rating removal (user chose the recommended option).
2. The buyers-only `403` body is identical for every reason in every field except the time of the answer, and its `message` holds exactly "Only customers who have bought this book can rate it.", naming no reason (user chose the recommended option).
3. The other write `400` reasons: a missing e-mail "Invalid email address"; a missing ISBN "ISBN must be a 13-digits value"; a `PUT` carrying a different e-mail or ISBN "A rating's e-mail and ISBN cannot be changed."; a body id that differs from the path's "bad rating id", as today (user chose the recommended option).
4. A buyers-only refusal's log entry names the reason and the ISBN and not the e-mail; one entry per refusal (user chose the recommended option).
5. A give's comment that is empty, absent or only spaces stores no comment; any other comment is stored as given (user chose the recommended option).
6. Stages 1–2 (header, Problem statement, Scope) confirmed as played back, inferred items included: the orders address set per environment in scope; criteria and test cases name the API's requests and statuses, while the problem statement, scope and stories stay free of them.
7. Stage 3 user stories confirmed: [U01] rate, [U02] change, [U03] know how a book is rated, [U04] comment, [U05] single-rating removal.
8. Stage 4 acceptance criteria confirmed as played back (42), inferred items included: a stored give or saved change answered with the rating's id, e-mail, ISBN, stars, Created At and Updated At; a write with several faults answered with the reason of one of them; a malformed e-mail or ISBN on a change answered with that field's reason rather than the move reason; the unknown-id `404` decided before the body id is compared, as today; the comment field left out of every answer rather than sent empty; reads keep their other fields and failure statuses; a removal with an id that is not a number refused `400`, `delete-all` included.
9. Stage 5 test-case approach confirmed: every criterion has a Happy path and a Negative / boundary case; every test drives the ratings service's API directly, with orders' answers as preconditions (orders stubbed); self-contained fixtures from the PRD-level specification; comments read in the ratings store and refusal entries in the ratings service's log.

## Fix-cycle decisions

10. BLOCKER — In-scope item 9 (the orders address set per environment) dropped as wiring for /design; epic.md's Dependencies and the ARD record it (user chose the recommended option).
11. BLOCKER — check order on a change: faults the body shows on its own ([U02] [AC09], [AC10]) are answered `400` first; then `404` for an unknown id ([AC12]); then the faults that need the stored rating ([AC11], [AC13]); [AC14] covers both branches; [AC12] [TC03] pins it (user chose the recommended option).
12. MAJOR — a give's `id` is disregarded: a give stores or changes only the rating of its own e-mail and ISBN and leaves every other rating as it was; the storefront's rating form sends `id: 0` on every give (`web/src/app/ratings/rating.ts`); new [U01] [AC10], the log criterion renumbered [AC11] (user chose the recommended option).
13. The other findings applied in this cycle (user chose the recommended option): the `403` body identical except the time of the answer and the request path; a `PUT` with no body id answered `400` "bad rating id", as today ([U02] [AC13] [TC03]); the comment-length out-of-scope line states the PRD-level rule; [U02] [AC04] back to "leave exactly one rating"; [U02] [AC15] [TC02] retitled.

## Stages

- Stage 1 Header + Problem statement — written
- Stage 2 Scope — written
- Stage 3 User stories — written
- Stage 4 Acceptance criteria — written
- Stage 5 Test cases — written
- Phase 5.5 pre-lint — clean (no placeholder, headings present, IDs contiguous, every criterion with a Happy path and a Negative / boundary case, open-questions count 0 = 0)
- Overlap read before the whole-artifact gate: [U02] [AC04] now says each racing give after the first is answered as a change ([AC02])
- Whole-artifact gate confirmed: 5 stories, 42 ACs, 129 TCs, 0 open questions; [U05] [AC03]/[AC04] regrouped and [U03] [AC01]/[AC02] split as played back
- Phase 6 spec-reviewer (1st) — BLOCK: 2 BLOCKER, 1 MAJOR, 3 MINOR, 2 NIT
- Fix cycle — decisions 10–13 applied, read against their overlaps; pre-lint clean; now 5 stories, 43 ACs, 134 TCs, 0 open questions
- Phase 6 spec-reviewer (re-review) — BLOCK: 1 BLOCKER (introduced by the fix cycle), 6 MINOR, 2 NIT; cap spent
- Escalation — BLOCKER: user chose "Provide manual fix notes" with the proposed notes (decision 14); the MINOR and NIT findings folded into the same bounded pass (decision 15); one decision taken (decision 16). Applied in one pass, read against its overlaps; [U02] [AC11]'s exclusion list also gained [AC10]. Now 5 stories, 43 ACs, 136 TCs, 0 open questions; no finding left open.
- **Verdict version:** BLOCK (re-review) was reached before the manual fix notes and the folded-in edits, which followed it; the current text has not been reviewed.

## Escalation decisions

14. BLOCKER — a repeat give fell under [U02] [AC12] (`404`) and [AC13] (`400` "bad rating id"): [AC12] and [AC13] now apply only to a `PUT /api/v1/ratings/{id}`, as do [AC14]'s [AC13] branch and [AC15]'s `404`; [AC03] applies a repeat give to its pair's rating whatever `id` it carries or lacks ([U01] [AC10]).
15. Folded-in edits: `403` stated in [U01] [AC04] and [U02] [AC16]; each single-fault criterion scoped to a request with no other fault ([U01] [AC07], [U02] [AC14]); "a non-blank ISBN" in [U03] [AC02]; [U01] [AC09] and [U02] [AC17] aligned with [AD#2] ("any status other than a success or not-found") with the bounded-time clause moved onto orders' answer; racing gives answered as a give or a change, never a server error ([U02] [AC04], [TC01] asserts both `200`); [AD#4]'s "empty comment" and "same values" readings recorded in `_glossary.md`; `isbn` named as a field in [U03] [AC01] and [AC02].
16. An answer from orders that is a success the ratings service cannot read is refused `503`, nothing saved, so a real buyer is never told they did not buy the book; [U01] [AC09] [TC06] and [U02] [AC17] [TC05] added (user chose the recommended option).

## Open questions

(none)

# /specify session: BOOK-2-01

- **Command:** /product-workflows:specify BOOK-2-01
- **Started:** 2026-10-13
- **Route:** BRD route — slice `BOOK-2-01`, parent `BOOK-2`; broad PRD-level specification (no Epics yet; multi-component PRD per the ARD's `components:`)
- **Feature folder / prd_dir:** `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/`
- **Gates:** PRD `pass` (row A); decision register `pass` (row A); ARD `found` (on `main`)
- **Config:** repo refresh = fetch + pull default branch; repos base = `/workspace`; docs grounding OFF (no markdown under `/workspace/docs`); architecture grounding OFF (`ARCHITECTURE_REPO_PATH` unset); team decisions OFF (no knowledge base yet)
- **Model routing:** SIGNIFICANT; authoring `claude-opus-5-5`; detection `claude-sonnet-5-5`; review `claude-opus-5-5`

## Inputs read

- `prd.md` — live requirements [US#1]–[US#5], [US#7]; [US#6] and [AC#22], [AC#25]–[AC#27] withdrawn
- `ard.md` — [AD#1]–[AD#7]; components `bookstore:ratings`, `bookstore:web`, `bookstore:orders`; contracts [AD#2] (exists), [AD#4], [AD#5], [AD#7] (changed), [AD#6] (new); landing order orders → ratings → web; deferred to /specify: the web's text for a `503` on a give or change and for a failed summary request, and the star input's detail ([VD#2])
- `decisions.md` — implementation-altitude input: [VD#2]; product/architecture records read for context; [CD#2], [CD#3] superseded; no open, reopened or [AS#n] gaps
- `grounding/code-grounding.md` — [CG#1]–[CG#12], every one with a verifier outcome; derivation matrix (EXISTS / DERIVED rows only)
- `grounding/design-grounding.md` — no [DG#n] (no frame sets)
- `spec-seed.md` — absent in the slice and in the parent
- Wireframe transcription (`../brd/brd-figures.md`): book page rating line `★ 4.2` `(37 ratings)`

## Decisions

1. Stages 1–2 (header, Problem statement, Scope) confirmed as played back, inferred items included.
2. Stage 3 user stories confirmed: [U01] rate ← [US#1]; [U02] change ← [US#2] (+ [AC#24]); [U03] book page ← [US#3]; [U04] catalogue list ← [US#4]; [U05] comment ← [US#7]; [U06] operator removal ← [US#5]. The any-caller rules ride inside each story's criteria, naming the ratings service and the [AD#N] interface crossed.
3. `503` on a give or change (orders cannot answer, [AD#2]): the rating form and the update form show exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." and stay open with the entries (user chose the recommended wording; ARD deferral settled).
4. A rating summary that cannot be loaded ([AD#6]): the book page shows "Rating unavailable" in place of the rating line, the rest of the book as usual; the catalogue list shows "Rating unavailable" in every book's Rating cell (user chose the recommended option; ARD deferral settled).
5. Star input ([VD#2] detail): the rating form offers five choices, the whole numbers 1 to 5, none chosen when it opens, and cannot be submitted until one is chosen; the update form preselects the rating's current stars (user chose the recommended option; ARD deferral settled).
6. Display format (from the Figure 1 wireframe): book page rating line "★ 4.2 (37 ratings)", "★ 5.0 (1 rating)", the average always with one decimal; the count is plain text; the catalogue list gains a "Rating" column after "OnVend" showing "★ 4.2", "Not yet rated" or "Rating unavailable" (user chose the recommended option).
7. `400` on a give or change: the form shows the ratings service's own reason, as today; the star reason becomes "Invalid rating. Must be a whole number from 1 to 5." (user chose the recommended option).
8. Timestamps on a change (Update or repeat add): Created At stays the moment the rating was first given; Updated At (the Ratings list's LastUpdated) becomes the time of the change; a refused change touches neither (user chose the recommended option; [CG#10] found every update overwriting the creation time).
9. After a successful give or change, repeat add included, the form opens the Ratings list with no extra message, as today (user chose the recommended option).

10. Stage 4 acceptance criteria confirmed as played back, including the inferred "Update Rating" heading, the forms keeping their entries on a refusal, and the invalid-before-purchase-check ordering ([AD#4]).
11. Stage 5 test-case approach confirmed: every AC has a Happy path and a Negative / boundary case; self-contained fixtures; storefront tests and direct ratings-service tests; orders unreachable as a precondition; stored comments read in the ratings store.
12. Whole-artifact gate confirmed: 6 stories, 54 ACs, 140 TCs, 0 open questions; [U02] item 14 split into [AC14]/[AC15]; [U03] [AC02] plural wording; [U03] [AC06] [TC04] added.

## Stages

- Stage 1 Header + Problem statement — written
- Stage 2 Scope — written
- Stage 3 User stories — written
- Stage 4 Acceptance criteria — written
- Stage 5 Test cases — written
- Phase 5.5 pre-lint — clean after one mechanical fix (a `<n>` placeholder in [U03] [AC02])
- Phase 6 spec-reviewer (1st) — BLOCK: 1 BLOCKER, 5 MAJOR, 12 MINOR, 2 NIT
- Fix cycle — user chose to apply every finding in this cycle so the one re-review covers them all; decisions 13–15 below; overlap read done (two criteria scoped to "a rating that exists"); now 6 stories, 61 ACs, 164 TCs, 0 open questions
- Phase 6 spec-reviewer (re-review) — BLOCK: 2 BLOCKER (both introduced by the fix cycle), 1 MAJOR, 7 MINOR, 1 NIT; cap spent
- Escalation — both BLOCKERs: user chose "Provide manual fix notes" and accepted the proposed notes: (1) the purchase check on a change uses the e-mail exactly as the change carries it, or the stored e-mail when it carries none ([AD#2]), letter-case tests name the spelling with no order; (2) new [U02] [AC23] "We can't load this rating right now. Please try again in a few minutes." for a load failure, "opens and loads" triggers, rating details / Ratings list load failures out of scope, [AC15] on open shows "This rating no longer exists." alone (the NIT folded in). Applied in one bounded pass, read against its overlaps.
- Open findings recorded in the spec as `- [ ]` (fix needs a product decision): MAJOR missing e-mail/ISBN reason ([U01] [AC09]); MINOR unavailable-refusal logging ([U01] [AC10]); MINOR whitespace-only comment ([U02] [AC04]); MINOR Updated At on an identical repeat ([U02] [AC06]); MINOR comment length limit (Scope).
- Open findings left for the report and the pull request (fix is only an edit): MINOR "otherwise valid" pointers on [U01] [AC10] / [U02] [AC12] and a ratings-service criterion for a change to a removed rating ([AD#4] `404`); MINOR [U02] [AC03] the losing simultaneous give answered as stored ([AD#3]); MINOR [U06] [AC03] "refuse it as invalid" only for the former remove-all request; MINOR exact reason text in [U01] [AC07] [TC03] and [U02] [AC09] [TC01]–[TC03].
- **Verdict version:** BLOCK (re-review) was reached before the manual fix notes for both BLOCKERs and the five open questions written above, which followed it; the current text has not been reviewed.
- Final: 6 stories, 62 ACs, 166 TCs, 5 open questions.
- Phase 7 — `consumed_by: specification` stamped on [CG#2]–[CG#10] (each was `ARD`; the field holds one value) and [VD#2] (was `none`).

## Fix-cycle decisions

13. BLOCKER — the update form, for a rating that no longer exists (`404` on Submit, or on opening from a stale Ratings list), shows exactly "This rating no longer exists. Nothing was saved."; on Submit it stays open with its entries, on opening it shows the text in place of its fields; a `400` on the update form shows the ratings service's reason, as the rating form does (user chose the recommended option).
14. MAJOR — when Submit cannot reach the ratings service or gets an answer no other criterion covers, both forms show exactly "We can't save your rating right now. Please try again in a few minutes." and stay open with the entries (user chose the recommended option).
15. MINOR (NFR) — response-time and load targets stated out of scope for this release (user chose the recommended option).
16. Edits settled by the ARD: orders hang / server error → unavailable ([AD#2]); letter case of the pair and the stored e-mail ([AD#3], [AD#4]); star values added to test steps; "No deletion from the storefront" and the per-rating removal open to anyone out of scope ([VD#1]); raters' e-mails visible and clearing a stored comment out of scope; [AD#4]/[AD#6] cited on crossing criteria; reason texts and the blank-ISBN summary refusal moved into their criteria; "otherwise valid" on [U01] [AC01] and [U02] [AC01]; Updated At terminology; test-case coverage and labels fixed.

## Open questions

(none)

# Design session: BOOK-2-01-02

- **Command**: /dev-workflows:design BOOK-2-01-02
- **Started**: 2026-10-13
- **Classification**: SIGNIFICANT (Opus session; model gate passed)
- **Specification**: `specification.md` on `main` at `7ab6e2c` (Published: no, Open questions: 0)
- **ARD**: `../ard.md` (PRD-level, found); invariants [AD#1]–[AD#7]; this Epic consumes [AD#4], [AD#5], [AD#6], [AD#7]
- **Architecture grounding**: OFF (ARCHITECTURE_REPO_PATH unset); team decisions OFF (no knowledge base yet)

## Confirmed repo set

- `bookstore` → `/workspace/bookstore` (mounted), target `bookstore:web`, paths `web/`

## Decisions

- **D1 — Bounded time:** one constant, 8 s, on all five ratings requests (summary, summaries, give, change, the update form's load). About 4 s headroom over a write's worst orders wait (1 s + 3 s) and safely under the specification's 10 s, so its "within 10 seconds" test cases do not race the timer. (user: "8 s for all five")
- **D2 — The ratings request-and-outcome interface: hybrid on take C.** Five named `RatingService` methods — `give`, `change`, `loadRating`, `bookLine`, `catalogueLines` — each a cold single-emission observable that never errors and carries the finished screen text; `RatingToEdit` flows from `loadRating` into `change`; B's idea taken: the classifiers (status → write outcome, readable reason, readable rating, readable summary / summaries) are exported pure functions, unit-tested without HTTP; a whole average renders "★ 4.0"; `getRatingById` stays (unchanged) for the rating details screen; the 8 s bound of D1. Candidates: C1 (outcome-typed) and C3 (text-producing) merged into the chosen shape; ~~C2 (raw service + classifiers in each screen)~~ struck — it keeps error logic in four screens. (user: "Hybrid on C")
- **D3 — Star choice:** a shared `StarChoiceComponent` (declared in `AppModule`, `standalone: false`): a `<fieldset>` with a `<legend>` and five native radio inputs 1–5, bound two-way as `[(stars)]`; no `ControlValueAccessor`. Keyboard operation and the 1–5 range come from the browser. Rejected: radios copied into both templates; a `ControlValueAccessor`; custom ARIA star glyphs with hand-written arrow keys. (user: "Shared native-radio component")
- **D4 — Catalogue summaries:** one `GET /summaries` per screen open (`ngOnInit`); the in-place `getBooks()` refreshes after the vend toggle, Delete, Delete All, Vend All and Unvend All reuse that answer, since they change books, not ratings. Recorded as a validation note on [U04] [AC06]. (user: "Once per screen open")
- **D5 — Test strategy:** the four screen specs drive the real `RatingService` behind `HttpTestingController` (`provideHttpClient()` + `provideHttpClientTesting()`), `fakeAsync`/`tick(8000)` for the bounded time; table-driven specs for the exported pure classifiers; a `StarChoiceComponent` spec; the book list and book page smoke specs gain `provideHttpClientTesting()`; keyboard-only and screen-reader behaviour checked by hand in release verification (Karma cannot trigger native arrow-key radio moves or run a screen reader). Rejected: stubbing `RatingService` in the screen specs; a new Playwright e2e suite. (user: "Real service + HttpTestingController")
- **D6 — Rollback signal:** in the 30 minutes after the web deploy, the release operator rolls back the web image (ratings stays) on any one of: (1) ratings `GET /summaries` count 0 while books `GET /api/v1/books` count ≥ 10; (2) ratings `GET /summary` count below 90% of books `GET /api/v1/books/find` count with ≥ 20 book pages; (3) ratings POST+PUT `400` share more than 10 percentage points above its share in the hour before the web deploy; (4) any failed step of the manual post-deploy smoke walk (rated and unrated book page, catalogue column, a non-buyer's refusal text, keyboard-only and screen-reader checks). Signals: Dynatrace service metrics per endpoint for ratings and books (existing); web carries no agent and no real-user monitoring. (user: "Three metric triggers + smoke walk")
- **D7 — Book page when the book fails:** the summary request still goes out in parallel on every load ([U03] [AC06]); the rating line renders only once the book itself has loaded, so a page whose book cannot be loaded stays as today (specification, Out of scope). (user: "Render only after the book loads")
- **Gate 1 (confirmed by the user, inferred items included):** a write is stored on any `200` whatever its body; `400` is invalid only with a string `message` holding a character other than a space, shown untrimmed by interpolation; readable rating = integer `id` equal to the requested one, non-empty string `email` and `isbn`, whole stars 1–5, any comment never read; readable summary = integer `count` ≥ 0 and, above 0, a finite `average` with at most one decimal place; readable summaries = an array of readable summaries each with a string `isbn`, a duplicate ISBN making it unreadable; `isbn` through `HttpParams`; a non-integer route id → cannot-load, no request; `observe: 'response'` + `timeout({first: 8000})`, expiry cancels the request; `takeUntilDestroyed` on every screen subscription; give body `{email, isbn, rating, comment}` untrimmed, `comment` `''` when empty; change body `{id, email, isbn, rating, comment}` with the loaded integer id and the loaded e-mail and ISBN; Submit gate = e-mail and ISBN each hold a character other than a space, a star chosen, nothing in flight; outcome text in a persistent `<p role="alert" aria-atomic="true">` beside Submit, cleared on each Submit; no `alert()`/`console.log`; update form shows only "Update Rating" while loading, e-mail and ISBN as plain text once loaded, comment input empty; rating line `"★ " + average.toFixed(1)` plus `" (1 rating)"`/`" (N ratings)"` on the book page only; `createRating`, `updateRating`, `deleteRating`, `deleteAllRatings` removed, `getRatingById` and the list getters unchanged; `HttpInterceptor` rejected (touches every service, no precedent).
- **D8 — Spec challenges:** four `## Engineering review` notes appended to `specification.md` — [ER1] questioned ([U04] [AC06], "loads" = a screen open), [ER2] questioned ([U02] [AC02]/[AC14], "the rating" = a readable rating), [ER3] validated (screen-reader and keyboard criteria: semantics unit-tested, behaviour checked by hand), [ER4] questioned (Out of scope's failed-book clause: the rating line waits for the book); no new spec `- [ ]` (the specification stays at 0 open questions). (user: "Engineering review notes only")
- **Gate 2 (confirmed by the user, inferred items included):** Error handling table; Test strategy file list; Risks (no landing → "Rating unavailable", stored-but-timed-out writes, summaries growth, exact ISBN join, manual accessibility, no web signal, cached old bundle); Migration (no data/config change, deploy precondition, rollback web only); Observability (Dynatrace per-endpoint metrics for ratings and books, pre-deploy-hour `400` baseline with a 20-write floor, ratio post-release checks, smoke walk); Out of scope; Open questions none.

## Interface candidates

The web's ratings request-and-outcome API (`web/src/app/ratings/rating.service.ts`), shared by the rating form, the update form (load and change), the book page and the catalogue list:

- **C1 — outcome-typed service.** `RatingService` applies the bounded time and never errors: each new or changed method emits one typed outcome (`give`/`change` → stored | buyers-only | invalid(reason) | unavailable | gone | cannot-save; `loadRating` → loaded(rating) | gone | failed; `summary`/`summaries` → readable figures | unavailable). Screens only render the outcome.
- **C2 — raw service + pure classifiers.** `RatingService` keeps raw `HttpClient` observables with the bounded time added; each screen subscribes with an error callback that calls pure helpers (`writeOutcomeText(error, form)`, `readSummary(body)`) to pick its text.
- **C3 — text-producing service.** `RatingService` maps every answer straight to the screen's text (`null` on success), so screens show what it returns.

### Interface fan-out (user chose "Design it three ways"; 3 of 3 takes returned)

- **Take A — minimise the interface.** One overloaded `call({op: 'give'|'change'|'load'|'summary'|'summaries', …})` entry point; outcomes `{kind, text}` (texts inside the service, not exported); `SummariesOutcome.of(isbn)`; single emission, never errors; 10 s timeout cancels the request.
- **Take B — maximise flexibility.** Five methods returning data-only outcome unions (`WriteOutcome`, `LoadOutcome`, `SummaryOutcome`, `SummariesOutcome` with `of(isbn)`); an exported generic `Exchange<T>` + `run()` layer (method/path/body + pure `classify`, `onFailure`, `boundedMs`); texts in a separate pure `rating-texts.ts` (`writeText`, `loadText`, `ratingLine(o, {withCount})`) that screens call; legacy delete methods left in place.
- **Take C — optimise for the most common caller.** Five methods — `give(entry)`, `change(loaded, stars, comment)`, `loadRating(id)`, `bookLine(isbn)`, `catalogueLines()` → `(isbn) => RatingLine` — each emitting one outcome carrying the finished screen text (from `rating-texts.ts` constants); `RatingToEdit {id, email, isbn, stars}` carries the loaded values into the change; readable rating requires an integer id equal to the one requested and stars 1–5; delete methods removed. Flaws noted: claims a whole average renders "4" (the specification says "★ 4.0"), and removes `getRatingById`, which the rating details screen still needs (its load stays as today).

## Reviews

### Review 1 — design-reviewer (Opus): PASS WITH RECOMMENDATIONS

- **MAJOR** — Observability: `GET /api/v1/books/find` and `GET /api/v1/books` are also called server to server by carts (`CartController.java:155-163`), orders (`OrderController.java:233-248`) and storage (`StorageController.java:93-94`, `146-147`), so the [U03]/[U04] ratio checks and rollback triggers (1)–(2) would misfire on a healthy deploy. Verified: in `web/` only `book-details` calls `/find` and only `book-list` calls the books list, so browser-originated requests are a clean denominator.
- **MINOR** — Risks: a not-yet-landed ratings answers `400` (not `404`) to `/summary` and `/summaries` (`@GetMapping("/{id}")` with a `Long` id), so trigger (1) would not fire; the Migration precondition and trigger (4) are the mitigation.
- **MINOR** — trigger (3) has no volume floor on the 30-minute post-deploy window.
- **MINOR** — the smoke walk names no fixture for a rated book and for the keyboard give and change on the live store, and a live change would overwrite a real customer's stars.
- **MINOR** — Requirements coverage: [U02] [AC02] and [U03] [AC06] say "validated" where the specification records [ER2] and [ER4] as questioned on them; [ER4] cited on [U03] [AC05] instead.
- **MINOR** — Risks: no entry for rendering the server's `message` (markup) into the page.
- **NIT** — Seams: the adapters named belong to Angular's `HttpBackend`, one layer below `RatingService`.

**Applied** (user: "Apply them"), each read against what it overlaps:
- Observability: books counts are browser-only (caller not a bookstore service) in Signals, the [U03]/[U04] checks and triggers (1)–(2), with the carts/orders/storage call sites cited; the "no signal of its own" risk now names the browser-originated ratios.
- Risks: not-landed ratings answers `400`; mitigation = the Migration precondition and the live walk's book-page step (trigger (4)); new bullet on rendering the server's `message` by interpolation only.
- Trigger (3) and the matching 24-hour check read only with at least 20 writes.
- D6 revised: the smoke walk split into a **stage walk** (fixture client with a completed order; keyboard give and change, screen-reader check, all screens; gates the live deploy — new Migration bullet "Stage first") and a **live walk** (read-only but for one refused give; trigger (4)). Both refusal steps accept the unavailable text while orders cannot answer, so an orders outage does not roll back a healthy web. Test strategy's by-hand checks moved to the stage walk.
- Requirements coverage: [U02] [AC02] questioned ([ER2]); [U03] [AC05], [AC06] questioned ([ER4]); the specification's [ER4] now names [U03] [AC05] too.
- Seams: the port is Angular's `HttpBackend` (`HttpXhrBackend` / `HttpClientTestingBackend`), `RatingService` the highest seam driven through it.

### Review 2 — design-reviewer (Opus), the cap's one re-review: PASS WITH RECOMMENDATIONS

First-review fixes 2, 5, 6, 7 confirmed; 1, 3, 4 applied but exposing:
- **MAJOR** — triggers (1)–(2) read in the first 30 minutes count old-bundle sessions (old book page calls `/find` but never `/summary`; old catalogue calls `/books` but never `/summaries`) in the browser-only denominators, so a healthy web can be rolled back.
- **MINOR** — the 24-hour `400` check compares against a pre-deploy hour with no floor, and "the hour before the web deploy" may predate the new ratings.
- **MINOR** — ratings' POST can answer `503` on its own, whatever orders does ([[BOOK-2-01-01]] design), so the walks' refused-give steps can fail on a healthy web; the Risks line "no write would ever get `403` or `503`" is wrong about `503`.
- **NIT** — storage calls only `/find`, not both books endpoints.
- **NIT** — the in-place refresh list omits Delete All (and names an "Unvend" single action the list does not have — the single action is the vend toggle).

**Applied** (user: "Apply them"; the cap is spent, so no review follows), each read against what it overlaps:
- Rollback triggers (1)–(2) read over minutes 60–90 after the web deploy, and the 24-hour post-release checks start one hour after it, once old-bundle sessions have ended; the Migration precondition and the live walk stand for (1)–(2) before then; the cached-bundle risk states the one-hour assumption.
- The `400` baseline is taken while the new ratings served the old web (from its start to the web deploy, at most the last hour); with fewer than 20 writes there is no baseline, and trigger (3) and the 24-hour `400` check are not read.
- Walks: a give or change showing the unavailable text is repeated up to three times; three in a row leave the step inconclusive (no rollback, no stage pass); the Risks line now reads "no write would ever get `403`".
- Signals: carts and orders call both books endpoints, storage calls `/find`.
- The refresh list reads "the vend toggle, Delete, Delete All, Vend All and Unvend All" in Data flow, the Error handling row, the specification's [ER1] and D4.

**Version line:** PASS WITH RECOMMENDATIONS (the re-review) was reached before the five edits answering its findings — the ratio triggers and 24-hour checks moved past the first hour; the `400` baseline taken while the new ratings served, with a 20-write floor on the 24-hour check; walks repeating an unavailable answer up to three times; storage's `/find` wording; the refresh list with Delete All — which followed it; the current text has not been reviewed. No finding is left open.

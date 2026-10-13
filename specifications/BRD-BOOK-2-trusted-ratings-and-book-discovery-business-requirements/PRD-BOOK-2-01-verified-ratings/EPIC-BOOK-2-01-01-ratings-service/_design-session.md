# /design session: BOOK-2-01-01

- **Command:** /dev-workflows:design BOOK-2-01-01
- **Started:** 2026-10-13
- **Route:** BRD route through the slice — Epic `BOOK-2-01-01` (target `bookstore:ratings`, paths `ratings`) in slice `BOOK-2-01`, parent `BOOK-2`; per-Epic design
- **Feature folder:** `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/EPIC-BOOK-2-01-01-ratings-service/`
- **Gates:** specification `pass` (row A, on `origin/main` @ 5e3ca3c); ARD `found` (PRD-level `ard.md`, on `main`; no Epic-level ARD) — invariants [AD#1]–[AD#7]
- **Run flags:** all default
- **Config:** feature folder confirmed; repo refresh = fetch + pull default branch; repos base = `/workspace`; architecture grounding OFF (`ARCHITECTURE_REPO_PATH` unset); team decisions OFF (no knowledge base yet)
- **Model routing:** SIGNIFICANT (REST contract change, data integrity, concurrent writes, bounded inter-service call); authoring `claude-opus-5-5`; detection `sonnet` (family-only harness); review `opus` (design-reviewer frontmatter pin); model gate: Opus session → proceed

## Confirmed repo set

- `bookstore` → `/workspace/bookstore` (target `bookstore:ratings`, paths `ratings`) — mounted

## Repository modules (enumerate-components)

`common`, `clients`, `books`, `carts`, `storage`, `orders`, `ratings`, `payments`, `dynapay`, `ingest`, `exceptions` (Gradle includes, code); `web` (code); `k8s`, `.github/workflows` (deploy)

## Code scan (Phase 4)

`code-scanner` (sonnet) on `bookstore` @ `master` (refreshed, clean, ahead 0 / behind 0): status OK. No service layer; validation in `Rating` setters (wrapped by Jackson during binding; null e-mail/ISBN → NPE); no `@ControllerAdvice`, no `server.error.include-message` (Boot default `never`, so reasons never reach `message`); clients built with `new RestTemplate()` (no timeouts, no test seam); every `Rating` setter moves `updatedAt`; H2 test URL has no `MODE`/`IGNORECASE`; the only ratings test is an empty `contextLoads`; existing log lines carry the e-mail (`RatingController.java:79`, `121`, `124`). Orders: `findByEmail` → `verifyClient` 404 for an unknown client (Spring default error body, `message` empty, `path` echoed); orders also serves `GET /api/v1/version` naming itself `orders`; local port 8085. `ClientRepository`/`BookRepository` used only by `RatingController`. Web forms read `error.error.message`.

## Decisions

- **D1 — Layering (user, 2026-10-13):** a new `RatingService` owns give / change / summaries (validation order, purchase check, upsert with race recovery, timestamps); a `PurchaseCheck` port wraps orders; `RatingController` keeps HTTP mapping, request DTO binding and the existing request hooks.

- **D2 — Existence calls (user, 2026-10-13):** drop `verifyClient` / `verifyBook` from the write path; delete `ClientRepository`, `BookRepository` and the `http.service.clients` / `http.service.books` properties in every ratings profile. Unknown e-mail → orders' unknown-customer 404 → identical 403; unknown ISBN → no qualifying order → identical 403 ([AD#4]). Writes depend on orders alone.
- **Fan-out (user, 2026-10-13):** "Design it three ways" chosen for the PurchaseCheck port; takes A/B/C dispatched to `interface-designer` (sonnet), blind.

- **D3 — Binding and validation (user, 2026-10-13):** POST/PUT bodies bind to a `RatingRequest` record (`Long id`, `String email`, `String isbn`, `JsonNode rating`, `String comment`) that never throws while it is read; `RatingService` validates in a fixed order with the specification's exact texts. Stars must be a JSON integer from 1 to 5 — missing, `null`, `0`, `6`, `-1`, `3.5`, `4.0` and `"4"` are all refused "Invalid rating. Must be a whole number from 1 to 5.". The `Rating` entity's setters lose their validation; the entity is never bound from a request.

- **D4 — PurchaseCheck port (user, 2026-10-13, fan-out → hybrid):** A's shape — `Eligibility check(String email, String isbn)` → `ELIGIBLE | NOT_A_CUSTOMER | NO_ORDER_FOR_BOOK | ORDER_NOT_COMPLETED`, throwing one `OrdersUnavailableException`; `RatingService` writes the refusal log entry (ISBN + reason, no e-mail) and throws the 403. From C: `OrdersUnavailableException` extends `ResponseStatusException(SERVICE_UNAVAILABLE)` (ratings-local); a `200` whose elements lack `isbn` or a boolean `completed` is unreadable → 503. From B: only a positive version-probe answer is cached; connect and read timeouts configured separately. A 404 counts as unknown-customer only when its body's `path` is the constant `/api/v1/orders/findByEmail` and `GET /api/v1/version` on the same origin names `orders`; otherwise 503.

- **D5 — Timeouts (user, 2026-10-13):** `ratings.orders.connect-timeout=1s`, `ratings.orders.read-timeout=3s` in `application.properties` (all profiles; overridable by environment); no retry. The version probe uses the same `RestTemplate`.
- **D6 — Error body (user, 2026-10-13):** a ratings-local `@RestControllerAdvice` renders every `ResponseStatusException` as Boot's own shape `{timestamp, status, error, message, path}` with the reason as `message`; any other exception still goes to Boot's default `/error` with `message` empty (`server.error.include-message` stays `never`), so no SQL text or e-mail leaks.
- **D7 — Upsert and race (user, 2026-10-13):** look up the pair (plain equality; the database collation decides letter case); none → insert and flush in its own transaction; on `DataIntegrityViolationException` re-read the pair in a new transaction and, where a row now exists, apply the give to it as a change (one retry); where none exists, rethrow. The purchase check runs before any transaction. Concurrent changes to one existing row are last-write-wins (no `@Version`), which [U02] [AC04] accepts.

- **D8 — Test database (user, 2026-10-13):** the test profile's URL becomes `jdbc:h2:mem:db;DB_CLOSE_DELAY=-1;MODE=MySQL;IGNORECASE=TRUE`, so H2 creates `email` as `VARCHAR_IGNORECASE` and the unique constraint ignores letter case, as MySQL's default collation does in production.
- **Fact — 503 reason:** `OrdersUnavailableException` carries the reason "Service Unavailable"; the storefront shows its own text on any 503 and never the body's `message` (BOOK-2-01-02 specification, lines 180, 459).
- **D9 — Refusal log (user, 2026-10-13):** one INFO entry on the named logger `com.dynatrace.ratings.refusals` — `Rating refused: isbn=<isbn> reason=<NOT_A_CUSTOMER|NO_ORDER_FOR_BOOK|ORDER_NOT_COMPLETED>` — with `logging.level.com.dynatrace.ratings.refusals=INFO` in every profile (prod's root is ERROR); the e-mail is dropped from every other ratings log line.

- **D10 — Summaries (user, 2026-10-13):** JPQL `SELECT isbn, COUNT(*), SUM(rating) … GROUP BY isbn` (with `WHERE isbn = :isbn` for one book); `RatingService` computes `BigDecimal(sum).divide(BigDecimal(count), 1, HALF_UP)`; the answer is `RatingSummary(String isbn, long count, BigDecimal average)` (`4.0` serialised as `4.0`, `null` when the count is 0); `@Index idx_ratings_isbn` on `isbn`.
- **D11 — Rollback triggers (user, 2026-10-13):** read from Dynatrace service metrics for ratings by the release operator, any of, over 15 minutes after deploy: (1) POST+PUT answered 500 above 1%; (2) POST+PUT answered 503 above 20% while orders' own failure rate is at its baseline; (3) every POST+PUT answered 403 for 30 minutes with at least 20 writes; (4) `GET /summary` or `/summaries` failure rate above 1%, or median response time above twice the 7-day baseline.
- **Fact — producer JSON:** orders' `GET /api/v1/version` answers `{"serviceId": "orders", …}` (`orders/src/main/java/com/dynatrace/orders/controller/VersionController.java:32`, `common/src/main/java/com/dynatrace/model/Version.java:32-33`); an order carries the boolean `completed` (`orders/src/main/java/com/dynatrace/orders/model/Order.java:114`).

- **D12 — Spec challenges (user, 2026-10-13):** `## Engineering review` [ER1]–[ER7] added to `specification.md`; [ER1] (a misrouted 404 → 503) and [ER2] (only a JSON integer is a whole number of stars) are proposals with a `- [ ]` each (under [U01] [AC09] and [U01] [AC05]); no `[Uxx]` / `[ACxx]` / `[TCxx]` rewritten; the design proceeds on the stated behaviour. Spec `Open questions`: 0 → 2.
- **Gate 1 (user, 2026-10-13):** context, coverage, architecture & components, interfaces, seams confirmed — inferred items confirmed: request hooks unchanged at the top of POST; `Clock`-driven timestamps instead of `@UpdateTimestamp`; `@JsonIgnore` on `Rating.getComment()`; no change outside `ratings/`.
- **Gate 2 (user, 2026-10-13):** data flow, error handling, tests, risks, rollout, observability, out of scope, Engineering review confirmed — inferred items confirmed: give validation order e-mail → ISBN → stars; change order stars → e-mail → ISBN → 404 → body id → move → purchase check; rollback leaves the constraint and index in place; an over-length comment or e-mail → 500.

- **Pre-lint (Phase 5.5):** 13/13 headings, header `Open questions: 0`, `- [ ]` under `## Open questions`: 0, [ER1]–[ER7] consistent across design and spec; one placeholder-pattern hit (the log template `isbn=<isbn> reason=<verdict>`) inline-fixed to SLF4J `{}`.
- **Whole-artifact gate (user, 2026-10-13):** confirmed before review.

## Review

- **Review 1 — `design-reviewer` (Opus), 2026-10-13: BLOCK** — 1 BLOCKER ([AD#7]: an advice over every `ResponseStatusException` adds `message` to the existing `GET /{id}` 404), 1 MAJOR (Boot's default open-in-view holds a connection across the orders call and answers the re-read by id from the request's persistence context), 4 MINOR (non-advice error bodies carry no `message` key; `Long`/`String` binding of id / email / isbn can throw or coerce; rollback triggers (1)/(2) and check 3 absolute where POST already answers some 503s; the cached version probe makes the IT order-dependent), 3 NIT (package-private timestamp setters not callable from `RatingService`; `IsbnTotals` missing from the component list; Hibernate's duplicate-entry log line names the e-mail on a lost race).
- **Fix cycle (user, 2026-10-13):** BLOCKER fixed by scoping the advice (below); the user chose to fold the MAJOR, MINOR and NIT fixes into the same cycle so the one re-review covers them. Each edit read against what it overlaps (the storefront shows its own text on a 404, so a PUT 404 without `message` affects no consumer).
  - **D6 revised:** `RatingsErrorAdvice` handles exactly `BadRequestException`, `PurchaseForbiddenException`, `OrdersUnavailableException`; every other exception keeps Boot's default body (no `message` key), so every existing read's failure body is unchanged; spec [ER6] reworded to "unchanged".
  - **D3 revised:** `RatingRequest(JsonNode id, JsonNode email, JsonNode isbn, JsonNode rating, String comment)`; a non-string e-mail / ISBN gets its reason; a body id that is not a JSON integer equal to the path id → "bad rating id"; POST ignores the id whatever it holds.
  - **D13 (new):** `spring.jpa.open-in-view=false` in `application.properties`; IT cases for both operator-removal races.
  - **D11 revised:** triggers (1) and (2) and post-release check 3 measured against ratings' own 7-day POST+PUT `500` / `503` baselines (margins kept: 1 point, 20 points).
  - **D9 narrowed:** no log line ratings' own code writes carries an e-mail; Hibernate's duplicate-key ERROR on a lost race (names the e-mail) accepted under Risks.
  - NITs: public `Rating(email, isbn, stars, comment, at)` and `applyChange(stars, comment, at)`; `IsbnTotals` listed; probe stubbed `between(0, 1)` in the IT, asserted only in the adapter test on a fresh adapter.

- **Review 2 (the one re-review) — `design-reviewer` (Opus), 2026-10-13: PASS WITH RECOMMENDATIONS** — all nine review-1 findings resolved; no [AD#1]–[AD#7] violation; 0 open questions. Left open: [MINOR] MockMvc never forwards to Boot's `/error`, so the stated default error bodies were unchecked; [MINOR] binding `isbn` as `JsonNode` and requiring a string refused a 13-digit JSON number, stricter than the spec and [AD#4]; [NIT] the timeout test could run against a `RestTemplate` whose request factory the IT's `MockRestServiceServer` replaced in a cached context.
- **Findings applied (user, 2026-10-13):** `RatingErrorBodiesIT` (`RANDOM_PORT` + `TestRestTemplate`) asserts each error body's exact key set, and `RatingControllerIT` now checks status, resolved exception and an empty advice body for non-advice failures; an ISBN is a JSON string or a JSON integer whose text matches `^\d{13}$`; `OrdersPurchaseCheckTimeoutTest` builds its `RestTemplate` from `OrdersClientConfig`'s bean method with no Spring context. Each edit read against what it overlaps.
- **Version line:** PASS WITH RECOMMENDATIONS was reached before these three edits (the error-body test, the ISBN-as-integer rule, the timeout test's isolation), which followed it; the current text has not been reviewed. The review cap is spent.

## Interface candidates

- **PurchaseCheck port** (ratings → orders, network boundary; adapters: HTTP to orders, in-memory fake for tests):
  - ~~C1 — decision port: `PurchaseVerdict check(String email, String isbn)` returning `ELIGIBLE | NOT_A_CUSTOMER | NO_ORDER_FOR_BOOK | ORDER_NOT_COMPLETED`, throwing `OrdersUnavailableException` (→ `503`) for timeout / unreachable / other non-2xx / unreadable 200 / misrouted 404.~~ (eliminated: superseded by the fan-out; its shape survives as take A)
  - ~~C2 — data port: `Optional<List<OrderLine>> ordersOf(String email)` (`OrderLine(email, isbn, completed)`; empty for orders' unknown-customer 404), throwing `OrdersUnavailableException`; `RatingService` decides eligibility and the logged reason.~~ (eliminated: leaks orders' shape into the service; B's `lookup` was its fan-out form and lost)
  - Fan-out takes (3 of 3 returned):
    - A (minimise) — **chosen as the base of the hybrid (D4)** — `Eligibility check(email, isbn)` → `ELIGIBLE | NOT_A_CUSTOMER | NO_ORDER_FOR_BOOK | ORDER_NOT_COMPLETED`; one unchecked `OrdersUnavailableException`; `RatingService` logs and throws the 403; 404 counted as unknown-customer only when the error body's `path` is `/api/v1/orders/findByEmail` and a `GET /api/v1/version` probe names `orders`; one 2 s timeout.
    - ~~B (maximise flexibility) — `OrdersAnswer lookup(email)` (sealed: `Orders(List<Purchase>)` / `UnknownCustomer` / `Unavailable(Cause, detail)`) with a default `check(email, isbn)` → sealed `PurchaseVerdict` (`Eligible` / `Refused(reason)` / `Unavailable(cause)`); no exceptions; inner `OrdersIdentity` seam for the version probe with a positive-only cache; connect 1 s / read 2 s.~~ (lost: two layers of sealed types for a yes/no/unavailable question; `OrdersIdentity` fails the two-adapters test)
    - ~~C (most common caller) — `void requireBuyer(email, isbn)` throwing `BuyersOnlyException` (403 `ResponseStatusException`, whose factory writes the log entry) or `OrdersUnavailableException` (503 `ResponseStatusException`); same 404 decision table as A, unreadable 200 including an element missing `isbn`/`completed`; connect 1 s / read 3 s.~~ (lost as a whole: logging and HTTP status inside the domain port; its 503 exception style and unreadable-200 rule kept in the hybrid)

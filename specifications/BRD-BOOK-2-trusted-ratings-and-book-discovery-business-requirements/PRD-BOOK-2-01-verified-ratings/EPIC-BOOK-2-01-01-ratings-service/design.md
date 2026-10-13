---
kind: design
key: BOOK-2-01-01
---

# Design

- **Feature name**: Ratings service: buyers-only ratings, one per customer and book
- **Spec**: `specification.md` (this folder) — Epic BOOK-2-01-01
- **Classification**: SIGNIFICANT
- **Version**: 1
- **Created**: 2026-10-13
- **Author**: ihudak
- **Repos**: bookstore
- **Target**: bookstore:ratings
- **Open questions**: 0

## Context & problem

Quillmark's shoppers ignore its star ratings because the ratings service accepts any rating from any caller, lets one customer rate a book many times, stores a missing star value as 0, lets a change move a rating to another customer or book, returns every comment, and lets one request delete every rating (`specification.md`, Problem statement). This design changes `bookstore:ratings` alone so that every rule holds for every caller ([AD#1]): a give or a change is saved only for a buyer as orders reports one ([AD#2]), each customer and book hold one rating ([AD#3]), the write API answers the statuses and texts [AD#4] and the specification fix, `delete-all` is gone ([AD#5]), two summary endpoints serve each book's count and average ([AD#6]), and no answer carries a comment ([AD#7]). The storefront ([BOOK-2-01-02](../EPIC-BOOK-2-01-02-storefront-ratings/)) consumes what this Epic produces and lands after it; orders is used as it already runs.

## Requirements coverage

Challenge notes cross-reference the specification's `## Engineering review` ([ER1]–[ER7]).

| Requirement | How this design addresses it | Challenge |
|---|---|---|
| [U01] Rate a book I bought | `RatingService.give` (Data flow, give) | validated |
| [U01] [AC01] A buyer's give is stored | purchase check `ELIGIBLE` → insert (D7 flow) | validated |
| [U01] [AC02] A stored give is answered with the saved rating | `200` with the saved `Rating`, serialised without its comment | validated |
| [U01] [AC03] A give without a qualifying order is refused | `NOT_A_CUSTOMER` / `NO_ORDER_FOR_BOOK` / `ORDER_NOT_COMPLETED` → `PurchaseForbiddenException` (403) | proposed-change — [ER1]: a `404` that orders' `findByEmail` did not answer is not "orders answers is not a customer" |
| [U01] [AC04] One refusal for every reason | one constant reason on `PurchaseForbiddenException`, rendered by `RatingsErrorAdvice` as `{timestamp, status, error, message, path}` | validated |
| [U01] [AC05] Stars outside the scale are refused | `RatingRequest.rating` is a `JsonNode`; only a JSON integer 1–5 passes | questioned — [ER2]: `4.0` and `"4"` are refused as not whole-number JSON integers |
| [U01] [AC06] A missing or malformed e-mail or ISBN is refused | explicit checks in `RatingService` on `RatingRequest`'s `JsonNode` fields — e-mail: missing, not a JSON string, or not matching the entity's pattern; ISBN: missing, or neither a JSON string nor a JSON integer whose text matches `^\d{13}$` | validated |
| [U01] [AC07] Several faults → one reason | fixed check order: e-mail, ISBN, stars (Error handling) | validated |
| [U01] [AC08] An invalid give makes no purchase check | every `400` is decided before `PurchaseCheck.check` | validated |
| [U01] [AC09] A give is refused while orders cannot answer | `OrdersUnavailableException` (503) from the adapter's decision table | proposed-change — [ER1]; bounded time validated — [ER4] |
| [U01] [AC10] A give's id changes no other rating | `RatingRequest.id` is ignored on POST, whatever JSON value it holds; the pair alone selects the rating | validated |
| [U01] [AC11] Each buyers-only refusal is logged | one INFO entry on logger `com.dynatrace.ratings.refusals`, ISBN and reason, no e-mail | validated — [ER3]: prod's root level is ERROR, so the logger's level is set explicitly |
| [U02] Change my rating of a book | `RatingService.change` and a give applied as a change | validated |
| [U02] [AC01] A buyer's change replaces the stars | purchase check under the carried e-mail as typed, else the stored one | validated |
| [U02] [AC02] A saved change is answered with the saved rating | `200` with the saved `Rating` | validated |
| [U02] [AC03] Rating a book again changes the existing rating | pair lookup (database collation decides case) → apply as a change | validated |
| [U02] [AC04] Simultaneous gives leave one rating | `uk_ratings_email_isbn` + insert-then-recover (D7) | validated |
| [U02] [AC05] An empty comment keeps the stored comment | `comment == null \|\| comment.isBlank()` keeps the stored value | validated |
| [U02] [AC06] A comment on a change replaces the stored comment | non-blank comment replaces, as given (not trimmed) | validated |
| [U02] [AC07] Updated At moves, Created At stays | `RatingService` sets `updatedAt` from the injected `Clock` on every saved change, identical repeats included | validated |
| [U02] [AC08] A change keeps the stored e-mail as stored | only stars, comment and `updatedAt` are written on a change | validated |
| [U02] [AC09] Stars outside the scale are refused on a change | same star check as a give | questioned — [ER2] |
| [U02] [AC10] A malformed e-mail or ISBN on a change is refused | e-mail / ISBN checked only where present | validated |
| [U02] [AC11] A change cannot move a rating | carried e-mail compared `equalsIgnoreCase`, carried ISBN compared exactly, against the stored rating | validated |
| [U02] [AC12] A change to a rating that does not exist is refused | `404` (Boot's default body) after the body checks and before the id / move checks | validated |
| [U02] [AC13] A body id that does not match its path is refused | `RatingRequest.id` not a JSON integer equal to the path id — absent, `null`, text or `3.7` — → `400` "bad rating id" | validated |
| [U02] [AC14] Several faults → one reason | fixed check order (Error handling) | validated |
| [U02] [AC15] An invalid change makes no purchase check | every `400` / `404` is decided before `PurchaseCheck.check` | validated |
| [U02] [AC16] A change without a qualifying order is refused | as [U01] [AC03] | proposed-change — [ER1] |
| [U02] [AC17] A change is refused while orders cannot answer | as [U01] [AC09] | proposed-change — [ER1] |
| [U02] [AC18] A rating outlives its order's cancellation | no purchase state is stored or re-checked on reads; summaries read only `ratings` | validated |
| [U03] Know how a book is rated | `GET /summary`, `GET /summaries` | validated |
| [U03] [AC01] One book's summary | `COUNT`/`SUM` by ISBN; `BigDecimal` half-up division to one decimal | validated |
| [U03] [AC02] An ISBN with no rating has a count of 0 | no row → `RatingSummary(isbn, 0, null)` | validated |
| [U03] [AC03] Every rated book's summary | `GROUP BY isbn` over `ratings` | validated |
| [U03] [AC04] Summaries count each write on the next request | computed per request; no stored aggregate or cache | validated |
| [U03] [AC05] A summary request with no ISBN is refused | missing parameter → Spring's `400`; blank → `BadRequestException` | validated |
| [U03] [AC06] Summaries name no rater | `RatingSummary` carries `isbn`, `count`, `average` only | validated |
| [U04] Only Quillmark sees my comment | `@JsonIgnore` on `Rating.getComment()` | validated |
| [U04] [AC01] A comment given with a new rating is stored | stored as given when not blank | validated |
| [U04] [AC02] A new rating given without a comment stores none | blank / absent → `null` | validated |
| [U04] [AC03] No answer carries a comment | every endpoint answers `Rating` or `RatingSummary`; neither serialises a comment | validated |
| [U04] [AC04] Reads keep their other fields and failure statuses | same entity serialiser; `RatingsErrorAdvice` renders only `BadRequestException`, `PurchaseForbiddenException` and `OrdersUnavailableException`, none of which an existing read throws, so every read's failure keeps its status and Boot's default body | validated — [ER6]: every existing read's failure body is unchanged |
| [U05] Remove a single rating | `DELETE /{id}` unchanged | validated |
| [U05] [AC01] An operator removes one rating | unchanged `deleteById` | validated |
| [U05] [AC02] Removing an id that holds no rating removes nothing | unchanged (`200`, empty body) | validated |
| [U05] [AC03] A removal with an id that is not a number is refused | `delete-all` mapping removed; `/{id}`'s `Long` conversion fails → `400` | validated |
| [U05] [AC04] No request removes more than one rating | `truncateTable` removed; no other bulk path | validated — [ER7]: `DELETE /api/v1/ratings` answers `405` |

## Architecture & components

All changes are inside `ratings/` — the target's paths. Nothing changes in `common`, `exceptions`, `k8s` or `orders`: the new exceptions are ratings-local, and `DT_ORDERS_SERVER` already reaches the ratings pod through `envFrom` (`k8s/configmap.yaml:15`, `k8s/ratings.yaml:32-36`).

Package `com.dynatrace.ratings` (paths below relative to `ratings/src/main/java/com/dynatrace/ratings/`):

- **`service/RatingService.java`** (new) — the deep module. Owns give, change, summary and summaries: the fixed validation order with the specification's exact texts, the purchase check, the refusal log entry, the pair lookup, insert-then-recover on the unique constraint, the change rules (stars and comment only; Created At kept; Updated At set), and the half-up average. Writes run in short transactions through a `TransactionTemplate`; the purchase check runs before any transaction opens, and open-in-view is off (Configuration), so each repository call and each write transaction holds a connection only for its own length and a slow orders never holds one.
- **`controller/RatingController.java`** (changed) — HTTP mapping only. POST and PUT bind `RatingRequest` and delegate to `RatingService`; POST keeps `runThreatScan(); applySecurityPolicy();` at its top exactly as today. Adds `GET /summary` and `GET /summaries`. Removes the `delete-all` mapping, `verifyClient`, `verifyBook` and the e-mail from every log line.
- **`controller/RatingsErrorAdvice.java`** (new) — `@RestControllerAdvice` with one `@ExceptionHandler` for exactly three types — `BadRequestException`, `PurchaseForbiddenException` and `OrdersUnavailableException` — rendering each as Boot's own error shape `{timestamp, status, error, message, path}` with the reason as `message`. No existing read throws any of the three. Every other exception — `ResourceNotFoundException` (the `404` of `GET /{id}` and of `PUT /{id}`), the request hooks' `503`, the `400`s and the `405` Spring raises itself, an unexpected `500` — keeps Boot's default rendering, which leaves the `message` key out (`server.error.include-message` stays `never`), so every existing read's failure body stays exactly as today ([AD#7]).
- **`model/RatingRequest.java`** (new) — `record RatingRequest(JsonNode id, JsonNode email, JsonNode isbn, JsonNode rating, String comment)`. The four `JsonNode` fields take whatever JSON value they carry, so a fault in any of them reaches `RatingService`'s checks and gets its reason; `comment` binds a string (Jackson turns a number or a boolean into its text). A body Jackson cannot read at all — malformed JSON, or an object or array as the comment — is Spring's `400` with Boot's default body.
- **`model/RatingSummary.java`** (new) — `record RatingSummary(String isbn, long count, BigDecimal average)`.
- **`model/Rating.java`** (changed) — `@Table(name = "ratings", uniqueConstraints = @UniqueConstraint(name = "uk_ratings_email_isbn", columnNames = {"email", "isbn"}), indexes = @Index(name = "idx_ratings_isbn", columnList = "isbn"))`; setters lose validation and no longer touch `updatedAt`; `getComment()` carries `@JsonIgnore`; a public creation constructor `Rating(String email, String isbn, int stars, String comment, Date at)` sets Created At and Updated At to `at`, and a public `applyChange(int stars, String comment, Date at)` sets the stars, applies the comment rule (Data flow) and sets Updated At to `at` — `RatingService` uses these two; today's no-op `setCreatedAt`/`setUpdatedAt` stay no-ops. Never bound from a request.
- **`repository/RatingRepository.java`** (changed) — `Optional<Rating> findByEmailAndIsbn(String email, String isbn)` (plain equality; the database collation decides letter case, so lookup and constraint agree by construction); two summary queries (Interfaces); `truncateTable` removed.
- **`repository/IsbnTotals.java`** (new) — `record IsbnTotals(String isbn, Long count, Long sum)`, the summary queries' projection.
- **`purchase/PurchaseCheck.java`, `purchase/Eligibility.java`** (new) — the port (Interfaces).
- **`purchase/OrdersPurchaseCheck.java`** (new) — the HTTP adapter over orders' `GET /api/v1/orders/findByEmail`, with a private consumer-side copy `record OrderLine(String isbn, Boolean completed)` (`@JsonIgnoreProperties(ignoreUnknown = true)`), the decision table, and the version probe with its positive-only cache.
- **`purchase/OrdersUnavailableException.java`** (new) — `extends ResponseStatusException`, `503`, reason "Service Unavailable".
- **`config/OrdersClientConfig.java`** (new) — the `ordersRestTemplate` bean from `RestTemplateBuilder` with `connectTimeout(ratings.orders.connect-timeout)` and `readTimeout(ratings.orders.read-timeout)`, and a `Clock` bean (`Clock.systemDefaultZone()`).
- **Deleted** — `repository/ClientRepository.java`, `repository/BookRepository.java`.
- **Configuration** (`ratings/src/main/resources/`):
  - `application.properties` — `ratings.orders.connect-timeout=1s`, `ratings.orders.read-timeout=3s`, `logging.level.com.dynatrace.ratings.refusals=INFO` (the base file every profile loads, so the refusal entry is written in prod too, whose root level is ERROR), and `spring.jpa.open-in-view=false` (Boot's default binds one persistence context and its connection to the whole request, which would hold a connection across the orders call and answer a re-read by id from that context instead of the database; `Rating` has no lazy association that needs it).
  - `application-{dev,stage,prod,test}.properties` — add `http.service.orders=http://${DT_ORDERS_SERVER:localhost:8085}/api/v1/orders`; remove `http.service.clients` and `http.service.books`.
  - `application-test.properties` — `spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1;MODE=MySQL;IGNORECASE=TRUE`.

Responsibilities, in one line each: the controller speaks HTTP; `RatingService` decides; `PurchaseCheck` answers "does this e-mail hold a qualifying order for this ISBN?"; the database guarantees one rating per pair; the advice puts the write and summary refusals' reasons into `message`.

### Alternatives considered

- **Take B (maximise flexibility)** — `OrdersAnswer lookup(email)` with a default `check` returning a sealed `PurchaseVerdict`, no exceptions, and an inner `OrdersIdentity` seam. Lost: two layers of sealed types (four records, two enums) for a yes / no / unavailable question, and `OrdersIdentity` has one real adapter, so it fails the two-adapters test.
- **Take C (most common caller)** — `void requireBuyer(email, isbn)` throwing `BuyersOnlyException` (403, which writes the log entry in its factory) or `OrdersUnavailableException`. Lost: the refusal policy and HTTP status move into the domain port, so a second caller would write spurious refusal entries and every fake must go through the logging factory. Its `ResponseStatusException` style for the 503 and its unreadable-200 rule were kept.
- **Rules kept in `RatingController`** — lost: one class mixing HTTP, persistence, orders I/O and the transaction boundary that race recovery needs.
- **`server.error.include-message=always`** — lost: every unexpected exception's text would reach callers, a duplicate-key message naming the e-mail among them.
- **An advice over every `ResponseStatusException`** — lost: it would add a `message` to the existing `GET /{id}` `404`, whose body [AD#7] keeps unchanged.
- **Native `INSERT … ON DUPLICATE KEY UPDATE`** — lost: bypasses JPA, needs a reload for the `200` answer, and ties the tests to H2's partial MySQL emulation.
- **`SELECT … FOR UPDATE` on the pair** — lost: locks nothing when the row is absent (MySQL gap locks deadlock under concurrency), so it still needs the constraint fallback.
- **Testcontainers MySQL for tests** — lost: new dependencies and Docker wherever `./gradlew :ratings:test` runs, for a collation H2 reproduces with `IGNORECASE=TRUE`.
- **A lower-cased key column** — lost: departs from [AD#3]'s "compared as the ratings database compares text".
- **Keeping the client and book existence calls** — lost: their `404` would disclose whether an e-mail is a customer ([AD#4]), and the purchase check already refuses an unknown e-mail or ISBN.
- **Hibernate `@UpdateTimestamp`** — lost: it moves Updated At only when the row is dirty, so an identical repeat would keep it ([U02] [AC07] TC04).

## Interfaces / contracts

### The purchase-check port (in-process side of a remote-but-owned seam)

```java
package com.dynatrace.ratings.purchase;

public interface PurchaseCheck {
    /** Read with no side effect. Throws OrdersUnavailableException for every outcome that is not an answer about the e-mail. */
    Eligibility check(String email, String isbn);
}

public enum Eligibility { ELIGIBLE, NOT_A_CUSTOMER, NO_ORDER_FOR_BOOK, ORDER_NOT_COMPLETED }

public class OrdersUnavailableException extends ResponseStatusException {
    public OrdersUnavailableException(Throwable cause) { super(HttpStatus.SERVICE_UNAVAILABLE, "Service Unavailable", cause); }
}
```

`email` is passed exactly as typed — no trim, no case folding — and URL-encoded by `UriComponentsBuilder` (`.encode()`); `isbn` is compared with `equals`. Neither the port nor its adapter logs the e-mail or puts it in an exception message. `RatingService` is the only production caller, and only on its write path, so no read depends on orders.

`OrdersPurchaseCheck`'s decision table, in order:

1. `200` whose body reads as a JSON array of objects each carrying a string `isbn` and a boolean `completed` (other fields ignored): an element with the ISBN and `completed == true` → `ELIGIBLE`; an element with the ISBN, none completed → `ORDER_NOT_COMPLETED`; no element with the ISBN → `NO_ORDER_FOR_BOOK` (a known customer with no orders answers `[]`).
2. `200` that does not read that way — not JSON, not an array, a `null` element, an element missing `isbn` or a boolean `completed` → `OrdersUnavailableException`.
3. `404` → `NOT_A_CUSTOMER` only when both hold: the body is a JSON object whose `path` equals the constant `/api/v1/orders/findByEmail`, and `GET <scheme://host:port of http.service.orders>/api/v1/version` answers `200` with `serviceId` `"orders"`. Only a positive probe answer is cached, for the life of the pod (configuration does not change at runtime). Anything else — a different `path`, a probe that fails, times out or names another service — → `OrdersUnavailableException`.
4. A connect or read timeout, a refused connection, any other status → `OrdersUnavailableException`.

Bound: one `findByEmail` call, at most 1 s to connect and 3 s to read; on the `404` path before the first positive probe, one more call under the same bounds. No retry. The call is a read: repeating it repeats no side effect.

### The ratings REST API ([AD#4]–[AD#7]; base path `/api/v1/ratings`, unchanged)

| Endpoint | Status of the change | Success | Failures (each saves nothing and leaves every rating as it was) |
|---|---|---|---|
| `POST ""` | changed | `200` the saved `Rating` — new, or the existing rating of the pair after the give is applied to it as a change | `400` "Invalid email address" / "ISBN must be a 13-digits value" / "Invalid rating. Must be a whole number from 1 to 5."; `403` "Only customers who have bought this book can rate it."; `503` "Service Unavailable" |
| `PUT /{id}` | changed | `200` the saved `Rating` | `400` the three texts above, "A rating's e-mail and ISBN cannot be changed.", "bad rating id"; `404` (Boot's default body); `403`; `503` |
| `DELETE /{id}` | unchanged | `200`, empty body, whether or not the id held a rating | `400` for an id that is not a number (`/delete-all` included), Boot's default body |
| `DELETE /delete-all` | removed | — | reaches `/{id}` → `400` |
| `GET /summary?isbn=` | new | `200` `{"isbn": "…", "count": n, "average": 4.3}` (`average` `null` when `count` is 0) | `400`: a blank `isbn` with "isbn is required"; a missing one with Boot's default body |
| `GET /summaries` | new | `200` `[ {isbn, count, average}, … ]`, one per ISBN holding a rating, in ISBN order | — |
| `GET ""`, `/findByEmail`, `/findByISBN`, `/{id}` | changed | unchanged but for the comment, which no answer carries | unchanged statuses and bodies |

`Rating` JSON is `{id, email, isbn, rating, updatedAt, createdAt}` — today's fields less `comment`. The `400`, `403` and `503` bodies `RatingsErrorAdvice` renders are `{timestamp, status, error, message, path}`, and two buyers-only refusals differ only in `timestamp` and `path`. Every other error body is Boot's default `{timestamp, status, error, path}`, with no `message` key: the `404` of `PUT /{id}` and of the reads, the `400`s Spring raises itself (a missing `isbn` parameter, a non-numeric id, `delete-all`, an unreadable body), the `405`, the request hooks' `503`, and a `500`. Repeating a give or a change leaves the same single rating with the same values and moves Updated At; repeating a removal removes nothing more and answers alike; a caller that gets `503` may repeat the request safely. Ratings answers no `500` for a race between gives of one pair.

`RatingRepository` summary queries:

```java
@Query("select new com.dynatrace.ratings.repository.IsbnTotals(r.isbn, count(r), sum(r.rating)) from Rating r where r.isbn = :isbn group by r.isbn")
Optional<IsbnTotals> totalsFor(String isbn);
@Query("select new com.dynatrace.ratings.repository.IsbnTotals(r.isbn, count(r), sum(r.rating)) from Rating r group by r.isbn order by r.isbn")
List<IsbnTotals> totalsForAll();
// record IsbnTotals(String isbn, Long count, Long sum) — wrapper types, as Hibernate 6 types count() and sum(int) as Long
```

`average = BigDecimal.valueOf(sum).divide(BigDecimal.valueOf(count), 1, RoundingMode.HALF_UP)` — the exact quotient rounded once, so 17/4 = 4.25 → 4.3 and 21/20 = 1.05 → 1.1; serialised as a JSON number with one decimal (`4.0`).

### Contract (Epic `## Contract`, PRD-level ARD `## Contracts`)

- **Produces [AD#4]** (changed) — met by the POST and PUT rows above and the Data flow; producer-side tests: `RatingControllerIT` (Test strategy).
- **Produces [AD#5]** (changed) — `delete-all` removed, `/{id}` unchanged; producer-side tests: `RatingControllerIT`.
- **Produces [AD#6]** (new) — `GET /summary`, `GET /summaries`; producer-side tests: `RatingControllerIT` and `RatingServiceTest`.
- **Produces [AD#7]** (changed) — `@JsonIgnore` on the comment; producer-side tests: `RatingControllerIT` reads every endpoint's raw JSON for a `comment` key.
- **Consumes [AD#2]** (exists) — orders' `findByEmail` as it already runs, plus its `GET /api/v1/version`. Ratings depends only on `isbn` and `completed` in the answer and `serviceId` in the version answer, and ignores every other field. Test doubles: `MockRestServiceServer` bound to `ordersRestTemplate` for the adapter, modelling every outcome of the decision table; `InMemoryPurchaseCheck` for `RatingService`, modelling each `Eligibility` and unavailability. Orders and ratings share a repository, and the contract row's `Artifact` is `—`: `OrderLine` is a consumer-side copy owned by ratings, as the ARD's Schema ownership states.

## Seams

| Seam | Component | Dependency category | Adapters |
|---|---|---|---|
| `PurchaseCheck` | bookstore:ratings → bookstore:orders | remote-but-owned | `OrdersPurchaseCheck` (HTTP, production); `InMemoryPurchaseCheck` (test sources) |
| `RatingRepository` / the `ratings` table | bookstore:ratings | local-substitutable | MySQL (production); H2 `MODE=MySQL;IGNORECASE=TRUE` (tests) |
| Summary arithmetic | `RatingService` | in-process | none — tested through `RatingService` |
| HTTP surface | `RatingController` + `RatingsErrorAdvice` | in-process (MockMvc) | none |
| `Clock` | `RatingService` | in-process | `Clock.systemDefaultZone()`; a fixed or stepped `Clock` in tests |

`PurchaseCheck` is the one seam that earns a port: it crosses a network boundary and has two real adapters. `RatingService` is the highest seam that still isolates the rules — tests drive it with the in-memory port and the H2 database, and the controller tests drive the whole stack through MockMvc with `MockRestServiceServer` behind the real adapter.

## Data flow

**Give — `POST /api/v1/ratings`.**

1. `RatingController.createRating` runs `runThreatScan(); applySecurityPolicy();` as today, then calls `RatingService.give(request)`.
2. Validation, in this order, each failure a `BadRequestException` (`400`) and no further step: e-mail missing, not a JSON string, or not matching the entity's e-mail pattern → "Invalid email address"; ISBN missing, or neither a JSON string nor a JSON integer whose text matches `^\d{13}$` → "ISBN must be a 13-digits value" (a 13-digit JSON integer is taken as its text, as Jackson's string binding took it before); `rating` missing, `null`, or not a JSON integer from 1 to 5 → "Invalid rating. Must be a whole number from 1 to 5.". `request.id` is ignored, whatever JSON value it holds.
3. `existing = ratingRepository.findByEmailAndIsbn(email, isbn)` — the database collation matches the e-mail ignoring letter case.
4. `purchaseCheck.check(email as typed, isbn)`, outside any transaction. `OrdersUnavailableException` propagates (`503`). Any verdict but `ELIGIBLE` writes one INFO entry on `com.dynatrace.ratings.refusals` — `Rating refused: isbn={} reason={}` (SLF4J; the ISBN, then the `Eligibility` constant) — and throws `PurchaseForbiddenException("Only customers who have bought this book can rate it.")` (`403`).
5. `existing` present → in one transaction, re-read it by id and apply the change (below); where the re-read finds nothing — an operator removed it meanwhile — continue at step 6.
6. `existing` absent → in its own transaction, insert `new Rating(email as typed, isbn, stars, comment blank → null, now)` — Created At and Updated At both `now` — and flush. On `DataIntegrityViolationException`, in a new transaction re-read the pair: a row → apply the change to it; none → rethrow (Error handling).
7. Answer `200` with the saved `Rating`.

**Change — `PUT /api/v1/ratings/{id}` → `RatingService.change(id, request)`.**

1. Validation, in this order: stars (as a give); e-mail only where present — in the body and not `null` — and then not a JSON string or not matching the pattern → "Invalid email address"; ISBN only where present, and then neither a JSON string nor a JSON integer whose text matches `^\d{13}$` → "ISBN must be a 13-digits value".
2. `ratingRepository.findById(id)` empty → `ResourceNotFoundException("Rating not found")` (`404`).
3. `request.id` not a JSON integer equal to the path id — absent, `null`, text or `3.7` included — → "bad rating id"; then a carried e-mail not `equalsIgnoreCase` the stored one, or a carried ISBN whose text is not equal to the stored one → "A rating's e-mail and ISBN cannot be changed." (each `400`).
4. `purchaseCheck.check(carried e-mail as typed, else the stored e-mail, stored ISBN)`, outside any transaction, as a give's step 4.
5. In one transaction, re-read by id (gone → `404`) and apply the change; answer `200`.

**Applying a change** (a `PUT`, or a give applied as one): `rating.applyChange(stars, comment, Date.from(clock.instant()))` sets the stars; a comment holding a non-space character replaces the stored comment as given, an empty, absent or all-space one keeps it; Updated At is set on every saved change, an identical repeat included; `email`, `isbn` and Created At stay as stored.

**Summary — `GET /summary?isbn=`** → `RatingService.summary(isbn)`: blank → `BadRequestException("isbn is required")`; `totalsFor(isbn)` → `RatingSummary(isbn, count, sum/count half-up to one decimal)`, or `RatingSummary(isbn, 0, null)` where no row matches. **Summaries — `GET /summaries`** → `totalsForAll()` mapped the same way, ordered by ISBN. Neither touches orders, and neither reads `email` or `comment`.

**Removal** — unchanged `deleteById`; nothing else removes ratings.

State: a rating is created by a give, altered only by a change (stars, comment, Updated At), and removed only by `DELETE /{id}`. Ratings stores no purchase state; eligibility is read from orders at the moment of each give or change.

## Error handling & edge cases

| Situation | Answer | State left |
|---|---|---|
| A give or change refused as invalid (`400`) | the reason in `message` | nothing saved; no call to orders |
| A change to an unknown id (`404`) | Boot's default body | nothing saved; no call to orders |
| A body Jackson cannot read — malformed JSON, or an object or array as the comment | `400`, Boot's default body | nothing saved; no call to orders |
| Orders: no qualifying order, unknown customer (confirmed `404`) | `403`, one identical body | nothing saved; one refusal log entry |
| Orders: timeout (1 s connect / 3 s read), refused connection, `5xx`, any status but `200`/`404`, unreadable `200`, misrouted `404` | `503` "Service Unavailable" | nothing saved; nothing logged by ratings (specification, Out of scope) |
| Version probe fails, times out or names another service after a `404` | `503` | nothing saved; the probe is retried on the next `404` |
| Two gives for one pair race | each `200` | exactly one rating; the loser's give applied to it as a change |
| Two changes to one rating race | each `200` | last write wins ([U02] [AC04] accepts either stars) |
| An operator removes a rating between a give's lookup and its write | `200` | the give inserts a new rating |
| An operator removes a rating between a change's checks and its write | `404` | nothing saved |
| A comment or an e-mail longer than its column (255 characters) | `500`, Boot's default body | nothing saved (the specification leaves this answer unset; [ER5]) |
| `GET /summary` with a non-blank ISBN that is not 13 digits | `200`, `count` 0, `average` `null` | — ([U03] [AC02]) |
| `GET /summary` with no or a blank `isbn` | `400` | — |
| `DELETE /api/v1/ratings/delete-all`, or any non-numeric id | `400` | nothing removed |
| `DELETE /api/v1/ratings` | `405` | nothing removed ([ER7]) |
| The unique constraint missing in an environment (table not emptied before start) | gives of one pair may store two rows under a race | detected by the post-release duplicate check; Migration says how it is prevented |

A give or change that several `400` checks refuse answers the first in the order of Data flow ([U01] [AC07], [U02] [AC14] allow any one of their reasons). Read endpoints never call orders, so they keep working while orders is down.

## Test strategy

Prior art: `ratings/src/test/java/com/dynatrace/ratings/RatingsApplicationTests.java` (an empty `contextLoads` on the `test` profile) is the only ratings test, and no test in the repository stubs another service. `spring-boot-starter-test` already supplies JUnit 5, MockMvc, `MockRestServiceServer` and `OutputCaptureExtension`; no dependency is added. Run with `./gradlew :ratings:test`. Every specification test case is implemented and named by its ID (`U01_AC05_TC03` …).

| Test (in `ratings/src/test/java/com/dynatrace/ratings/`) | Seam / category | Covers |
|---|---|---|
| `service/RatingServiceTest` — `@SpringBootTest(webEnvironment = NONE)` on the `test` profile, `InMemoryPurchaseCheck` and a stepped `Clock` as `@Primary` beans | `RatingService` with the local-substitutable database (H2 `MODE=MySQL;IGNORECASE=TRUE`) and the in-memory port | validation order and exact texts, a non-string e-mail, an ISBN sent as a 13-digit JSON integer (accepted, and on a change equal to the stored one proceeding) or as a 12-digit one (refused), and a body id that is text, `3.7` or absent included; give and change rules; the pair compared ignoring letter case ([U02] [AC03] TC04); stored e-mail kept ([AC08]); comment rules; Created At / Updated At with the stepped clock ([AC07]); summaries — 4.25 → 4.3, 1.05 → 1.1, 4.0, 4.7, count 0 → `null`; a rating outliving its order ([AC18]) |
| `service/RatingServiceConcurrencyTest` — not transactional, table emptied after each test | same | 10 threads giving one pair at once, mixed capitals included ([U02] [AC04] TC01–TC04): one row, no exception, every give answered |
| `purchase/OrdersPurchaseCheckTest` — a fresh adapter per test, `MockRestServiceServer` bound to the `RestTemplate` passed to it | `PurchaseCheck` HTTP adapter (remote-but-owned, the port's production adapter) | every row of the decision table: buyer, not completed, no order, `[]`, unknown-customer `404` + probe `orders`, `404` with another `path`, `404` + probe naming `books`, `404` + probe `500`, `500`, `503`, unreadable `200` (HTML, an object, an element missing `completed`), a timeout (`withException(new SocketTimeoutException())`); e-mail URL-encoded; a second `404` after a positive probe makes no probe call |
| `purchase/OrdersPurchaseCheckTimeoutTest` — no Spring context: the `RestTemplate` comes from `OrdersClientConfig`'s bean method with 1 s / 3 s, so no test's `MockRestServiceServer` can have replaced its request factory | that `RestTemplate` and the adapter against a local `ServerSocket` that accepts and never answers | the bound: `check` throws `OrdersUnavailableException` in under 4.5 s ([U01] [AC09] TC03, [U02] [AC17] TC03) |
| `controller/RatingControllerIT` — `@SpringBootTest` + `@AutoConfigureMockMvc`, `MockRestServiceServer` bound to the `ordersRestTemplate` bean | the whole HTTP surface, the real adapter behind the stub | every specification TC at HTTP level: statuses and exact `message` texts; two refusals' bodies equal in every field but `timestamp` and `path`; no `comment` key in any answer's raw JSON; `ExpectedCount.never()` on orders for every `400`/`404` ([U01] [AC08], [U02] [AC15]); `delete-all` `400` and `DELETE /api/v1/ratings` `405`; summary JSON `4.3`, `4.0`, `null`; for each failure the advice does not render, the status, `getResolvedException()`'s type and that the advice wrote no body — MockMvc does not forward to Boot's `/error`, so the bodies themselves are checked in `RatingErrorBodiesIT`; both operator-removal races, the rating removed from inside the orders stub's answer (a `ResponseCreator` that deletes it before it responds), answered `200` with a new rating for a give and `404` for a change; the version probe stubbed `ExpectedCount.between(0, 1)` with `ignoreExpectOrder`, because the adapter bean's positive cache outlives a test method — the probe and its cache are asserted only in `OrdersPurchaseCheckTest`; the refusal log entry through `OutputCaptureExtension` — one line on `com.dynatrace.ratings.refusals` naming the ISBN and the reason and holding no `@` |
| `controller/RatingErrorBodiesIT` — `@SpringBootTest(webEnvironment = RANDOM_PORT)` + `TestRestTemplate`, a real server so Boot's `/error` renders | the HTTP surface as a caller sees it | the exact key set of each kind of error body: `{timestamp, status, error, path}` with no `message` for the `GET /{id}` `404` (as today), the `PUT /{id}` `404`, a Spring `400` (`DELETE /delete-all`) and the `405` (`DELETE /api/v1/ratings`); `{timestamp, status, error, message, path}` for an advice-rendered `400`. None of these cases calls orders |
| `RatingsApplicationTests` (unchanged) | context | the context starts with the new beans and the H2 URL |

Producer-side tests for each boundary interface: `RatingControllerIT` drives the real `RatingService`, repository and adapter through the REST API for [AD#4], [AD#5], [AD#6] and [AD#7], their failure cases included. The consumed [AD#2] is modelled by `MockRestServiceServer` (every outcome orders can give, failures included) and by `InMemoryPurchaseCheck` (each `Eligibility` and unavailability); neither claims orders behaves so — orders is unchanged and its behaviour is the one the ARD grounds (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:59-65`, `221-231`).

## Risks & mitigations

- **Every give and change now waits on orders, and orders on clients twice.** Accepted under [VD#5]; bounded by the 1 s / 3 s timeouts, with no retry, so a slow orders costs at most about 4 s per write (about 8 s on the first `404` path while the probe is unconfirmed) and never holds a database connection, open-in-view being off.
- **A wrong orders URL or host.** Refusing every buyer with `403` is prevented by the `404` confirmation, which answers `503` instead; trigger 2 or 3 of the rollback signal catches it within 15–30 minutes.
- **Orders changes its error body or its version endpoint.** Unknown e-mails would then be answered `503` rather than `403` — the safe direction — and the adapter test documents the shape it relies on.
- **The unique constraint is not created.** Under `ddl-auto=update` Hibernate cannot add it over duplicate rows and starts anyway; insert-then-recover then has nothing to recover from. Mitigated by the release steps (Migration) and the post-release duplicate and index checks.
- **H2 and MySQL compare text alike only where it matters.** MySQL's `utf8mb4_0900_ai_ci` also ignores accents; the e-mail pattern admits ASCII only, so H2's `IGNORECASE` reproduces every comparison the pair can meet.
- **`summaries` grows with the catalogue.** One `GROUP BY` over `idx_ratings_isbn` per request, unpaged ([CD#10]); trigger 4 watches its response time.
- **Rollback leaves the constraint in place.** The previous ratings answers `500` to a repeat add rather than storing a second rating; accepted, since data stays clean and nothing needs reversing.
- **A lost race writes the e-mail to the log.** On a lost race (Data flow, give step 6) Hibernate's `SqlExceptionHelper` logs the database's duplicate-key message, which names the e-mail and the ISBN, at ERROR, which prod keeps. Accepted: it happens only when two gives for one pair arrive at once, and silencing that logger would hide every other SQL error; no log line ratings' own code writes carries an e-mail.
- **Test environments with the load generator.** Most of its gives are refused, each costing an orders call, and its `delete-all` calls fail and are swallowed (ARD, Edge cases & risks); accepted.

No `Target span`, no ARD deviation, and no architecture deviation (architecture grounding was off for this run).

## Migration / rollout / backward-compatibility

- **Schema.** No column changes. `ddl-auto=update` adds `uk_ratings_email_isbn` (unique, on `email`, `isbn`) and `idx_ratings_isbn` when the new ratings starts. No migration tool and no start-up check are added (ARD, Edge cases & risks).
- **Release steps, by the operators, in each environment** (ARD, Edge cases & risks; outside this Epic's code): stop the old ratings and, where it runs, the load generator; empty the `ratings` table; start the new ratings; confirm `SHOW INDEX FROM ratings` lists `uk_ratings_email_isbn` with `Non_unique = 0` and `idx_ratings_isbn`; restart the load generator where it runs; then deploy the storefront (landing order: orders, unchanged → ratings → web).
- **Configuration.** `http.service.orders` resolves from `DT_ORDERS_SERVER`, already in `bookstore-configmap`; no Kubernetes change. The timeouts and the refusal logger's level ship in `application.properties`.
- **Compatibility.** `/api/v1/ratings` changes in place, as the ARD's Versioning section states: every caller is in this repository; the storefront moves in the same release; the load generator tolerates the failures. Between the ratings and the web deploys the old storefront's Delete All fails with `400` and its comment column is empty; acceptable for the length of a deploy. Every existing read's failure status and body are unchanged ([ER6]), since the advice renders only the write and summary refusals.
- **Rollback.** Roll the storefront back first where it is already deployed (the new storefront needs the summary endpoints), then redeploy the previous ratings image. Leave the constraint and index in place; there is no data to reverse.

## Observability & release verification

- **Signals.**
  - Dynatrace service metrics for the ratings service, per endpoint — request count, failure rate, response time, and response-code mix (`200`/`400`/`403`/`404`/`503`/`500`) for `POST /api/v1/ratings`, `PUT /api/v1/ratings/{id}`, `GET /api/v1/ratings/summary`, `GET /api/v1/ratings/summaries` and `GET /api/v1/ratings/findByISBN` — exist today (every ratings image is instrumented); the two summary endpoints appear with this change.
  - Traced outbound calls from ratings to orders (`GET /api/v1/orders/findByEmail`, `GET /api/v1/version`) — exist as soon as ratings makes them; they show orders' latency and `404` rate behind each write.
  - Refusal log entries on `com.dynatrace.ratings.refusals`, one per buyers-only refusal, carrying the ISBN and the reason — added by this change (`RatingService`).
- **Baseline.** The ratings service's POST and PUT failure rate, `500` share, `503` share and median response time, `GET /findByISBN`'s median response time (the stand-in baseline for the two new summary endpoints), and the orders service's failure rate, each read in Dynatrace for the 7 days before the deploy, per environment, by the release operator on the day of the deploy. None can be read now: this run has no Dynatrace access, and the summary endpoints do not exist yet.
- **Blast radius.** Orders, and clients behind it — one more `findByEmail` per write (signal: orders' request count and failure rate). The ratings database — two new indexes and `GROUP BY` queries (signal: ratings' database statement time). The storefront — the consumer of every changed answer (signal: ratings' `4xx` mix per endpoint). The load generator in test environments — `delete-all` now `400`, most gives refused (signal: its own log). Ratings' reads must not depend on orders (signal: read endpoints' failure rate while orders' failure rate rises).
- **Post-release checks** (each over the 24 hours after the deploy, per environment):
  - [U01] [AC01], [AC02]; [U02] [AC01]–[AC03] — POST+PUT `500` share no higher than its 7-day baseline, and at least one `200` once real ratings arrive.
  - [U01] [AC03], [AC04], [AC11]; [U02] [AC16] — refusal log entries present with their reasons, never an `@`; POST+PUT answered `403` below 100% whenever there were at least 20 writes.
  - [U01] [AC09]; [U02] [AC17] — POST+PUT `503` share no more than one percentage point above its own 7-day baseline plus orders' failure share (POST already answers some `503`s today, from its request hooks).
  - [U02] [AC04] — `SELECT email, isbn, COUNT(*) c FROM ratings GROUP BY email, isbn HAVING c > 1` returns no row, and `SHOW INDEX FROM ratings WHERE Key_name = 'uk_ratings_email_isbn'` shows `Non_unique = 0`.
  - [U03] [AC01]–[AC04], [AC06] — `GET /summary` and `/summaries` failure rate 0%, median response time at most twice `GET /findByISBN`'s baseline; for one rated book, `/summary` equals `SELECT COUNT(*), SUM(rating) FROM ratings WHERE isbn = ?` divided and rounded half up.
  - [U05] [AC03], [AC04] — in a test environment with the load generator, `DELETE /delete-all` answered `400`, and the ratings row count never falls by more than one per removal request.
  - n/a — no runtime signal tells these apart; they rest on Test strategy: [U01] [AC05]–[AC08], [AC10]; [U02] [AC05]–[AC15], [AC18]; [U03] [AC05]; [U04] [AC01]–[AC04]; [U05] [AC01], [AC02].
- **Alert / SLO impact.** No service-level objective is defined for the bookstore services in the repository. Dynatrace counts a `503` as a server-side failure, so an orders outage now raises the ratings service's failure rate too; this is expected and adds no alert. No alert is added or retuned.
- **Rollback signal.** Read by the release operator from Dynatrace service metrics for ratings; any one of these, within 15 minutes of the deploy unless stated, triggers the rollback in Migration: (1) POST+PUT `500` share more than 1 percentage point above its 7-day baseline; (2) POST+PUT `503` share more than 20 percentage points above its 7-day baseline while orders' own failure rate is at its baseline; (3) every POST+PUT answered `403` for 30 minutes with at least 20 writes; (4) `GET /summary` or `/summaries` failure rate above 1%, or their median response time above twice `GET /findByISBN`'s 7-day baseline.

## Out of scope

- Any change to `bookstore:orders`, `bookstore:web`, the load generator, `common`, `exceptions` or `k8s` — this design changes `ratings/` alone.
- Logging a `503` or the orders failure behind it (specification, Out of scope).
- A limit on a comment's length, and what an over-length comment or e-mail is answered (specification, Out of scope; [ER5]).
- Paging `summaries` ([CD#10]); summaries anywhere but the two endpoints.
- Matching a typed e-mail to an order's e-mail ignoring letter case, or a lighter purchase lookup — orders changes ([VD#5]).
- Restricting `DELETE /{id}` to operators, sign-in, and an operator endpoint for comments ([VD#1], [CD#14], [AD#7]).
- A migration tool, a start-up check of the constraint, emptying the table, or Testcontainers.

## Open questions

None.

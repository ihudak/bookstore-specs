---
kind: ard
key: BOOK-2-01
title: Verified ratings — ARD
scope: epic
prd: BOOK-2
epic: BOOK-2-01
area: null
status: draft
grounded_repos:
  - bookstore @ /workspace/bookstore
components:
  - id: bookstore:ratings
  - id: bookstore:web
  - id: bookstore:orders
inherits: null
derived_from: specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/prd.md
---

# Verified ratings — ARD

## Context

Shoppers on Quillmark's online store ignore its star ratings because the store does nothing to make them trustworthy (`prd.md`, Problem). This slice of BRD BOOK-2 makes a rating count only when it comes from a buyer: a customer with a completed, paid and not since cancelled order for the book ([CD#1]). Each customer holds one rating per book, of whole stars from 1 to 5, which they can change but not repeat or remove. Operators remove a single rating for an erasure request or fraud. The book page shows a book's average rating and the number of ratings it rests on, the catalogue list shows each book's average, and a book with no ratings shows "Not yet rated" ([US#1]–[US#5]). Comments are still taken and kept, and shown to no shopper ([US#7], [CD#17]).

The architecture is framed by what the PRD and the decision register already settle. The rating rules hold however a rating reaches the store, not only through the storefront's forms (the PRD's non-functional requirement). The rater is whoever's e-mail is typed, because the store has no sign-in in this release ([CD#14]). Orders are unchanged, and eligibility is checked against the orders the store holds when a rating is given or changed ([VD#5]). No rating predates this release: every existing rating was removed before go-live ([CD#19], [CD#20]). The load generator is unchanged and never runs against Quillmark's live store ([CD#16]). Average ratings anywhere other than the book page and the catalogue list belong to the sibling slice BOOK-2-02 ([CD#10]).

## Grounding findings (architecture as-is)

All evidence is in `bookstore` at `e439178550371f2bc4f5a9acfee9a1c4f2289ae4`, the commit the slice's code grounding is pinned to and the commit this run's scan read. Paths below are relative to the repository root.

- **A rating is saved with no purchase check.** `createRating` checks only that the book and the client exist, then saves (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:72-80`, `120-132`). Ratings is configured with the clients and books URLs only (`ratings/src/main/resources/application-prod.properties:17-18`), and nothing in ratings consults orders ([CG#2]).
- **Nothing enforces one rating per customer and book.** The `ratings` table has no unique constraint (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-9`), `findByEmailAndIsbn` returns a list and is never called (`ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:16`), and a repeat POST inserts a second row ([CG#3], [CG#10]).
- **The star range is checked only in the entity setter.** `setRating` rejects values outside 1 to 5 (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:86-91`), but the no-arg constructor that request binding uses leaves the `int` at 0 (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:36-38`), so a POST or PUT that omits the stars stores 0. No bean validation or database check backs the setter, and the web inputs have no min, max or step ([CG#5]).
- **A change overwrites the whole rating.** `PUT /api/v1/ratings/{id}` saves the request body under the id after checking only that the rating exists and the ids match (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:84-97`), so it can change the e-mail and the ISBN, which the update form leaves editable (`web/src/app/ratings/update-rating/update-rating.component.html:6`, `11`). It re-checks neither client nor book ([CG#10]).
- **Ratings can be deleted one at a time or all at once.** `DELETE /api/v1/ratings/{id}` and `DELETE /api/v1/ratings/delete-all` exist (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:101-111`), the latter a native `TRUNCATE` (`ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:17-20`). The storefront's Ratings list offers Delete on every rating and Delete All (`web/src/app/ratings/rating-list/rating-list.component.html:23`, `33`) through `deleteRating` and `deleteAllRatings` (`web/src/app/ratings/rating.service.ts:40-46`) ([CG#4]). The load generator calls `delete-all` at the start of every cycle and swallows any failure (`ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:337`, `ingest/src/main/java/com/dynatrace/ingest/repository/IngestRepository.java:15-19`), and its deployment calls it once at start-up (`k8s/ingest.yaml:124`).
- **No average or count exists anywhere.** The ratings API is list, by e-mail, by ISBN, by id, create, update, delete and delete-all, and nothing computes an aggregate (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:38`, `45`, `52`, `59`, `72`, `84`, `101`, `108`). The book page loads only the book from the books service (`web/src/app/books/book-details/book-details.component.ts:18-23`), and the catalogue list loads only the books list (`web/src/app/books/book-list/book-list.component.ts:18-21`) ([CG#6]–[CG#9]).
- **The browser calls each service directly.** Each service has its own environment URL (`web/src/environments/environment.ts:7-15`), reached through an ingress path rewrite (`k8s/ingress.yaml:58-62`), and ratings allows requests from any origin (`ratings/src/main/java/com/dynatrace/ratings/config/WebConfig.java:10-16`). There is no backend for the frontend.
- **Every ratings read returns the comment.** The comment is a nullable column serialised on every `Rating` response (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:25-26`, `94-96`), shown in the Ratings list (`web/src/app/ratings/rating-list/rating-list.component.html:8`, `19`) and on the rating details screen (`web/src/app/ratings/rating-details/rating-details.component.html:16`).
- **Orders can answer the buyers-only question as it stands.** `GET /api/v1/orders/findByEmail` returns a customer's orders, and 404s for an unknown customer through `verifyClient` (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:59-65`, `221-231`); it matches the e-mail exactly (`orders/src/main/java/com/dynatrace/orders/repository/OrderRepository.java:14`), in a PostgreSQL database (`orders/src/main/resources/application-prod.properties:4`). `GET /api/v1/orders/findByISBN` returns a book's orders, and 404s for an unknown book or one not currently for sale (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:68-74`, `241-245`). An order is unique per e-mail and ISBN and carries one state flag, `completed` (`orders/src/main/java/com/dynatrace/orders/model/Order.java:11`, `30-31`). `completed` is set only once the books are taken from storage and the payment succeeds, and is reset if the payment fails (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:274-276`, `290-298`); a cancellation resets it (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:311-313`). A create carrying `completed: true`, or an update that moves an order into `completed`, runs that same purchase flow (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:93-95`, `128-131`). An update over an order that is already completed runs neither branch and saves the body as sent, its `email` and `isbn` included (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:128-137`), and the storefront's update-order form leaves both editable and sends that update (`web/src/app/orders/update-order/update-order.component.html:6`, `11`; `web/src/app/orders/update-order/update-order.component.ts:18-19`). So `completed == true` is [CD#1]'s qualifying order for every order that has not since been moved to another customer or book that way ([CG#2]; Edge cases & risks).
- **Ratings' outbound calls have no timeouts and handle only a 404.** Each client class builds its own `RestTemplate` and maps a 404 to `null` (`ratings/src/main/java/com/dynatrace/ratings/repository/BookRepository.java:19-21`, `35-37`); any other failure propagates. Ratings has no orders client, but `DT_ORDERS_SERVER` is in the shared configmap (`k8s/configmap.yaml:15`), which the ratings pod loads whole (`k8s/ratings.yaml:32-36`).
- **The load generator is not a monitor.** It posts ratings for a random client and book, unrelated to any order, and logs and ignores a refused create (`ingest/src/main/java/com/dynatrace/ingest/repository/RatingRepository.java:40-51`), back to back while it runs, with no schedule ([CG#11]).

## Architecture decisions

### [AD#1]: The ratings service enforces every rating rule

- **Binds:** every write path of bookstore:ratings — `POST /api/v1/ratings`, `PUT /api/v1/ratings/{id}`, the `DELETE` endpoints and any write path added later — and what bookstore:web may rely on for a rule.
- **Prevents:** a rule held only by a web form or screen, which any direct caller of the API bypasses; today no rule beyond book and client existence is checked on create, and the star range only in a setter that an omitted value skips ([CG#2], [CG#5]).
- **Rule:** Buyers only ([AD#2]), one rating per e-mail and ISBN ([AD#3]), whole stars from 1 to 5 and a fixed e-mail and ISBN ([AD#4]), and single-rating removal only ([AD#5]) are each enforced by bookstore:ratings on every write it accepts, whoever sends it; a constraint in bookstore:web — the 1-to-5 input, the read-only e-mail and ISBN, the absent Delete buttons — is a convenience for the customer and is never the only guard of a rule.
- **Alternatives:** Enforce the rules in the web forms only — lost: the PRD's non-functional requirement holds them however a rating reaches the store, and the load generator already calls the ratings API directly (`ingest/src/main/java/com/dynatrace/ingest/repository/RatingRepository.java:40-51`). Enforce them in front of the service — lost: the ingress only rewrites paths (`k8s/ingress.yaml:58-62`), and uniqueness and eligibility need ratings' own data and its call to orders.

### [AD#2]: Purchase eligibility is read from orders at write time

- **Binds:** bookstore:ratings' give and change paths — a POST that creates, a POST applied as a change under [AD#3], a PUT — and bookstore:orders' `GET /api/v1/orders/findByEmail` and `GET /api/v1/orders/findByISBN` as their only source of purchase state.
- **Prevents:** a rating given or changed without a qualifying order as orders records it ([BR#1], [CD#1]); a copy of purchase state in ratings that drifts from orders; a real buyer told they did not buy the book because orders could not answer, or because they typed their e-mail with different capitals from their order's.
- **Rule:** Before saving a give or a change, bookstore:ratings calls `GET /api/v1/orders/findByEmail?email=<EMAIL>` on bookstore:orders, with a bounded timeout, and accepts when the response holds an order whose `isbn` is the rating's ISBN and whose `completed` is `true`. Where that response holds no such order, or orders answers it `404`, ratings calls `GET /api/v1/orders/findByISBN?isbn=<ISBN>` and accepts when that response holds an order whose e-mail equals the rating's e-mail once both are lowercased and whose `completed` is `true`. No qualifying order from either call — a `404` from the second included — is the buyers-only refusal of [AD#4]. A timeout, a connection failure or any other non-2xx response from either call, other than those two `404`s, saves nothing, leaves an existing rating exactly as it was, and is answered `503`. Both calls are reads and repeat no side effect. Eligibility is checked only at that moment: a later cancellation or deletion of the order changes no saved rating ([VD#5], [AC#10]), and bookstore:ratings stores no purchase data of its own.
- **Alternatives:** `GET /api/v1/orders/findByEmail` alone — lost: orders matches the e-mail exactly (`orders/src/main/java/com/dynatrace/orders/repository/OrderRepository.java:14`), so a buyer who types different capitals is refused. `GET /api/v1/orders/findByISBN` alone — lost: every write pays the book's whole order list, which grows with its popularity, and a buyer of a book since taken off sale is refused (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:241-245`); it is kept only for the miss. Orders matching the e-mail ignoring letter case, or a new orders endpoint by e-mail and ISBN — lost: each changes bookstore:orders, which [VD#5] keeps unchanged in this release. Ratings reading the orders service's database directly — lost: it couples ratings to another service's schema. A copy of each purchase kept in ratings — lost: [VD#5] rejects duplicating the orders service's data for one edge case. When orders cannot answer, the same `403` as a refusal — lost: it tells a real buyer during an outage that they did not buy the book. When orders cannot answer, saving the rating unverified — lost: it breaks [SM#1]'s 100% target.

### [AD#3]: One rating per customer e-mail and ISBN, held by the database

- **Binds:** bookstore:ratings' `ratings` table and its create path.
- **Prevents:** a second rating for one customer and book ([BR#2], [SM#2]), whether from a repeated add or from two adds racing each other; a repeat add refused rather than applied as a change ([AC#7]).
- **Rule:** The `ratings` table carries a unique constraint on (`email`, `isbn`), two e-mails that match once lowercased being one customer, and two adds for one pair, concurrent or not, leave exactly one rating: an add for a pair that already holds a rating is applied to it as a change, as [AD#4] states. The constraint is the guarantee, and no write path relies on a lookup alone.
- **Alternatives:** A lookup by e-mail and ISBN before insert, with no constraint — lost: two concurrent adds both find nothing and both insert. The constraint, with a repeat add refused as a conflict — lost: [AC#7] has a repeat add change the existing rating.

### [AD#4]: The ratings write API's behaviour

- **Binds:** `POST /api/v1/ratings` and `PUT /api/v1/ratings/{id}` as bookstore:web calls them (`web/src/app/ratings/rating.service.ts:32-38`), and every response web relies on from them.
- **Prevents:** web and ratings disagreeing on what a refused, invalid or failed write returns; a refusal that tells a caller whether an e-mail is a Quillmark customer or what they bought; a change that moves a rating to another customer or book ([VD#4]).
- **Rule:** A `POST` for an e-mail and ISBN that hold no rating creates one; a `POST` for a pair that already holds a rating is applied to that rating as a change, exactly as a `PUT` of it would be, so a repeat add never creates a second rating ([AD#3]). A successful give or change answers `200` with the saved rating, without its comment ([AD#7]). E-mails are compared once lowercased, here as in [AD#2] and [AD#3]. On a give or a change, a missing star value, or one that is not a whole number from 1 to 5, is answered `400`; a `PUT` whose body carries an `email` or `isbn` different from the stored rating's is answered `400`, while one that repeats them or leaves them out proceeds; a `PUT` for an id that holds no rating, or whose body id differs from the path's, is answered as today, `404` and `400` (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:87-96`); a buyers-only refusal ([AD#2]) is answered `403` with one identical body for every reason — an unknown e-mail, an unknown ISBN, no order for the book, an unpaid order, a cancelled or deleted order — and the reason is written only to ratings' own log; orders failing to answer is `503` ([AD#2]). Each of these failures saves nothing and leaves an existing rating exactly as it was. The `400` and `404` checks — the stars, a `PUT`'s id, a changed `email` or `isbn` — are decided before any call to orders: none of them depends on what an e-mail bought, and a write refused with one makes no call to orders. On a change, through `PUT` or through a `POST` applied as one, only the stars and the comment change: an absent or empty comment keeps the stored comment, and a non-empty one replaces it ([AC#30]). Repeating an identical give or change leaves the same single rating with the same values.
- **Alternatives:** A `403` carrying a reason code — lost: with no sign-in ([CD#14]), anyone typing e-mails could use it to learn whether an e-mail with no qualifying order is a Quillmark customer at all, and the state of an order — unpaid, cancelled or deleted — which the identical `403` keeps hidden; whether a typed e-mail bought a book stays learnable by other means, accepted in Edge cases & risks. Keeping today's `404` for an unknown e-mail or book (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:120-132`) — lost: it discloses whether an e-mail is a customer. On a change, ignoring a different `email` or `isbn` — lost: the caller would see success for a move that did not happen.

### [AD#5]: No shopper deletion; operators remove one rating at a time

Seeded from [VD#1].

- **Binds:** bookstore:ratings' `DELETE` endpoints and bookstore:web's Ratings list.
- **Prevents:** a book's average and count moving for any reason but a customer giving or changing their own rating, or an operator removing one rating ([VD#1], [CD#6]); a bulk wipe of every rating (`ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:17-20`).
- **Rule:** `DELETE /api/v1/ratings/delete-all` no longer exists — a request to it is answered `404` and removes nothing — and no route removes more than one rating per request; `DELETE /api/v1/ratings/{id}` stays as the operators' removal path for erasure requests and fraud ([AC#21]), answers as today on a removal, and answers `404` for an id that holds no rating, so repeating a removal removes nothing further and answers `404`; no bookstore:web screen calls either. With no sign-in the per-rating path is reachable by anyone who calls the API, a risk [VD#1] accepts until sign-in exists.
- **Alternatives:** Also build an operator removal of every rating of one customer, and change the load generator's deletion paths — lost: no requirement asks for it, and removing one customer's ratings is the per-rating removal repeated ([VD#1]). Keep bulk delete as an operator-only maintenance action — lost: without sign-in nothing tells an operator's request from a shopper's ([VD#1]). Leave deletion as it is today — lost: any shopper can delete any rating, or all of them ([CG#4]).

### [AD#6]: A book's rating summary is computed by ratings on each read

- **Binds:** bookstore:ratings' two new summary endpoints, and bookstore:web's book page and catalogue list.
- **Prevents:** the book page and the catalogue list showing different averages for one book ([AC#19]); a stale figure after a rating is given, changed or removed ([AC#16], [AC#20]); the rounding rule implemented twice; raters' e-mails and comments pulled into the browser to compute a figure.
- **Rule:** `GET /api/v1/ratings/summary?isbn=<ISBN>` returns `{isbn, count, average}` for one book, and `GET /api/v1/ratings/summaries` returns one such entry for every book holding at least one rating. `count` is the number of stored ratings for the ISBN, and `average` their mean rounded half up to one decimal place from the exact quotient of the star sum by the count, so 4.25 is 4.3 ([VD#3]); `average` is `null` when `count` is 0. Both are computed from the stored ratings on each request, with no stored or cached aggregate, so a write is reflected on the next read. Neither response carries an e-mail or a comment. A `summary` request with a missing or blank `isbn` is answered `400`; an `isbn` that holds no rating, whether or not the book exists, answers `count` 0 and `average` `null`. A consumer shows "Not yet rated" only for a count of 0, or for a book absent from a successful `summaries` response, and never for a request that failed.
- **Alternatives:** A per-book sum and count stored in ratings and updated on every write — lost: every write path, deletion included, has to keep it consistent, for a read cost the catalogue does not need yet. The books service adding the figures to its book responses — lost: it adds a books-to-ratings dependency and a fourth changed component. The web computing them from `GET /api/v1/ratings/findByISBN` — lost: one call per book on the catalogue list, each returning raters' e-mails and comments.

### [AD#7]: Comments are write-only on the ratings API

- **Binds:** every bookstore:ratings response — the reads `GET /api/v1/ratings`, `/findByEmail`, `/findByISBN`, `/{id}` and [AD#6]'s two summary endpoints, and the responses to `POST` and `PUT` — and bookstore:web's Ratings list and rating details screen.
- **Prevents:** a comment reaching a shopper through the API although no screen shows it, which would break the form's promise "Only Quillmark sees your comment" ([AC#5], [AC#12], [CD#17]); today every read returns it (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:94-96`), and so does every write, which answers with the saved rating (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:80`, `97`) — a change that keeps the stored comment would hand it to whoever sent the change.
- **Rule:** `POST` and `PUT` accept a rating's comment and bookstore:ratings stores it, but no bookstore:ratings response carries it — no read and no write response; every other part of the read responses, their failure statuses included, is unchanged from today. Quillmark reads comments in the ratings database, and this release builds no operator screen or endpoint for them.
- **Alternatives:** Keep returning comments and hide them in the web only — lost: with no sign-in any shopper can call the API, so the promise would not hold. A separate operator-only read endpoint — lost: without sign-in it cannot be kept from shoppers, so it would be public.

## Cross-repo / component approach

Every rating rule lives in bookstore:ratings ([AD#1]). bookstore:web changes only what it shows and sends, and bookstore:orders changes no code: it is the producer of an interface that already exists ([AD#2]).

| Capability | Components | Decisions |
|---|---|---|
| Buyers only on give and change ([US#1], [US#2]) | bookstore:ratings (the check), bookstore:orders (read, unchanged), bookstore:web (shows [AC#2]'s text on the refusal) | [AD#1], [AD#2], [AD#4] |
| One rating per customer and book; a repeat add is a change ([US#2]) | bookstore:ratings, bookstore:web (add form) | [AD#3], [AD#4] |
| Whole stars from 1 to 5 ([US#1], [US#2]) | bookstore:ratings (the guard), bookstore:web (a 1-to-5 input) | [AD#1], [AD#4] |
| A fixed e-mail and ISBN on a change ([US#2], [US#5]) | bookstore:ratings (the guard), bookstore:web (read-only fields on the update form) | [AD#4] |
| No shopper deletion; operators remove one rating ([US#5]) | bookstore:ratings, bookstore:web (the Ratings list loses Delete and Delete All) | [AD#5] |
| A book's average and count on its page, and its average on the catalogue list ([US#3], [US#4]) | bookstore:ratings (the summaries), bookstore:web (book page and catalogue list) | [AD#6] |
| Comments kept and shown to no shopper ([US#7]) | bookstore:ratings (write-only), bookstore:web (the list and details screen drop the comment; the forms carry the note) | [AD#7] |

## Contracts

| AD | Producer | Consumers | Kind | Status | Artifact |
|---|---|---|---|---|---|
| [AD#2] | bookstore:orders | bookstore:ratings | REST | exists | — |
| [AD#4] | bookstore:ratings | bookstore:web | REST | changed | — |
| [AD#5] | bookstore:ratings | bookstore:web | REST | changed | — |
| [AD#6] | bookstore:ratings | bookstore:web | REST | new | — |
| [AD#7] | bookstore:ratings | bookstore:web | REST | changed | — |

No contract is a code file: the services publish no OpenAPI or schema file, and each side keeps its own model of the shapes it reads.

### Schema ownership

- bookstore:ratings owns the rating (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-38`), the refusal body and the summary shape. bookstore:web's `Rating` model (`web/src/app/ratings/rating.ts:1-8`) is a consumer copy, which loses its comment on read under [AD#7].
- bookstore:orders owns `Order` (`orders/src/main/java/com/dynatrace/orders/model/Order.java:11-31`). bookstore:ratings reads only `email`, `isbn` and `completed` from it, through a consumer-side copy of its own.

### Versioning and compatibility

- `/api/v1/ratings` changes in place, with no `/v2`: every caller of it is in this repository — bookstore:web (`web/src/app/ratings/rating.service.ts:12`), which moves in the same release, and the load generator (`ingest/src/main/resources/application-prod.properties:46`, `k8s/ingest.yaml:124`), which does not and tolerates the failures (below).
- [AD#2] (exists): ratings depends only on `email`, `isbn` and `completed` in both responses and ignores every other field, so an additive change to orders' responses is safe. Renaming or removing `completed`, or changing what sets it, is a breaking change for ratings.
- [AD#4], [AD#5] and [AD#7] (changed): web stops calling `delete-all`, offers no Delete, and stops showing comments in the same release. The load generator keeps its calls and tolerates their failure (Edge cases & risks).
- [AD#6] (new): additive.

### Landing order

1. bookstore:orders — the producer of [AD#2], which exists; nothing lands.
2. bookstore:ratings
3. bookstore:web

## Stack & invariants

- Java 21 (`build.gradle:121-124`) and Spring Boot 3.5.3 (`build.gradle:3`), so `jakarta.*` throughout.
- bookstore:ratings runs on MySQL (`build.gradle:84`), database `dt_books_ratings` (`k8s/ratings.yaml:44-45`). Hibernate generates its schema with `ddl-auto=update` (`ratings/src/main/resources/application-prod.properties:9`); there is no migration tool.
- Services call each other with `RestTemplate` at URLs of the form `http://${DT_BOOKS_SERVER:localhost:8082}/api/v1/books` (`ratings/src/main/resources/application-prod.properties:17-18`), filled from the `DT_*_SERVER` entries of `bookstore-configmap` (`k8s/configmap.yaml:11-18`). The orders URL follows the same pattern with `DT_ORDERS_SERVER`, already in the configmap (`k8s/configmap.yaml:15`).
- Error statuses come from the shared exceptions module: `PurchaseForbiddenException` is `403` (`exceptions/src/main/java/com/dynatrace/exception/PurchaseForbiddenException.java:7`) and `BadRequestException` is `400` (`exceptions/src/main/java/com/dynatrace/exception/BadRequestException.java:7`).
- bookstore:web is Angular 21 (`web/package.json:16`) and calls each service directly at its environment URL (`web/src/environments/environment.ts:7-15`), under the base path `/api/v1/ratings` for ratings (`web/src/app/ratings/rating.service.ts:12`), which stays.
- E-mails are case-insensitive: every comparison of two e-mails on the rating path is made once both are lowercased ([AD#2], [AD#3], [AD#4]).
- The store has no sign-in anywhere in this release ([CD#14]).

## Edge cases & risks

- **The unique constraint can be missing without anyone noticing.** Under `ddl-auto=update`, Hibernate adds a new unique constraint to an existing table but cannot add it over duplicate rows, and it logs the failure and starts anyway. The live store's table is empty ([CD#19]), but a test environment the load generator ran in holds duplicate pairs ([CG#3]). The release has to make sure the constraint exists in each environment before [AD#3] can be relied on (Deferred).
- **Every give and change now costs a call to orders, and through it two to clients; a miss costs a second.** `findByEmail` runs `verifyClient`, which fetches the client and then every client (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:63`, `221-231`), and where it finds no qualifying order [AD#2] calls `findByISBN`, which fetches the book's whole order list and then every book (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:72-73`, `246`). Every refusal pays both. That is accepted to keep orders unchanged ([VD#5]); a lighter lookup would be an orders change for a later release.
- **One letter-case case is still refused.** E-mails are compared ignoring letter case, but orders matches them exactly, so [AD#2] reaches an order stored with different capitals only through `findByISBN`, which answers `404` for a book not currently for sale (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:241-245`). A buyer who types different capitals from their order's, for a book since taken off sale, is refused with `403` and counts in [SMC#1]. Accepted under [VD#5]; closing it is an orders change.
- **A paid order can be moved to another customer or book and still read as completed.** An update over an order that is already completed saves its `email` and `isbn` as sent, with no new payment, and the storefront's update-order form offers exactly that (`orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:128-137`; `web/src/app/orders/update-order/update-order.component.html:6`, `11`). [AD#2] would then accept a rating for a book the customer did not pay for, and such a rating counts against [SM#1]'s 100% target. This is accepted under [VD#5], which keeps orders unchanged in this release; closing it is an orders change.
- **A wrong orders URL would refuse every buyer.** Ratings gains its first orders URL in this release, and if that URL or path is wrong in an environment, orders answers `404` to every lookup, which [AD#2] reads as an unknown customer: every give and change is then refused with `403` and [AC#2]'s text. How ratings tells orders' unknown-customer `404` from a wrong route's is deferred to `/dev-workflows:design`.
- **Orders becomes a dependency of rating writes, not of reads.** While orders cannot answer, no rating can be given or changed ([AD#2]'s `503`), but the book page and the catalogue list keep working, because [AD#6] does not call orders.
- **The load generator is unchanged ([CD#16]).** Its `delete-all` calls fail and are swallowed (`ingest/src/main/java/com/dynatrace/ingest/repository/IngestRepository.java:15-19`), and the start-up `curl` at `k8s/ingest.yaml:124` sits in an `&&` chain, but plain `curl` exits 0 on an HTTP error status, so the chain carries on. Most of its random ratings are refused under [AD#2] and logged (`ingest/src/main/java/com/dynatrace/ingest/repository/RatingRepository.java:40-51`), yet each refusal still costs both orders calls of [AD#2], which adds load in test environments. The few it gets accepted accumulate, because nothing wipes the table any more. None of this reaches the live store, where the generator never runs ([CD#16]).
- **Accepted exposures, all from the typed-e-mail identity.** Anyone can type a buyer's e-mail and rate, or change the rating of, a book that buyer bought ([CD#14]). The operators' per-rating `DELETE` is reachable by anyone who calls the API ([VD#1], [AD#5]). Raters' e-mails stay visible on the Ratings list and in the ratings read API ([VD#6]); the summaries of [AD#6] carry none. Under buyers-only, each visible rating also shows that its rater's e-mail bought that book, and a `200` to a give for a typed e-mail and ISBN shows the same; this disclosure is accepted as part of the typed-e-mail exposure [VD#6] accepts. Because the per-rating `DELETE` is open to anyone ([AD#5]), such a give can then be removed, so whether a typed e-mail bought a book can be learned without leaving the rating behind; that combination is accepted under [CD#14] and [VD#1].
- **Rounding must start from the exact quotient.** A floating-point mean can land just below an exact tie, so the average is rounded half up from the star sum and the count ([AD#6]), and 4.25 shows as 4.3 ([VD#3]).
- **`summaries` returns every rated book in one response**, so it grows with the catalogue.
- **The deploy window between ratings and web.** Once ratings lands and before web does, the old web's Delete All fails and its Comment column is empty. That is acceptable for the length of a deploy, and the landing order keeps the new web from calling a summary endpoint that does not exist yet.

## Open questions

None. No repository was descoped, architecture grounding was off for this run, and the grill found no contradiction with a settled decision in the register. The gaps the reviews raised were settled by the architect, each recorded in Edge cases & risks: a paid order moved to another customer or book, and the one letter-case case still refused, accepted under [VD#5]; purchases disclosed through visible ratings, accepted under [VD#6], and the give-then-remove probe, under [CD#14] and [VD#1]; and e-mails compared ignoring letter case, by lowercasing ([AD#2]–[AD#4]).

## Deferred

- **To `/product-workflows:specify`:** what the web says when ratings answers `503` ([AD#2]) and when a summary request fails ([AD#6]) — neither text is in the PRD. The detail of the star input that [VD#2] decides at implementation altitude.
- **To `/dev-workflows:design`:**
  - The orders call's timeout value, and a configured `RestTemplate` for it in place of today's per-class construction.
  - How ratings tells orders' unknown-customer `404` from a `404` for a wrong orders URL or path ([AD#2]), for example by checking the orders URL at start-up or readiness, or by matching orders' not-found response.
  - How the unique constraint of [AD#3] is created and verified in each environment, including emptying a test environment's ratings before the deploy.
  - Whether ratings keeps its own client and book existence calls on a write, now that orders' `findByEmail` checks the client.
  - How a create that loses the race on the constraint is applied as a change ([AD#3]).
  - How the lowercase comparison of e-mails is applied to the constraint of [AD#3] and the changed-`email` check of [AD#4] — the e-mail stored lowercased, or a comparison that lowercases both sides.
  - The `404` [AD#5] fixes for the removed `delete-all` path needs an explicit route or handler: with the mapping removed, `DELETE /api/v1/ratings/delete-all` falls to `DELETE /{id}`, which answers `400` for an id that is not a number, and the service has no exception handler of its own to change that.
  - The summaries query, and the web's join of the summaries onto the catalogue list ([AD#6]).
  - Paging `summaries` if the catalogue grows.

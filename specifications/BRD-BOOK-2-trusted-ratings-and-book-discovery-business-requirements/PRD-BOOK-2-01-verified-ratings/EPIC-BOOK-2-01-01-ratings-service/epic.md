---
kind: epic
key: BOOK-2-01-01
target: bookstore:ratings
kept_whole: "It's a small change, and that's my decision to keep one Epic per module"
---

# The ratings service counts only buyers' ratings, one per customer and book

## Goal
The ratings service enforces, for every caller, that a rating counts only when it is 1 to 5 whole stars from a customer who bought the book, held once per customer and book, so that shoppers can trust the stars in Quillmark's store (PRD [[BOOK-2-01]], "Verified ratings").

## Business value
Shoppers ignore the store's star ratings today because nothing makes them trustworthy. With every rule held in the ratings service, no direct API caller can bypass the buyers-only, one-rating and star-scale rules, and the service supplies the averages and counts the book page and catalogue list need.

## Scope

### In scope
- Buyers-only check on every give and change: before saving, call orders `GET /api/v1/orders/findByEmail?email=<EMAIL>` with a bounded timeout, with the e-mail exactly as typed, and accept only when the response holds an order whose `isbn` is the rating's ISBN and whose `completed` is `true`; a consumer-side copy of the order reading only `email`, `isbn` and `completed`; an orders URL property in each profile property file of ratings ([AD#2]).
- One identical `403` body for every buyers-only refusal (unknown e-mail, unknown ISBN, no order for the book, unpaid order, cancelled or deleted order), with the reason written only to ratings' own log; `503` when orders times out, cannot be reached or answers with a non-2xx other than the unknown-customer `404` ([AD#2], [AD#4]).
- One rating per e-mail and ISBN: a unique constraint on (`email`, `isbn`) compared ignoring letter case, and a `POST` for a pair that already holds a rating applied to it as a change, including when two adds race ([AD#3]).
- The write API's behaviour: `200` with the saved rating on success; `400` for a missing or out-of-scale star value, a missing or malformed `email` or `isbn` on `POST`, a `PUT` carrying a different `email` (compared ignoring letter case) or `isbn`, and a body id that differs from the path's, each `400` answer carrying its reason, word for word, in the `message` field of the error body (the field the storefront's forms read today), including a reason raised while the request body is read; `404` for an unknown id; these `400` and `404` checks decided before any call to orders; on a change only the stars and the comment change, the stored e-mail stays as stored, and an empty or absent comment keeps the stored one ([AD#4]).
- Removal: `DELETE /api/v1/ratings/delete-all` and its truncating repository method removed, so the request reaches `/{id}` and is refused `400`; `DELETE /api/v1/ratings/{id}` unchanged ([AD#5]).
- `GET /api/v1/ratings/summary?isbn=<ISBN>` and `GET /api/v1/ratings/summaries`, each entry `{isbn, count, average}`, computed on each request, with the average rounded half up to one decimal place from the exact quotient of the star sum by the count ([AD#6]).
- Comments accepted and stored on `POST` and `PUT`, and left out of every ratings response ([AD#7]).
- Automated tests for the above, with the orders call stubbed (for example with `MockRestServiceServer`) and the case-insensitive pair compared the way the ratings database compares it.

### Out of scope
- Any change to bookstore:orders, the producer of [AD#2], which exists and does not change.
- Any change to the load generator (ingest); its `delete-all` calls fail and are tolerated.
- Cleaning up existing ratings and any migration tool or start-up check for the constraint; each environment starts on an emptied ratings table, done by operators at deploy.
- Any screen or endpoint for operators to read comments; Quillmark reads them in the ratings database.
- An operator removal of every rating of one customer; that is the per-rating removal repeated.
- Averages anywhere other than the two summary endpoints ([[BOOK-2-02]]), sign-in, notifications, a comment length limit, paging of `summaries`, and logging of `503` refusals.
- Every storefront screen and text; those belong to [[BOOK-2-01-02]].

## Acceptance criteria
- Given an e-mail for which orders reports no completed order for the ISBN (including an e-mail orders does not know), when a give or a change reaches the ratings service, then it answers `403` with one identical body whatever the reason, saves nothing, leaves any existing rating exactly as it was, and writes the reason to its own log.
- Given an otherwise valid give or change for an e-mail with a completed order for the ISBN, when it reaches the ratings service, then it answers `200` with the saved rating (without its comment, as below).
- Given orders times out, cannot be reached or answers with an error other than not-found, when a give or a change reaches the ratings service, then it answers `503`, saves nothing and leaves any existing rating exactly as it was.
- Given a give or a change with no star value or one that is not a whole number from 1 to 5, a give with a missing or malformed e-mail, or a give with a missing ISBN or one that is not 13 digits, when it reaches the ratings service, then it answers `400` (with the reason "Invalid rating. Must be a whole number from 1 to 5.", "Invalid email address" or "ISBN must be a 13-digits value" as it applies, word for word in the `message` field of the error body, including a reason raised while the request body is read), saves nothing, and makes no call to orders.
- Given an e-mail and ISBN that already hold a rating (compared ignoring letter case), when a give reaches the ratings service, then it is applied to that rating as a change under every rule of a change.
- Given an e-mail and ISBN that hold no rating or already hold one (compared ignoring letter case), when simultaneous gives for that pair reach the ratings service, then exactly one rating is left for the pair.
- Given a change, when it carries an e-mail other than the stored rating's (compared ignoring letter case) or an ISBN other than the stored rating's, then the ratings service answers `400` and leaves the rating exactly as it was.
- Given a change, when it carries the stored e-mail in other capitals, then the stored e-mail stays as stored.
- Given a saved change, when its comment is empty, absent or only spaces, then the stored comment is kept; when it holds a character other than a space, then it replaces the stored comment.
- Given a saved change, then the rating's Updated At is set to the time of the change (an identical repeat included) and its Created At is left as first given.
- Given a rating, when a request to `DELETE /api/v1/ratings/delete-all` arrives, then the ratings service refuses it `400` and removes nothing.
- Guard: when `DELETE /api/v1/ratings/{id}` is requested, then the ratings service removes that rating and no other and answers `200` with an empty body, and an id that holds no rating removes nothing and answers the same `200`.
- Given stored ratings, when a rating summary is requested for one ISBN or for all books, then the service answers `{isbn, count, average}` computed from the stored ratings on that request, with the average rounded half up to one decimal place from the exact quotient (so 4.25 is 4.3), `null` when the count is 0, and no e-mail or comment; `summaries` holds one entry for every book that holds at least one rating.
- Given a summary request for an ISBN that is missing or blank, then the ratings service answers `400`.
- Given any ratings response, when it is a read, a `POST` answer or a `PUT` answer, then it carries no comment.
- Given a comment given with a give or a change, when the rating is stored, then the comment is stored with the rating.
- Guard: a rating given or changed stays and keeps counting in its book's summary when the qualifying order is cancelled or deleted afterwards.

## Independent Test
This Epic is verifiable standalone by calling the ratings API directly with the orders service stubbed (a buyer, a non-buyer, an unknown customer and an unavailable orders), and delivers rating rules that hold for every caller, with correct summaries, without any not-yet-built Epic.

## Dependencies
- bookstore:orders (outside this PRD, no Epic): the existing `GET /api/v1/orders/findByEmail`, used as it runs; ratings needs `email`, `isbn` and `completed` from it. Orders is unchanged in this release.
- The existing clients and books services, which ratings already calls on a create; whether ratings keeps those calls on a write is a design decision.
- Deploy step by operators, outside this Epic's code: stop the old ratings service and, where it runs, the load generator, empty the `ratings` table, then start the new ratings so the unique constraint is created on an empty table; operators confirm from the table's indexes that the constraint exists before the rules are relied on in an environment.
- Operator-set configuration: `DT_ORDERS_SERVER` already exists in `bookstore-configmap`, which the ratings pod loads whole.
- [[BOOK-2-01-02]] consumes what this Epic produces; this Epic does not depend on it, and lands first.

## Contract
- Produces: [AD#4] — the changed write API: statuses, one identical `403`, `503`, fixed e-mail and ISBN on a change, comment kept when empty, the saved rating returned without its comment.
- Produces: [AD#5] — `delete-all` removed (refused `400`), per-id `DELETE` unchanged.
- Produces: [AD#6] — the new `summary` and `summaries` endpoints.
- Produces: [AD#7] — no ratings response carries the comment.
- Consumes: [AD#2] — exists — used as it runs (orders `findByEmail`).

## Covers
- Stories, ratings side: [US#1], [US#2], [US#3], [US#4], [US#5], [US#7]
- [AC#1], [AC#2], [AC#4], [AC#7], [AC#8], [AC#9], [AC#10], [AC#13], [AC#14], [AC#16], [AC#19], [AC#20], [AC#21], [AC#23], [AC#24], [AC#30], [AC#12]
- [SM#1], [SM#2], [SM#3], [SMC#1], [UC#1], [UC#2], [UC#3], [UC#4]
- [U01], [U01/AC01], [U01/AC02], [U01/AC04], [U01/AC07], [U01/AC08], [U01/AC10], [U01/AC15]
- [U02], [U02/AC01], [U02/AC02], [U02/AC03], [U02/AC04], [U02/AC05], [U02/AC06], [U02/AC07], [U02/AC09], [U02/AC10], [U02/AC11], [U02/AC12], [U02/AC21]
- [U03], [U03/AC06], [U03/AC07], [U03/AC08], [U04], [U04/AC06], [U05], [U05/AC05], [U05/AC06], [U06], [U06/AC01], [U06/AC02], [U06/AC03]

## Suggested stories
- Refuse ratings from non-buyers: the orders client and eligibility decision, the identical `403`, the `503` when orders cannot answer, and the logged reason.
- Hold one rating per customer and book: the unique constraint, a repeat add applied as a change, and the fixed e-mail and ISBN, empty-comment and timestamp rules of a change.
- Enforce the scale and the shape of a write for every caller: whole stars 1 to 5, missing or malformed e-mail and ISBN, and `400` and `404` checks before any orders call.
- Answer rating summaries: `summary` and `summaries`.
- Keep comments write-only: no comment in any response.
- Retire `delete-all`: refused `400`, per-id `DELETE` unchanged.

## References
- Parent PRD: [[BOOK-2-01]]
- [Source: specs/specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/ard.md#Architecture decisions] — [AD#1] to [AD#7]
- [Source: specs/specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/specification.md] — storefront-independent ratings criteria, exact reason texts
- [Source: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java] — create, update, delete and delete-all handlers
- [Source: ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java] — `findByEmailAndIsbn`, truncate
- [Source: ratings/src/main/java/com/dynatrace/ratings/repository/BookRepository.java] — client-class pattern for the orders client
- [Source: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java] — entity, star setter, comment serialisation
- [Source: orders/src/main/java/com/dynatrace/orders/controller/OrderController.java] — `findByEmail`

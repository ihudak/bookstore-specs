---
kind: specification
key: BOOK-2-01-01
---

# Specification

- **Feature name**: Ratings service: buyers-only ratings, one per customer and book
- **Version**: 1
- **Created**: 2026-10-13
- **Author**: ihudak
- **Published**: no
- **Open questions**: 2

## Problem statement

Quillmark's ratings service accepts any rating from anyone who calls it. It does not check that the rater bought the book, it lets one customer rate the same book again and again, and a rating sent without a star value counts as zero, below the scale shoppers see. A change to a rating can move it to another customer or another book, and it loses the time the rating was first given. Every answer the service gives hands back the rating's comment, although Quillmark does not publish reviews, and one request removes every rating at once. The storefront is not the service's only caller: the store's test-traffic generator, and anyone who knows its address, call it directly, so a rule held only in the storefront's forms would not hold. Nothing computes a book's average rating or how many ratings it rests on, so the book page and the catalogue list have nothing trustworthy to show. Quillmark's shoppers ignore its ratings for these reasons, and Quillmark wants the next release of its store to count only buyers' ratings, once per customer and book, and to show each book's average and count from those ratings alone.

## Scope

**In scope**

- Buyers only: the ratings service saves a give or a change only when orders reports a qualifying order for the book under the e-mail at that moment; every refusal for that reason gets one identical answer, and its reason and ISBN are written to the ratings service's own log.
- One rating per customer and book, the e-mail compared ignoring letter case: a give for a pair that already holds a rating changes it, also when gives for the pair arrive at the same time.
- Whole stars from 1 to 5 on every give and change, and a well-formed e-mail and a 13-digit ISBN on every give, each refused with its stated reason before any purchase check, whoever sends the rating.
- A change alters only the stars and the comment: the customer e-mail and the ISBN stay fixed, an empty comment keeps the stored one, Created At stays, and Updated At moves to the time of the change.
- While orders cannot answer the purchase check, a give or a change is refused as unavailable and nothing is saved.
- A book's rating summary, its rating count and its average rounded half up to one decimal place, for one book and for every rated book, computed from the stored ratings on each request and naming no rater.
- Comments stored with their rating and returned by no answer of the ratings service.
- Single-rating removal kept as it is, as the operators' path for erasure requests and fraud; the request that removed every rating at once is gone.

**Out of scope**

- Every storefront screen, text and form behaviour, including what the storefront shows for each answer of the ratings service: [BOOK-2-01-02](../EPIC-BOOK-2-01-02-storefront-ratings/).
- Any change to orders, including matching a typed e-mail to an order's e-mail regardless of letter case, or a lighter purchase lookup ([VD#5]).
- Restricting single-rating removal to operators: until the store has sign-in, anyone who calls the ratings service can reach it ([VD#1], [CD#14]).
- Removing every rating of one customer in one request: an operator removes each of them in turn ([VD#1]).
- An operator screen or endpoint for reading comments; Quillmark reads them where the ratings are stored ([AD#7]).
- Emptying the stored ratings in each environment when the release is deployed, and confirming there that one rating per customer and book is enforced, which are operator steps of the release (`ard.md`, Edge cases & risks).
- Any change to the store's test-traffic generator, whose requests to remove every rating now fail and are tolerated ([CD#16]).
- Rating summaries anywhere other than the two summary requests of [U03], and paging the summaries of every book ([CD#10]).
- A limit on a comment's length: a give or a change whose comment is longer than the ratings store holds saves nothing, and what the ratings service answers it is not set in this release (PRD-level specification, Scope).
- A customer clearing a stored comment: an empty comment, or one of spaces only, on a change keeps the stored one ([U02] [AC05]), and an erasure request removes the whole rating ([U05]).
- Logging a refusal as unavailable, or the orders failure behind it: only a buyers-only refusal's reason is logged.
- Response-time or load targets for a give, a change or a summary: none is set in this release, and the extra purchase check on every write is accepted (`ard.md`, Edge cases & risks).
- Sign-in, or any check that the person typing an e-mail owns it ([CD#14]).

## User stories

### [U01]: Rate a book I bought

As a customer who bought a book, I want to give it a star rating, so that other shoppers can see what buyers think of it.

Done when a give from a customer with a qualifying order for the book is stored, and every other give, whoever sends it, is refused with nothing stored.

Traces to: [US#1]; [AC#1], [AC#2], [AC#4]; PRD-level specification [U01/AC01], [U01/AC02], [U01/AC04], [U01/AC07], [U01/AC08], [U01/AC10], [U01/AC15]; [VD#2]; [CG#2], [CG#5]; [AD#1], [AD#2], [AD#4].

#### [AC01]: A buyer's give is stored

When a give (`POST /api/v1/ratings`) that [AC05] and [AC06] do not refuse reaches the ratings service for an e-mail and ISBN that hold no rating, the pair compared ignoring letter case ([AD#3]), and orders reports a qualifying order for that ISBN under the e-mail exactly as the give carries it ([AD#2]), the ratings service shall store one rating for that e-mail and ISBN with the given stars.

##### Test cases

**[TC01]: A buyer's give is stored — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`; the ratings service holds no rating for that e-mail and ISBN.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Send `GET /api/v1/ratings/findByEmail?email=reader@example.com`.
- *Expected result:* The ratings service holds exactly one rating for `reader@example.com` and `9780000000017`, with 4 stars.

**[TC02]: A buyer of one book cannot rate another — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader2@example.com` and no order for `9780000000031`; the ratings service holds no rating for that e-mail.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader2@example.com`, `isbn` `9780000000031` and `rating` 4. 2. Send `GET /api/v1/ratings/findByEmail?email=reader2@example.com`.
- *Expected result:* The ratings service stores no rating for `reader2@example.com` and `9780000000031`.

**[TC03]: The purchase is checked under the e-mail exactly as typed — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `Reader9@Example.com` and no order under `reader9@example.com`; the ratings service holds no rating for `9780000000017` under either spelling.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader9@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000017`.
- *Expected result:* The ratings service stores no rating for `9780000000017`.

#### [AC02]: A stored give is answered with the saved rating

When the ratings service stores a give, the ratings service shall answer `200` with the saved rating's id, e-mail, ISBN, stars, Created At and Updated At.

##### Test cases

**[TC01]: A buyer's give is answered with the saved rating — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000024` under `direct@example.com`; the ratings service holds no rating for that e-mail and ISBN.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `direct@example.com`, `isbn` `9780000000024` and `rating` 5.
- *Expected result:* The ratings service answers `200` with an id, `direct@example.com`, `9780000000024`, 5 stars, a Created At and an Updated At.

**[TC02]: A refused give is answered with no rating — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer for whom orders reports no order for ISBN `9780000000017`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service does not answer `200`, and its answer carries no rating id.

#### [AC03]: A give without a qualifying order is refused

If a give that [AC05] and [AC06] do not refuse reaches the ratings service for an e-mail that, as orders reports it, holds no qualifying order for the ISBN, or that orders answers is not a customer ([AD#2]), then the ratings service shall answer `403`, storing no rating.

##### Test cases

**[TC01]: A customer with no order for the book is refused — Happy path:**
- *Preconditions:* `browser@example.com` is a customer for whom orders reports no order for ISBN `9780000000017`; the ratings service holds no rating for that e-mail and ISBN.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Send `GET /api/v1/ratings/findByEmail?email=browser@example.com`.
- *Expected result:* The ratings service answers `403` and stores no rating for `browser@example.com` and `9780000000017`.

**[TC02]: An unpaid order does not qualify — Negative / boundary:**
- *Preconditions:* Orders reports an order for ISBN `9780000000017` under `unpaid@example.com` that is not completed, because it was placed and never paid.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `unpaid@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `403` and stores no rating for `unpaid@example.com` and `9780000000017`.

**[TC03]: A cancelled order does not qualify — Negative / boundary:**
- *Preconditions:* Orders reports the order for ISBN `9780000000017` under `cancelled@example.com` as not completed, because it was completed and paid and then cancelled.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `cancelled@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `403` and stores no rating for `cancelled@example.com` and `9780000000017`.

**[TC04]: A deleted order does not qualify — Negative / boundary:**
- *Preconditions:* `deleted@example.com` is a customer whose completed, paid order for ISBN `9780000000017` was deleted, so orders reports no order for it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `deleted@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `403` and stores no rating for `deleted@example.com` and `9780000000017`.

**[TC05]: An e-mail orders does not know is refused — Negative / boundary:**
- *Preconditions:* Orders answers not-found for `stranger@example.com`, which is not a customer of the store.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `stranger@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `403` and stores no rating for `stranger@example.com`.

**[TC06]: An ISBN the store does not know is refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order under `reader@example.com` for ISBN `9780000000017` only; ISBN `9789999999999` is 13 digits and names no book in the store.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9789999999999` and `rating` 4.
- *Expected result:* The ratings service answers `403` and stores no rating for `9789999999999`.

#### [AC04]: One refusal for every reason

The ratings service shall answer every buyers-only refusal ([AC03], [U02] [AC16]) with `403` and a body identical in every field except the time of the answer and the request path, whose `message` is exactly "Only customers who have bought this book can rate it."

##### Test cases

**[TC01]: An unknown e-mail and a customer with no order get the same answer — Happy path:**
- *Preconditions:* Orders answers not-found for `stranger@example.com`; `browser@example.com` is a customer for whom orders reports no order for ISBN `9780000000017`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `stranger@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Send the same give with `email` `browser@example.com`.
- *Expected result:* The ratings service answers both gives `403`, with bodies identical in every field except the time of the answer, each `message` exactly "Only customers who have bought this book can rate it."

**[TC02]: Unpaid, cancelled, deleted and unknown-book refusals get the same answer — Negative / boundary:**
- *Preconditions:* For ISBN `9780000000017`, orders reports an unpaid order under `unpaid@example.com`, a cancelled order under `cancelled@example.com`, and no order under `deleted@example.com`, whose completed order was deleted; orders reports a qualifying order under `reader@example.com` for `9780000000017` only; ISBN `9789999999999` names no book.
- *Steps:* 1. Send `POST /api/v1/ratings` with `isbn` `9780000000017` and `rating` 4 for each of `unpaid@example.com`, `cancelled@example.com` and `deleted@example.com`. 2. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9789999999999` and `rating` 4.
- *Expected result:* The ratings service answers all four gives `403`, with bodies identical in every field except the time of the answer.

**[TC03]: The refusal names no reason — Security / privacy:**
- *Preconditions:* Orders reports the order for ISBN `9780000000017` under `cancelled@example.com` as not completed, because it was cancelled.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `cancelled@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Read the whole answer.
- *Expected result:* The answer's `message` is exactly "Only customers who have bought this book can rate it.", and nothing in the answer says whether the e-mail is a customer, whether an order exists, or the order's state.

**[TC04]: A refused change gets the same answer as a refused give — State / lifecycle:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while its order qualified; orders now reports that order as not completed, because it was cancelled; `browser@example.com` is a customer for whom orders reports no order for `9780000000017`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for the rating of `lapsed@example.com`, with its `id` and `rating` 5. 2. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers both `403`, with bodies identical in every field except the time of the answer and the request path, each `message` exactly "Only customers who have bought this book can rate it."

#### [AC05]: Stars outside the scale are refused

If a give reaches the ratings service with no star value, or with a star value that is not a whole number from 1 to 5, and with no fault of [AC06] ([AC07]), then the ratings service shall answer `400` with exactly "Invalid rating. Must be a whole number from 1 to 5." in the `message` field of the error body, storing no rating.

##### Test cases

**[TC01]: A give with no star value is refused, not stored as zero — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `nostars@example.com`; the ratings service holds no rating for that e-mail and ISBN.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `nostars@example.com` and `isbn` `9780000000017` and no `rating` field. 2. Send `GET /api/v1/ratings/findByEmail?email=nostars@example.com`.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5." and stores no rating.

**[TC02]: Zero, six and minus one are refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `edges@example.com`; the ratings service holds no rating for that e-mail and ISBN.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `edges@example.com`, `isbn` `9780000000017` and `rating` 0. 2. Send the same give with `rating` 6. 3. Send the same give with `rating` -1.
- *Expected result:* The ratings service answers all three `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5." and stores no rating.

**[TC03]: A fractional star value is refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `fraction@example.com`; the ratings service holds no rating for that e-mail and ISBN.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `fraction@example.com`, `isbn` `9780000000017` and `rating` 3.5.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5." and stores no rating.

**[TC04]: One and five stars are not refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `low@example.com` and under `high@example.com`; the ratings service holds no rating for either.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `low@example.com`, `isbn` `9780000000017` and `rating` 1. 2. Send `POST /api/v1/ratings` with `email` `high@example.com`, `isbn` `9780000000017` and `rating` 5.
- *Expected result:* The ratings service answers neither give `400`, and stores a 1-star rating for `low@example.com` and a 5-star rating for `high@example.com`.

##### Open questions

- [ ] Engineering review [ER2]: accept that "a whole number from 1 to 5" here and in [U02] [AC09] means a JSON integer, so a give or change whose `rating` is `4.0` or the string `"4"` is refused with "Invalid rating. Must be a whole number from 1 to 5.", and add the proposed test case to each?

#### [AC06]: A missing or malformed e-mail or ISBN is refused

If a give reaches the ratings service with a missing or malformed e-mail, or with a missing ISBN or one that is not 13 digits, and with no other fault of [AC05] and [AC06] ([AC07]), then the ratings service shall answer `400` with exactly "Invalid email address" in the `message` field of the error body for the e-mail, or exactly "ISBN must be a 13-digits value" for the ISBN, storing no rating.

##### Test cases

**[TC01]: A malformed e-mail is refused — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader-at-example`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid email address" and stores no rating.

**[TC02]: An ISBN that is not 13 digits is refused — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `978000000001` (12 digits) and `rating` 4. 2. Send the same give with `isbn` `97800000000170` (14 digits). 3. Send the same give with `isbn` `978000000001X`.
- *Expected result:* The ratings service answers all three `400` with `message` exactly "ISBN must be a 13-digits value" and stores no rating.

**[TC03]: A give with no e-mail is refused — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Send `POST /api/v1/ratings` with `isbn` `9780000000017` and `rating` 4 and no `email` field.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid email address" and stores no rating.

**[TC04]: A give with no ISBN is refused — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com` and `rating` 4 and no `isbn` field.
- *Expected result:* The ratings service answers `400` with `message` exactly "ISBN must be a 13-digits value" and stores no rating.

#### [AC07]: A give with several faults is refused with one of their reasons

If a give reaches the ratings service with more than one fault of [AC05] and [AC06], then the ratings service shall answer `400` with the reason of one of those faults in the `message` field of the error body, storing no rating.

##### Test cases

**[TC01]: A malformed e-mail and zero stars — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader-at-example`, `isbn` `9780000000017` and `rating` 0.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid email address" or exactly "Invalid rating. Must be a whole number from 1 to 5.", and stores no rating.

**[TC02]: No ISBN and six stars — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com` and `rating` 6 and no `isbn` field.
- *Expected result:* The ratings service answers `400` with `message` exactly "ISBN must be a 13-digits value" or exactly "Invalid rating. Must be a whole number from 1 to 5.", and stores no rating.

#### [AC08]: An invalid give makes no purchase check

If the ratings service answers a give `400` ([AC05], [AC06], [AC07]), then the ratings service shall make no call to orders for that give.

##### Test cases

**[TC01]: A non-buyer's zero stars get the star reason — Happy path:**
- *Preconditions:* `browser@example.com` is a customer for whom orders reports no order for ISBN `9780000000017`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 0. 2. Read the requests orders received.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5.", not `403`, and orders received no request for the give.

**[TC02]: An invalid give while orders is unreachable is refused as invalid — Negative / boundary:**
- *Preconditions:* The orders service is unreachable from the ratings service.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader-at-example`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid email address", not `503`.

#### [AC09]: A give is refused while orders cannot answer

If, for a give that [AC05] and [AC06] do not refuse, orders does not answer the purchase check within the ratings service's bounded time, cannot be reached, answers with any status other than a success or not-found, or answers a success the ratings service cannot read ([AD#2]), then the ratings service shall answer `503`, storing no rating.

##### Test cases

**[TC01]: A buyer's give is refused while orders is unreachable — Happy path:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service holds no rating for that e-mail and ISBN; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader5@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `503` and stores no rating.

**[TC02]: A non-buyer's give while orders is unreachable is not refused as not bought — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `503`, not `403`, and stores no rating.

**[TC03]: A give is refused in bounded time while orders never answers — Negative / boundary:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service holds no rating for that e-mail and ISBN; the orders service accepts connections from the ratings service but never answers.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader5@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `503` within its bounded time, rather than waiting on orders, and stores no rating.

**[TC04]: An orders server error is not read as not bought — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`; the orders service answers every request with a server error.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `503`, not `403`, and stores no rating.

**[TC05]: The same give is stored once orders answers again — State / lifecycle:**
- *Preconditions:* `reader6@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service holds no rating for that e-mail and ISBN; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader6@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Make the orders service reachable again. 3. Send the same give again.
- *Expected result:* The ratings service answers the first give `503` and stores one rating, with 4 stars, from the second.

**[TC06]: An unreadable answer from orders is not read as not bought — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`; the orders service answers the purchase check with a success whose body the ratings service cannot read.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `503`, not `403`, and stores no rating.

##### Open questions

- [ ] Engineering review [ER1]: accept that "not-found" here and in [U02] [AC17] means orders' answer that the e-mail is not a customer, so a `404` that orders' `findByEmail` did not answer (a wrong orders host or path) is refused `503`, not `403`, and add the proposed test case to each?

#### [AC10]: A give's id changes no other rating

When a give carries an `id`, the ratings service shall disregard it, storing or changing only the rating of the give's e-mail and ISBN and leaving every other rating exactly as it was.

##### Test cases

**[TC01]: A give for a new pair carrying another rating's id stores a new rating — Happy path:**
- *Preconditions:* `second@example.com` holds a 3-star rating of ISBN `9780000000017` with the id R2; orders reports a qualifying order for ISBN `9780000000024` under `reader10@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `id` R2, `email` `reader10@example.com`, `isbn` `9780000000024` and `rating` 4. 2. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000024`. 3. Send `GET /api/v1/ratings/R2`.
- *Expected result:* The ratings service holds a 4-star rating for `reader10@example.com` and `9780000000024` with an id other than R2, and the rating R2 is still 3 stars from `second@example.com` for `9780000000017`.

**[TC02]: A repeat give carrying another rating's id changes only its own pair's rating — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it with the id R1; `second@example.com` holds a 3-star rating of it with the id R2.
- *Steps:* 1. Send `POST /api/v1/ratings` with `id` R2, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 5. 2. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000017`.
- *Expected result:* The rating R1 has 5 stars, and the rating R2 is still 3 stars from `second@example.com`.

**[TC03]: A give carrying the id 0 is stored as a new rating — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader11@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `id` 0, `email` `reader11@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Send `GET /api/v1/ratings/findByEmail?email=reader11@example.com`.
- *Expected result:* The ratings service holds one 4-star rating for `reader11@example.com` and `9780000000017`.

#### [AC11]: Each buyers-only refusal is logged

When the ratings service answers a give or a change with the buyers-only refusal, the ratings service shall write one entry to its own log naming the ISBN and the reason, with no e-mail: the e-mail is not a customer, the customer holds no order for the book, or the customer's order for the book is not completed.

##### Test cases

**[TC01]: An unpaid order's refusal is logged with its reason — Happy path:**
- *Preconditions:* Orders reports an order for ISBN `9780000000017` under `unpaid@example.com` that is not completed, because it was never paid.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `unpaid@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds one entry for the refused give, naming `9780000000017` and that the customer's order for the book is not completed.

**[TC02]: An unknown e-mail's refusal is logged with its own reason — Negative / boundary:**
- *Preconditions:* Orders answers not-found for `stranger@example.com`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `stranger@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds one entry for the refused give, naming `9780000000017` and that the e-mail is not a customer.

**[TC03]: A customer with no order for the book is logged with that reason — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer for whom orders reports no order for ISBN `9780000000017`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds one entry for the refused give, naming `9780000000017` and that the customer holds no order for the book.

**[TC04]: A refused change is logged — Audit / observability:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds one entry for the refused change, naming `9780000000017` and that the customer's order for the book is not completed.

**[TC05]: The log holds no e-mail — Security / privacy:**
- *Preconditions:* Orders reports the order for ISBN `9780000000017` under `cancelled@example.com` as not completed, because it was cancelled.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `cancelled@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Read the ratings service's log entry for the refusal.
- *Expected result:* The entry names `9780000000017` and its reason, and does not hold `cancelled@example.com`.

**[TC06]: The reason reaches the log and not the caller — Audit / observability:**
- *Preconditions:* Orders reports the order for ISBN `9780000000017` under `cancelled@example.com` as not completed, because it was cancelled.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `cancelled@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Compare the answer with the ratings service's log entry for the refusal.
- *Expected result:* The log entry names the reason, and the answer carries only the buyers-only refusal of [AC04].

---

### [U02]: Change my rating of a book

As a customer who rated a book, I want to change my rating whenever I change my mind, so that my rating says what I think now and still counts only once.

Done when a change from a customer with a qualifying order for the book replaces the stars, each customer and book hold at most one rating, and no change moves a rating to another customer or another book.

Traces to: [US#2], [US#5]; [AC#7]–[AC#10], [AC#24], [AC#30]; PRD-level specification [U02/AC01]–[U02/AC07], [U02/AC09]–[U02/AC12], [U02/AC21]; [VD#2], [VD#4], [VD#5]; [CG#3], [CG#4], [CG#10]; [AD#2], [AD#3], [AD#4].

#### [AC01]: A buyer's change replaces the stars

When a change (`PUT /api/v1/ratings/{id}`) that [AC09]–[AC14] do not refuse reaches the ratings service for a rating that exists, and orders reports a qualifying order for the rating's ISBN under the e-mail exactly as the change carries it, or under the stored e-mail when the change carries none ([AD#2]), the ratings service shall replace the rating's stars with the stars of the change.

##### Test cases

**[TC01]: A buyer changes their stars — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 4. 2. Send `GET /api/v1/ratings/findByEmail?email=reader@example.com`.
- *Expected result:* The ratings service holds one rating for `reader@example.com` and `9780000000017`, with 4 stars.

**[TC02]: A change carrying no e-mail or ISBN is checked under the stored e-mail — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000024` under `direct@example.com`, who holds a 5-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 3 and no `email` or `isbn` field. 2. Read the requests orders received.
- *Expected result:* The ratings service holds one rating for `direct@example.com` and `9780000000024`, with 3 stars, and orders was asked about `direct@example.com`.

**[TC03]: Repeating an identical change leaves one rating with the same stars — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 4-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 4. 2. Send the same change again.
- *Expected result:* The ratings service holds one rating for `reader@example.com` and `9780000000017`, with 4 stars.

#### [AC02]: A saved change is answered with the saved rating

When the ratings service saves a change, the ratings service shall answer `200` with the saved rating's id, e-mail, ISBN, stars, Created At and Updated At.

##### Test cases

**[TC01]: A change is answered with the saved rating — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 4.
- *Expected result:* The ratings service answers `200` with the rating's id, `reader@example.com`, `9780000000017`, 4 stars, its Created At and its Updated At.

**[TC02]: A give applied as a change is answered with the existing rating — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 5.
- *Expected result:* The ratings service answers `200` with the existing rating's id and 5 stars.

#### [AC03]: Rating a book again changes the existing rating

When a give reaches the ratings service for an e-mail and ISBN that already hold a rating, the pair compared ignoring letter case ([AD#3]), the ratings service shall apply the give to that rating as a change, whatever `id` the give carries or lacks ([U01] [AC10]), under every rule of a change in this story but [AC12] and [AC13], which apply only to a `PUT /api/v1/ratings/{id}`.

##### Test cases

**[TC01]: A buyer rates the same book again — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 5. 2. Send `GET /api/v1/ratings/findByEmail?email=reader@example.com`.
- *Expected result:* The ratings service holds one rating for `reader@example.com` and `9780000000017`, with 5 stars.

**[TC02]: A repeat give after the order was cancelled is refused as a change — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `lapsed@example.com`, `isbn` `9780000000017` and `rating` 5.
- *Expected result:* The ratings service answers with the buyers-only refusal and still holds one rating for `lapsed@example.com` and `9780000000017`, with 3 stars.

**[TC03]: A give from another buyer of the same book is a rating of its own — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; orders reports a qualifying order for it under `second@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `second@example.com`, `isbn` `9780000000017` and `rating` 1.
- *Expected result:* The ratings service holds two ratings for `9780000000017`, one for each e-mail, and the rating of `reader@example.com` is unchanged.

**[TC04]: A repeat give typed in other capitals changes the same rating — Negative / boundary:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; orders reports a qualifying order for it under `Reader@Example.com` and none under `reader@example.com`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `Reader@Example.com`, `isbn` `9780000000017` and `rating` 5. 2. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000017`.
- *Expected result:* The ratings service holds one rating of `9780000000017` for that customer, with 5 stars.

#### [AC04]: Simultaneous gives leave one rating

If gives for one e-mail and ISBN, the pair compared ignoring letter case, reach the ratings service at the same time ([AD#3]), then the ratings service shall leave exactly one rating for that pair ([AC03]), answering each of those gives that no other criterion refuses as [U01] [AC02] or [AC02] describes, never with a server error.

##### Test cases

**[TC01]: Two simultaneous gives from one buyer leave one rating — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `racer@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `racer@example.com`, `isbn` `9780000000017` and `rating` 3, and the same give with `rating` 5, at the same moment.
- *Expected result:* The ratings service answers both gives `200` and holds exactly one rating for `racer@example.com` and `9780000000017`, with 3 or 5 stars.

**[TC02]: Simultaneous gives from two buyers leave two ratings — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `racer1@example.com` and under `racer2@example.com`; neither holds a rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` for `racer1@example.com` and for `racer2@example.com`, each with `isbn` `9780000000017` and `rating` 4, at the same moment.
- *Expected result:* The ratings service holds exactly one rating for each of the two e-mails for `9780000000017`.

**[TC03]: Simultaneous gives typed in different capitals leave one rating — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `racer3@example.com` and under `RACER3@example.com`; the ratings service holds no rating of it under either spelling.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `racer3@example.com`, `isbn` `9780000000017` and `rating` 2, and the same give with `email` `RACER3@example.com` and `rating` 4, at the same moment.
- *Expected result:* The ratings service holds exactly one rating of `9780000000017` for that customer.

**[TC04]: Many simultaneous gives from one buyer leave one rating — State / lifecycle:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `burst@example.com`, who holds no rating of it.
- *Steps:* 1. Send ten `POST /api/v1/ratings` gives for `burst@example.com` and `9780000000017`, with stars from 1 to 5, at the same moment.
- *Expected result:* The ratings service holds exactly one rating for `burst@example.com` and `9780000000017`.

#### [AC05]: An empty comment on a change keeps the stored comment

When a change whose comment is empty, absent or only spaces is saved, the ratings service shall keep the rating's stored comment.

##### Test cases

**[TC01]: An empty comment keeps the stored comment — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it with the comment "Great plot".
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `rating` 4 and `comment` "". 2. Read the rating in the ratings store.
- *Expected result:* The rating has 4 stars, and its stored comment is still "Great plot".

**[TC02]: A change with no comment field keeps the stored comment — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000024` under `direct@example.com`, who holds a 5-star rating of it with the comment "Slow start".
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 4 and no `comment` field. 2. Read the rating in the ratings store.
- *Expected result:* The rating's stored comment is still "Slow start".

**[TC03]: A comment of spaces only keeps the stored comment — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000024` under `direct@example.com`, who holds a 5-star rating of it with the comment "Slow start".
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `rating` 3 and a `comment` of three spaces. 2. Read the rating in the ratings store.
- *Expected result:* The rating has 3 stars, and its stored comment is still "Slow start".

**[TC04]: A repeat give with no comment keeps the stored comment — State / lifecycle:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it with the comment "Great plot".
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 4 and no `comment` field. 2. Read the rating in the ratings store.
- *Expected result:* The rating's stored comment is still "Great plot".

#### [AC06]: A comment on a change replaces the stored comment

When a change carrying a comment that holds a character other than a space is saved, the ratings service shall replace the rating's stored comment with that comment.

##### Test cases

**[TC01]: A new comment replaces the old one — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it with the comment "Great plot".
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `rating` 3 and `comment` "Weaker second half". 2. Read the rating in the ratings store.
- *Expected result:* The rating's stored comment is "Weaker second half".

**[TC02]: A refused change leaves the stored comment — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017` with the comment "Good read", given while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `rating` 5 and `comment` "Changed my mind". 2. Read the rating in the ratings store.
- *Expected result:* The rating's stored comment is still "Good read".

**[TC03]: A repeat give with a comment replaces the stored comment — State / lifecycle:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it with the comment "Great plot".
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017`, `rating` 3 and `comment` "Weaker second half". 2. Read the rating in the ratings store.
- *Expected result:* The rating's stored comment is "Weaker second half".

#### [AC07]: A change updates Updated At and keeps Created At

When a change is saved, an identical repeat included, the ratings service shall set the rating's Updated At to the time of the change, leaving its Created At as the time the rating was first given.

##### Test cases

**[TC01]: A change moves Updated At only — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it first given at time T1.
- *Steps:* 1. At a later time T2, send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 3. 2. Send `GET /api/v1/ratings/{id}` for it.
- *Expected result:* The rating's Created At is T1 and its Updated At is T2.

**[TC02]: A refused change moves neither time — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017` given and last updated at time T1, while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. At a later time T2, send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5. 2. Send `GET /api/v1/ratings/{id}` for it.
- *Expected result:* The rating's Created At and Updated At are both still T1.

**[TC03]: A repeat give keeps Created At — State / lifecycle:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it first given at time T1.
- *Steps:* 1. At a later time T2, send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 2. 2. Send `GET /api/v1/ratings/{id}` for that rating.
- *Expected result:* The rating's Created At is T1 and its Updated At is T2.

**[TC04]: An identical repeated change moves Updated At — State / lifecycle:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 4-star rating of it with the comment "Great plot", first given at time T1 and last changed at time T2.
- *Steps:* 1. At a later time T3, send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `rating` 4 and `comment` "Great plot". 2. Send `GET /api/v1/ratings/{id}` for it.
- *Expected result:* The rating still has 4 stars, its Created At is T1 and its Updated At is T3.

#### [AC08]: A change keeps the stored e-mail as stored

When a change carrying the rating's e-mail in other capitals is saved, the ratings service shall keep the rating's e-mail exactly as stored ([AD#4]).

##### Test cases

**[TC01]: A change typed in other capitals keeps the stored e-mail — Happy path:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; orders reports a qualifying order for it under `Reader@Example.com` and none under `reader@example.com`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `email` `Reader@Example.com` and `rating` 4. 2. Send `GET /api/v1/ratings/{id}` for it.
- *Expected result:* The rating has 4 stars, and its e-mail is still `reader@example.com`.

**[TC02]: A repeat give typed in capitals keeps the stored e-mail — Negative / boundary:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; orders reports a qualifying order for it under `READER@EXAMPLE.COM` and none under `reader@example.com`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `READER@EXAMPLE.COM`, `isbn` `9780000000017` and `rating` 5. 2. Send `GET /api/v1/ratings/{id}` for that rating.
- *Expected result:* The rating has 5 stars, and its e-mail is still `reader@example.com`.

#### [AC09]: Stars outside the scale are refused on a change

If a change reaches the ratings service with no star value, or with a star value that is not a whole number from 1 to 5, and with no fault of [AC10], [AC11] and [AC13] ([AC14]), then the ratings service shall answer `400` with exactly "Invalid rating. Must be a whole number from 1 to 5." in the `message` field of the error body, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A change to zero stars is refused — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 4-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 0.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5.", and the rating still has 4 stars.

**[TC02]: A change with no star value is refused, not stored as zero — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 4-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and no `rating` field.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5.", and the rating still has 4 stars.

**[TC03]: A change to six or to a fraction is refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 4-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 6. 2. Send the same change with `rating` 2.5.
- *Expected result:* The ratings service answers both `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5.", and the rating still has 4 stars.

#### [AC10]: A malformed e-mail or ISBN on a change is refused

If a change reaches the ratings service carrying a malformed e-mail, or an ISBN that is not 13 digits, with no other fault of [AC09], [AC10], [AC11] and [AC13] ([AC14]), then the ratings service shall answer `400` with exactly "Invalid email address" in the `message` field of the error body for the e-mail, or exactly "ISBN must be a 13-digits value" for the ISBN, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A malformed e-mail on a change is refused — Happy path:**
- *Preconditions:* `reader@example.com` holds a 2-star rating of ISBN `9780000000017`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `email` `reader-at-example` and `rating` 4.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid email address", and the rating still has 2 stars and the e-mail `reader@example.com`.

**[TC02]: A short ISBN on a change gets the ISBN reason, not the move reason — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a 2-star rating of ISBN `9780000000017`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `isbn` `978000000001` and `rating` 4.
- *Expected result:* The ratings service answers `400` with `message` exactly "ISBN must be a 13-digits value", and the rating is still for `9780000000017` with 2 stars.

#### [AC11]: A change cannot move a rating

If a change reaches the ratings service for a rating that exists, carrying a well-formed e-mail different from the stored rating's, compared ignoring letter case, or a 13-digit ISBN different from the stored rating's, with no fault of [AC09], [AC10] and [AC13] ([AC14]), then the ratings service shall answer `400` with exactly "A rating's e-mail and ISBN cannot be changed." in the `message` field of the error body, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A change to another book is refused — Happy path:**
- *Preconditions:* Orders reports qualifying orders for ISBNs `9780000000017` and `9780000000024` under `reader@example.com`, who holds a rating of `9780000000017`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `isbn` `9780000000024` and `rating` 3.
- *Expected result:* The ratings service answers `400` with `message` exactly "A rating's e-mail and ISBN cannot be changed.", and the rating is still for `9780000000017`.

**[TC02]: A change to another customer is refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it, and under `other@example.com`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `email` `other@example.com` and `rating` 3.
- *Expected result:* The ratings service answers `400` with `message` exactly "A rating's e-mail and ISBN cannot be changed.", and the rating is still for `reader@example.com`.

**[TC03]: A change that repeats or leaves out the e-mail and ISBN proceeds — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 3. 2. Send `PUT /api/v1/ratings/{id}` for it with its `id` and `rating` 4 and no `email` or `isbn` field.
- *Expected result:* The ratings service answers neither change `400`, and the rating has 4 stars for `reader@example.com` and `9780000000017`.

#### [AC12]: A change to a rating that does not exist is refused

If a `PUT /api/v1/ratings/{id}` that [AC09] and [AC10] do not refuse reaches the ratings service for an id that holds no rating, then the ratings service shall answer `404`, storing no rating.

##### Test cases

**[TC01]: A change to a removed rating is refused — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it with the id R.
- *Steps:* 1. Send `DELETE /api/v1/ratings/R`. 2. Send `PUT /api/v1/ratings/R` with `id` R, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 4. 3. Send `GET /api/v1/ratings/findByEmail?email=reader@example.com`.
- *Expected result:* The ratings service answers the change `404` and holds no rating of `9780000000017` for `reader@example.com`.

**[TC02]: An unknown id is refused before the body's id is compared — Negative / boundary:**
- *Preconditions:* The id 999999 holds no rating.
- *Steps:* 1. Send `PUT /api/v1/ratings/999999` with `id` 999998, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `404`, not `400`, and stores no rating.

**[TC03]: A body fault is answered before an unknown id — Negative / boundary:**
- *Preconditions:* The id 999999 holds no rating.
- *Steps:* 1. Send `PUT /api/v1/ratings/999999` with `id` 999999, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 0. 2. Send the same change with `rating` 4 and `email` `reader-at-example`.
- *Expected result:* The ratings service answers the first `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5." and the second `400` with `message` exactly "Invalid email address", not `404`, and stores no rating.

#### [AC13]: A change whose body id does not match its path is refused

If a `PUT /api/v1/ratings/{id}` reaches the ratings service for a rating that exists, carrying in its body no id, or an id different from the id it is sent to, with no other fault of [AC09], [AC10] and [AC11] ([AC14]), then the ratings service shall answer `400` with exactly "bad rating id" in the `message` field of the error body, leaving every rating exactly as it was.

##### Test cases

**[TC01]: A body id that names another rating is refused — Happy path:**
- *Preconditions:* `reader@example.com` holds a 2-star rating of ISBN `9780000000017` with the id R1; `second@example.com` holds a 3-star rating of it with the id R2.
- *Steps:* 1. Send `PUT /api/v1/ratings/R1` with `id` R2 and `rating` 5.
- *Expected result:* The ratings service answers `400` with `message` exactly "bad rating id", and the rating R1 still has 2 stars and R2 still has 3 stars.

**[TC02]: A body id that matches the path proceeds — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it with the id R1.
- *Steps:* 1. Send `PUT /api/v1/ratings/R1` with `id` R1 and `rating` 5.
- *Expected result:* The ratings service does not answer `400`, and the rating R1 has 5 stars.

**[TC03]: A body with no id is refused — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a 2-star rating of it with the id R1.
- *Steps:* 1. Send `PUT /api/v1/ratings/R1` with `rating` 5 and no `id` field.
- *Expected result:* The ratings service answers `400` with `message` exactly "bad rating id", and the rating R1 still has 2 stars.

#### [AC14]: A change with several faults is refused with one of their reasons

If a change reaches the ratings service with more than one fault of [AC09] and [AC10], or, for a rating that exists, of [AC09], [AC10] and [AC11] and, for a `PUT /api/v1/ratings/{id}`, [AC13], then the ratings service shall answer `400` with the reason of one of those faults in the `message` field of the error body, leaving every rating exactly as it was.

##### Test cases

**[TC01]: Zero stars and another ISBN — Happy path:**
- *Preconditions:* `reader@example.com` holds a 4-star rating of ISBN `9780000000017`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `isbn` `9780000000024` and `rating` 0.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5." or exactly "A rating's e-mail and ISBN cannot be changed.", and the rating is still for `9780000000017` with 4 stars.

**[TC02]: Another rating's id and six stars — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a 2-star rating of ISBN `9780000000017` with the id R1; `second@example.com` holds a 3-star rating of it with the id R2.
- *Steps:* 1. Send `PUT /api/v1/ratings/R1` with `id` R2 and `rating` 6.
- *Expected result:* The ratings service answers `400` with `message` exactly "bad rating id" or exactly "Invalid rating. Must be a whole number from 1 to 5.", and the rating R1 still has 2 stars and R2 still has 3 stars.

#### [AC15]: An invalid change makes no purchase check

If the ratings service answers a change `400` ([AC09]–[AC11], [AC13], [AC14]) or a `PUT /api/v1/ratings/{id}` `404` ([AC12]), then the ratings service shall make no call to orders for that change.

##### Test cases

**[TC01]: A lapsed buyer's zero stars get the star reason — Happy path:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 0. 2. Read the requests orders received.
- *Expected result:* The ratings service answers `400` with `message` exactly "Invalid rating. Must be a whole number from 1 to 5.", not `403`, and orders received no request for the change.

**[TC02]: A change to an unknown id while orders is unreachable is not found — Negative / boundary:**
- *Preconditions:* The id 999999 holds no rating; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `PUT /api/v1/ratings/999999` with `id` 999999, `email` `reader@example.com`, `isbn` `9780000000017` and `rating` 4.
- *Expected result:* The ratings service answers `404`, not `503`.

**[TC03]: A move while orders is unreachable is refused as invalid — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `isbn` `9780000000024` and `rating` 3.
- *Expected result:* The ratings service answers `400` with `message` exactly "A rating's e-mail and ISBN cannot be changed.", not `503`.

#### [AC16]: A change without a qualifying order is refused

If a change that [AC09]–[AC14] do not refuse reaches the ratings service for a rating that exists, and orders reports no qualifying order for the rating's ISBN under the e-mail exactly as the change carries it, or under the stored e-mail when the change carries none, or answers that this e-mail is not a customer ([AD#2]), then the ratings service shall answer `403` with the buyers-only refusal of [U01] [AC04], leaving the rating exactly as it was.

##### Test cases

**[TC01]: A change after the order was cancelled is refused — Happy path:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5.
- *Expected result:* The ratings service answers `403` with the buyers-only refusal, and the rating still has 3 stars.

**[TC02]: A change after the order was deleted is refused — Negative / boundary:**
- *Preconditions:* `gone@example.com` holds a 4-star rating of ISBN `9780000000017`, given while its order qualified; that order has since been deleted, so orders reports no order for it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 1.
- *Expected result:* The ratings service answers `403` with the buyers-only refusal, and the rating still has 4 stars.

**[TC03]: A change typed under a spelling with no order is refused — Negative / boundary:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; orders reports a qualifying order for it under `reader@example.com` and none under `READER@EXAMPLE.COM`.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id`, `email` `READER@EXAMPLE.COM` and `rating` 4.
- *Expected result:* The ratings service answers `403` with the buyers-only refusal, and the rating still has 2 stars.

#### [AC17]: A change is refused while orders cannot answer

If, for a change that [AC09]–[AC14] do not refuse, orders does not answer the purchase check within the ratings service's bounded time, cannot be reached, answers with any status other than a success or not-found, or answers a success the ratings service cannot read ([AD#2]), then the ratings service shall answer `503`, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A buyer's change is refused while orders is unreachable — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 4.
- *Expected result:* The ratings service answers `503`, and the rating still has 2 stars.

**[TC02]: A repeat give while orders is unreachable leaves the rating — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it with the comment "Great plot"; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `reader@example.com`, `isbn` `9780000000017`, `rating` 5 and `comment` "Even better". 2. Read the rating in the ratings store.
- *Expected result:* The ratings service answers `503`, and the rating still has 2 stars and the comment "Great plot".

**[TC03]: A change is refused in bounded time while orders never answers — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it; the orders service accepts connections from the ratings service but never answers.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 4.
- *Expected result:* The ratings service answers `503` within its bounded time, and the rating still has 2 stars.

**[TC04]: An orders server error is not read as not bought — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017` whose order has since been cancelled; the orders service answers every request with a server error.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5.
- *Expected result:* The ratings service answers `503`, not `403`, and the rating still has 3 stars.

**[TC05]: An unreadable answer from orders leaves the rating — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it; the orders service answers the purchase check with a success whose body the ratings service cannot read.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 4.
- *Expected result:* The ratings service answers `503`, and the rating still has 2 stars.

#### [AC18]: A rating outlives its order's cancellation

While a rating's qualifying order has been cancelled or deleted since the rating was last given or changed, the ratings service shall count the rating in its book's rating summary ([U03]).

##### Test cases

**[TC01]: A rating still counts after its order is cancelled — Happy path:**
- *Preconditions:* ISBN `9780000000031` holds one rating, 4 stars from `keeper@example.com`, given while its order qualified; orders now reports that order as not completed, because it was cancelled.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000031`.
- *Expected result:* The rating of `keeper@example.com` still counts: the summary has the count 1 and the average 4.0.

**[TC02]: A rating still counts after its order is deleted — Negative / boundary:**
- *Preconditions:* ISBN `9780000000048` holds two ratings, 5 stars from `keeper2@example.com` and 3 stars from another buyer, each given while its order qualified; the order of `keeper2@example.com` has since been deleted, so orders reports no order for it.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000048`.
- *Expected result:* The rating of `keeper2@example.com` still counts: the summary has the count 2 and the average 4.0.

**[TC03]: A refused change leaves the rating counting as it was — State / lifecycle:**
- *Preconditions:* ISBN `9780000000055` holds one rating, 2 stars from `lapsed2@example.com`, whose order has since been cancelled.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000055`.
- *Expected result:* The rating still counts with its 2 stars: the summary has the count 1 and the average 2.0.

---

### [U03]: Know how a book is rated

As a shopper choosing a book, I want each book's average rating and rating count computed from the ratings that count, so that the book page and the catalogue list show me the same figures and I can trust them.

Done when the ratings service answers one book's rating summary and every rated book's rating summary with the count and the average rounded half up to one decimal place, naming no rater.

Traces to: [US#3], [US#4]; [AC#13], [AC#14], [AC#16], [AC#19], [AC#20]; PRD-level specification [U03/AC06], [U03/AC07], [U03/AC08], [U04/AC06]; [VD#3]; [CG#6], [CG#7], [CG#8], [CG#9]; [AD#6].

#### [AC01]: The ratings service answers one book's summary

When `GET /api/v1/ratings/summary?isbn=<ISBN>` reaches the ratings service for an ISBN that holds at least one rating, the ratings service shall answer `200` with `isbn`, the ISBN asked for, `count`, the number of the ISBN's stored ratings, and `average`, their mean rounded half up to one decimal place from the exact quotient of the star sum by the count.

##### Test cases

**[TC01]: An average of 4.25 is answered as 4.3 — Happy path:**
- *Preconditions:* ISBN `9780000000062` holds four ratings, of 5, 4, 4 and 4 stars.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000062`.
- *Expected result:* The ratings service answers `200` with the ISBN `9780000000062`, the count 4 and the average 4.3.

**[TC02]: An exact 1.05 rounds up — Negative / boundary:**
- *Preconditions:* ISBN `9780000000086` holds twenty ratings: nineteen of 1 star and one of 2 stars.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000086`.
- *Expected result:* The ratings service answers the count 20 and the average 1.1.

**[TC03]: A whole average and a repeating one — Negative / boundary:**
- *Preconditions:* ISBN `9780000000079` holds three ratings, each of 4 stars; ISBN `9780000000192` holds three ratings, of 4, 5 and 5 stars.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000079`. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000192`.
- *Expected result:* The ratings service answers the count 3 and the average 4.0 for `9780000000079`, and the count 3 and the average 4.7 for `9780000000192`.

#### [AC02]: An ISBN with no rating has a count of 0

When the summary is asked for a non-blank ISBN that holds no rating, whether or not a book has that ISBN, the ratings service shall answer `200` with `isbn`, the ISBN asked for, `count` 0 and `average` `null`.

##### Test cases

**[TC01]: A book nobody has rated — Happy path:**
- *Preconditions:* ISBN `9780000000123` names a book that holds no rating.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000123`.
- *Expected result:* The ratings service answers `200` with the ISBN `9780000000123`, the count 0 and the average `null`.

**[TC02]: An ISBN that names no book — Negative / boundary:**
- *Preconditions:* ISBN `9789999999999` names no book in the store.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9789999999999`.
- *Expected result:* The ratings service answers `200` with the count 0 and the average `null`, not `400` or `404`.

**[TC03]: A book whose only rating was removed — State / lifecycle:**
- *Preconditions:* ISBN `9780000000130` holds one rating.
- *Steps:* 1. Send `DELETE /api/v1/ratings/{id}` for that rating. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000130`.
- *Expected result:* The ratings service answers the count 0 and the average `null`.

#### [AC03]: The ratings service answers every rated book's summary

When `GET /api/v1/ratings/summaries` reaches the ratings service, the ratings service shall answer `200` with one summary, as [AC01] defines it, for every ISBN that holds at least one rating, and none for any other ISBN.

##### Test cases

**[TC01]: Two rated books and one unrated — Happy path:**
- *Preconditions:* The ratings service holds ratings for ISBNs `9780000000062` (5, 4, 4 and 4 stars) and `9780000000109` (5 stars) only; ISBN `9780000000123` names a book with no rating.
- *Steps:* 1. Send `GET /api/v1/ratings/summaries`.
- *Expected result:* The ratings service answers `200` with exactly two summaries: `9780000000062` with the count 4 and the average 4.3, and `9780000000109` with the count 1 and the average 5.0.

**[TC02]: No ratings at all — Negative / boundary:**
- *Preconditions:* The ratings service holds no rating.
- *Steps:* 1. Send `GET /api/v1/ratings/summaries`.
- *Expected result:* The ratings service answers `200` with no summary.

**[TC03]: An entry equals the book's own summary — Negative / boundary:**
- *Preconditions:* ISBN `9780000000086` holds nineteen ratings of 1 star and one of 2 stars.
- *Steps:* 1. Send `GET /api/v1/ratings/summaries`. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000086`.
- *Expected result:* The entry for `9780000000086` and the book's own summary both hold the count 20 and the average 1.1.

#### [AC04]: Summaries count each give, change and removal on the next request

When a rating has been given, changed or removed, the ratings service shall count that give, change or removal in every summary it answers after it.

##### Test cases

**[TC01]: A new rating counts on the next request — Happy path:**
- *Preconditions:* ISBN `9780000000147` holds one rating, of 4 stars; orders reports a qualifying order for it under `newbuyer@example.com`, who holds no rating of it.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000147`. 2. Send `POST /api/v1/ratings` with `email` `newbuyer@example.com`, `isbn` `9780000000147` and `rating` 2. 3. Send `GET /api/v1/ratings/summary?isbn=9780000000147` again.
- *Expected result:* The second summary has the count 2 and the average 3.0.

**[TC02]: A changed rating counts on the next request — State / lifecycle:**
- *Preconditions:* ISBN `9780000000154` holds one rating, 2 stars from `reader8@example.com`, for whom orders reports a qualifying order for it.
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000154`.
- *Expected result:* The summary has the count 1 and the average 5.0.

**[TC03]: A removed rating drops out on the next request — State / lifecycle:**
- *Preconditions:* ISBN `9780000000161` holds two ratings, of 5 and 1 stars.
- *Steps:* 1. Send `DELETE /api/v1/ratings/{id}` for the 1-star rating. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000161`.
- *Expected result:* The summary has the count 1 and the average 5.0.

**[TC04]: A refused give changes nothing — Negative / boundary:**
- *Preconditions:* ISBN `9780000000178` holds one rating, of 4 stars; `browser@example.com` is a customer for whom orders reports no order for it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `browser@example.com`, `isbn` `9780000000178` and `rating` 1. 2. Send `GET /api/v1/ratings/summary?isbn=9780000000178`.
- *Expected result:* The summary has the count 1 and the average 4.0.

**[TC05]: The summaries of every book count a first rating — State / lifecycle:**
- *Preconditions:* ISBN `9780000000123` holds no rating; orders reports a qualifying order for it under `firstbuyer@example.com`.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `firstbuyer@example.com`, `isbn` `9780000000123` and `rating` 4. 2. Send `GET /api/v1/ratings/summaries`.
- *Expected result:* The summaries include `9780000000123` with the count 1 and the average 4.0.

#### [AC05]: A summary request with no ISBN is refused

If a request for one book's summary reaches the ratings service with no ISBN or a blank one, then the ratings service shall answer `400`.

##### Test cases

**[TC01]: A summary request with no ISBN — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Send `GET /api/v1/ratings/summary` with no `isbn` parameter.
- *Expected result:* The ratings service answers `400`.

**[TC02]: A summary request with a blank ISBN — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=` with an empty value. 2. Send it with an `isbn` of three spaces.
- *Expected result:* The ratings service answers both `400`, rather than the count 0.

#### [AC06]: Summaries name no rater

The ratings service shall leave every rater's e-mail out of each summary it answers, for one book or for every rated book.

##### Test cases

**[TC01]: One book's summary carries no e-mail — Happy path:**
- *Preconditions:* ISBN `9780000000116` holds ratings from `reader@example.com` and `second@example.com`.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000116`. 2. Read the whole answer.
- *Expected result:* The answer carries the ISBN, the count and the average, and no e-mail.

**[TC02]: A single-rating book's summary carries no e-mail — Negative / boundary:**
- *Preconditions:* ISBN `9780000000109` holds one rating, from `solo@example.com`.
- *Steps:* 1. Send `GET /api/v1/ratings/summary?isbn=9780000000109`. 2. Read the whole answer.
- *Expected result:* The answer does not carry `solo@example.com` or any other e-mail.

**[TC03]: The summaries of every book carry no e-mail — Security / privacy:**
- *Preconditions:* At least three books hold ratings from at least three different e-mails.
- *Steps:* 1. Send `GET /api/v1/ratings/summaries`. 2. Read the whole answer.
- *Expected result:* The answer carries no e-mail.

---

### [U04]: Know that only Quillmark sees my comment

As a customer who rates a book, I want my comment stored with my rating and returned to no one who asks the ratings service, so that only Quillmark sees it.

Done when a comment given with a new rating is stored, and no answer of the ratings service carries a comment.

Traces to: [US#7]; [AC#12]; [CD#17]; PRD-level specification [U05/AC05], [U05/AC06]; [CG#10]; [AD#7].

#### [AC01]: A comment given with a new rating is stored

When a give that creates a rating carries a comment holding a character other than a space, the ratings service shall store the comment with the rating, as given.

##### Test cases

**[TC01]: A comment is stored with a new rating — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `commenter2@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `commenter2@example.com`, `isbn` `9780000000017`, `rating` 4 and `comment` "Loved the ending". 2. Read the stored rating in the ratings store, where Quillmark reads comments.
- *Expected result:* The stored rating holds the comment "Loved the ending".

**[TC02]: A comment with leading spaces is stored as given — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `commenter3@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `commenter3@example.com`, `isbn` `9780000000017`, `rating` 4 and `comment` "  Loved it". 2. Read the stored rating in the ratings store.
- *Expected result:* The stored rating holds the comment "  Loved it".

#### [AC02]: A new rating given without a comment stores none

When a give that creates a rating carries a comment that is empty, absent or only spaces, the ratings service shall store the rating with no comment.

##### Test cases

**[TC01]: A give with no comment field — Happy path:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `quiet@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `quiet@example.com`, `isbn` `9780000000017` and `rating` 3 and no `comment` field. 2. Read the stored rating in the ratings store.
- *Expected result:* The stored rating holds 3 stars and no comment.

**[TC02]: A give with a comment of spaces only — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `quiet2@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `quiet2@example.com`, `isbn` `9780000000017`, `rating` 3 and a `comment` of three spaces. 2. Read the stored rating in the ratings store.
- *Expected result:* The stored rating holds no comment.

**[TC03]: A give with an empty comment — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `quiet3@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `quiet3@example.com`, `isbn` `9780000000017`, `rating` 3 and `comment` "". 2. Read the stored rating in the ratings store.
- *Expected result:* The stored rating holds no comment.

#### [AC03]: No answer of the ratings service carries a comment

The ratings service shall leave the comment field out of every answer it gives: to `GET /api/v1/ratings`, `/findByEmail`, `/findByISBN`, `/{id}`, `/summary` and `/summaries`, and to a give and a change ([AD#7]).

##### Test cases

**[TC01]: Reading ratings returns no comment — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017` with the comment "Great plot".
- *Steps:* 1. Send `GET /api/v1/ratings/{id}` for that rating. 2. Send `GET /api/v1/ratings/findByEmail?email=reader@example.com`. 3. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000017`. 4. Send `GET /api/v1/ratings`. 5. Send `GET /api/v1/ratings/summary?isbn=9780000000017`. 6. Send `GET /api/v1/ratings/summaries`.
- *Expected result:* None of the six answers carries a comment field or the text "Great plot".

**[TC02]: The answer to a give carries no comment — Negative / boundary:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `commenter@example.com`, who holds no rating of it.
- *Steps:* 1. Send `POST /api/v1/ratings` with `email` `commenter@example.com`, `isbn` `9780000000017`, `rating` 4 and `comment` "Loved the ending".
- *Expected result:* The ratings service's answer to the give carries no comment field and not "Loved the ending".

**[TC03]: The answer to a change that keeps the stored comment does not return it — Security / privacy:**
- *Preconditions:* Orders reports a qualifying order for ISBN `9780000000017` under `reader@example.com`, who holds a rating of it with the comment "Great plot".
- *Steps:* 1. Send `PUT /api/v1/ratings/{id}` for that rating, with its `id` and `rating` 5 and no `comment` field.
- *Expected result:* The ratings service's answer to the change carries no comment field and not "Great plot".

#### [AC04]: Reads keep their other fields and failure statuses

The ratings service shall answer each read of ratings with the fields and the failure statuses it answered before this release, apart from the comment ([AD#7]).

##### Test cases

**[TC01]: A rating read by its id keeps its other fields — Happy path:**
- *Preconditions:* `reader@example.com` holds a 4-star rating of ISBN `9780000000017`.
- *Steps:* 1. Send `GET /api/v1/ratings/{id}` for that rating.
- *Expected result:* The ratings service answers `200` with the rating's id, `reader@example.com`, `9780000000017`, 4 stars, its Created At and its Updated At.

**[TC02]: A read of an id that holds no rating is still not found — Negative / boundary:**
- *Preconditions:* The id 999999 holds no rating.
- *Steps:* 1. Send `GET /api/v1/ratings/999999`.
- *Expected result:* The ratings service answers `404`.

---

### [U05]: Remove a single rating for an erasure request or fraud

As a Quillmark operator, I want to remove a single rating, and no one able to remove every rating at once, so that I can meet an erasure request or clean up fraud without a book's ratings vanishing in bulk.

Done when removing a rating by its id removes that rating alone, and the request that removed every rating before this release is refused with nothing removed.

Traces to: [US#5]; [AC#21], [AC#23]; [VD#1]; PRD-level specification [U06/AC01], [U06/AC02], [U06/AC03]; [CG#4]; [AD#5].

#### [AC01]: An operator removes one rating

When `DELETE /api/v1/ratings/{id}` reaches the ratings service for an id that holds a rating, the ratings service shall remove that rating and no other, answering `200` with an empty body.

##### Test cases

**[TC01]: One rating is removed and the others stay — Happy path:**
- *Preconditions:* ISBN `9780000000017` holds ratings from `reader@example.com` and `second@example.com`.
- *Steps:* 1. Send `GET /api/v1/ratings/findByEmail?email=reader@example.com` to find the rating's id. 2. Send `DELETE /api/v1/ratings/{id}` with that id. 3. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000017`.
- *Expected result:* The ratings service answers the removal `200` with an empty body; the rating of `reader@example.com` for `9780000000017` is gone, and the rating of `second@example.com` remains.

**[TC02]: Every rating of one customer is removed for an erasure request — State / lifecycle:**
- *Preconditions:* `erase@example.com` holds ratings of three books.
- *Steps:* 1. Send `GET /api/v1/ratings/findByEmail?email=erase@example.com`. 2. Send `DELETE /api/v1/ratings/{id}` for each of the three ratings. 3. Send `GET /api/v1/ratings/findByEmail?email=erase@example.com` again.
- *Expected result:* The ratings service holds no rating for `erase@example.com`.

**[TC03]: Removing the same id twice removes nothing more — Negative / boundary:**
- *Preconditions:* ISBN `9780000000017` holds ratings from `reader@example.com` and `second@example.com`.
- *Steps:* 1. Send `DELETE /api/v1/ratings/{id}` for the rating of `reader@example.com`. 2. Send the same request again. 3. Send `GET /api/v1/ratings/findByISBN?isbn=9780000000017`.
- *Expected result:* The rating of `second@example.com` remains after both removals.

#### [AC02]: Removing an id that holds no rating removes nothing

If `DELETE /api/v1/ratings/{id}` reaches the ratings service for an id that holds no rating, then the ratings service shall answer `200` with an empty body, removing no rating.

##### Test cases

**[TC01]: An unused id — Happy path:**
- *Preconditions:* The ratings service holds two ratings; the id 999999 holds no rating.
- *Steps:* 1. Send `DELETE /api/v1/ratings/999999`. 2. Send `GET /api/v1/ratings`.
- *Expected result:* The ratings service answers `200` with an empty body, and both ratings remain.

**[TC02]: A repeated removal is answered alike — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Send `DELETE /api/v1/ratings/{id}` for that rating. 2. Send the same request again.
- *Expected result:* The ratings service answers both removals `200` with an empty body.

#### [AC03]: A removal with an id that is not a number is refused

If `DELETE /api/v1/ratings/{id}` reaches the ratings service with an id that is not a number, `DELETE /api/v1/ratings/delete-all` included, then the ratings service shall answer `400`, removing no rating.

##### Test cases

**[TC01]: The former remove-all request removes nothing — Happy path:**
- *Preconditions:* The ratings service holds five ratings.
- *Steps:* 1. Send `DELETE /api/v1/ratings/delete-all`. 2. Send `GET /api/v1/ratings`.
- *Expected result:* The ratings service answers `400`, and all five ratings remain.

**[TC02]: Another id that is not a number removes nothing — Negative / boundary:**
- *Preconditions:* The ratings service holds five ratings.
- *Steps:* 1. Send `DELETE /api/v1/ratings/all`. 2. Send `GET /api/v1/ratings`.
- *Expected result:* The ratings service answers `400`, and all five ratings remain.

#### [AC04]: No request removes more than one rating

The ratings service shall remove at most one rating per request.

##### Test cases

**[TC01]: A single-rating removal removes exactly one — Happy path:**
- *Preconditions:* The ratings service holds five ratings.
- *Steps:* 1. Send `DELETE /api/v1/ratings/{id}` for one of them. 2. Send `GET /api/v1/ratings`.
- *Expected result:* Four ratings remain.

**[TC02]: A removal sent to the whole collection removes nothing — Negative / boundary:**
- *Preconditions:* The ratings service holds five ratings.
- *Steps:* 1. Send `DELETE /api/v1/ratings`. 2. Send `GET /api/v1/ratings`.
- *Expected result:* All five ratings remain.

---

## Engineering review

Recorded by `/dev-workflows:design` on 2026-10-13 while designing this Epic (`design.md`, this folder). No `[Uxx]`, `[ACxx]` or `[TCxx]` is rewritten here; a proposed change waits on its open question.

- **[ER1] — proposed-change: a misrouted `404` is orders unavailable.** [U01] [AC09] and [U02] [AC17] answer `503` for "any status other than a success or not-found", and [U01] [AC03] and [U02] [AC16] answer `403` only where orders "answers that this e-mail is not a customer". A `404` from a wrong orders host or path is neither, so the specification gives it no outcome, and `ard.md` (Edge cases & risks) names the danger: a wrong orders URL would refuse every buyer. The design answers `503` for any `404` that is not orders' unknown-customer answer — its body's `path` is not `/api/v1/orders/findByEmail`, or orders' `GET /api/v1/version` does not name `orders`. Proposed wording for [U01] [AC09] and [U02] [AC17]: "… answers with any status other than a success or its answer that the e-mail is not a customer …". Proposed test case for each: *A misrouted orders URL is not read as not bought — Negative / boundary:* Preconditions: `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service's orders URL names a service that answers every request `404`. Steps: 1. Send the give (or the change). Expected result: the ratings service answers `503`, not `403`, and saves nothing. Open question under [U01] [AC09].
- **[ER2] — questioned: only a JSON integer is a whole number of stars.** [U01] [AC05] and [U02] [AC09] refuse a star value "that is not a whole number from 1 to 5". The design reads the field as JSON and accepts only an integer from 1 to 5, so `4.0` and `"4"` are refused like `3.5` — and `3.5` is never silently truncated to 3, which the service's JSON binding does by default today. Proposed test case for each: *A star value written as `4.0` or as text is refused — Negative / boundary:* Steps: send the give (or the change) with `rating` `4.0`, then with `rating` `"4"`. Expected result: both answered `400` with "Invalid rating. Must be a whole number from 1 to 5.", and nothing saved. Open question under [U01] [AC05].
- **[ER3] — validated: [U01] [AC11]'s log entry is written in the live store.** The prod profile sets the root log level to ERROR (`ratings/src/main/resources/application-prod.properties:11`), which would drop an INFO or WARN entry; the design writes the entry on its own logger, `com.dynatrace.ratings.refusals`, whose level is INFO in every profile.
- **[ER4] — validated: the bounded time.** [U01] [AC09] TC03 and [U02] [AC17] TC03 wait on "the ratings service's bounded time": 1 s to connect and 3 s to read, with no retry — about 4 s.
- **[ER5] — validated: an over-length comment.** The scope leaves unset what a give or change with a comment longer than the ratings store holds is answered; in this design it is `500` with Boot's default error body, which carries no `message`, and nothing is saved. An e-mail longer than its column is answered alike.
- **[ER6] — validated: [U04] [AC04]'s failure statuses and bodies.** The design's error handling renders only the write and summary refusals (`400`, `403`, `503`) with their reasons; none of them is raised by an existing read, so every read's failure keeps its status and its body exactly as today.
- **[ER7] — validated: [U05] [AC04] TC02.** `DELETE /api/v1/ratings` maps to no route and is answered `405`; nothing is removed.

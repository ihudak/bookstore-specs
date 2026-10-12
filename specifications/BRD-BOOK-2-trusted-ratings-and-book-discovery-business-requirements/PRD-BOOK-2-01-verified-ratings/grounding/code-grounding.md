---
kind: code-grounding
key: BOOK-2-01
---

# Code grounding: BOOK-2-01

Grounded on 2026-10-12 against `bookstore` pinned at
`e439178550371f2bc4f5a9acfee9a1c4f2289ae4` (see the grounding baselines document). Every finding below
was re-derived blind by an independent verifier and compared before it was written; `outcome` records
the result.
Ground tier: `claude-opus-5-5`.

## Findings

- id: [CG#1]
  claim: baseline integrity: `bookstore` is pinned at a verified, unmodified commit
  verdict: CONFIRMED
  evidence:
    - git -C /workspace/bookstore rev-parse HEAD → e439178550371f2bc4f5a9acfee9a1c4f2289ae4
    - git -C /workspace/bookstore diff --ignore-cr-at-eol --stat → (empty)
    - git -C /workspace/bookstore status --porcelain → (empty; no entry to line-count compare)
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: implementation
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    I re-ran the three pin-check commands before comparing. rev-parse HEAD gave e439178550371f2bc4f5a9acfee9a1c4f2289ae4, diff --stat printed nothing, and status --porcelain printed nothing and exited 0 (with and without --no-optional-locks). This is a baseline finding, so under the grounding rules it owes no control search.

- id: [CG#2]
  claim: [BR#1] — Only a customer who has bought a book can rate it. The store works this way today, and it must keep doing so.
  verdict: REWRITTEN
  evidence:
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [72, 73, 74, 77, 78, 80]
      note: createRating checks only that the book exists (verifyBook, line 77) and that the client exists (verifyClient, line 78), then saves. Nothing consults orders or payments, so any existing client can rate any existing book whether or not they bought it. The operation summary on line 73 says the call does not require client or book to exist, which lines 77-78 contradict (stale text)
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [120, 122, 130, 132]
      note: verifyClient looks the client up by email in the clients service and verifyBook looks the book up by ISBN in the books service; neither reads order or purchase state
    - path: ratings/src/main/resources/application-dev.properties
      lines: [17, 18]
      note: the ratings service is configured with clients and books URLs only (the same at application-prod.properties lines 17-18), so it has no orders endpoint to ask whether a purchase happened
    - path: orders/src/main/java/com/dynatrace/orders/controller/OrderController.java
      lines: [264, 274, 275, 290, 291]
      note: a purchase in this system is an order that buyFromStorage marks completed and pays; that state lives only in the orders service, which ratings never calls
    - path: common/src/main/java/com/dynatrace/controller/SecurityController.java
      lines: [24, 25, 26, 30, 34, 49, 50, 53]
      note: false friend — applySecurityPolicy(), which createRating calls at RatingController.java:76, reads like an eligibility gate on who may rate. It throws SecurityException only when the dt.simulate.crash config row is turned on, inside its time window and probability; it never looks at the customer, the book or any purchase
    - path: web/src/app/ratings/create-rating/create-rating.component.html
      lines: [6, 11]
      note: the web create form takes a free-text email and ISBN; the UI applies no purchase gate either
    - path: ratings
      note: searched `git grep -n -i 'order\|purchas\|bought\|paid' <commit> -- ratings web/src/app/ratings` — no match
    - path: "*.java outside ratings/ and ingest/"
      note: Searched `git grep -n -i 'rating' <commit> -- '*.java' ':!ratings' ':!ingest'`. No match. So the orders, payments and carts services never call ratings, and no purchase gate exists on that side either.
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Rating.java
      lines: [36, 37, 38, 42]
      note: The generator posts ratings for a random client and a random book, unrelated to any order, and the endpoint above accepts them. Ratings from people who never bought the book are written routinely.
  control: same grep shape (keyword grep over ratings for the sibling service a rating precondition would consult), pointed at the client-existence precondition createRating is known to enforce (read at RatingController.java lines 120-128) — `git grep -n -i 'client' <commit> -- ratings` matched ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:122
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Control: I ran `git grep -n -i 'client' <commit> -- ratings`. It matched ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:122, one of 39 hits. I also re-ran the original's absence search, `'order\|purchas\|bought\|paid'` over `ratings web/src/app/ratings`: no match, exit 1. I checked that the same BRE alternation does match under orders/, so the empty result is not a syntax failure.
    Why extend: two own_evidence entries are new.
    - Entry 5 is a reverse-direction search. No Java outside ratings/ and ingest/ mentions ratings, so no purchase gate exists on the calling side either. The original never checked that side.
    - Entry 7 is ingest/model/Rating.java:36-42. The load generator routinely posts ratings for random client/ISBN pairs, and the endpoint accepts them.
    Entry 8 (Order.java:11) restates where the purchase fact lives, which the original's OrderController entry already covers. The blind result does not reproduce the original's applySecurityPolicy() false-friend entry and does not dispute it either.
    The blind control `verify(Book|Client)\(` over ratings/src fired, at lines 77, 78, 120 and 130.
    Caller: the two new entries (5 and 7) are appended above; entry 8 is not, as the comparison found it a restatement.

- id: [CG#3]
  claim: [BR#2] — A customer can rate a given book only once.
  verdict: FALSE-FRIEND
  evidence:
    - path: ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java
      lines: [16]
      note: "`List<Rating> findByEmailAndIsbn(String email, String isbn)` is declared. The name looks like a one-rating-per-customer-per-book lookup. It returns a List, which itself expects several rows per pair. Orders returns a single entity instead (orders/.../OrderRepository.java:16)."
    - path: ratings/src
      note: Searched `git grep -n 'findByEmailAndIsbn' <commit>` across the repository. Callers exist only in carts (CartController.java:85,104,121) and orders (OrderController.java:97,147,179). Nothing in ratings calls it.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [74, 77, 78, 80]
      note: createRating saves without looking for an existing rating with the same e-mail and ISBN.
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [8, 9]
      note: "`@Table(name=\"ratings\")` has no uniqueConstraints. Hibernate generates the schema (spring.jpa.hibernate.ddl-auto=update, ratings/src/main/resources/application-prod.properties:9), and database/mysql/init/01-create-db.sql:5 only creates the database. So the database does not enforce uniqueness either. Searched `git grep -n -i 'unique' <commit> -- '*.java'`: no hit under ratings/."
    - path: carts/src/main/java/com/dynatrace/carts/model/Cart.java
      lines: [13]
      note: "For contrast, carts declares `@UniqueConstraint(columnNames = {\"email\",\"isbn\"})`, and so does orders at orders/src/main/java/com/dynatrace/orders/model/Order.java:11. Ratings has no equivalent."
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [338, 339, 340]
      note: Each iteration posts numRatings (700 in k8s/ingest.yaml:132) ratings, each for a random pair (ingest/.../model/Rating.java:36-42). Duplicate pairs are accepted.
  control: Same `git grep -n 'findByEmailAndIsbn' <commit>` search, pointed at orders, whose create path is known (read) to look up an existing order by e-mail and ISBN — orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:97; same `git grep -n -i 'unique' <commit> -- '*.java'` search, pointed at the orders e-mail+ISBN constraint (read) — orders/src/main/java/com/dynatrace/orders/model/Order.java:11
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: contradict
  notes: |
    Rewritten in place by verification: pre-rewrite verdict REWRITTEN.
    The verdicts differ: the original is REWRITTEN and the blind result is FALSE-FRIEND. That makes this a contradict. Both agree on the substance: nothing in ratings enforces one rating per customer per book, and RatingRepository.findByEmailAndIsbn (line 16) is declared but never called. The original listed that method as a "false friend" evidence entry under REWRITTEN. The blind result made it the verdict, which is the case the FALSE-FRIEND verdict exists for.
    Control: `git grep -n 'findByEmailAndIsbn' <commit>` matched OrderController.java:97, plus the other carts and orders callers. `git grep -n -i 'unique' <commit> -- orders` matched Order.java:11. I re-ran the original's absence search, `unique` over `ratings common web/src/app/ratings k8s`: no match.
    The blind evidence supports its own verdict. The method is declared, ratings never calls it (a repo-wide grep whose control fired), create does no lookup, and there is no unique constraint (control fired at Order.java:11).

- id: [CG#4]
  claim: [BR#3] — A rating, once given, is final.
  verdict: REWRITTEN
  evidence:
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [84, 86, 97]
      note: PUT /api/v1/ratings/{id} overwrites a stored rating with the request body
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [101, 103, 104, 108, 111]
      note: DELETE /api/v1/ratings/{id} removes a single rating and DELETE /api/v1/ratings/delete-all truncates the table
    - path: web/src/app/ratings/rating-list/rating-list.component.html
      lines: [22, 23, 33]
      note: the ratings list offers Update and Delete for every rating, plus Delete All
    - path: web/src/app/app-routing.module.ts
      lines: [63]
      note: the update-rating/:id route is registered
    - path: web/src/app/ratings/update-rating/update-rating.component.ts
      lines: [18, 19]
      note: the update form's onSubmit sends the PUT
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [335, 336, 337]
      note: every generator cycle deletes all ratings before creating new ones, through IngestRepository.deleteAll (ingest/src/main/java/com/dynatrace/ingest/repository/IngestRepository.java:15-18) calling delete-all, so while the load generator runs, ratings are not even durable
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    The original and the blind result both rest on update and delete paths being present, not on anything being absent, so neither owes a control. The blind result also cites RatingController.java:87 and :92 and rating.service.ts:36-37. These back up paths the original already cites rather than adding anything new, so the outcome is agree, not extend. I read all of these lines at the pinned commit.

- id: [CG#5]
  claim: [BR#4] — A rating is a whole number of stars from 1 to 5.
  verdict: AMENDED
  evidence:
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [22, 23]
      note: The value is stored in an `int` column, so a stored rating is always a whole number.
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [86, 87, 88, 90]
      note: setRating rejects values <= 0 or > 5, so 1–5 is enforced whenever the setter runs. The error text at line 88 says "between 0 and 5", but 0 is rejected.
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [36, 37, 38]
      note: The no-arg constructor that request-body binding uses does not set `rating`, so it starts at the int default 0. The setter runs only when the request includes a `rating` property.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [74, 80, 86, 97]
      note: Both POST and PUT save the bound object with no further range check. A request that leaves out `rating` therefore stores 0. The column is nullable=false (Rating.java:22), and 0 is not null.
    - path: ratings/src
      note: Searched `git grep -n -E '@(Min|Max|Valid|Validated|Check|Pattern|NotNull|Range)\b' <commit> -- '*.java'`. No hit under ratings/. No bean validation and no database CHECK backs the setter.
    - path: web/src/app/ratings/create-rating/create-rating.component.html
      lines: [16]
      note: "`<input type=\"number\">` has no min, max or step, and the update form is the same (update-rating.component.html:16). The UI allows 0, negative numbers and fractions, so the server is the only gate."
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Rating.java
      lines: [42]
      note: The generator emits `random.nextInt(4) + 1`, i.e. 1–4. It never produces a 5.
  control: Same `git grep -n -E '@(Min|Max|Valid|Validated|Check|Pattern|NotNull|Range)\b' <commit> -- '*.java'` search, pointed at the numeric range constraint on the carts quantity field (read) — carts/src/main/java/com/dynatrace/carts/model/Cart.java:30
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: implementation
  horizon: current
  consumed_by: none
  outcome: contradict
  notes: |
    Rewritten in place by verification: pre-rewrite verdict AMENDED (unchanged verdict; the search behind the correction was repaired with a positive control).
    The contradict comes from the control rule, not from a different verdict. The blind result is AMENDED, the same as the original.
    The original's correction rests on an absence. It says "the range check lives only in the setter" and that create and update "save the bound body with no further check on rating". In other words, no bean-validation annotation, entity hook or database constraint backs the setter. That is a negative over the codebase. No search in the record reaches it and the record carries no control, so under the grounding rules a control search is owed and missing.
    Because the control rule forces this contradict, it cannot be disputed. The re-derived finding can carry the blind result's evidence and its own_control.
    I checked the blind result's own absence and control. The validation grep has no hit under ratings/ and does hit carts/src/main/java/com/dynatrace/carts/model/Cart.java:30 (`@Min(1)`), so the blind control fires.
    Both sides agree on the substance: a POST or PUT that omits `rating` stores 0.

- id: [CG#6]
  claim: [BR#5] — The book page must show … the book's average rating, rounded to one decimal place
  verdict: REWRITTEN
  evidence:
    - path: web/src/app/books/book-details/book-details.component.html
      lines: [4, 8, 12, 16, 20, 24]
      note: the book page shows ISBN, title, author, price, language and vend status only; no rating, average or count. The average is new work at this commit
    - path: web/src/app/books/book-details/book-details.component.ts
      lines: [18, 19, 21]
      note: the page loads only the book from the books service and calls nothing in ratings
    - path: web/src/app/books/book.ts
      lines: [2, 3, 4, 5, 6, 7, 8]
      note: the web Book model has no rating field
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [38, 45, 52, 59, 72, 84, 101, 108]
      note: the whole ratings API is list all, by email, by ISBN, by id, create, update, delete and delete-all; nothing computes or rounds an average
    - path: .
      note: searched `git grep -n -i -E 'average|avg\(|\bavg\b|mean' <commit>` over java, ts, html, sql and properties files, which matched only the word means in the Config.java schema descriptions, and `git grep -n -i 'rating' <commit> -- books common web/src/app/books`, which found no match
  control: same grep shape `git grep -n -i 'rating' <commit> -- <web component dir>`, pointed at the ratings list template, which is known to show a rating value (read), matched web/src/app/ratings/rating-list/rating-list.component.html:18
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    Control: `git grep -n -i 'rating' <commit> -- web/src/app/ratings/rating-list` matched rating-list.component.html:18 (`{{rating.rating}}`). I re-ran the original's absence search, `rating` over `books common web/src/app/books`: no match.
    Each side has a small inaccuracy, and neither affects the verdict:
    - The original's average grep also hits common/src/main/java/com/dynatrace/controller/SecurityController.java:85 ("means" in a comment), not only the Config.java descriptions.
    - The blind result's aggregate grep also hits `country` in books/.../model/Author.java and books/.../model/Book.java:127, so VersionController row counts are not "the only `count` hits".
    The blind result's extra citations (app-routing.module.ts:44 and the wider cross-service Java grep) back up the original's evidence rather than add to it.

- id: [CG#7]
  claim: [BR#6] — The book page must show … the number of ratings the average is based on
  verdict: REWRITTEN
  evidence:
    - path: web/src/app/books/book-details/book-details.component.html
      lines: [4, 8, 12, 16, 20, 24]
      note: the book page shows no rating count; the count is new work at this commit
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [52, 53, 55]
      note: GET /api/v1/ratings/findByISBN returns a book's ratings, from which a count could be derived, but nothing on the book page calls it
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java
      lines: [30, 32]
      note: false friend — the Count message built from ratingRepository.count() reads like a rating count, but it is the total number of rows in the whole ratings table, carried in the version message
    - path: web/src/app/ingest/versions/version-list/version-list.component.html
      lines: [23]
      note: that message reaches the web only as the Message column of the Services Versions page, through ingest/src/main/java/com/dynatrace/ingest/controller/VersionController.java:71 and web/src/app/ingest/versions/version.service.ts:12-16 — it is not per book and is not on the book page
    - path: .
      note: searched `git grep -n -i -E 'count' <commit> -- ratings books web/src/app/books web/src/app/ratings common`, which matched only the two VersionController count() lines and the books country field, so there is no per-book count
  control: the count grep's own scope includes the table-wide rating count read at ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32, and it matched there; the 'rating' grep over web/src/app/books, pointed at the ratings list template (read), matched web/src/app/ratings/rating-list/rating-list.component.html:18
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    Control: the original's count grep over `ratings books web/src/app/books web/src/app/ratings common` matched ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32, as the record states. It also matched books VersionController.java:32 and the books `country` field. The 'rating' arm matched rating-list.component.html:18.
    Both sides call the VersionController count a service-wide total, not a per-book count. The only extra citation in the blind result is the route (app-routing.module.ts:44), which adds nothing new.

- id: [CG#8]
  claim: [BR#7] — Wherever an average rating would be shown, a book with no ratings yet must show "Not yet rated" instead.
  verdict: REWRITTEN
  evidence:
    - path: .
      note: searched `git grep -n -i 'not yet rated\|unrated\|no ratings' <commit>` across the whole repo — no match
    - path: web/src/app/books/book-details/book-details.component.html
      lines: [4, 8, 12, 16, 20, 24]
      note: no average is shown on the book page (CG#6) or on the book list (CG#9), so there is nowhere for the fallback to apply; the average and its empty state are both new work
  control: same case-insensitive literal grep, pointed at a conditional fallback label known on the book list (read) — `git grep -n -i 'NotVended' <commit>` matched web/src/app/books/book-list/book-list.component.html:22
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    Control: `git grep -n -i 'NotVended' <commit>` matched web/src/app/books/book-list/book-list.component.html:22. I re-ran the original's repo-wide absence search, `not yet rated\|unrated\|no ratings`: no match.
    The blind control also fired: 'Ratings List' matched rating-list.component.html:1.

- id: [CG#9]
  claim: [BR#8] — The list of books must show each book's average rating, rounded to one decimal place.
  verdict: REWRITTEN
  evidence:
    - path: web/src/app/books/book-list/book-list.component.html
      lines: [5, 6, 7, 8, 9, 10, 11, 17, 18, 19, 20, 21, 22]
      note: the list's columns are ISBN, Title, Author, Price, Lang, OnVend and Actions; there is no rating column, so the average is new work at this commit
    - path: web/src/app/books/book-list/book-list.component.ts
      lines: [18, 19, 20]
      note: the list loads books only and makes no ratings call
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [38, 45, 52, 59, 72, 84, 101, 108]
      note: no endpoint returns a per-book aggregate, for one book or for many
    - path: web/src/app/books
      note: searched `git grep -n -i 'rating' <commit> -- books common web/src/app/books` — no match
  control: same grep shape `git grep -n -i 'rating' <commit> -- <web component dir>`, pointed at the ratings list template, which is known to show a rating value (read), matched web/src/app/ratings/rating-list/rating-list.component.html:18
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    Control: the 'rating' grep matched rating-list.component.html:18. I re-ran the original's absence search over `books common web/src/app/books`: no match.
    The blind result's extra citations (the route at app-routing.module.ts:43 and the wider Java grep) back up the original's evidence rather than add to it.

- id: [CG#10]
  claim: [BR#9] — A customer must be able to change their rating of a book at any time. The new rating replaces the old one.
  verdict: AMENDED
  evidence:
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [84, 86, 87, 92, 97]
      note: PUT /api/v1/ratings/{id} saves the submitted rating under the same id, so it replaces the stored values in place. The only checks are that the rating exists and the id matches. There is no time window.
    - path: web/src/app/ratings/rating-list/rating-list.component.html
      lines: [22]
      note: The Update button goes to `update-rating/:id` (app-routing.module.ts:63). UpdateRatingComponent loads the rating (update-rating.component.ts:27-30) and submits a PUT (update-rating.component.ts:18-19; rating.service.ts:36-37).
    - path: web/src/app/ratings/update-rating/update-rating.component.html
      lines: [6, 11, 16]
      note: The edit form also lets the user change the e-mail and ISBN. The edit works on any rating record by id. It is not limited to a customer changing their own rating.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [74, 77, 78, 80]
      note: "A customer who submits a new rating through POST gets a second row. Nothing replaces the old one: create does no lookup by e-mail and ISBN, and RatingRepository.findByEmailAndIsbn (line 16) is never called."
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [86, 97]
      note: Unlike create (lines 77-78), the update path does not re-check that the book and client exist.
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [37, 114, 115]
      note: The bound object's createdAt is set to now (line 37), and setCreatedAt does nothing (114-115). Every update therefore overwrites the original creation time.
  control: Same `git grep -n -E 'verify(Book|Client)\(' <commit> -- ratings/src` search for checks run on a write path, pointed at createRating where they are known (read) to run — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:77; same `git grep -n 'findByEmailAndIsbn' <commit>` search, pointed at the orders create lookup (read) — orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:97
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: contradict
  notes: |
    Rewritten in place by verification: pre-rewrite verdict AMENDED (unchanged verdict; the searches behind the corrections were repaired with positive controls).
    The contradict comes from the control rule, not from a different verdict. The blind result is AMENDED, the same as the original.
    The original's corrections assert several absences, and the record carries no control for any of them:
    - "nothing restricts when"
    - "the update path checks no ownership and re-verifies neither client nor book"
    - "posting another rating ... adds a second row instead of replacing the first"
    - "so any caller can rewrite any rating"
    The last one is a negative over the whole stack: it assumes no authentication or authorisation layer exists anywhere. No search in the record reaches it, so under the grounding rules a control search is owed and missing. This contradict cannot be disputed.
    The blind controls fire: verify(Book|Client)\( at RatingController.java:77, and findByEmailAndIsbn at OrderController.java:97.
    Do not carry "any caller can rewrite any rating" into the re-derived record. The blind result found no positive control for a filter, interceptor or guard search and explicitly does not rest on "there is no authentication". What its evidence supports is narrower: the edit works on any rating by id, and the method itself has no ownership check.

- id: [CG#11]
  claim: [BR#10] — Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who buys a book and then rates it.
  verdict: FALSE-FRIEND
  evidence:
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [327, 331, 333, 337, 340]
      note: false friend — each generator cycle creates orders, pays some of them, then wipes and recreates ratings, which reads as a customer buying and then rating
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Rating.java
      lines: [37, 38, 42]
      note: each rating draws its own random client email and ISBN, independently of the orders, which draw theirs separately (ingest/src/main/java/com/dynatrace/ingest/model/Order.java:53-54), so the rater is generally not someone who bought that book. Rating values are random.nextInt(4)+1, i.e. 1 to 4 and never 5
    - path: ingest/src/main/java/com/dynatrace/ingest/repository/OrderRepository.java
      lines: [70, 73, 78, 83]
      note: the pay step submits a random previously generated order, which is also unlinked to the later rating
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [291, 292, 294]
      note: cycles repeat back to back while isWorking, with no interval or clock; cycle length depends on the requested volumes, so nothing sets or guarantees a once-a-minute cadence
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [38, 42, 56, 58]
      note: generation starts only when someone POSTs /api/v1/ingest, and the next POST stops it. This is the only start path read; the finding does not rest on there being no other
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Ingest.java
      lines: [35, 36]
      note: continuousLoad, described as an infinite loop until called again, reads like always-on monitoring but is that manual toggle
    - path: k8s/ingest.yaml
      lines: [2, 9]
      note: ingest is deployed as a plain one-replica Deployment, not as a CronJob or any other schedule
    - path: .
      note: searched `git grep -n -i -E 'synthetic|browser.?monitor|http.?monitor|clickpath' <commit>` outside markdown — no match. A synthetic monitor, if one exists, is configured in an external monitoring tool, which this repository does not contain, so its existence cannot be established here
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [287, 291, 292, 294]
      note: The loop runs batch iterations back to back while isWorking is true, with no sleep or schedule (`git grep -n -i -E 'sleep|Thread\.sleep|TimeUnit' <commit> -- ingest/src` found only HTTP timeouts in ConfigController and VersionController). Each batch is about 1000 carts, 2000 storage items, 500 orders, 455 payments and 700 ratings over HTTP (k8s/ingest.yaml:132). Nothing guarantees a buy-then-rate at least once a minute.
    - path: k8s/ingest.yaml
      lines: [113, 132, 134]
      note: The pod starts the generator with one POST after a sleep. Its `while true` loop afterwards only polls /version and /config. There is nothing scheduled and nothing that checks or alerts, so this is a load generator, not a monitor.
  control: the schedule grep `git grep -n -i -E 'CronJob|schedule:|cron' <commit> -- k8s .github '*.yaml' '*.yml' '*.sh'` matched a cron trigger known present in a YAML example (read) at .github/dtctl.instructions.md:149; the synthetic-monitor grep shape, pointed at a Dynatrace monitoring setting known in this repo (read), matched k8s/ingest.yaml:18; the Spring startup and scheduling-annotation grep `git grep -n -E 'ApplicationReadyEvent|CommandLineRunner|ApplicationRunner|@PostConstruct|@EventListener|@Scheduled|@EnableScheduling' <commit>` has no known case of the same kind anywhere in this repo to point it at — no match
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: architecture
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Why extend: three own_evidence items are new.
    - Entry 1 cites CLAUDE.md:79, .github/copilot-instructions.md:68 and .kiro/instructions.md:68. These are where the "synthetic" name that makes ingest look like the answer comes from.
    - Entry 5 cites k8s/ingest.yaml:113, :132 and :134. The Deployment's second container sleeps 360 s, then POSTs /api/v1/ingest with continuousLoad:true and numRatings 700, then only polls /version and /config.
    - Entry 4 adds the sleep/TimeUnit grep over ingest/src.
    Entry 5 qualifies two of the original's points. The start-path note (IngestController.java:38-58) and the "manual toggle" reading of Ingest.java:35-36 should not be read as "never runs unattended". In the k8s deployment, the pod's own startup script switches the generator on automatically and leaves it looping. The original said it does not rest on there being no other start path, so its verdict still stands.
    The control has three arms:
    - Arm 1, the schedule grep over `k8s .github '*.yaml' '*.yml' '*.sh'`, matched .github/dtctl.instructions.md:148-149. It fired.
    - Arm 2's pattern is not stated in the record. I re-ran the same grep shape with markdown excluded, aimed at the named setting (`metrics\.dynatrace\.com/scrape`), and it matched k8s/ingest.yaml:18. That line is a Prometheus scrape annotation, which is a weak "same kind" for a synthetic-monitor definition.
    - Arm 3, the startup/scheduling-annotation grep, returned no match. That repeats the failure the record itself states.
    The verdict does not rest on arm 3's absence. FALSE-FRIEND stands on direct reads: ratings use random pairs (Rating.java:37-38), orders use their own separate random pairs (Order.java:53-54), and the loop runs without a pause. Even a scheduler would not make ingest rate the book it bought. So the owed control is reported as fired.
    The blind controls fired too: dtctl.instructions.md:149 and CLAUDE.md:79.
    Caller: entries 4 and 5 are appended above. Entry 1 is not: it cites repository documentation (CLAUDE.md and two agent-instruction files), and under the grounding rules a documentation page is never evidence for a code finding.

- id: [CG#12]
  claim: [BR#11] — Clicking the rating count on the book page opens the list of the book's ratings, as the sketch's note says: "Clicking the count opens the list of this book's ratings."
  verdict: FALSE-FRIEND
  evidence:
    - path: web/src/app/books/book-details/book-details.component.html
      note: Read the whole file (26 lines). The book page has no rating count and no link or click handler that goes to ratings.
    - path: web/src/app/ratings/rating-list/rating-list.component.ts
      lines: [33, 34, 35]
      note: "RatingListComponent.getRatingsByISBN looks like \"the list of this book's ratings\". Nothing calls it: `git grep -n -E 'getRatingsByISBN' <commit> -- web/src ':!*.spec.ts'` finds only the definition."
    - path: web/src/app/ratings/rating-list/rating-list.component.html
      lines: [22, 23, 24]
      note: The template calls only updateRating, deleteRating and ratingDetails. It never calls getRatingsByISBN.
    - path: web/src/app/app-routing.module.ts
      lines: [61]
      note: The `ratings` route takes no ISBN parameter, so no navigation can open the list filtered to one book.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [52, 54, 55]
      note: The backend GET /api/v1/ratings/findByISBN does return a book's ratings, with a client wrapper at web/src/app/ratings/rating.service.ts:28-29. It is a real building block, but no book page uses it.
  control: Same `git grep -n -E '<RatingListComponent method>' <commit> -- web/src ':!*.spec.ts'` search, pointed at the sibling method updateRating, which is known (read) to be called from the template — web/src/app/ratings/rating-list/rating-list.component.html:22
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: contradict
  notes: |
    Rewritten in place by verification: pre-rewrite verdict REWRITTEN.
    The verdicts differ: the original is REWRITTEN and the blind result is FALSE-FRIEND. That makes this a contradict. Both agree on the substance:
    - the book page has no count to click and no link to ratings;
    - the `ratings` route takes no ISBN;
    - getRatingsByISBN is defined and never called;
    - the backend findByISBN endpoint exists.
    The original listed getRatingsByISBN as a "false friend" evidence entry under REWRITTEN. The blind result made it the verdict.
    Control: `git grep -n 'updateRating' <commit> -- web/src/app/ratings` matched rating-list.component.html:22. I re-ran the original's absence search, `getRatingsByISBN\|getRatingByISBN` over web: it matched only the definitions at rating-list.component.ts:33-34 and rating.service.ts:28.
    The blind evidence supports its own verdict: it names the decoy, and the decoy's lack of callers is backed by a control that fired.

## Derivation matrix

Ran by default: the claim list asks the store to hold or display data elements (BR#4–BR#8, BR#11 —
the star value, a per-book average, a per-book count, an empty state and a per-book list). One
implementation-altitude row per data element, classed EXISTS (already captured and stored) or
DERIVED (computed from data that already exists), each resting on the verified finding it cites.

| element | asked by | class | source | finding |
|---|---|---|---|---|
| Rating value — whole stars, 1 to 5 | [BR#4] | EXISTS | `ratings.rating` int column (`ratings/.../model/Rating.java:22-23`); range enforced only in the setter, so an omitted value stores 0 | [CG#5] |
| Who rated which book (customer e-mail, ISBN) | [BR#1], [BR#2] | EXISTS | `ratings.email` and `ratings.isbn` columns; no uniqueness constraint on the pair | [CG#3] |
| Whether the rater bought the book | [BR#1] | EXISTS | stored only in the orders service (order e-mail + ISBN, completed and paid); the ratings service never reads it | [CG#2] |
| A book's average rating, rounded to one decimal place | [BR#5], [BR#8] | DERIVED | computable from that book's ratings rows (`GET /api/v1/ratings/findByISBN`); nothing computes, stores or rounds it today | [CG#6], [CG#9] |
| The number of ratings a book's average is based on | [BR#6] | DERIVED | count of that book's ratings rows; the only existing count is table-wide (`VersionController.java:32`) | [CG#7] |
| "Not yet rated" state | [BR#7] | DERIVED | the zero-count case of the per-book count | [CG#8] |
| The list of a book's ratings | [BR#11] | EXISTS | `GET /api/v1/ratings/findByISBN` returns it; no screen requests it | [CG#12] |

Every `DERIVED` row is computed over the ratings table, which the running load generator empties and
refills on every cycle ([CG#4]).

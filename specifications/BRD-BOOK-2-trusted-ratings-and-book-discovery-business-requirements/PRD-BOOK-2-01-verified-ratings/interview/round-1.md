# Interview round 1: BOOK-2-01

Opened 2026-10-12 by `/product-workflows:brd-interview BOOK-2-01` — the first round, generated from
the verified grounding (`grounding/code-grounding.md`, `bookstore` pinned at `e439178`) and the
parent BOOK-2's requirement-defect log. No design grounding exists for this slice, so no `[DG#n]`
was read.

generated against: [CG#1]=CONFIRMED/agree, [CG#2]=REWRITTEN/extend, [CG#3]=FALSE-FRIEND/contradict, [CG#4]=REWRITTEN/agree, [CG#5]=AMENDED/contradict, [CG#6]=REWRITTEN/agree, [CG#7]=REWRITTEN/agree, [CG#8]=REWRITTEN/agree, [CG#9]=REWRITTEN/agree, [CG#10]=AMENDED/contradict, [CG#11]=FALSE-FRIEND/extend, [CG#12]=FALSE-FRIEND/contradict

## Scope

This round covers 11 rows — [BR#1] to [BR#11], every one `covered-here` in `coverage-ledger.md`.
No row was left to another BRD (none is `covered-by`), and no orphan row was passed over.

Sources walked: every row's finding moved its premise (REWRITTEN, AMENDED or FALSE-FRIEND), so each
row raised a question; no finding is `NOT-PROVABLE`; no finding carries a `will-change` horizon; no
design divergence exists; no row is `rejected` or `deferred-to`; twelve open requirement defects
joined to these rows are this slice's to ask; nothing is asserted without evidence, so no `[AS#n]`
was recorded. No register was on file before the round.

The unsourced defects on these rows — [DEF#2], [DEF#12] and [DEF#15] — are grounding's to settle and
raise no question here; [CG#2], [CG#11] and [CG#12] bear on them.

## Questions

### 1. [G] — [BR#1]

Does the store today let only a customer who has bought a book rate it, as requirement 3.1 says?

- **State:** answered from findings — [CG#2] REWRITTEN: no. Rating checks only that the book and the
  customer exist; nothing consults orders or payments, and the load generator routinely writes
  ratings from customers who never bought the book. 3.1 describes new work.

### 2. [C] — [BR#1]

Which orders entitle a customer to rate a book ("has bought")?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 2
- **State:** answered by the customer — [CD#1], from `customer-review-20261012.md` (reconciled 20261012)

### 3. [C] — [BR#1]

When the buyers-only rule takes effect, what happens to ratings already given by customers who never
bought the book?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 3
- **State:** answered by the customer — [CD#2], from `customer-review-20261012.md` (reconciled 20261012)
- **State:** answered by the customer — [CD#19], from Quillmark's follow-up meeting of 20261012 (recorded by hand 20261012)

### 4. [G] — [BR#2]

Does the store today stop a customer rating the same book twice?

- **State:** answered from findings — [CG#3] FALSE-FRIEND: no. `findByEmailAndIsbn` reads like a
  once-only lookup but ratings never calls it; create does no lookup, the ratings table has no
  uniqueness constraint, and duplicate pairs are accepted.

### 5. [C] — [BR#2]

What happens to the repeat ratings already given?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 5
- **State:** answered by the customer — [CD#3], from `customer-review-20261012.md` (reconciled 20261012)
- **State:** answered by the customer — [CD#20], from Quillmark's follow-up meeting of 20261012 (recorded by hand 20261012)

### 6. [C] — [BR#2], [BR#3]

Do the two obligations split from requirement 3.2 stand independently?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 6
- **State:** answered by the customer — [CD#4], from `customer-review-20261012.md` (reconciled 20261012)

### 7. [G] — [BR#3]

Is a rating final once given, today?

- **State:** answered from findings — [CG#4] REWRITTEN: no. Any rating can be updated and deleted,
  a delete-all exists in the API and the ratings list, and the load generator deletes every rating
  at the start of each cycle.

### 8. [C] — [BR#3], [BR#9]

Requirement 3.2 (final) against 3.7 (changeable at any time): which governs?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 8
- **State:** answered by the customer — [CD#5], from `customer-review-20261012.md` (reconciled 20261012)

### 9. [C] — [BR#3]

May a customer remove their own rating?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 9
- **State:** answered by the customer — [CD#6], from `customer-review-20261012.md` (reconciled 20261012)

### 10. [V] — [BR#3]

How are ratings protected from bulk deletion — the storefront's delete-all and the load generator's
per-cycle wipe?

- **Re-puts:** none
- **State:** decided — [VD#1]

### 11. [G] — [BR#4]

Is a rating a whole number of stars from 1 to 5 today?

- **State:** answered from findings — [CG#5] AMENDED: mostly. The value is an `int` and the setter
  rejects values at or below 0 and above 5, but a request that omits the value stores 0; no
  validation or database check backs the setter, and the UI input allows 0, negatives and fractions.

### 12. [C] — [BR#4]

Is the 1-to-5 whole-star scale a requirement, or a description of today's scale?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 12
- **State:** answered by the customer — [CD#7], from `customer-review-20261012.md` (reconciled 20261012)

### 13. [V] — [BR#4]

How is the 1-to-5 whole-star value enforced?

- **Re-puts:** none
- **State:** decided — [VD#2]

### 14. [G] — [BR#5]

Does the book page show a book's average rating today?

- **State:** answered from findings — [CG#6] REWRITTEN: no. The book page shows no rating, and
  nothing in the store computes or rounds an average. New work.

### 15. [C] — [BR#5], [BR#6]

Do the two obligations split from requirement 3.4 stand as separate requirements?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 15
- **State:** answered by the customer — [CD#8], from `customer-review-20261012.md` (reconciled 20261012)

### 16. [V] — [BR#5], [BR#8]

How is the average rounded to one decimal place?

- **Re-puts:** none
- **State:** decided — [VD#3]

### 17. [G] — [BR#6]

Does the book page show the number of ratings today?

- **State:** answered from findings — [CG#7] REWRITTEN: no. The only rating count in the store is a
  table-wide total in the version message, not per book. New work.

### 18. [G] — [BR#7]

Does the store show "Not yet rated" anywhere today?

- **State:** answered from findings — [CG#8] REWRITTEN: no. No average is shown anywhere, so the
  empty state is new work with it.

### 19. [C] — [BR#7]

Does "Not yet rated" replace the number of ratings too?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 19
- **State:** answered by the customer — [CD#9], from `customer-review-20261012.md` (reconciled 20261012)

### 20. [G] — [BR#8]

Does the list of books show each book's average today?

- **State:** answered from findings — [CG#9] REWRITTEN: no. The catalogue list has no rating column
  and no endpoint returns a per-book aggregate. New work.

### 21. [C] — [BR#8]

Which lists of books does requirement 3.6 cover?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 21
- **State:** answered by the customer — [CD#10], from `customer-review-20261012.md` (reconciled 20261012)

### 22. [G] — [BR#9]

Can a customer change their rating today, the new one replacing the old?

- **State:** answered from findings — [CG#10] AMENDED: partly. A stored rating can be overwritten in
  place by id at any time, but the edit works on any rating, may change its customer e-mail and
  ISBN, and overwrites its creation time; posting another rating adds a second row instead of
  replacing the first.

### 23. [V] — [BR#9], [BR#3]

What may a change to a rating alter?

- **Re-puts:** none
- **State:** decided — [VD#4]

### 24. [G] — [BR#10]

Does the store run synthetic monitoring that buys a book and then rates it?

- **State:** answered from findings — [CG#11] FALSE-FRIEND: no. The load generator reads like it but
  rates random customer and book pairs unrelated to its orders, runs back to back with no schedule,
  never rates 5, and checks or alerts on nothing; no synthetic monitor is defined in the store's code.

### 25. [C] — [BR#10]

Does requirement 3.8 ask the supplier to build a new monitor, or to support one Quillmark runs?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 25
- **State:** answered by the customer — [CD#11], from `customer-review-20261012.md` (reconciled 20261012)

### 26. [C] — [BR#10]

Do the monitoring's purchases and ratings count toward averages, the shelf and suggestions?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 26
- **State:** answered by the customer — nothing frozen. The customer declined it as moot in `customer-review-20261012.md`, because [CD#11] withdrew [BR#10] and its requirement defect [DEF#10] is `withdrawn`. To be put again if 3.8 returns (closed 20261012, with round 2)

### 27. [C] — [BR#10]

Which customer and books does the monitoring use?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 27
- **State:** answered by the customer — nothing frozen. The customer declined it as moot in `customer-review-20261012.md`, because [CD#11] withdrew [BR#10] and its requirement defect [DEF#11] is `withdrawn`. To be put again if 3.8 returns, split as [SR#1] proposes ([CD#13]) (closed 20261012, with round 2)

### 28. [G] — [BR#11]

Does clicking the rating count on the book page open the book's ratings today?

- **State:** answered from findings — [CG#12] FALSE-FRIEND: no. The book page has no count and no
  link to ratings; `getRatingsByISBN` reads like that list but nothing calls it, and the ratings
  route takes no book. The ratings-by-ISBN endpoint exists as a building block.

### 29. [C] — [BR#11]

Is the sketch's note a requirement of this release?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 29
- **State:** answered by the customer — [CD#12], from `customer-review-20261012.md` (reconciled 20261012)

### 30. [C] — [BR#11]

What does the list of a book's ratings show for each rating?

- **State:** held for the customer — `interview/customer-questions.md`, Round 1, question 30
- **State:** answered by the customer — nothing frozen. The customer declined it as moot in `customer-review-20261012.md`, because [CD#12] withdrew [BR#11] and its requirement defect [DEF#14] is `withdrawn`. To be put again if the list returns (closed 20261012, with round 2)

## Account

requirement defects: [DEF#1], [DEF#3], [DEF#4], [DEF#5], [DEF#6], [DEF#7], [DEF#8], [DEF#9], [DEF#10], [DEF#11], [DEF#13], [DEF#14] asked
code defects: none
Status: open — waiting on held for the customer
Status: closed 20261012 — the last held questions, 26, 27 and 30, closed with nothing frozen: their requirements were withdrawn by [CD#11] and [CD#12]

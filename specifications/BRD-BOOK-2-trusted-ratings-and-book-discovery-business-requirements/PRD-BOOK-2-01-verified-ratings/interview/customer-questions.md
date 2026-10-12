# Customer questions: BOOK-2-01

Questions only Quillmark Booksellers can settle, held for the review package. Each is put against
what the store does today, as established from its code at commit `e439178`.

## Round 1, question 2

Requirement 3.1 (`source/quillmark-trusted-ratings-brd.md › § 3.1`) lets only a customer who has
bought a book rate it, but "has bought" does not say which order qualifies — one that is placed,
paid or delivered — or whether an order later cancelled still does. In the store, a purchase is an
order for the book that is completed and paid, and the order records are the only place that says
who bought what. Which orders should entitle a customer to rate the book?

Options:
- An order for that book that is completed and paid, and not since cancelled (the reading we propose)
- Any order placed for that book, paid or not
- An order for that book that has been delivered

- **Findings:** [CG#2] REWRITTEN
- **Altitude:** product
- **Requirement defect:** [DEF#1]

## Round 1, question 3

Requirement 3.1 (`source/quillmark-trusted-ratings-brd.md › § 3.1`) says the store already lets only
a buyer rate a book. It does not: today any registered customer can rate any book, and ratings from
customers who never bought the book are on file. So 3.1 describes new work, which we read as still
required in this release. When the buyers-only rule takes effect, what should happen to the ratings
already given by customers who never bought the book?

Options:
- Remove them
- Keep them, but leave them out of every average and count
- Keep them and count them, as today

- **Findings:** [CG#2] REWRITTEN
- **Altitude:** product

## Round 1, question 5

Requirement 3.2 (`source/quillmark-trusted-ratings-brd.md › § 3.2`) lets a customer rate a given book
only once, and the introduction to section 3
(`source/quillmark-trusted-ratings-brd.md › 3. Verified ratings`) names repeat ratings as a reason
shoppers distrust them. Today nothing stops a customer rating the same book again, and repeat
ratings are on file. 3.2 does not say what happens to the repeat ratings already given, and the
averages in 3.4 and 3.6 depend on the answer. What should happen to them?

Options:
- Keep only the latest rating each customer gave each book, and remove the rest (the reading we propose)
- Keep only the first rating each customer gave each book, and remove the rest
- Keep all of them, and apply the once-only rule to new ratings only

- **Findings:** [CG#3] FALSE-FRIEND
- **Altitude:** product
- **Requirement defect:** [DEF#3]

## Round 1, question 6

Requirement 3.2 (`source/quillmark-trusted-ratings-brd.md › § 3.2`) holds two obligations in one
numbered item, and we have recorded them as two requirements: [BR#2] "A customer can rate a given
book only once" and [BR#3] "A rating, once given, is final". Do the two stand independently — so
that, if 3.7's change-at-any-time is the rule that governs (round 1, question 8), a customer still
has exactly one rating per book, which they change rather than add to?

Options:
- Yes: two independent requirements, and one rating per customer per book holds either way (the reading we propose)
- No: 3.2 is one rule, and it stands or falls as a whole

- **Findings:** [CG#3] FALSE-FRIEND, [CG#4] REWRITTEN
- **Altitude:** product
- **Requirement defect:** [DEF#4]

## Round 1, question 8

Requirement 3.2 (`source/quillmark-trusted-ratings-brd.md › § 3.2`) says a rating, once given, is
final. Requirement 3.7 (`source/quillmark-trusted-ratings-brd.md › § 3.7`) says a customer must be
able to change their rating of a book at any time, the new rating replacing the old one. Both
cannot hold. Today a stored rating can be changed, or deleted, at any time. Which rule governs?

Options:
- 3.7 governs: a customer can change their rating at any time, and "final" in 3.2 is withdrawn
- 3.2 governs: a rating is final once given, and 3.7 is withdrawn
- A rating can be changed for a limited period after it is given, then becomes final (please give the period)

- **Findings:** [CG#4] REWRITTEN, [CG#10] AMENDED
- **Altitude:** product
- **Requirement defect:** [DEF#5]

## Round 1, question 9

Today any rating can be deleted. Neither requirement 3.2
(`source/quillmark-trusted-ratings-brd.md › § 3.2`) nor 3.7
(`source/quillmark-trusted-ratings-brd.md › § 3.7`) says whether a customer may remove a rating they
gave. Should a customer be able to remove their own rating of a book?

Options:
- No: once given, a rating can be changed (where 3.7 allows it) but not removed
- Yes: a customer can remove their own rating, and the book's average and count update accordingly

- **Findings:** [CG#4] REWRITTEN
- **Altitude:** product

## Round 1, question 12

Requirement 3.3 (`source/quillmark-trusted-ratings-brd.md › § 3.3`) says a rating is a whole number
of stars from 1 to 5. It carries no "must", which section 1
(`source/quillmark-trusted-ratings-brd.md › 1. About this document`) makes the mark of a requirement,
yet it is numbered as one. Is the 1-to-5 whole-star scale a requirement of this release, or a
description of the scale the store uses today? Today the scale is whole stars 1 to 5, although a
rating submitted with no value is stored as 0; we will refuse such a rating either way.

Options:
- A requirement: every rating is a whole number of stars from 1 to 5 (the reading we propose)
- A description of today's scale, not a requirement of this release

- **Findings:** [CG#5] AMENDED
- **Altitude:** product
- **Requirement defect:** [DEF#6]

## Round 1, question 15

Requirement 3.4 (`source/quillmark-trusted-ratings-brd.md › § 3.4`) holds two obligations in one
numbered item, and we have recorded them as two requirements: [BR#5] the book page shows the book's
average rating, rounded to one decimal place, and [BR#6] it shows the number of ratings the average
is based on. Do the two stand as separate requirements, with the number shown always being the
count of exactly the ratings the average is computed from?

Options:
- Yes: two requirements, and the number is always the count of the ratings in the average (the reading we propose)
- No: 3.4 is one requirement (please say how it differs)

- **Findings:** [CG#6] REWRITTEN, [CG#7] REWRITTEN
- **Altitude:** product
- **Requirement defect:** [DEF#7]

## Round 1, question 19

Requirement 3.5 (`source/quillmark-trusted-ratings-brd.md › § 3.5`) says a book with no ratings yet
shows "Not yet rated" instead of an average. It does not say what happens to the number of ratings
that 3.4 (`source/quillmark-trusted-ratings-brd.md › § 3.4`) shows beside the average. For a book
with no ratings, what should the book page show?

Options:
- "Not yet rated" alone, with no count (the reading we propose)
- "Not yet rated" together with "(0 ratings)"

- **Findings:** [CG#8] REWRITTEN, [CG#7] REWRITTEN
- **Altitude:** product
- **Requirement defect:** [DEF#8]

## Round 1, question 21

Requirement 3.6 (`source/quillmark-trusted-ratings-brd.md › § 3.6`) says "the list of books" shows
each book's average rating. Which lists does that cover? Today the store has one list of books, the
catalogue list, and it shows no ratings. This release adds two more: the book-page suggestions
(`source/quillmark-trusted-ratings-brd.md › § 4.1`, BOOK-2 [BR#12]) and the Top rated shelf
(`source/quillmark-trusted-ratings-brd.md › § 4.3`, BOOK-2 [BR#16]).

Options:
- The catalogue list only
- Every list of books in the store: the catalogue list, search results wherever the store shows them, the book-page suggestions and the Top rated shelf

- **Findings:** [CG#9] REWRITTEN
- **Altitude:** product
- **Requirement defect:** [DEF#9]

## Round 1, question 25

Requirement 3.8 (`source/quillmark-trusted-ratings-brd.md › § 3.8`) speaks of "our synthetic
monitoring" acting as a customer who buys a book and then rates it. The store contains no synthetic
monitor. Its load generator resembles one but is not: it generates traffic with unrelated random
customers and books, on no schedule, and the customer who rates a book is generally not the one who
bought it. A monitor configured in a monitoring tool outside the store's code cannot be seen from
the store. Is 3.8 asking us to build and run a new synthetic monitor, or to support a monitor
Quillmark already runs?

Options:
- Build: we build and run a new synthetic monitor that buys a book and then rates it
- Support: Quillmark already runs synthetic monitoring (please name the tool and who operates it), and the store must let it buy and rate

- **Findings:** [CG#11] FALSE-FRIEND
- **Altitude:** product

## Round 1, question 26

Requirement 3.8 (`source/quillmark-trusted-ratings-brd.md › § 3.8`) has the monitoring buy and rate
a book at least once a minute, around the clock — more than half a million purchases and ratings a
year. It does not say whether these count toward a book's average and number of ratings
(`source/quillmark-trusted-ratings-brd.md › § 3.4`, `source/quillmark-trusted-ratings-brd.md › § 3.6`),
the Top rated shelf (`source/quillmark-trusted-ratings-brd.md › § 4.3`,
`source/quillmark-trusted-ratings-brd.md › § 4.4`; BOOK-2 [BR#16], BOOK-2 [BR#17]) and the
also-bought suggestions (`source/quillmark-trusted-ratings-brd.md › § 4.1`, BOOK-2 [BR#12]). Should
the monitoring's purchases and ratings:

Options:
- Be kept out of every average, count, shelf and suggestion (the reading we propose)
- Count like any other customer's purchases and ratings

- **Findings:** [CG#11] FALSE-FRIEND
- **Altitude:** product
- **Requirement defect:** [DEF#10]

## Round 1, question 27

Requirement 3.8 (`source/quillmark-trusted-ratings-brd.md › § 3.8`) does not say which customer and
which books the monitoring uses, or whether its purchases are paid orders. Under 3.2
(`source/quillmark-trusted-ratings-brd.md › § 3.2`) a customer can rate a given book only once, so a
monitor that rates once a minute needs a new customer and book pairing on every run — unless
changing a rating (3.7, round 1, question 8) is allowed and counts as the monitor's "rate". What
should the monitoring use?

Options:
- One dedicated monitoring customer and one dedicated test book, buying with a paid order and changing its rating on each run (needs 3.7 to govern)
- A new monitoring customer on each run, buying a catalogue book with a paid order and rating it once

- **Findings:** [CG#11] FALSE-FRIEND
- **Altitude:** product
- **Requirement defect:** [DEF#11]

## Round 1, question 29

The only statement that clicking the rating count opens the list of the book's ratings is the note
on the book-page sketch, Figure 1
(`source/assets/book-page-wireframe.svg › "Clicking the count opens the list of this book's ratings."`).
Section 1 (`source/quillmark-trusted-ratings-brd.md › 1. About this document`) makes a sentence that
says something must happen the mark of a requirement, and the note is a picture's annotation with no
"must". Today the book page shows no rating count and links to no list of its ratings. Is the note a
requirement of this release?

Options:
- Yes: clicking the count on the book page opens the list of that book's ratings
- No: the note illustrates an idea, not a requirement of this release

- **Findings:** [CG#12] FALSE-FRIEND
- **Altitude:** product
- **Requirement defect:** [DEF#13]
- **Defect image:** source/assets/book-page-wireframe.svg

## Round 1, question 30

If the list of a book's ratings is required (round 1, question 29), what should it show for each
rating? The sketch (`source/assets/book-page-wireframe.svg`) does not say, and written reviews are
out of scope (`source/quillmark-trusted-ratings-brd.md › 5. Out of scope`).

Options:
- The stars and the date only, showing no customer details (the reading we propose)
- The stars, the date and who gave the rating (please say how the customer should be named)

- **Findings:** [CG#12] FALSE-FRIEND
- **Altitude:** product
- **Requirement defect:** [DEF#14]
- **Defect image:** source/assets/book-page-wireframe.svg

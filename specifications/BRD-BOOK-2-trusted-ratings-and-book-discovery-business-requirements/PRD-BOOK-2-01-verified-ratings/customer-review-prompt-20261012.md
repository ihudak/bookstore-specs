> **Banner, 20261012 — superseded in part by `/brd-reconcile` (`reconciliation-20261012.md`).** This file records the BOOK-2-01 customer review prompt as sent on 20261012, and nothing beneath this banner has changed. Quillmark's returned review (`customer-review-20261012.md`) answered its part 7. Customer decisions [CD#1] to [CD#18] are frozen in `decisions.md`, and [VD#1] and [VD#4] are reopened there: [VD#1] by [CD#2], [CD#3], [CD#5], [CD#6] and [CD#16], and [VD#4] by [CD#17]. Three requirements are withdrawn: 3.2's finality ([BR#3], by [CD#5]), 3.8 ([BR#10], by [CD#11]) and the Figure 1 note ([BR#11], by [CD#12]). Round 1, questions 26, 27 and 30 were declined and remain held for the customer. The current position is recorded in `decisions.md`, `coverage-ledger.md`, `interview/customer-questions.md`, the parent's `brd/brd-defect-log.md` and `reconciliation-20261012.md`. The copy of this prompt inside `bundle-20261012/` is the unbannered original sent to the customer, and is the one to quote from.

# BOOK-2-01 — customer review prompt

You have been sent a review package from your supplier for **BOOK-2-01**, the part of Quillmark
Booksellers' business requirements *Trusted ratings and book discovery* (BOOK-2) that covers verified
ratings. This prompt tells you how to review it and what single file to send back. Work through its
eleven parts in order.

## 1. Setup

**What this prompt assumes.** This prompt assumes an agent that can read files in a folder and search
for a file by name; the pin check in part 4 additionally needs a terminal, and a reviewer whose tool
has none says so in their section 1 and skips it.

**The tier of this package: Full.** Alongside the documents, you are given the `bookstore` repository
pinned to commit `e439178550371f2bc4f5a9acfee9a1c4f2289ae4`, the commit every code claim in this
package was checked against. The tier is a fact about what you were given, not a grade, and it is
not yours to change: thoroughness does not raise it, and if you could not in fact obtain that commit
you say so rather than claim it. Because the tier is Full, your review's section 1 must state:

> That code claims were independently verifiable, and what the pin-verification procedure returned when they ran it

**What to put on your machine.** The bundle: a folder of markdown files and images. Point your tool
at the folder you were given access to; if it arrived as an archive, extract it to a real folder
first. And a checkout of the `bookstore` repository at commit
`e439178550371f2bc4f5a9acfee9a1c4f2289ae4` — part 4 says how to confirm you have exactly that commit.

**If you extracted an archive:** a file browser will show you the contents of a `.zip` without
extracting it and a tool that opens files by name will find nothing there, and macOS puts the
documents one level down inside a folder of the same name. Either way, filenames contain spaces, so
quote them.

**If the repository cannot be obtained after all:** review the documents and record in your section 1
that no code claim was independently verified; do not skip the review.

**The rule for the whole session: read the bundle, write exactly one new file, and modify nothing in
the package.** Do not edit, rename, move, delete, reformat or re-save any document or image you were
sent. Changes you want made go into your review as instructions (part 10 says where).

## 2. What each package in the bundle is for

The bundle holds one package, **BOOK-2-01**, and all of it is for review. It covers the requirements
of section 3 of your document, *Verified ratings*, and the note on its Figure 1 sketch. No other
package is copied in, so nothing in this bundle is marked *not for re-review*.

## 3. Documents to review

Every document is named here by its filename. **Search the bundle folder for the filename**; do not
rely on any folder path, which may differ on your machine.

- `BOOK-2-01-manifest.md` — start here: what every file in the bundle is, and a map from the file
  names your own document uses (such as `source/assets/book-page-wireframe.svg`) to the names those
  files carry in this bundle.
- `BOOK-2-01-quillmark-trusted-ratings-brd.md` — your business requirements document, exactly as you
  sent it.
- `BOOK-2-01-book-page-wireframe.svg` — your Figure 1 sketch of the book page.
- `BOOK-2-01-brd-figures.md` — our transcription of that sketch: its text, and what its note points
  at.
- `BOOK-2-01-brd-inventory.md` — the requirements we read out of your document for this package, one
  row per obligation, each with where in your document it is stated.
- `BOOK-2-01-brd-defect-log.md` — the problems we found in the wording of your requirements. It is
  the log for the whole of BOOK-2, so it also lists entries on requirements outside this package.
- `BOOK-2-01-coverage-ledger.md` — where each requirement in this package is handled.
- `BOOK-2-01-code-grounding.md` — what the store's code does today, requirement by requirement, with
  file-and-line evidence in `bookstore` at the pinned commit.
- `BOOK-2-01-design-grounding.md` — a short note that no comparison against design frames was run,
  and why.
- `BOOK-2-01-baselines.md` — the commit the code was read at, and how that was verified.
- `BOOK-2-01-decisions.md` — the decisions we have taken ourselves, each with the options we weighed
  and our reasoning.
- `BOOK-2-01-customer-questions.md` — the questions only you can answer; part 7 repeats them.

## 4. Code baselines and the verification procedure

| Repository | Pinned commit | How the pin was verified |
|---|---|---|
| `bookstore` | `e439178550371f2bc4f5a9acfee9a1c4f2289ae4` | On 2026-10-12, the three commands below were run against the checkout the code was read from: the first printed this commit, and the other two printed nothing — a verified, unmodified commit (`BOOK-2-01-baselines.md`). |

Run these three commands in a terminal against your own checkout. They are written for a checkout in
a folder named `bookstore` in the current directory; if yours is elsewhere, replace `bookstore` with
its path.

```
git -C "bookstore" rev-parse HEAD                      # must print e439178550371f2bc4f5a9acfee9a1c4f2289ae4
git -C "bookstore" diff --ignore-cr-at-eol --stat      # must be empty
git -C "bookstore" status --porcelain                  # any entry needs a line-count comparison
```

- **The first** confirms your checkout is at the commit this package cites. Any other output means
  you are reading a different snapshot of the code: every code claim you then confirm or challenge is
  about code we did not read. Check out that commit, or record the mismatch in section 1.
- **The second** confirms that no tracked file in your checkout has been changed, ignoring
  line-ending differences. Any output means the lines you read may not be the lines the package
  cites.
- **The third** lists every changed or untracked file. For each entry it prints, compare that file's
  line count in your checkout with its line count at the pinned commit; a file whose count differs
  can move every line number the package cites in it.

Report what each of the three returned in your section 1.

## 5. The single most important claim to verify first

**[CG#4]**, our finding on requirement [BR#3], *"A rating, once given, is final."* Two of our
decisions rest on it — [VD#1] and [VD#4] — more than on any other finding, so if it is wrong, both
are reopened.

Its verdict is **REWRITTEN**: the store does not work that way today. Its evidence says any stored
rating can be overwritten and deleted (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java`,
lines 84–111, including a delete-all at `DELETE /api/v1/ratings/delete-all`), the ratings list offers
Update, Delete and Delete All (`web/src/app/ratings/rating-list/rating-list.component.html`, lines 22,
23 and 33), and every cycle of the load generator deletes all ratings before creating new ones
(`ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java`, lines 335–337).

Search `BOOK-2-01-code-grounding.md` for `[CG#4]` to read the full finding, then read those lines at
the pinned commit. Record what you found in your section 5.

## 6. Review scope

**This package is answerable for eleven requirements**, every row of `BOOK-2-01-coverage-ledger.md`,
and every one of them is handled here — none is deferred, rejected, superseded or held by another
package:

- **3.1** — only a customer who has bought a book can rate it: [BR#1]
- **3.2** — one rating per customer per book, and that rating's finality: [BR#2], [BR#3]
- **3.3** — a rating is a whole number of stars from 1 to 5: [BR#4]
- **3.4** — the book page shows the average rating and the number of ratings behind it: [BR#5], [BR#6]
- **3.5** — "Not yet rated" for a book with no ratings: [BR#7]
- **3.6** — the list of books shows each book's average rating: [BR#8]
- **3.7** — a customer can change their rating at any time: [BR#9]
- **3.8** — synthetic monitoring buys a book and rates it, at least once a minute: [BR#10]
- **Figure 1's note** — clicking the rating count opens the book's ratings: [BR#11]

`BOOK-2-01-brd-inventory.md` gives each requirement's exact wording and where your document states
it. Anything else in your document, section 4 included, is not in this package. No requirement in
this package is held by another package.

**Code defects we have undertaken to repair inside this scope:** none — this package records no code
defect.

## 7. The decisions you must make

Answer every item in this part in your review's section 7, one row each: your answer, and your
reason. Cite each question by the round and position in its heading, and each finding of ours by its
bracketed identifier. If you cannot answer an item yet, give it a row anyway, saying so and saying
what you would need in order to answer it.

### 7.1 Questions held for you

These are the questions from our interview that only you can settle. Each one states what the store
does today, as read from its code at the pinned commit, and names the findings that read rests on.
Where an option says *the reading we propose*, it is the reading we would take if you agree, not a
decision already made.

#### Round 1, question 2

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

> Our own review raised a point bearing on this question that only you can settle: [SR#11], in 7.2 below. Please read it before you answer.

#### Round 1, question 3

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

> Our own review raised a point bearing on this question that only you can settle: [SR#2], in 7.2 below. Please read it before you answer.

#### Round 1, question 5

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

> Our own review raised a point bearing on this question that only you can settle: [SR#5], in 7.2 below. Please read it before you answer.

#### Round 1, question 6

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

#### Round 1, question 8

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

#### Round 1, question 9

Today any rating can be deleted. Neither requirement 3.2
(`source/quillmark-trusted-ratings-brd.md › § 3.2`) nor 3.7
(`source/quillmark-trusted-ratings-brd.md › § 3.7`) says whether a customer may remove a rating they
gave. Should a customer be able to remove their own rating of a book?

Options:
- No: once given, a rating can be changed (where 3.7 allows it) but not removed
- Yes: a customer can remove their own rating, and the book's average and count update accordingly

- **Findings:** [CG#4] REWRITTEN
- **Altitude:** product

#### Round 1, question 12

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

#### Round 1, question 15

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

#### Round 1, question 19

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

#### Round 1, question 21

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

> Our own review raised a point bearing on this question that only you can settle: [SR#2], in 7.2 below. Please read it before you answer.

#### Round 1, question 25

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

#### Round 1, question 26

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

> Our own review raised a point bearing on this question that only you can settle: [SR#8], in 7.2 below. Please read it before you answer.

#### Round 1, question 27

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

> Our own review raised a point bearing on this question that only you can settle: [SR#1], in 7.2 below. Please read it before you answer.

#### Round 1, question 29

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

#### Round 1, question 30

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

> Our own review raised a point bearing on this question that only you can settle: [SR#10], in 7.2 below. Please read it before you answer.

### 7.2 Points from our own review that only you can settle

Before sending this package, we attacked it ourselves. The points below are what that review found
that only you can settle. Each is in the review's own words.

#### [SR#1]

**What it is about:** The questions held for the customer: round 1, question 27 (which customer and which books the monitoring uses)

**The attack:**

This question asks Quillmark to choose between two designs for the monitor's test data. That choice is a technical trade-off the delivery team owns and should decide and argue itself. What separates the two options is how the store behaves, and Quillmark has no way to weigh that.

- **The first option cannot run as written.** The store keeps one order per customer per book: the orders table carries a uniqueness rule on customer e-mail and ISBN (orders/src/main/java/com/dynatrace/orders/model/Order.java:11, cited in [CG#3]). Submitting an order that is already completed returns it without buying again (orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:151-153). So one customer cannot buy the same book "with a paid order ... on each run". The monitor would have to cancel and re-buy every minute, moving stock each time. The question does not say so.
- **The second option has an unmentioned cost.** It creates a new customer account every minute, more than half a million a year.
- **Which option is possible depends on two other open questions.** One is whether requirement 3.7 governs (round 1, question 8). The other is whether Quillmark's own tool runs the monitor (round 1, question 25). If Quillmark runs it, Quillmark configures it. If the supplier builds it, choosing its test data is the supplier's job.

Only two parts of this are Quillmark's to decide. First, whether the monitor's purchases may be real paid orders, with money moving through the payment service. Second, whether test customers or a test book may appear anywhere shoppers can see them. Neither is offered as a choice: both options assume "a paid order".

Put as it stands, Quillmark's answer would carry its sign-off on a design whose consequences it was never shown.

**What would settle it:**

Split the question. The supplier decides and argues the monitor's test-data design, once Quillmark has said it wants the supplier to build the monitor (round 1, question 25). Quillmark is asked only what it alone can settle:
- whether the monitor's purchases may be real paid orders;
- whether its test customers and books may be visible to shoppers.

Alternatively, the package shows that both options work with the store as it stands, and that the choice between them turns on something only Quillmark knows.

#### [SR#2]

**What it is about:** The decision register and the questions held for the customer: the unstated premise that the web application in the repository is the storefront Quillmark's shoppers use

**The attack:**

Throughout the package, the web application in the repository is treated as Quillmark's shopper-facing store:
- [VD#1] removes "the storefront's Delete All" from it.
- [VD#2] limits its rating input so that shoppers do not meet errors.
- Round 1, question 3 says that "today any registered customer can rate any book" through it.
- Round 1, question 21 calls its book list "the catalogue list".

No finding establishes this, and the decision register records no assumptions at all. The application itself points the other way:
- Its menu offers Clients, Books, Carts, Storage, Orders, Ratings, Versions, Configs and Ingest. The last starts and stops the load generator (web/src/app/app.component.html:3-11).
- Every list carries a Delete All button, the book list included (web/src/app/books/book-list/book-list.component.html:35).
- The rating form accepts any e-mail address typed into it ([CG#2]).
- The code grounding deliberately left open whether any sign-in exists ([CG#10]).

Quillmark's document says its online store "runs on the BookStore platform". That does not say its shoppers use this application rather than a front end of Quillmark's own.

Both possibilities leave a gap:
- If shoppers do not use it, the user-interface halves of [VD#1] and [VD#2] change a back-office screen. The "book page" and "list of books" the grounding examined may also not be the pages Quillmark means.
- If shoppers do use it, they can today delete every book and start the load generator. A buyers-only rule checked against a typed e-mail address could be beaten by typing a buyer's address.

The package says neither. Quillmark could settle this in one sentence, and is never asked.

**What would settle it:**

Either Quillmark confirms which front end its shoppers use, or the package records, as an assumption put to Quillmark, that it is this web application. In both cases, every decision and question that depends on the answer says that it does.

#### [SR#5]

**What it is about:** The questions held for the customer: round 1, question 5, the sentence saying that repeat ratings are on file

**The attack:**

[CG#3] establishes that nothing in the code stops a repeat rating. It also establishes that the load generator posts ratings for random customer-and-book pairs, and that the store accepts the repeats this produces. It establishes nothing about Quillmark's live ratings data.

Quillmark's introduction to section 3 says a customer can rate the same book again and again, which matches what the code shows. It does not say the store holds shoppers' repeat ratings.

Where the load generator runs, every repeat on file is a random repeat of its own, deleted and replaced every cycle ([CG#4]). The package's proposed reading is to keep only the latest rating each customer gave each book and remove the rest. In that case it would be a clean-up of generated data, presented as a decision about shoppers' ratings. The question puts that choice to Quillmark as a fact about its customers, and the package does not hold that fact.

**What would settle it:**

Either evidence from Quillmark's live data that shoppers' repeat ratings exist, or the sentence is restated as what the code permits, and the question asks what should happen to any repeat ratings Quillmark's store holds.

#### [SR#8]

**What it is about:** [VD#1]

**The attack:**

The decision changes one thing the load generator does, deleting ratings, and says nothing about the rest. Take the decision's own premise, that the generator runs against the live store:
- **Its ratings would pile up.** Each cycle writes about 700 ratings (k8s/ingest.yaml:132, per [CG#3]). They are for random customer-and-book pairs unrelated to any purchase, and never a 5 ([CG#2], [CG#5], [CG#11]). Cycles run back to back ([CG#11]). Today each cycle's ratings are wiped. Under this decision they stay, and every average and count shoppers will see is made of them.
- **It would keep erasing the records the buyers-only rule needs.** It still deletes every order at the start of every cycle, and on cycles that regenerate them, every customer and book (ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:256-274, called at line 309).
- **Its rating step would start failing.** Once requirements 3.1 and 3.2 are built, the store will refuse the generator's random ratings, as coming from non-buyers or as repeats. The decision does not say what the generator does then.

In round 1, question 26, the package treats the monitor's single rating a minute as needing Quillmark's decision on whether it should count. Here it lets hundreds of generated ratings per cycle count, without saying so.

**What would settle it:**

The decision states what the load generator does with ratings and orders after the change, and how its ratings are kept out of what shoppers see. Alternatively, it shows that none of these consequences reaches Quillmark's live store.

#### [SR#10]

**What it is about:** The questions held for the customer: round 1, question 30 (what the list of a book's ratings shows for each rating)

**The attack:**

The question tells Quillmark that written reviews are out of scope and proposes showing "the stars and the date only". It leaves out two facts about the store as it stands:
- **Ratings already carry comments.** Every rating has an optional free-text comment, taken by the create and edit forms and shown in the ratings list and on the rating details page (ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:25-26; web/src/app/ratings/rating-list/rating-list.component.html:8 and 19). In practice that is a short written review. Quillmark's section 5 may have been written in the belief that none exist.
- **The original date is not kept.** Every change to a rating today overwrites the time it was first given ([CG#10]). "The date" could mean when the rating was given or when it was last changed, and the store does not keep the first.

If Quillmark accepts the proposed reading, it will have decided to hide comments its customers have already written, without being told they exist. It will also have chosen a date the store may not be able to show.

**What would settle it:**

The question:
- tells Quillmark that ratings already carry comments, and asks what happens to them (shown, hidden or removed);
- says which date it means;
- says that today the original date is lost whenever a rating changes.

#### [SR#11]

**What it is about:** The questions held for the customer: round 1, question 2 (which orders entitle a customer to rate a book)

**The attack:**

The third option, "an order for that book that has been delivered", names a state the store does not record. An order holds a customer, a book, a quantity, a price, a completed flag and two timestamps, and nothing about delivery (orders/src/main/java/com/dynatrace/orders/model/Order.java:18-39). No finding examined delivery at all.

The question does not tell Quillmark this. If Quillmark chooses that option, it commits to recording deliveries in the order flow without knowing it. Its own section 5 puts changes to checkout, payment and order cancellation out of scope, so it may be choosing an option its own scope forbids. The package could then only ask the question again.

**What would settle it:**

The question says which options the store can evaluate from what it records today, and that choosing "delivered" means new work in the order flow. Alternatively, a finding shows that the store does record delivery.

### 7.3 Open assumptions

None: our decision register records no assumption, so there is none here for you to confirm or
correct.

## 8. What could still move

Nothing in this package rests on another package's decisions. It depends on no other package, no
decision in it is conditional on one, and it records no code defect whose scope is still unsettled.
So no approval you give here is contingent on another review. If your section 3 marks nothing as
contingent, that is correct.

## 9. Where to attack us hardest

These are the weakest points in this package as our own review found them, in that review's own
words. We have chosen to show them to you rather than hide them: attack them, and say in your review
where you agree.

**Open assumptions:** none — our decision register records no assumption.

#### [SR#3]

**What it is about:** [VD#1]

**The attack:**

The reasoning says the load generator's per-cycle wipe "silently resets every average a shopper sees". The finding it cites, [CG#4], says something narrower: every generator cycle deletes all ratings, "so while the load generator runs, ratings are not even durable". The decision drops "while the load generator runs" and widens the claim to the store Quillmark's shoppers use.

Nothing in the package establishes that the load generator runs against Quillmark's live store. [CG#11] shows that the repository's own Kubernetes manifest starts it automatically (k8s/ingest.yaml:113). That is the repository's deployment, not a finding about Quillmark's. No average exists today ([CG#6]), so the sentence also describes a harm that does not yet occur.

This sentence is the only justification for half of the chosen option: stopping the load generator from deleting ratings. Quillmark's operations team will know at once whether a load generator runs against its live store.

**What would settle it:**

Quillmark states that its live store runs the load generator. Failing that, the reasoning keeps the condition its finding carries ("while the load generator runs"), and the package records, as an assumption put to Quillmark, that the generator runs against the live store.

**Where we stand:** we agree with this finding; it is recorded for repair outside this package rather than accepted as it stands.

#### [SR#4]

**What it is about:** The questions held for the customer: round 1, question 3, the sentence saying that ratings from customers who never bought the book are on file

**The attack:**

[CG#2] establishes two things about the code:
- it accepts a rating from any existing customer for any existing book;
- the load generator routinely writes ratings for random customer-and-book pairs.

It establishes nothing about what Quillmark's live ratings data holds: grounding reads code at a commit, not a live database. Where the load generator runs, every rating on file is its own output, deleted and recreated every cycle ([CG#4]). None of it was "given by customers".

The question still asks Quillmark what should happen to "the ratings already given by customers who never bought the book". Its first option is to remove them, as though real shoppers' ratings of that kind were known to exist. Quillmark's own requirement 3.1 says its store already limits ratings to buyers. Its reviewer will ask for evidence that such ratings exist, and the package has none.

**What would settle it:**

Either evidence from Quillmark's live data, which only Quillmark holds, that such ratings exist. Or the sentence is restated to say what the code permits and what the load generator writes wherever it runs, and the question asks what should happen to any such ratings Quillmark's store holds.

**Where we stand:** we agree with this finding; it is recorded for repair outside this package rather than accepted as it stands.

#### [SR#6]

**What it is about:** The code-grounding findings, derivation matrix: the row "Whether the rater bought the book", classed as already captured and stored, unchanged

**The attack:**

[CG#2] establishes where the fact of a purchase lives: in an order that is completed and paid, held only by the orders service. It does not establish that the fact is kept, and the code at the pinned commit shows it is not:
- An order can be deleted on its own or all at once (orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:206-219).
- Every load generator cycle begins by deleting every order (ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:256-266, called at line 309).
- Cancelling clears the same completed flag in place (OrderController.java:173-203 and 302-313). The order record therefore keeps no trace that a cancelled order was ever completed.

The note under the matrix warns that the load generator empties the ratings table. It says nothing about the generator emptying the orders table, which the buyers-only rule depends on. Requirement 3.1, and requirement 4.4 in the sibling package, both need to know who bought a book after the purchase. "Unchanged" asserts that nothing has to be built for that. A reviewer with the repository will contradict the row by reading those lines.

**What would settle it:**

Either a finding that order records are kept on every path that runs in Quillmark's live store, or the row is classed as needing new work, with the retention gap stated beside it.

**Where we stand:** we agree with this finding; it is recorded for repair outside this package rather than accepted as it stands.

#### [SR#7]

**What it is about:** [VD#1]

**The attack:**

The decision says ratings cannot be deleted in bulk. Its reasoning says this "leaves no route by which ratings disappear, except individual removal if the customer allows it (round 1, question 9)". The package contradicts that in three places:
- **Round 1, question 5.** The package's own proposed reading is to keep only the latest rating each customer gave each book "and remove the rest". That is a bulk removal.
- **Round 1, question 3.** It offers "Remove them" for every rating from a non-buyer.
- **Round 1, question 9.** If Quillmark picks its first option ("changed ... but not removed"), this decision leaves no way to remove any rating, for anyone, for any reason.

The reasoning itself names the case that would change the answer: Quillmark needing to purge ratings for legal erasure or fraud clean-up. Only Quillmark knows whether it does, and the package never asks. Every rating stores the customer's e-mail address (ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:16-17), so a request to erase a customer's data reaches this table.

The reasoning also opens on requirement 3.2's finality. Round 1, question 8 offers Quillmark the option of withdrawing that finality, and the reasoning does not say whether the decision still stands if Quillmark does.

**What would settle it:**

- The decision says whether the one-off removals that questions 3 and 5 may call for fall outside "bulk deletion".
- Quillmark is asked whether it must be able to remove ratings, for erasure requests or fraud.
- The reasoning shows that the decision holds under every answer to questions 8 and 9.

**Where we stand:** we agree with this finding; it is recorded for repair outside this package rather than accepted as it stands.

#### [SR#9]

**What it is about:** [VD#4]

**The attack:**

The decision says a change to a rating may alter "only its stars". But every rating also carries a free-text comment:
- it is stored with the rating (ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:25-26);
- the create and edit forms both take it (web/src/app/ratings/create-rating/create-rating.component.html:20-21 and web/src/app/ratings/update-rating/update-rating.component.html:20-21);
- the ratings list and the rating details page show it.

"Only its stars" silently forbids changing the comment, and the decision never names that field. Its second option describes today's edit as changing "the stars, the customer e-mail and the ISBN", but today's edit also changes the comment. No finding records the comment at all.

The decision is also inconsistent about which paths it binds. The chosen option fixes the customer and the book "on every path". The reasoning then says that an operator correcting a rating recorded against the wrong book "would still be a separate correction path, not the shopper's edit". A reader cannot tell whether such a correction is allowed, which is exactly what reasoning for a decision has to make clear. Quillmark's reviewer will raise both points by opening the edit form.

**What would settle it:**

The decision names every field a change may alter, and says what happens to the comment. It states whether "every path" includes an operator correction path, with reasoning that matches.

**Where we stand:** we agree with this finding; it is recorded for repair outside this package rather than accepted as it stands.

## 10. The file you send back

**Write exactly one new file**, named:

`BOOK-2-01 Customer Review <YYYYMMDD>.md`

Replace `<YYYYMMDD>` with **the date you finish your review**, not any date printed in this package.
A review of BOOK-2-01 finished on 20 October 2026, for example, is
`BOOK-2-01 Customer Review 20261020.md`. State that same date in your section 1.

Copy this line, exactly as written, into your section 1 as a line of its own. It identifies the
package you reviewed and is not your review's date:

```
Package reviewed: BOOK-2-01 20261012
```

That file is the only file to send back.

**Again: read the bundle, write exactly one new file, and modify nothing in the package.** Not the
documents, not this prompt, not the images. Every change you want made to a document is written as
an instruction in your section 12, never applied.

Your review must satisfy the rules below.

### 10.1 Rule — evidence limitations come first

**The review's section 1 states what the reviewer actually had, before the review makes any claim
that rests on it.** Not the last section, not an appendix, not a footnote under the verdict.

The prompt tells the reviewer which degradation tier they are in and what their section 1 must
therefore state. A review written against unpinned repositories is a review whose code claims are
true of *an unidentified snapshot*, and a review written against no repositories at all has verified
no code claim independently. Both are perfectly good reviews — provided the reader meets that
sentence before the findings, not after them.

**A reader who meets the limitations after the findings has already believed them.** By the time the
caveat arrives, the verdict has been read, the traceability table has been skimmed, and the
confirmations have been taken as confirmations. The caveat then reads as modesty rather than as the
scope statement it is, and nobody goes back to re-weigh what they already accepted. Ordering is the
whole of the rule: a limitations section at the end satisfies a checklist and fails the reader.

Section 1 also records what the reviewer could not open at all — a document that would not render,
an image that did not arrive, a repository they could not obtain — and, when the prompt supplied a
pin-verification procedure, whether it was run and what it returned.

### 10.2 Rule — exactly one new file, and nothing in the package is modified

**The reviewer writes one new file and modifies nothing else.** Not the documents, not the prompt,
not the images. Nothing is renamed, moved, deleted, reformatted or re-saved. No "corrected copy" is
attached alongside the review.

This rule is stated twice in the rendered text — once at the top and once again where the output is
described — because an agent asked to review documents will otherwise helpfully edit them.

Two reasons, and both survive the file being read in isolation:

- **The two copies stay byte-identical.** The delivery team's copy of the bundle and the customer's
  are the same bytes, so every claim the review makes is checkable against a known document — months
  later, by somebody who was not in the room.
- **Nobody can otherwise tell what was sent from what was changed.** A returned package with edits
  in it has two authors and no way to separate them; the sentence under discussion may be the one
  that was delivered or the one the reviewer rewrote, and there is no third copy to adjudicate.

**Required changes to package documents are requested, never applied** — they go in the review's
section 12 as instructions to edit. That section of the review exists because this rule closes every
other channel.

The output file is named `<BRD-KEY> Customer Review <YYYYMMDD>.md`, where `<YYYYMMDD>` is **the date
you finish this review** — not any date already printed in the package. If you finish on 22 April
2026 and the key is `EPIC-008`, the file is `EPIC-008 Customer Review 20260422.md`. State that same
date in your section 1. It is the only file to send back.

### 10.3 The twelve sections

Numbered 1 to 12, in this order, in the returned review. (This file's own section numbers are not
the review's; the table below is the review's spine.)

| No. | Section | What it must carry |
|---|---|---|
| 1 | Review identity and evidence limitations | The `Package reviewed:` line the prompt gives, copied exactly as its own line; who reviewed, in what role, on what date; the documents and repositories actually available; the tier statement the evidence-limitations rule obliges; anything that could not be opened |
| 2 | Executive verdict | One of `approved`, `approved-with-required-changes`, `not-approved`, and one paragraph saying why |
| 3 | Approved and deferred scope | What is approved to proceed now, what is deferred, and what a deferral is waiting for |
| 4 | Requirement traceability | Per requirement in scope, exactly one of four values and no other: `confirmed` — the package's reading matches the customer's intent, including a requirement the package says will not be built where the customer agrees it should not be; `corrected` — it does not, and the row says what the customer meant; `missing` — a requirement the package left out entirely, stated as its own row; `withdrawn` — the customer drops the requirement, saying why in this row; where the package put a question about that requirement, the answer to that question goes in section 7 as every answer does, and a withdrawal the package asked about nowhere is stated here alone. Where the package includes its transcription of the customer's images, that transcription — for each image bearing on a requirement in scope — is part of its reading, and the reviewer confirms or corrects it as they do a requirement. A correction the transcription itself must carry is also entered in section 12, as a required change naming the document that holds the transcriptions by its filename and the image by its heading there |
| 5 | Code-grounding confirmations and challenges | Both: which code claims the reviewer confirms and how, and which they challenge and on what evidence |
| 6 | Design review | The design positions the package takes, confirmed or challenged, including anything that contradicts the customer's own environment |
| 7 | The decision log | One row per decision the package asked the customer to take: the answer, and the reason for it |
| 8 | Accepted assumptions and rejected alternatives | One row per assumption the package surfaced, and every alternative the customer rules out, with the reason |
| 9 | Ownership | Who on the customer side owns each approved area and each decision, and who to ask when a later question arises |
| 10 | Unresolved blockers | What prevents this proceeding, what would clear it, and who clears it |
| 11 | Readiness statement | One explicit sentence: what the delivery team is clear to proceed with, and what it is not |
| 12 | Required changes to downstream documents | Per change: which document, which section or identifier, and what must change — instructions, not edits |

**Every section is present, even when it is empty.** An omitted section is indistinguishable from an
overlooked one. A section with nothing in it says `none` and says why — *no design position was
challenged*, *no repository was available so no code claim was confirmed* — because those two
sentences mean very different things and an absent section means neither.

**The review cites identifiers; it does not mint them.** Every row that answers something names the
package's own identifier for it, copied exactly as it appears — every identifier the package uses is
bracketed, a short prefix, a `#` and the record's number, shown here as `[BR#n]` or `[CG#n]` with
`n` standing for that number, whatever kind of record it names, and a question is cited by the
round and position that identify it. The review never assigns identifiers of its own in
those namespaces: the registers on the delivery-team side own that numbering, and an identifier
whose origin is ambiguous is worse than no identifier. Plain list numbers inside a review section
are local to the review and are not identifiers.

**Every claim in the review is traceable to something the reviewer had.** A statement that rests on
the reviewer's own knowledge of the customer's environment rather than on a bundle document says so
— that is often the most valuable thing in the review, and it is worthless if it cannot be told
apart from a reading of the package.

### 10.4 Rules the individual sections carry

**Section 2 — the verdict is not a summary.** It is one of the three values, chosen deliberately,
and it has to survive being read next to sections 10 and 12: `approved` alongside an unresolved
blocker is a contradiction, and so is `approved` alongside a required change to a document.

**Section 4 — the section is about intent, not about bookkeeping.** The question is whether the
package understood what the customer asked for. Requirements the package missed entirely are the
highest-value rows here and are stated as their own rows rather than folded into a correction. The
review does not set the delivery team's coverage bookkeeping.

**Section 5 — confirmations without challenges are not a review.** A section holding only
confirmations has either been rubber-stamped or has hit a limit the reviewer has not admitted, and
section 1 is where that limit belongs. Each confirmation says *how* it was checked, or states
plainly that it was accepted on the package's word. Each challenge names what the package claims,
what the reviewer believes instead, and the best evidence available for it — a filename and a line,
a screenshot, or the fact that the reviewer runs the system and it does not behave that way. The
review does not assign the package's own verdict vocabulary to a claim; it confirms or challenges,
and the delivery team re-adjudicates.

**Section 7 — a decision without a reason cannot be applied.** The answer alone leaves the delivery
team unable to tell an instruction from a preference, and unable to defend the position when it is
challenged later. A question the customer declines to answer this round still gets a row, saying so
and saying what they would need in order to answer it: silence is otherwise indistinguishable from
oversight, and it will be re-asked.

**Section 8 — every assumption the package surfaced gets exactly one row**, marked accepted,
corrected, or cannot-say. Cannot-say is a real answer and is not a failure to respond. An assumption
the customer corrects is the cheapest correction in the whole loop, and it is only cheap while it is
still an assumption.

**Rejected alternatives are recorded with their reasons**, in the same section, so the option is not
re-proposed next round by somebody who never learned why it was ruled out.

**Section 9 — a review nobody owns cannot be acted on.** Roles and names on the customer side, per
approved area and per decision, so a question three weeks from now has a destination.

**Section 11 — the readiness statement is not the verdict again.** The verdict judges the package;
the readiness statement authorises work. It says what may start, and it says what may not, and it is
one sentence long so it can be quoted without being softened.

**Section 12 — this is the only channel for a change to a package document.** The one-new-file rule
closes every other one. Each row names the document by filename, the section or identifier inside
it, and what must change. A change described only as "fix the architecture section" cannot be
applied by anyone who was not the reviewer.

### 10.5 Where a returned review may still be provisional

A package may ship while a prerequisite it depends on is not yet customer-reviewed, provided the
prompt says so and names which positions could still move. Where it does, the review's section 3
states which of its approvals are contingent on that prerequisite, and section 10 does **not** list
the prerequisite as a blocker — it is not blocking, it is unsettled, and the two get very different
treatment on the delivery side.

A reviewer who was told nothing could move writes no such rows, and their absence then means what it
says.

## 11. What this session cannot settle

- **Scope still open elsewhere:** none. Every requirement in this package is handled here, and
  nothing in it is deferred, rejected or left to another package.
- **Other packages:** none. This package depends on no other.
- **Code defects we will not repair:** none recorded.
- **The limits of this review.** It is a reading of the bundle and of the `bookstore` repository at
  the pinned commit. It cannot settle anything about your live store — its data, how it is deployed
  and operated, or tooling configured outside the store's code — because none of that is in either.
  Several items in part 7 ask you for exactly those facts. Where an answer of yours rests on your own
  knowledge of your store rather than on a document in this bundle, say so in that row.

# BOOK-2-01 Customer Review 20261012

## 1. Review identity and evidence limitations

```
Package reviewed: BOOK-2-01 20261012
```

- **Reviewed by:** Quillmark Booksellers, jointly by the Head of E-commerce (owner of sections 2 to 5
  of our requirements) and the Operations Lead (owner of the monitoring requirements). No names are
  given; see section 9.
- **How the review was produced:** an AI review agent (Claude Code) ran the review prompt as an
  interview with us, read the bundle and the repository, and wrote this file. **Every answer and
  position in this review is ours**, chosen during the interview. The *reasons* in sections 7 and 8
  were written up by the agent from the options, consequences and evidence it showed us when we
  chose. Where a reason does not match our intent, the answer still stands and the reason should
  be raised with us (section 9).
- **Review finished:** 2026-10-12.
- **Documents available:** every file the manifest lists. All twelve were opened and read in full:
  the prompt, the manifest, our requirements document, the Figure 1 sketch, the figures
  transcription, the inventory, the defect log, the coverage ledger, the code grounding, the
  design-grounding note, the baselines and the decision register, plus the customer questions. The
  sketch `BOOK-2-01-book-page-wireframe.svg` was read as SVG source (its text, shapes and
  coordinates), not as a rendered picture. We also read the covering delivery note, which sits
  outside the bundle.
- **Repositories available:** `bookstore`, checked out locally at the pinned commit.
- **Tier: Full.** Code claims were independently verifiable: we had the `bookstore` repository at
  commit `e439178550371f2bc4f5a9acfee9a1c4f2289ae4`. We ran the pin-verification procedure on
  2026-10-12 against that checkout, and it returned:
  1. `git -C "bookstore" rev-parse HEAD` printed `e439178550371f2bc4f5a9acfee9a1c4f2289ae4`, the
     pinned commit.
  2. `git -C "bookstore" diff --ignore-cr-at-eol --stat` printed nothing.
  3. `git -C "bookstore" status --porcelain` printed nothing, so no file needed a line-count
     comparison.

  Every code claim we confirm or challenge in section 5 is therefore about the same lines the
  package cites.
- **Could not open:** nothing. Everything we were sent opened.
- **What we did not do:** we did not build, run or test the store. We checked code claims by
  reading the cited lines at the pinned commit and re-running a few of the package's searches.
  Section 5 lists which claims we took on the package's word.
- **What rests on our own knowledge, not on the bundle:** four facts about our live store, each
  marked *(our own knowledge)* where it is used:
  - which front end our shoppers use;
  - that the load generator never runs against our live store;
  - our plans for third-party monitoring;
  - that our shops' systems use no other rounding rule.

  Nothing in the bundle or the repository can confirm these.

## 2. Executive verdict

**approved-with-required-changes**

The package read our requirements faithfully. Its code grounding is accurate wherever we checked
it, and the questions it put to us were the right ones. Its self-review found most of its own
weak points before we did. We are not approving it as it stands, for four reasons:
- Our answers withdraw three of the eleven requirements in scope ([BR#3], [BR#10], [BR#11]) and add
  three it did not have (section 4).
- Two of the supplier's own decisions, [VD#1] and [VD#4], must be restated. [VD#1] must cover an
  operator removal path, the go-live clean-up and rating-deletion paths the package did not cite.
  [VD#4] must name the comment.
- The derivation matrix overstates how safely purchase records are kept, a gap that reaches our
  live store because shoppers use the `web/` application.
- The decision register records none of the assumptions this review settles.

These are corrections to documents, listed in section 12. None of them needs another decision from
us, so nothing blocks the work (section 10).

## 3. Approved and deferred scope

**Approved to proceed now**, on the answers in section 7 and the positions in section 6:
- **3.1** (buyers only, [BR#1]): a qualifying order is completed and paid, and not since
  cancelled. Eligibility is checked when a rating is given or changed.
- **3.2** (one rating per customer per book, [BR#2]): repeat ratings already stored are reduced to
  each customer's latest.
- **3.3** (whole stars 1 to 5, [BR#4]), enforced on every write path and in the rating input
  ([VD#2]).
- **3.4** (average and count on the book page, [BR#5] and [BR#6]), rounded half up ([VD#3]).
- **3.5** ("Not yet rated", [BR#7]), shown alone, with no count.
- **3.6** (average on the list of books, [BR#8]): the catalogue list only.
- **3.7** (change a rating at any time, [BR#9]): a change may alter the stars and the comment, never
  the customer or the book ([VD#4] as corrected in section 6).
- **The three additions in section 4:**
  - eligibility is checked when a rating is given or changed;
  - operators can remove ratings for erasure requests or fraud;
  - comments stay stored and hidden from shoppers.
- **The one-off clean-up at go-live:** remove ratings from customers with no qualifying order, and
  all but the latest of each customer's repeat ratings of a book.
- **[VD#1] as amended in section 6.**

**Withdrawn from this release** (not deferred: nothing in this release waits on them):
- **3.2's second sentence, "A rating, once given, is final"** ([BR#3]): 3.7 governs.
- **3.8, synthetic monitoring** ([BR#10]): Quillmark will set up third-party monitoring itself later
  *(our own knowledge)*. We expect 3.8 to return in a later version of our requirements once the
  tool is chosen, and round 1, questions 26 and 27 with it.
- **The Figure 1 note** ([BR#11]): it illustrates an idea and is not a requirement.

**Deferred:** none.

**Contingent approvals:** none. The prompt says nothing in this package rests on another package,
and we mark nothing as contingent.

Requirement 4.6, the monitoring of suggestions and the Top rated shelf, is in the other BOOK-2
package. Our answer on 3.8 does not decide it.

## 4. Requirement traceability

| Requirement | Value | Our intent |
|---|---|---|
| [BR#1] | confirmed | The package's reading is right. Its grounding ([CG#2]) shows the store does not limit ratings to buyers today, so the buyers-only rule is new work, and our sentence "The store works this way today" was mistaken. Which orders qualify: section 7, round 1, question 2. |
| [BR#2] | confirmed | One rating per customer per book. It stands independently of [BR#3] (round 1, question 6). |
| [BR#3] | withdrawn | "A rating, once given, is final" is dropped: 3.7 governs (round 1, question 8). One rating per customer per book ([BR#2]) still holds. A customer changes their rating; they do not add to it or remove it. |
| [BR#4] | confirmed | A requirement, not a description of today's scale (round 1, question 12). |
| [BR#5] | confirmed | Average on the book page, one decimal place, rounded half up ([VD#3]). |
| [BR#6] | confirmed | The count shown is always the count of exactly the ratings in the average (round 1, question 15). |
| [BR#7] | confirmed | "Not yet rated" alone, with no count (round 1, question 19). It applies wherever an average is shown, so on the catalogue list too. |
| [BR#8] | confirmed | The catalogue list only (round 1, question 21). |
| [BR#9] | confirmed | A customer can change their rating at any time, and the new rating replaces the old one. A change may alter the stars and the comment ([VD#4] as corrected in section 6). |
| [BR#10] | withdrawn | 3.8 is withdrawn from this release: Quillmark will set up third-party monitoring later *(our own knowledge)*. Our answer to round 1, question 25 is in section 7. |
| [BR#11] | withdrawn | The Figure 1 note is an illustration, not a requirement (round 1, question 29). No list of a book's ratings is required in this release. |
| missing | missing | **A rating's eligibility is checked when it is given and when it is changed.** If the customer later cancels the qualifying order, the rating stays and keeps counting. A customer with no qualifying order at the time cannot change their rating. The package did not ask this; we stated it in the interview. It does not decide how 4.4 (in the other BOOK-2 package) treats such ratings. |
| missing | missing | **Operators, never shoppers, must be able to remove a single rating, or every rating of one customer, to meet an erasure request or to clean up fraud.** A removed rating leaves every average and count. This answers [SR#7]. It is the only route by which a rating disappears after go-live. |
| missing | missing | **Rating comments are stored but never shown to shoppers.** The rating forms keep taking a comment, and a customer may change their comment when they change their rating. Comments already stored are kept and hidden. This answers [SR#10]. Written reviews stay out of scope (our section 5): a hidden comment is not a written review. |

**The Figure 1 transcription** (`BOOK-2-01-brd-figures.md`, heading
`source/assets/book-page-wireframe.svg`): **confirmed.** We compared it with the SVG source. The
following all match the sketch:
- the title bar text;
- the five placeholder lines;
- the gold star, `4.2` and the blue, underlined `(37 ratings)`;
- the note's wording, and its arrow ending at the right end of the count's underline.

Its text block puts `★ 4.2` and `(37 ratings)` on separate lines where the sketch draws them on one
line. This does not change any reading, so no correction is required. Since [BR#11] is withdrawn,
the sketch now only illustrates [BR#5] and [BR#6]. The format `★ 4.2 (37 ratings)` is an example,
not a required format.

## 5. Code-grounding confirmations and challenges

### Confirmations

| Finding | How we checked it |
|---|---|
| [CG#1] | We re-ran the three pin-check commands ourselves (section 1); the results match. |
| [CG#2] | We read `RatingController.java` lines 72–80 and 120–135: create checks only that the book and client exist, and the summary on line 73 is stale. We also read `create-rating.component.html` lines 6 and 11 (free-text e-mail and ISBN) and `ingest/.../model/Rating.java` lines 36–42 (random client and book). We took the property-file lines and the cross-service grep on the package's word. |
| [CG#3] | We read `RatingRepository.java` line 16, which returns a `List`. `Rating.java` lines 8–9 carry no unique constraint, and `Order.java` line 11 does. `git grep -i unique` over `ratings` returns nothing. We took the carts contrast on the package's word. |
| [CG#4] | We read every cited line at the pinned commit. Update and its overwrite are at `RatingController.java` 84–98 and delete at 101–105. Delete-all at 108–111 calls `truncateTable`, a native `TRUNCATE TABLE ratings` (`RatingRepository.java` 17–20). The list's Update, Delete and Delete All are at `rating-list.component.html` 22, 23 and 33, the route at `app-routing.module.ts` 63, and the per-cycle wipe at `IngestController.java` 335–337. **The single most important claim holds:** a stored rating can be overwritten and deleted, one at a time or all at once. Our challenge below concerns what [VD#1] builds on this finding, not the finding itself. |
| [CG#5] | We read `Rating.java` lines 22–23 (`int`), 36–38 (the no-arg constructor leaves the rating at 0) and 86–92: the setter rejects values ≤ 0 or > 5, and its message wrongly says "between 0 and 5". We read `ingest/.../model/Rating.java` line 42: `random.nextInt(4) + 1` never produces a 5. We took the validation-annotation grep on the package's word. |
| [CG#6], [CG#7], [CG#9] | We read `book-details.component.html` lines 1–26 and `book-list.component.html` lines 1–37: neither shows a rating, an average or a count. We took the `VersionController` count and the average grep on the package's word. |
| [CG#8] | We re-ran `git grep -i 'not yet rated\|unrated\|no ratings'` at the pinned commit: no match. |
| [CG#10] | We read `Rating.java` line 37 and lines 114–115: the creation time is overwritten on every update. In `update-rating.component.html`, lines 6, 11 and 16 let the e-mail, ISBN and stars be edited, and lines 20–21 the comment. This finding is the basis for our [VD#4] correction. |
| [CG#11] | We read `ingest/.../model/Rating.java` 37–38, `ingest/.../model/Order.java` 53–54, `IngestController.java` 287–299 and `k8s/ingest.yaml` 105–140. The verdict is sound. It no longer bears on this release, because [BR#10] is withdrawn. |
| [CG#12] | We read `rating-list.component.ts` lines 33–37, and `git grep getRatingsByISBN` finds only the definition. `app-routing.module.ts` line 61 takes no ISBN. The verdict is sound, and it no longer bears on this release, because [BR#11] is withdrawn. |

### Challenges

1. **[CG#4], and [VD#1], which rests on it: the load generator deletes ratings in three more places
   than the package cites.**
   - **What the package claims:** [CG#4] cites only the per-cycle wipe (`IngestController.java`
     335–337). [VD#1] concludes that stopping it means "the load generator does not delete
     ratings".
   - **What we found at the pinned commit:**
     - Starting the generator first clears every service's data, ratings included
       (`IngestController.java` line 55, calling `clearData(true, true)`, which deletes ratings at
       lines 257–259).
     - `DELETE /api/v1/ingest`, when the generator is idle, does the same (lines 193–202).
     - The ingest pod's startup script deletes all ratings directly with
       `curl -X DELETE .../api/v1/ratings/delete-all` (`k8s/ingest.yaml` line 124).
     - Every generator delete-all goes through `IngestRepository.deleteAll`, which swallows every
       exception (`IngestRepository.java` lines 15–19). Once [VD#1] removes the ratings delete-all
       endpoint, these calls will fail **silently** rather than visibly.
   - **Why it matters less than it might:** the load generator never runs against our live store
     *(our own knowledge)*, so none of these paths reaches our shoppers' ratings. It still means the
     decision as written is not delivered by the change it describes.
2. **The derivation matrix row "Whether the rater bought the book", classed EXISTS: the purchase
   record is not kept, and in our case the gap reaches the live store.**
   - **What the package claims:** the row classes the purchase record as already captured and
     stored, so nothing needs building for it.
   - **What we found:** we agree with [SR#6], and our answer to [SR#2] makes it worse, because
     `web/` is our storefront *(our own knowledge)*:
     - its orders list offers Delete (`order-list.component.html` line 32) and Delete All
       (line 42) to anyone who uses it;
     - those buttons call `OrderController.java` lines 206–211 and 214–219, which delete one order
       or truncate the table;
     - cancelling (line 26 of the same template; `OrderController.java` 173–203) clears the
       completed flag in place (lines 311–313).

     A shopper who deletes an order, or all of them, erases the only record that a purchase
     happened, and with it the right to rate or to change a rating under 3.1.
3. **[SR#3] cites the wrong line for where the manifest starts the generator.**
   - **What the package claims:** [SR#3] cites `k8s/ingest.yaml:113`.
   - **What we found:** line 113 is `sleep_period=360`. The POST that starts the generator is on
     line 132. [CG#11] itself cites both lines correctly. This is minor and needs no change to a
     package document.

## 6. Design review

| Position | Our position |
|---|---|
| [VD#1] | **Challenged; amended as follows.** (a) No route that shoppers can reach deletes all ratings: drop the ratings delete-all from the storefront API and from the Ratings list's Delete All (`rating-list.component.html` line 33). (b) Shoppers cannot remove any rating, including their own (round 1, question 9), so the storefront must not offer the per-rating Delete either (line 23). (c) Operators get a removal path for one rating, or for all of one customer's ratings, for erasure requests and fraud ([SR#7], section 4). (d) The one-off go-live clean-up is not "bulk deletion" in the sense this decision forbids. It removes ratings from customers with no qualifying order (round 1, question 3) and all but the latest repeat rating of each customer and book (round 1, question 5). (e) The load generator never runs against our live store *(our own knowledge)*. So the reasoning must keep the condition its finding carries, "while the load generator runs" ([SR#3]), and must not claim the generator resets averages shoppers see. How the generator behaves in test environments is the supplier's call. If the supplier keeps the generator change, it must cover all four deletion paths in section 5, challenge 1, and say what the generator's rating step does once 3.1 and 3.2 refuse its random ratings ([SR#8]). (f) The reasoning's phrase "except individual removal if the customer allows it (round 1, question 9)" no longer holds: customers may not remove ratings, and operators may. **A design point we leave to the supplier:** our storefront has no sign-in (we accept the typed e-mail, section 8), so the supplier must say how the operator path is kept out of shoppers' reach. |
| [VD#2] | **Confirmed.** It matches our answer that 3.3 is a requirement (round 1, question 12). It matters more than the package knew, because the rating input is in our shoppers' storefront. |
| [VD#3] | **Confirmed.** Our shops' systems use no other rounding rule for ratings *(our own knowledge)*, so the condition that would change this decision does not arise. |
| [VD#4] | **Challenged; corrected as follows.** A change to a rating may alter its **stars and its comment**. The customer e-mail and the ISBN never change, **on any path, operator paths included**. A rating recorded against the wrong book is removed through the operator removal path, never reassigned to another book. This settles both points of [SR#9]. It also replaces the reasoning's "separate correction path", which we do not want. |
| [SR#2]'s premise: `web/` is the store shoppers use | **Confirmed** *(our own knowledge)*. So the "book page" is the book-details page and "the list of books" is the Books list, and the UI parts of [VD#1] and [VD#2] change the screens our shoppers use. |
| The storefront's Ratings list | **Not a position the package takes; raised for the delivery team.** The `web/` Ratings list shows every rating's comment (`rating-list.component.html` line 19), which our answer on comments forbids. It offers Delete (line 23), which our answer on round 1, question 9 forbids. It also shows every rater's e-mail address (line 16) and offers Update on every customer's rating (line 22). We have not decided here what of that screen remains for shoppers. Please propose it in the PRD and put it to the Head of E-commerce. |
| Design grounding (`BOOK-2-01-design-grounding.md`) | **Confirmed.** No design frames were sent, so there was nothing to reconcile, and the sketch is now illustration only. |

## 7. The decision log

The answers are ours. The reasons were written up during the interview from the consequences shown
to us (section 1).

### 7.1 Questions held for us

| Question | Our answer | Reason |
|---|---|---|
| Round 1, question 2 | An order for that book that is completed and paid, and not since cancelled (the reading proposed). | A customer has bought a book when they have paid for it and kept it. An unpaid order is not a purchase. "Delivered" is a state the store does not record ([SR#11]), and adding it would change the order flow, which our section 5 excludes. Checked when the rating is given or changed (section 4). |
| Round 1, question 3 | **Remove them** at go-live: every rating from a customer with no qualifying order. Asked as [SR#4] reframes it, about any such ratings our live store holds. | Only buyers' ratings are trustworthy, which is the point of section 3. A rating kept but excluded would hold a customer's e-mail address for no purpose. The code has allowed any customer to rate any book, so such ratings may well exist in our live data. We hold no count of them, and the answer applies to whatever exists. |
| Round 1, question 5 | **Keep only the latest rating each customer gave each book, and remove the rest** (the reading proposed), at go-live. Asked as [SR#5] reframes it, about any repeat ratings our live store holds. | It matches 3.7: a customer's newest rating replaces their earlier one. Keeping the first would contradict that, and keeping all would leave the repeat ratings that section 3 blames for shoppers' distrust. |
| Round 1, question 6 | Yes: two independent requirements, and one rating per customer per book holds either way (the reading proposed). | Repeat ratings are what we named as the problem, not changed ones. One rating per customer per book must hold even though ratings can be changed. |
| Round 1, question 8 | **3.7 governs:** a customer can change their rating at any time, and "final" in 3.2 is withdrawn. | Shoppers change their minds. A changed rating still counts once, so allowing changes does not bring back the repeat-rating problem. A time limit would add a rule shoppers cannot see. |
| Round 1, question 9 | **No:** a customer can change their rating but not remove it. | A change already covers a customer who has changed their mind. Letting customers remove ratings would make counts swing for no reason shoppers can see. Removal exists, but only for operators handling erasure requests and fraud (section 4). |
| Round 1, question 12 | A requirement: every rating is a whole number of stars from 1 to 5 (the reading proposed). | It is the scale shoppers see on the page. A 0 or a fraction would distort every average in 3.4 and 3.6. |
| Round 1, question 15 | Yes: two requirements, and the number is always the count of the ratings in the average (the reading proposed). | The count is there to show how many people the average rests on (section 3's introduction). Any other number would mislead. |
| Round 1, question 19 | "Not yet rated" alone, with no count (the reading proposed). | "(0 ratings)" says the same thing a second time and draws the eye to an empty state. |
| Round 1, question 21 | **The catalogue list only.** | 3.6 is about the list of books we have today. The suggestions and the Top rated shelf belong to section 4 and the other BOOK-2 package, and 3.6 should not widen them. |
| Round 1, question 25 | **Neither, in this release:** 3.8 is withdrawn. Quillmark will set up third-party monitoring itself later *(our own knowledge)*. | The choice of monitoring tool is ours, and it is not made yet. We will bring 3.8 back, with the tool and who operates it, in a later version of our requirements. |
| Round 1, question 26 | **Not answered: moot**, because 3.8 is withdrawn. | When 3.8 returns, we will need to know how our chosen tool's purchases and ratings can be told apart from shoppers'. Please re-ask then. |
| Round 1, question 27 | **Not answered: moot**, because 3.8 is withdrawn. | When 3.8 returns, please split the question as [SR#1] proposes. Ask us only whether the monitor's purchases may be real paid orders, and whether its test customers and books may be visible to shoppers. The test-data design is the supplier's to argue. |
| Round 1, question 29 | **No:** the note illustrates an idea, not a requirement of this release. | We drew it to show what the count is for. We do not need a list of a book's ratings now. |
| Round 1, question 30 | **Not answered: moot**, because no list of a book's ratings is required (question 29). | If the list returns, ask us again as [SR#10] proposes: which date, and what of the comments, which stay hidden from shoppers (section 4). |

### 7.2 Points from the package's own review

| Point | Our answer | Reason |
|---|---|---|
| [SR#1] | Moot for this release (3.8 is withdrawn). We agree the question must be split when it returns. | See round 1, question 27. |
| [SR#2] | **Settled:** our shoppers use the `web/` application in the repository *(our own knowledge)*. For this release we accept that the rater is the customer whose e-mail is entered (section 8). | The package was right to depend on this, and wrong not to say so. Every decision that depends on it now does (section 6). |
| [SR#5] | Settled by our answer to round 1, question 5, asked in the reframed form. | The load generator never runs in our live store *(our own knowledge)*, so any repeat ratings there are real shoppers'. We hold no count of them. |
| [SR#8] | Settled: none of these consequences reaches our live store, because the load generator never runs there *(our own knowledge)*. | The generator's behaviour in test environments is the supplier's call. The decision must say so rather than claim to protect shoppers' averages ([VD#1] in section 6). |
| [SR#10] | **Settled:** comments are kept but hidden from shoppers, the forms keep taking them, and a change may alter them (section 4 and [VD#4]). The date question is moot with question 30. | We had not known ratings carry comments. Hiding them keeps written reviews out of scope without throwing away what customers wrote. |
| [SR#11] | Settled by our answer to round 1, question 2. | We chose knowing that "delivered" is not recorded and would mean order-flow work our section 5 excludes. |

## 8. Accepted assumptions and rejected alternatives

### Assumptions

The decision register records none (part 7.3 of the prompt). The package's self-review asked for
two to be recorded and put to us ([SR#2], [SR#3]). The interview and [SR#4]/[SR#5] surfaced two
more. Each gets one row:

| Assumption | Marked | Note |
|---|---|---|
| The `web/` application is the storefront Quillmark's shoppers use ([SR#2]) | accepted | *(Our own knowledge.)* |
| The load generator runs against Quillmark's live store ([SR#3], [VD#1]'s reasoning) | corrected | It never runs against our live store, only in test and demonstration environments *(our own knowledge)*. |
| The store may identify the rater by the e-mail address entered, with no sign-in, for this release | accepted | We accept that someone could type a buyer's e-mail. Sign-in is not part of this release. |
| Our live data holds ratings from non-buyers, and repeat ratings ([SR#4], [SR#5]) | cannot-say | We hold no count. Our answers to round 1, questions 3 and 5 apply to whatever exists. |

### Rejected alternatives

| Item | Rejected | Why |
|---|---|---|
| Round 1, question 2 | Any order placed, paid or not | An unpaid order is not a purchase. |
| Round 1, question 2 | An order that has been delivered | Not recorded by the store; it would change the order flow, which our section 5 excludes. |
| Round 1, question 3 | Keep, excluded from averages and counts | Keeps customers' e-mail addresses for no purpose. |
| Round 1, question 3 | Keep and count, as today | Defeats 3.1. |
| Round 1, question 5 | Keep the first rating | Contradicts 3.7, under which the newest rating replaces the old. |
| Round 1, question 5 | Keep all; apply the rule to new ratings only | Leaves the repeat ratings section 3 blames for distrust. |
| Round 1, question 6 | 3.2 is one rule | One rating per customer per book must survive the withdrawal of "final". |
| Round 1, question 8 | 3.2 governs | Shoppers must be able to change their minds. |
| Round 1, question 8 | Changeable for a limited period | A rule shoppers cannot see. |
| Round 1, question 9 | Customers can remove their own rating | A change covers the change of mind. Removal is for operators only. |
| Round 1, question 12 | A description of today's scale | The scale must hold. |
| Round 1, question 15 | 3.4 as one requirement | The count must always match the average. |
| Round 1, question 19 | "Not yet rated" with "(0 ratings)" | Redundant. |
| Round 1, question 21 | Every list of books | Section 4's lists are not 3.6's concern. |
| Round 1, question 25 | Build; Support | Neither in this release: 3.8 is withdrawn. |
| Round 1, question 29 | The note is a requirement | We do not need the list now. |
| [VD#1] | Accept as written | Ignores the erasure and fraud need and the go-live clean-up, and rests on the generator running in our live store, which it does not. |
| [VD#1] | Keep bulk delete as an operator-only maintenance action | Operators need targeted removal, not a table-wide delete. |
| [VD#1] | Leave bulk deletion as it is today | Anyone in the storefront can wipe every rating. |
| [VD#2] | Enforce on the server only; or leave as today | Shoppers should not meet errors the form could prevent, and today a missing value stores 0. |
| [VD#3] | Round half to even; truncate | Neither is the rule shoppers expect. |
| [VD#4] | Keep today's edit | It lets a rating move to another customer or book. |
| [VD#4] | Stars only, comment fixed | We keep taking comments, so customers may change them. |
| [VD#4] | Operators may move a rating to the correct book | Operators remove a wrong rating; they never reassign it. |
| Comments ([SR#10]) | Show them as today | Written reviews are out of scope. |
| Comments ([SR#10]) | Remove the field and delete existing comments | We keep what customers wrote. |
| Comments | Stop taking new comments | We keep taking them, stored hidden. |
| A rating after its order is cancelled | Removed; or kept out of the counts | Eligibility is checked when a rating is given or changed, not afterwards. |
| Rater identity | Signed-in shoppers only | Not part of this release. |
| Rater identity | Out of scope, handled by Quillmark | We have no sign-in layer of our own to rely on. |
| Operator removal ([SR#7]) | No removal ever after go-live | Erasure requests reach the ratings table, and fraud must be cleanable. |

## 9. Ownership

| Area or decision | Owner at Quillmark |
|---|---|
| The rating rules and their display: [BR#1], [BR#2], [BR#4] to [BR#9]; round 1, questions 2, 3, 5, 6, 8, 9, 12, 15, 19, 21 and 29; [VD#1] to [VD#4]; the three additions in section 4; the storefront Ratings list raised in section 6 | Head of E-commerce |
| Facts about the live store and its operation: the storefront in use ([SR#2]), where the load generator runs ([SR#3], [SR#8]), the go-live clean-up, the operating side of the erasure and fraud removal path | Operations Lead |
| The withdrawn 3.8 and its return with a third-party monitoring tool; round 1, questions 25 to 27 | Operations Lead |

Names were not given in this review. Send later questions through the account manager, quoting the
requirement number, as section 6 of our requirements asks.

## 10. Unresolved blockers

**None.** Every question put to us is answered, or recorded as moot because the requirement behind
it is withdrawn. Every assumption is marked. The changes in section 12 are corrections the
delivery team applies to its own documents from our answers, and none of them waits on a further
decision from us.

Three things remain for the delivery team to propose, and none of them stops the work from
starting:
- how the operator removal path is kept out of shoppers' reach in a storefront with no sign-in
  ([VD#1], section 6);
- how purchase records survive the storefront's order Delete and Delete All (section 5,
  challenge 2);
- what remains of the storefront's Ratings list for shoppers (section 6).

Each goes to the owner in section 9 once proposed.

## 11. Readiness statement

The delivery team is clear to write the PRD and design for requirements 3.1, 3.2 (one rating per
customer per book), 3.3, 3.4, 3.5, 3.6 (catalogue list only) and 3.7, together with the three
additions in section 4, on the answers in this review and with [VD#1] and [VD#4] as amended in
section 6, and is not clear to build anything for 3.8 or for a list of a book's ratings, both
withdrawn from this release.

## 12. Required changes to downstream documents

| Document | Section or identifier | What must change |
|---|---|---|
| `BOOK-2-01-decisions.md` | [VD#1] | Restate the decision and its reasoning per section 6, points (a) to (f). (a) No storefront route deletes all ratings. (b) No storefront per-rating Delete. (c) An operator-only removal path for one rating or all of one customer's ratings. (d) The go-live clean-up is outside "bulk deletion". (e) The reasoning keeps "while the load generator runs", records that the generator never runs in Quillmark's live store, and no longer says the wipe resets averages shoppers see. If the generator change is kept, cover `IngestController.java` lines 55, 193–202 and 335–337 and `k8s/ingest.yaml` line 124, note the exceptions swallowed in `IngestRepository.java` lines 15–19, and say what the generator's rating step does under 3.1 and 3.2. (f) Delete "except individual removal if the customer allows it (round 1, question 9)". Add the question of how the operator path is kept from shoppers as an open point. |
| `BOOK-2-01-decisions.md` | [VD#4] | Change the statement to: a change to a rating may alter its stars and its comment; the customer e-mail and the ISBN never change on any path, operator paths included; a rating against the wrong book is removed through the operator removal path, never reassigned. In the second option, add the comment to what today's edit changes. Replace the sentence about "a separate correction path" with the no-reassignment rule. |
| `BOOK-2-01-decisions.md` | assumptions | Record the four assumptions in section 8 with our marks: `web/` is the storefront (accepted); the generator runs in live (corrected: it never does); typed-e-mail identity (accepted); non-buyer and repeat ratings exist in live data (cannot-say). |
| `BOOK-2-01-brd-defect-log.md` | [DEF#1] to [DEF#15] | Record the resolutions from this review. [DEF#1]: completed and paid, not since cancelled. [DEF#2]: not true today, and the rule is new work. [DEF#3]: keep the latest, remove the rest. [DEF#4]: independent. [DEF#5]: 3.7 governs and "final" is withdrawn. [DEF#6]: a requirement. [DEF#7]: two requirements, and the count matches the average. [DEF#8]: "Not yet rated" alone. [DEF#9]: catalogue list only. [DEF#10] to [DEF#12]: moot, 3.8 withdrawn. [DEF#13]: not a requirement. [DEF#14] and [DEF#15]: moot. Leave [DEF#16] to [DEF#25] untouched: they are outside this package. |
| `BOOK-2-01-brd-inventory.md` | new rows | Add the three requirements marked `missing` in section 4 (eligibility checked when given or changed; operator removal for erasure and fraud; comments stored, hidden and editable), each sourced to this review. The delivery team assigns their identifiers. Record [BR#3], [BR#10] and [BR#11] as withdrawn by Quillmark in this review, in whatever form the inventory and ledger use. |
| `BOOK-2-01-code-grounding.md` | [CG#4] | Add the further rating-deletion paths as evidence: `IngestController.java` line 55 with lines 256–259, lines 193–202, and `k8s/ingest.yaml` line 124. Note that `IngestRepository.java` lines 15–19 swallow delete failures. |
| `BOOK-2-01-code-grounding.md` | Derivation matrix, row "Whether the rater bought the book" | Reclass the row as needing new work. State the retention gap beside it: `OrderController.java` lines 206–219; the storefront's order Delete and Delete All at `order-list.component.html` lines 32 and 42; cancel clearing the completed flag in place at `OrderController.java` lines 311–313; and, in environments where it runs, the load generator clearing all orders. |
| `BOOK-2-01-code-grounding.md` | Note under the derivation matrix | Change "which the running load generator empties and refills on every cycle" to say this happens only where the load generator runs, which is never Quillmark's live store. |
| `BOOK-2-01-customer-questions.md` | Round 1, questions 3 and 5 | Record that these were answered in the reframed form [SR#4] and [SR#5] ask for: about any such ratings Quillmark's store holds, not as a fact about Quillmark's data. |
| `BOOK-2-01-customer-questions.md` | Round 1, questions 26, 27 and 30 | Mark them moot for this release, with the reason (3.8 withdrawn; Figure 1 note not a requirement). When 3.8 returns, re-ask question 26 and re-ask question 27 split as [SR#1] proposes. |

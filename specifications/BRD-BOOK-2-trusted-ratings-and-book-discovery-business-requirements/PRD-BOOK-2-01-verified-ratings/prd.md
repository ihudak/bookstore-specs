---
kind: prd
key: BOOK-2-01
title: Verified ratings
summary: Make Quillmark's star ratings trustworthy — buyers only, one rating per customer and book, whole stars 1 to 5 — and show each book's average and rating count on the book page and its average on the catalogue list.
status: draft
brd_key: BOOK-2-01
brd_parent: BOOK-2
---

# Verified ratings

## Problem

Shoppers on Quillmark's online store tell shop staff that the star ratings look made up, and they ignore them. A rating that shoppers ignore does not help them choose a book, and a shopper who cannot choose leaves without buying (Quillmark's requirements, sections 2 and 3).

The store today does nothing that would earn that trust:

- Any customer can rate any book, whether or not they bought it.
- A customer can rate the same book again and again, so one person can move a book's standing on their own.
- A rating saved without a star value counts as zero stars, which is outside the scale shoppers see.
- Neither the book page nor the catalogue list shows a book's average rating or how many ratings it rests on, so shoppers cannot tell a single opinion from a broad one.
- Anyone using the storefront can delete any rating, or every rating at once, so a book's ratings can vanish without explanation.

Quillmark wants this fixed in the next release of its online store.

## Goal

Shoppers can trust the star ratings in Quillmark's online store, because every rating that counts is between 1 and 5 whole stars and comes from a customer who had bought the book when the store last checked it (when the rating was given or changed, or at go-live for a rating that predates this release). Each customer holds one rating per book, which they can change but not repeat or remove. The book page shows each book's average rating with the number of ratings it rests on, and the catalogue list shows each book's average rating.

## Target audience

- **Shoppers browsing the store** — read a book's average rating and rating count on the book page and the catalogue list to decide what to buy.
- **Customers who bought a book** — give a rating to a book they bought, and change it later if they change their mind.
- **Quillmark operators** — remove a rating to meet an erasure request or to clean up fraud, and run the one-off clean-up of existing ratings at go-live.

Quillmark's Head of E-commerce owns the rating rules and how they are shown. The Operations Lead owns the operating side of rating removal and the go-live clean-up.

## User Stories

### [US#1]: Rate a book I bought

As a customer who bought a book, I want to give it a star rating, so that other shoppers can see what buyers think of it.

Traces to: [BR#1], [BR#4]; [CD#1], [CD#7], [VD#2].

### [US#2]: Change my rating of a book

As a customer who rated a book, I want to change my rating whenever I change my mind, so that my rating says what I think now and still counts only once.

Traces to: [BR#2], [BR#9]; [CD#4], [CD#5], [CD#6], [CD#17], [VD#2], [VD#4], [VD#5], [VD#6].

### [US#3]: See how a book is rated on its book page

As a shopper looking at a book, I want to see its average rating and how many ratings it rests on, so that I can tell a broad opinion from a single one.

Traces to: [BR#5], [BR#6], [BR#7]; [CD#8], [CD#9], [VD#3].

### [US#4]: Compare books by rating on the catalogue list

As a shopper browsing the catalogue list, I want to see each book's average rating, so that I can tell which books buyers rate highly before I open one.

Traces to: [BR#7], [BR#8]; [CD#10], [VD#3].

### [US#5]: Remove a rating for an erasure request or fraud

As a Quillmark operator, I want to remove a rating, so that I can meet a customer's erasure request or clean up fraudulent ratings.

Traces to: [CD#6], [VD#1], [VD#4]; the operator-removal addition in section 4 of Quillmark's review.

### [US#6]: Bring existing ratings in line with the rules at go-live

As a Quillmark operator, I want existing ratings brought in line with the new rules once, at go-live, so that every average shoppers see from the first day rests only on ratings the rules allow.

Traces to: [BR#1], [BR#2]; [CD#2], [CD#3], [VD#1]. Contested by a later meeting with Quillmark, and the live store's ratings have already been removed: see `## Assumptions & open questions`, item 1.

### [US#7]: Know that only Quillmark sees my comment

As a customer who rates a book, I want to know that only Quillmark sees my comment, so that I am not misled into thinking I am publishing a review.

Traces to: [CD#17]; the comments addition in section 4 of Quillmark's review.

## Acceptance Criteria

**[US#1] Rate a book I bought**

- [AC#1] A new rating is saved only when the e-mail entered belongs to a customer who, at that moment, has a qualifying order for the book: an order for that book that is completed and paid, and not since cancelled.
- [AC#2] When [AC#1] refuses a rating, nothing is saved and the customer sees exactly "Only customers who have bought this book can rate it." The same text appears whatever the reason: an e-mail the store does not know, no order for the book, an unpaid order, or an order since cancelled or deleted.
- [AC#3] The rating input offers only whole stars from 1 to 5.
- [AC#4] The store refuses a rating submitted with no star value, or with any value other than a whole number from 1 to 5, and saves nothing.

**[US#2] Change my rating of a book**

- [AC#6] The update form opened from the Ratings list shows the customer e-mail and the ISBN but does not let either be edited, does not show the rating's existing comment ([AC#12]), and carries the comment note of [AC#5].
- [AC#7] When a customer submits the add form for a book they have already rated, the store changes their existing rating to the new stars, and to the new comment as [AC#30] states, exactly as an Update would. A book never holds two ratings from the same customer.
- [AC#8] The store saves a change, made through Update or through the add form, only when the customer has a qualifying order for the book at the moment of the change. Otherwise it refuses the change with the message of [AC#2], and the existing rating stays exactly as it was and keeps counting.
- [AC#9] A change is held to the star scale of [AC#3] and [AC#4].
- [AC#10] Guard: a rating given or changed after go-live, or kept by the go-live clean-up, stays, and keeps counting in its book's average and count, when its qualifying order is cancelled or deleted after go-live.
- [AC#11] Guard: the Ratings list keeps each rating's rater e-mail and its Update action.
- [AC#29] Guard: from the Ratings list, a customer can still select Update on a rating and change its stars and its comment.
- [AC#30] On a change, made through Update or through the add form, an empty comment field leaves the rating's stored comment as it is, and a comment entered in the field replaces it.

**[US#3] See how a book is rated on its book page**

- [AC#13] The book page shows the book's average rating, rounded half up to one decimal place, so that an average of 4.25 shows as 4.3.
- [AC#14] Beside the average, the book page shows the number of ratings the average is computed from, always exactly those ratings, worded in the singular for one rating and in the plural otherwise.
- [AC#15] A book with no ratings shows "Not yet rated" in place of the average, alone, with no count.
- [AC#16] A rating that is given, changed or removed is reflected in the average and the count the next time the book page is loaded after the rating is saved or removed.

**[US#4] Compare books by rating on the catalogue list**

- [AC#17] Each book on the catalogue list shows its average rating, rounded half up to one decimal place as in [AC#13].
- [AC#18] A book with no ratings shows "Not yet rated" on the catalogue list in place of the average.
- [AC#19] The average the catalogue list shows for a book always equals the average its book page shows at the same moment.
- [AC#20] The catalogue list reflects a rating that is given, changed or removed the next time it is loaded, as [AC#16] does for the book page.

**[US#5] Remove a rating for an erasure request or fraud**

- [AC#21] Guard: an operator can still remove any single rating, and so can remove every rating one customer has given.
- [AC#22] A removed rating no longer counts in its book's average or count. **Withdrawn:** restates [AC#16] and [AC#20], which already cover a removed rating.
- [AC#23] No storefront screen lets anyone remove a rating, either one at a time or all at once. After go-live, no route, an operator's included, removes every rating at once: a rating leaves the store only through single-rating removal ([AC#21]) or the go-live clean-up ([AC#25] to [AC#27]).
- [AC#24] No path, an operator's included, changes the customer e-mail or the book of an existing rating. A rating recorded against the wrong book is removed under [AC#21], never moved to another book.

**[US#6] Bring existing ratings in line with the rules at go-live**

- [AC#25] At go-live, the clean-up removes every existing rating whose customer has no qualifying order for the book.
- [AC#26] After [AC#25], for each customer and book that still hold more than one rating, the clean-up keeps only the latest and removes the rest.
- [AC#27] The clean-up runs once, at go-live, started by operators. No storefront screen offers it.

**[US#7] Know that only Quillmark sees my comment**

- [AC#5] The comment field on the rating form shows exactly "Only Quillmark sees your comment. It is not shown to other shoppers."
- [AC#12] No screen a shopper can reach shows a rating's comment.
- [AC#28] Guard: the rating form still accepts a comment.

## Scope

**In scope**

- Buyers-only rating: a rating is given or changed only against a qualifying order ([US#1], [US#2]).
- One rating per customer and book, where rating the same book again changes the existing rating ([US#2]).
- Whole stars from 1 to 5 on every rating ([US#1], [US#2]).
- Changing a rating's stars and comment at any time while the customer holds a qualifying order, with its customer e-mail and book fixed ([US#2], [US#5]).
- Comments still taken by the forms and kept, shown to no shopper, with a note on the forms saying so ([US#7]).
- The Ratings list without comments ([US#7]) and without Delete or Delete All ([US#5]).
- The average rating and the rating count on the book page, with "Not yet rated" for an unrated book ([US#3]).
- The average rating, or "Not yet rated", on the catalogue list ([US#4]).
- Operator removal of a rating for erasure requests and fraud ([US#5]).
- The one-off clean-up of existing ratings at go-live ([US#6]), contested (`## Assumptions & open questions`, item 1).

**Out of scope**

- Synthetic monitoring that buys a book and then rates it (Quillmark's requirement 3.8, withdrawn: [CD#11]).
- A list of one book's ratings opened from its rating count (the Figure 1 note, withdrawn: [CD#12]). The existing Ratings list stays, as [US#2] describes.
- Written reviews, their moderation, and showing rating comments to shoppers (Quillmark's section 5; [CD#17]).
- A customer removing their own rating ([CD#6]).
- Moving a rating to another customer or another book ([VD#4]).
- Sign-in, or any check that the person typing an e-mail owns it ([CD#14]).
- Average ratings anywhere other than the book page and the catalogue list, including search results, book suggestions and the Top rated shelf ([CD#10]; the sibling slice [BOOK-2-02](../PRD-BOOK-2-02-book-discovery/)).
- Any change to orders, checkout, payment or order cancellation, including recording delivery ([VD#5], [CD#18]; Quillmark's section 5).
- Notifications of any kind, and a mobile app (Quillmark's section 5).
- Changes to the store's test-traffic generator ([VD#1]).

## Success Metrics

- [SM#1] Buyers-only integrity: the share of ratings counted in any book's average that passed the buyers-only rule when the store last checked them. A rating kept by the go-live clean-up passes when its customer had a qualifying order at go-live ([AC#25]); any later rating passes when its customer had a qualifying order when it was last given or changed ([AC#1], [AC#8]). Target: 100%, checked at go-live and monthly for the first three months after go-live.
- [SM#2] One rating per customer and book: the number of customer and book pairs that hold more than one rating. Target: 0, at go-live and at every monthly check over the first three months after go-live.
- [SM#3] Buyer participation: the number of distinct customer and book pairs whose rating was given or changed in the month, so that repeated changes to one rating count once. Baseline: the first full calendar month after go-live. Target: at or above the baseline in each of the two months that follow.
- [SMC#1] Counterbalances [SM#3]: ratings refused by the buyers-only rule. Not to be optimized: a rise is investigated, for example as a sign of a confusing form, and never brought down by accepting ratings from customers without a qualifying order.

## Use cases & user journey

- [UC#1] **Rate a book after buying it.** A customer who bought a book opens the rating form, enters the e-mail they bought it with, picks a number of stars and may add a comment, reading that only Quillmark sees it. The store confirms the purchase and saves the rating ([US#1]). The next time anyone opens the book page or the catalogue list, the book's average and count include it ([US#3], [US#4]).
- [UC#2] **Change their mind.** Weeks later the same customer finds their rating in the Ratings list under their e-mail and updates its stars, or rates the book again from the rating form. Either way the book still holds one rating from them, now with the new stars ([US#2]).
- [UC#3] **Try to rate without a purchase.** A shopper types an e-mail that has no completed, paid order for the book, for example a friend's e-mail whose owner never bought the book. The store saves nothing and shows the rule, and the book's average is untouched ([US#1]).
- [UC#4] **Handle an erasure request.** A customer asks Quillmark to erase their data. An operator removes each of that customer's ratings ([US#5]). Each affected book's average and count drop those ratings on the next load, and a book left with no ratings shows "Not yet rated" ([US#3], [US#4]).

## Non-functional requirements

- The rating rules hold however a rating reaches the store, not only through the storefront's forms: buyers only ([AC#1], [AC#8]), one rating per customer and book ([AC#7]), whole stars from 1 to 5 ([AC#4], [AC#9]), a fixed customer and book ([AC#24]), and, apart from the go-live clean-up, removal one rating at a time only ([AC#23]). A rating that arrives by any other route is treated on exactly the same terms: refused where the storefront refuses it, and a repeat rating applied as a change under [AC#7] and [AC#8].

## Assumptions & open questions

1. **Contradiction — the go-live clean-up ([US#6]).** At a follow-up meeting after its review, Quillmark agreed that every existing rating is removed at go-live, rather than only the ratings of customers with no qualifying order ([CD#2]) and every earlier repeat ([CD#3]). The delivery team reported this while authoring this PRD, and it is not yet in the decision register, so this PRD carries [CD#2] and [CD#3] as the register states them. If the meeting's position stands, [US#6] changes: every book starts at "Not yet rated", the comments customers already wrote go with their ratings, and [VD#1]'s reasoning that the go-live clean-up is not bulk deletion, which rests on [CD#2] and [CD#3], must be decided again. Route: `/product-workflows:brd-package` on this slice puts the question to Quillmark, and `/product-workflows:brd-reconcile` on this slice, given Quillmark's reply, records its answer as a decision superseding [CD#2] and [CD#3]. `/product-workflows:brd-interview` on this slice then decides [VD#1] again, and `/product-workflows:update-prd` refreshes this PRD. Owners at Quillmark: the Head of E-commerce (the rating rules) and the Operations Lead (the go-live clean-up). **Already done in the live store:** the delivery team reports, on 2026-10-12, that every rating in Quillmark's live store has already been removed, as the meeting agreed. So in the live store [US#6]'s clean-up finds no existing rating to act on, every book starts at "Not yet rated", the comments customers had written are gone with their ratings, although section 4 of the review kept comments already stored ([CD#17]), and [SM#1] and [SM#2] hold trivially at go-live. [CD#2] and [CD#3] still stand in the register until Quillmark's answer is reconciled. Should they stand, Quillmark must also say what "latest" in [CD#3] means for [AC#26] (the rating given last, or the one given or changed last), and what becomes of an existing rating outside whole stars 1 to 5, which neither decision covers; in the live store both questions are moot, since no rating remains there.
2. **Accepted: the rater is whoever's e-mail is typed** ([CD#14]). The store has no sign-in in this release, so someone could type a buyer's e-mail and rate, or change the rating of, a book that buyer bought. Quillmark accepted this in section 8 of its review.
3. **The operators' removal path is not protected from shoppers** ([VD#1]). With no sign-in, the store cannot tell an operator's removal from the same request made by anyone else outside the storefront screens. No screen offers removal ([AC#23]), but the path itself can be reached. Quillmark asked how this path is kept from shoppers (section 10 of its review), and [VD#1] accepts the risk until sign-in exists. That answer goes, with [VD#5] and [VD#6], to the owners section 9 of the review names.
4. **Raters' e-mails stay visible on the Ratings list** ([VD#6]). Any shopper can see which e-mail gave which rating. This is part of the typed-e-mail identity Quillmark accepted, and it can change once the store has sign-in.
5. **A cancelled or deleted order ends the right to change a rating, not the rating** ([VD#5]). By Quillmark's decision (section 4 of its review), eligibility is checked when a rating is given or changed, not afterwards. A customer whose qualifying order is later cancelled or deleted keeps their rating, which keeps counting, but cannot change it ([AC#8], [AC#10]). Such a rating counts although its customer no longer holds the purchase [AC#1] requires.

## References / linked issues

- Quillmark's requirements, sections 2, 3 and 5: `../brd/source/quillmark-trusted-ratings-brd.md`
- This slice's requirement inventory and coverage ledger: `brd/brd-inventory.md`, `coverage-ledger.md`
- Decision register: `decisions.md`
- Quillmark's review and its reconciliation: `customer-review-20261012.md`, `reconciliation-20261012.md`
- Interview rounds: `interview/round-1.md`, `interview/round-2.md`
- Code grounding of `bookstore` at `e439178`: `grounding/code-grounding.md`
- Sibling slice for book suggestions and the Top rated shelf: [BOOK-2-02](../PRD-BOOK-2-02-book-discovery/)

## Documentation impact

- **Shopper help:** who can rate a book, how to change a rating, that comments are seen by Quillmark only, what the average and the count mean, and what "Not yet rated" means.
- **Operator guide:** removing a rating for an erasure request or for fraud, including every rating of one customer, and running the one-off clean-up at go-live.
- **Reference for the store's ratings operations:** removing every rating at once is no longer offered, and a rating that breaks the rules is refused.
- The product documentation set holds no pages yet, so each of these is new.

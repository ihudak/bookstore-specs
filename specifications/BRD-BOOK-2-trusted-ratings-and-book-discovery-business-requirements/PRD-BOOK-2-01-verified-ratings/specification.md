---
kind: specification
key: BOOK-2-01
---

# Specification

- **Feature name**: Verified ratings
- **Version**: 1
- **Created**: 2026-10-13
- **Author**: ihudak
- **Published**: no
- **Open questions**: 5

## Problem statement

Shoppers on Quillmark's online store ignore its star ratings, because nothing about them earns trust. Any customer can rate any book whether or not they bought it, and the same customer can rate one book again and again, so one person can move a book's standing alone. A rating saved without a star value counts as zero, below the scale shoppers see. Anyone using the storefront can delete any rating, or every rating at once, so ratings can vanish without explanation. The comments customers write with a rating appear on the storefront, although Quillmark does not publish reviews. Neither the book page nor the catalogue list shows a book's average rating or how many ratings it rests on, so a shopper cannot tell one opinion from many. Ratings that shoppers ignore do not help them choose, and a shopper who cannot choose leaves without buying. Quillmark wants this fixed in the next release of its store, which starts with no ratings: every existing rating was removed before go-live.

## Scope

**In scope**

- Buyers only: a rating is given or changed only when the e-mail entered holds a qualifying order for the book at that moment (completed and paid, and not since cancelled); while orders can answer, every otherwise valid attempt without one is refused with one message, whatever the reason.
- One rating per customer and book: rating a book again changes the existing rating.
- Whole stars from 1 to 5 on every rating, with a rating input that offers only those values.
- The book page shows the book's average rating, rounded half up to one decimal place, and the number of ratings it rests on, or "Not yet rated".
- The catalogue list shows each book's average rating, or "Not yet rated".
- Changing a rating's stars and comment from the Ratings list, with its customer e-mail and book fixed.
- Comments still taken and kept, shown to no shopper: a note on the rating forms, and no comment on the Ratings list, the rating details screen or any response of the ratings service.
- No deletion from the storefront: the Ratings list loses Delete and Delete All, and removing every rating at once no longer exists anywhere.
- Operators remove a single rating, for an erasure request or fraud, through the ratings service's per-rating removal.
- The rating rules hold for every caller of the ratings service, not only for the storefront's forms.
- What the storefront shows when the purchase check cannot be made, when a rating to be changed cannot be loaded or no longer exists, when a rating cannot be saved, and when a book's rating cannot be loaded.

**Out of scope**

- Restricting the per-rating removal to operators: until the store has sign-in, anyone who calls the ratings service can reach it ([VD#1], [CD#14]).
- Opening the list of a book's ratings by clicking its rating count ([CD#12]).
- Average ratings anywhere other than the book page and the catalogue list, including search results, book suggestions and the Top rated shelf ([CD#10]; the sibling slice [BOOK-2-02](../PRD-BOOK-2-02-book-discovery/)).
- Showing rating comments to shoppers, written reviews and their moderation ([CD#17]).
- An operator screen or endpoint for reading comments; Quillmark reads them where the ratings are stored ([AD#7]).
- A customer removing their own rating ([CD#6]).
- Moving a rating to another customer or another book ([VD#4]).
- Sign-in, or any check that the person typing an e-mail owns it ([CD#14]).
- Hiding raters' e-mails: they stay visible on the Ratings list ([VD#6]).
- What the rating details screen and the Ratings list show when a rating cannot be loaded or no longer exists, which stays as it is today.
- A customer clearing a stored comment: an empty comment on a change keeps the stored one ([AC#30]), and an erasure request removes the whole rating ([U06]).
- Any change to orders, checkout, payment or order cancellation, including matching a typed e-mail to an order's e-mail regardless of letter case ([VD#5]).
- Any clean-up of existing ratings: every rating the store held was removed before go-live ([CD#19], [CD#20]).
- Emptying the stored ratings in each environment when the release is deployed, which is an operator step of the release rather than storefront behaviour (`ard.md`, Edge cases & risks).
- Synthetic monitoring that buys a book and then rates it ([CD#11]).
- Any change to the store's test-traffic generator ([CD#16]).
- Response-time or load targets for the book page, the catalogue list, or a give or change: none is set in this release, and the costs are recorded in `ard.md`, Edge cases & risks.

### Open questions

- [ ] Re-review finding (MINOR, cardinality): what is the longest comment a rating may carry? No limit is stated, so a comment longer than the ratings store holds would fail on every retry with the "try again" text of [U01] [AC12]. Either set a limit that the rating form and the update form enforce and the ratings service refuses beyond as invalid with a named reason, or state it out of scope.

## User stories

### [U01]: Rate a book I bought

As a customer who bought a book, I want to give it a star rating, so that other shoppers can see what buyers think of it.

Done when a rating given with a qualifying order for the book is stored, and every other give is refused with nothing stored.

Traces to: [US#1]; [AC#1]–[AC#4]; [VD#2]; [CG#2], [CG#5].

#### [AC01]: A give from a buyer is stored

When an otherwise valid give ([AC07], [AC08]) for an e-mail and ISBN that hold no rating, the pair compared ignoring letter case ([AD#3]), reaches the ratings service through the ratings write interface ([AD#4]), and orders reports a qualifying order for that ISBN under the e-mail exactly as the give carries it ([AD#2]), the ratings service shall store one rating for that e-mail and ISBN with the given stars.

##### Test cases

**[TC01]: A buyer rates through the rating form — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Open the Ratings list and select New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The ratings service stores exactly one rating for `reader@example.com` and `9780000000017`, with 4 stars.

**[TC02]: A buyer's give sent straight to the ratings service is stored — Happy path:**
- *Preconditions:* `direct@example.com` holds a qualifying order for ISBN `9780000000024`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Send a give for `direct@example.com`, `9780000000024` and 5 stars to the ratings write interface ([AD#4]), with no storefront involved.
- *Expected result:* The ratings service stores exactly one rating for `direct@example.com` and `9780000000024`, with 5 stars.

**[TC03]: A buyer's purchase of one book does not let them rate another — Negative / boundary:**
- *Preconditions:* `reader2@example.com` holds a qualifying order for ISBN `9780000000017` and no order for `9780000000031`; no rating exists for that e-mail.
- *Steps:* 1. Send a give for `reader2@example.com`, `9780000000031` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service stores no rating for `reader2@example.com` and `9780000000031`.

#### [AC02]: A give without a qualifying order is refused

If an otherwise valid give ([AC07], [AC08]) reaches the ratings service ([AD#4]) for an e-mail that, as orders reports it ([AD#2]), holds no qualifying order for the ISBN, then the ratings service shall refuse the give with the buyers-only refusal, storing no rating.

##### Test cases

**[TC01]: A customer with no order for the book is refused — Happy path:**
- *Preconditions:* `browser@example.com` is a customer of the store with no order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Send a give for `browser@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and stores no rating for `browser@example.com` and `9780000000017`.

**[TC02]: An unpaid order does not qualify — Negative / boundary:**
- *Preconditions:* `unpaid@example.com` has an order for ISBN `9780000000017` that was placed but never completed and paid.
- *Steps:* 1. Send a give for `unpaid@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and stores no rating for `unpaid@example.com` and `9780000000017`.

**[TC03]: A cancelled order does not qualify — Negative / boundary:**
- *Preconditions:* `cancelled@example.com` had a completed, paid order for ISBN `9780000000017`, since cancelled.
- *Steps:* 1. Send a give for `cancelled@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and stores no rating for `cancelled@example.com` and `9780000000017`.

**[TC04]: A deleted order does not qualify — Negative / boundary:**
- *Preconditions:* `deleted@example.com` had a completed, paid order for ISBN `9780000000017`, since deleted from the store.
- *Steps:* 1. Send a give for `deleted@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and stores no rating for `deleted@example.com` and `9780000000017`.

**[TC05]: An e-mail the store does not know is refused — Negative / boundary:**
- *Preconditions:* `stranger@example.com` is not a customer of the store.
- *Steps:* 1. Send a give for `stranger@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and stores no rating for `stranger@example.com`.

**[TC06]: An ISBN the store does not know is refused — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for `9780000000017`; ISBN `9789999999999` is 13 digits and names no book in the store.
- *Steps:* 1. Send a give for `reader@example.com`, `9789999999999` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and stores no rating for `9789999999999`.

#### [AC03]: The rating form names the buyers-only rule

When the ratings service refuses a give from the rating form with the buyers-only refusal ([AD#4]), the rating form shall display exactly "Only customers who have bought this book can rate it."

##### Test cases

**[TC01]: A customer with no order sees the rule — Happy path:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The rating form displays exactly "Only customers who have bought this book can rate it."

**[TC02]: A customer whose order was cancelled sees the same text — Negative / boundary:**
- *Preconditions:* `cancelled@example.com` had a completed, paid order for ISBN `9780000000017`, since cancelled.
- *Steps:* 1. Open the rating form. 2. Enter `cancelled@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The rating form displays exactly "Only customers who have bought this book can rate it.", with no text naming the cancellation.

#### [AC04]: One refusal for every reason

The ratings service shall answer every buyers-only refusal with one identical response, whatever the reason for the refusal ([AD#4]).

##### Test cases

**[TC01]: An unknown e-mail and a customer with no order get the same answer — Happy path:**
- *Preconditions:* `stranger@example.com` is not a customer; `browser@example.com` is a customer with no order for ISBN `9780000000017`.
- *Steps:* 1. Send a give for `stranger@example.com`, `9780000000017` and 4 stars to the ratings write interface. 2. Send a give for `browser@example.com`, `9780000000017` and 4 stars.
- *Expected result:* The ratings service answers both gives with one identical response, status and body alike.

**[TC02]: Unpaid, cancelled, deleted and unknown-book refusals get the same answer — Negative / boundary:**
- *Preconditions:* `unpaid@example.com` has an order for ISBN `9780000000017` that was placed but never completed and paid; `cancelled@example.com` had a completed, paid order for it, since cancelled; `deleted@example.com` had a completed, paid order for it, since deleted; `reader@example.com` holds a qualifying order for `9780000000017` only; ISBN `9789999999999` is 13 digits and names no book.
- *Steps:* 1. Send a give for each of `unpaid@example.com`, `cancelled@example.com` and `deleted@example.com` with `9780000000017` and 4 stars. 2. Send a give for `reader@example.com` with the unknown ISBN `9789999999999` and 4 stars.
- *Expected result:* The ratings service answers all four gives with one identical response, status and body alike.

**[TC03]: The refusal discloses nothing about the customer or the order — Security / privacy:**
- *Preconditions:* `cancelled@example.com` had a completed, paid order for ISBN `9780000000017`, since cancelled.
- *Steps:* 1. Send a give for `cancelled@example.com`, `9780000000017` and 4 stars to the ratings write interface. 2. Read the whole response.
- *Expected result:* The ratings service's response names no reason: it does not say whether the e-mail is a customer, whether an order exists, or the order's state.

#### [AC05]: The rating form offers whole stars 1 to 5 only

The rating form shall offer exactly five star choices, the whole numbers 1, 2, 3, 4 and 5, with none chosen when the form opens.

##### Test cases

**[TC01]: The form opens with five star choices and none chosen — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating.
- *Expected result:* The rating form offers exactly the star choices 1, 2, 3, 4 and 5, with none chosen.

**[TC02]: No value outside the scale can be chosen — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form. 2. Try to choose or enter 0, 6, -1 and 3.5 as the star value.
- *Expected result:* The rating form offers no way to choose any value other than 1, 2, 3, 4 or 5.

#### [AC06]: The rating form needs a star choice

While no star choice is chosen, the rating form shall keep Submit from sending the rating.

##### Test cases

**[TC01]: With a star chosen, Submit sends the rating — Happy path:**
- *Preconditions:* `reader3@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Open the rating form. 2. Enter `reader3@example.com` and `9780000000017`, and choose 2 stars. 3. Select Submit.
- *Expected result:* The rating form sends the rating, and the ratings service stores it with 2 stars.

**[TC02]: With no star chosen, nothing is sent — Negative / boundary:**
- *Preconditions:* `reader4@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Open the rating form. 2. Enter `reader4@example.com` and `9780000000017`, and choose no stars. 3. Select Submit.
- *Expected result:* The rating form sends nothing, and the ratings service stores no rating for `reader4@example.com` and `9780000000017`.

#### [AC07]: Stars outside the scale are refused for every caller

If a give reaches the ratings service ([AD#4]) with no star value, or with a star value that is not a whole number from 1 to 5, then the ratings service shall refuse it as invalid with the reason "Invalid rating. Must be a whole number from 1 to 5.", storing no rating.

##### Test cases

**[TC01]: A give with no star value is refused, not stored as zero — Happy path:**
- *Preconditions:* `nostars@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Send a give for `nostars@example.com` and `9780000000017` with no star value to the ratings write interface.
- *Expected result:* The ratings service refuses the give as invalid with the reason "Invalid rating. Must be a whole number from 1 to 5." and stores no rating.

**[TC02]: Zero and six are refused — Negative / boundary:**
- *Preconditions:* `edges@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Send a give for `edges@example.com` and `9780000000017` with 0 stars. 2. Send the same give with 6 stars.
- *Expected result:* The ratings service refuses both gives as invalid with the reason "Invalid rating. Must be a whole number from 1 to 5." and stores no rating.

**[TC03]: A fractional star value is refused — Negative / boundary:**
- *Preconditions:* `fraction@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Send a give for `fraction@example.com` and `9780000000017` with 3.5 stars.
- *Expected result:* The ratings service refuses the give as invalid and stores no rating.

**[TC04]: One and five stars are accepted — Negative / boundary:**
- *Preconditions:* `low@example.com` and `high@example.com` each hold a qualifying order for ISBN `9780000000017`; neither has a rating.
- *Steps:* 1. Send a give for `low@example.com` and `9780000000017` with 1 star. 2. Send a give for `high@example.com` and `9780000000017` with 5 stars.
- *Expected result:* The ratings service refuses neither give as invalid, and stores a 1-star rating for `low@example.com` and a 5-star rating for `high@example.com`.

**[TC05]: Invalid stars from a customer with no order are refused as invalid — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`.
- *Steps:* 1. Send a give for `browser@example.com` and `9780000000017` with 0 stars.
- *Expected result:* The ratings service refuses the give as invalid with the reason "Invalid rating. Must be a whole number from 1 to 5.", not with the buyers-only refusal.

#### [AC08]: A missing or malformed e-mail or ISBN is refused

If a give reaches the ratings service ([AD#4]) with a missing or malformed e-mail, or with a missing ISBN or one that is not 13 digits, then the ratings service shall refuse it as invalid, storing no rating, with the reason "Invalid email address" for a malformed e-mail and "ISBN must be a 13-digits value" for an ISBN that is not 13 digits.

##### Test cases

**[TC01]: A malformed e-mail is refused — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Send a give for the e-mail `reader-at-example`, ISBN `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the give as invalid with the reason "Invalid email address" and stores no rating.

**[TC02]: A 12-digit ISBN is refused — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017`.
- *Steps:* 1. Send a give for `reader@example.com`, ISBN `978000000001` and 4 stars.
- *Expected result:* The ratings service refuses the give as invalid with the reason "ISBN must be a 13-digits value" and stores no rating.

**[TC03]: A give with no e-mail or no ISBN is refused — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Send a give with ISBN `9780000000017` and 4 stars and no e-mail. 2. Send a give with the e-mail `reader@example.com` and 4 stars and no ISBN.
- *Expected result:* The ratings service refuses both gives as invalid and stores no rating.

#### [AC09]: The rating form shows an invalid write's reason

When the ratings service refuses a give from the rating form as invalid ([AD#4]), the rating form shall display the reason the ratings service gives.

##### Test cases

**[TC01]: A malformed e-mail shows its reason — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form. 2. Enter `reader-at-example` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form displays "Invalid email address".

**[TC02]: A short ISBN shows its reason, not the buyers-only text — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017`.
- *Steps:* 1. Open the rating form. 2. Enter `reader@example.com` and `978000000001`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form displays "ISBN must be a 13-digits value" and not "Only customers who have bought this book can rate it."

##### Open questions

- [ ] Re-review finding (MAJOR): what does the rating form show when the e-mail or the ISBN is left empty? [AC08] names no reason for a missing e-mail or a missing ISBN, so the text [AC09] displays for them is undefined. Either name both reasons in [AC08], or keep Submit from sending while either field is empty, as [AC06] does for the stars.

#### [AC10]: A give is refused while orders cannot answer

If orders does not answer the purchase check for an otherwise valid give within a bounded time, cannot be reached, or answers with an error other than not-found ([AD#2]), then the ratings service shall refuse the give as unavailable, storing no rating.

##### Test cases

**[TC01]: A buyer's give is refused while orders is unreachable — Happy path:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send a give for `reader5@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the give as unavailable and stores no rating.

**[TC02]: A non-buyer's give while orders is unreachable is not refused as not bought — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send a give for `browser@example.com`, `9780000000017` and 4 stars.
- *Expected result:* The ratings service refuses the give as unavailable, not with the buyers-only refusal, and stores no rating.

**[TC03]: A give is refused in bounded time while orders never answers — Negative / boundary:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN; the orders service accepts connections from the ratings service but never answers.
- *Steps:* 1. Send a give for `reader5@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the give as unavailable within its bounded time, rather than waiting on orders, and stores no rating.

**[TC04]: An orders server error is not read as not bought — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`; the orders service answers every request with a server error.
- *Steps:* 1. Send a give for `browser@example.com`, `9780000000017` and 4 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the give as unavailable, not with the buyers-only refusal, and stores no rating.

**[TC05]: The same give is stored once orders answers again — State / lifecycle:**
- *Preconditions:* `reader6@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN; the orders service is unreachable.
- *Steps:* 1. Send a give for `reader6@example.com`, `9780000000017` and 4 stars. 2. Make the orders service reachable again. 3. Send the same give again.
- *Expected result:* The ratings service refuses the first give as unavailable and stores one rating, with 4 stars, from the second.

##### Open questions

- [ ] Re-review finding (MINOR, observability): should the ratings service write each refusal as unavailable to its own log, naming the orders failure behind it (a timeout, a connection failure or the status orders answered), as [AC15] logs a buyers-only refusal's reason? This puts [AC10] and [U02] [AC12] in doubt; the alternative is to state it out of scope.

#### [AC11]: The rating form says the purchase check is unavailable

When the ratings service refuses a give from the rating form as unavailable ([AD#4]), the rating form shall display exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

##### Test cases

**[TC01]: A buyer sees the unavailable text while orders is unreachable — Happy path:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the orders service is unreachable from the ratings service.
- *Steps:* 1. Open the rating form. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form displays exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

**[TC02]: A customer with no order sees the unavailable text, not the rule — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`; the orders service is unreachable from the ratings service.
- *Steps:* 1. Open the rating form. 2. Enter `browser@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form displays exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." and not "Only customers who have bought this book can rate it."

#### [AC12]: The rating form says when a rating cannot be saved

If the rating form cannot reach the ratings service, or the ratings service answers a give with anything [AC03], [AC09], [AC11] and [AC14] do not cover ([AD#4]), then the rating form shall display exactly "We can't save your rating right now. Please try again in a few minutes."

##### Test cases

**[TC01]: The ratings service is unreachable — Happy path:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service is unreachable from the storefront.
- *Steps:* 1. Open the rating form. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form displays exactly "We can't save your rating right now. Please try again in a few minutes."

**[TC02]: An unexpected server error is not shown as a refusal — Negative / boundary:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service answers every give with an unexpected server error.
- *Steps:* 1. Open the rating form. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form displays exactly "We can't save your rating right now. Please try again in a few minutes.", and neither "Only customers who have bought this book can rate it." nor the purchase-check text of [AC11].

#### [AC13]: The rating form keeps what was entered after a refusal

When the ratings service refuses a give from the rating form ([AD#4]), or the rating form cannot save it ([AC12]), the rating form shall stay open with the e-mail, ISBN, star choice and comment as entered.

##### Test cases

**[TC01]: A buyers-only refusal keeps the entries — Happy path:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`.
- *Steps:* 1. Open the rating form. 2. Enter `browser@example.com` and `9780000000017`, choose 3 stars and enter the comment "Lovely cover". 3. Select Submit.
- *Expected result:* The rating form stays open showing `browser@example.com`, `9780000000017`, 3 stars and "Lovely cover".

**[TC02]: An unavailable refusal keeps the entries — Negative / boundary:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the orders service is unreachable from the ratings service.
- *Steps:* 1. Open the rating form. 2. Enter `reader5@example.com` and `9780000000017`, choose 5 stars and enter the comment "Loved it". 3. Select Submit.
- *Expected result:* The rating form stays open showing `reader5@example.com`, `9780000000017`, 5 stars and "Loved it", and the Ratings list is not opened.

**[TC03]: A save that cannot reach the ratings service keeps the entries — Negative / boundary:**
- *Preconditions:* `reader5@example.com` holds a qualifying order for ISBN `9780000000017`; the ratings service is unreachable from the storefront.
- *Steps:* 1. Open the rating form. 2. Enter `reader5@example.com` and `9780000000017`, choose 2 stars and enter the comment "Too long". 3. Select Submit.
- *Expected result:* The rating form stays open showing `reader5@example.com`, `9780000000017`, 2 stars and "Too long".

#### [AC14]: A stored give opens the Ratings list

When the ratings service stores a give from the rating form ([AD#4]), the rating form shall open the Ratings list.

##### Test cases

**[TC01]: A buyer's stored give opens the Ratings list — Happy path:**
- *Preconditions:* `reader7@example.com` holds a qualifying order for ISBN `9780000000017`; no rating exists for that e-mail and ISBN.
- *Steps:* 1. Open the rating form. 2. Enter `reader7@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form opens the Ratings list, which lists the rating of `reader7@example.com` for `9780000000017` with 4 stars.

**[TC02]: A refused give does not open the Ratings list — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`.
- *Steps:* 1. Open the rating form. 2. Enter `browser@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form does not open the Ratings list.

#### [AC15]: Each buyers-only refusal's reason is logged

When the ratings service refuses a give or a change with the buyers-only refusal, the ratings service shall write the reason for the refusal to its own log.

##### Test cases

**[TC01]: An unpaid order's refusal is logged with its reason — Happy path:**
- *Preconditions:* `unpaid@example.com` has an order for ISBN `9780000000017` that was placed but never completed and paid.
- *Steps:* 1. Send a give for `unpaid@example.com`, `9780000000017` and 4 stars to the ratings write interface. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds an entry for the refused give that names its reason, that the customer holds no qualifying order for the book.

**[TC02]: An unknown e-mail's refusal is logged with its own reason — Negative / boundary:**
- *Preconditions:* `stranger@example.com` is not a customer of the store.
- *Steps:* 1. Send a give for `stranger@example.com`, `9780000000017` and 4 stars. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds an entry for the refused give that names its reason, that the e-mail is not a customer of the store.

**[TC03]: The reason reaches the log and not the caller — Audit / observability:**
- *Preconditions:* `cancelled@example.com` had a completed, paid order for ISBN `9780000000017`, since cancelled.
- *Steps:* 1. Send a give for `cancelled@example.com`, `9780000000017` and 4 stars. 2. Compare the response with the ratings service's log.
- *Expected result:* The ratings service's log names the refusal's reason, and the response carries only the buyers-only refusal of [AC04].

**[TC04]: A refused change's reason is logged — Audit / observability:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while their order qualified; that order has since been cancelled.
- *Steps:* 1. Send a change of that rating to 5 stars to the ratings write interface. 2. Read the ratings service's log.
- *Expected result:* The ratings service's log holds an entry for the refused change that names its reason, that the customer holds no qualifying order for the book.

---

### [U02]: Change my rating of a book

As a customer who rated a book, I want to change my rating whenever I change my mind, so that my rating says what I think now and still counts only once.

Done when a change from a customer who holds a qualifying order replaces the stars, and every rating stays one per customer and book with its customer e-mail and book fixed.

Traces to: [US#2], [US#5]; [AC#6]–[AC#11], [AC#24], [AC#29], [AC#30]; [VD#2]; [CG#3], [CG#10].

#### [AC01]: A buyer's change replaces the stars

When an otherwise valid change ([AC09], [AC10]) to a rating reaches the ratings service through the ratings write interface ([AD#4]), and orders reports a qualifying order for the rating's ISBN under the e-mail exactly as the change carries it, or under the stored rating's e-mail when the change carries none ([AD#2]), the ratings service shall replace the rating's stars with the stars of the change.

##### Test cases

**[TC01]: A buyer changes their stars through Update — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it.
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The ratings service holds one rating for `reader@example.com` and `9780000000017`, with 4 stars.

**[TC02]: A change sent straight to the ratings service replaces the stars — Happy path:**
- *Preconditions:* `direct@example.com` holds a qualifying order for ISBN `9780000000024` and a 5-star rating of it.
- *Steps:* 1. Send a change of that rating to 3 stars to the ratings write interface, with no storefront involved.
- *Expected result:* The ratings service holds one rating for `direct@example.com` and `9780000000024`, with 3 stars.

**[TC03]: Repeating an identical change leaves one unchanged rating — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 4-star rating of it.
- *Steps:* 1. Send a change of that rating to 4 stars. 2. Send the same change again.
- *Expected result:* The ratings service holds one rating for `reader@example.com` and `9780000000017`, with 4 stars.

#### [AC02]: Rating a book again changes the existing rating

When a give reaches the ratings service ([AD#4]) for an e-mail and ISBN that already hold a rating, the pair compared ignoring letter case ([AD#3]), the ratings service shall apply the give to that rating as a change, under every rule of a change through Update.

##### Test cases

**[TC01]: A buyer rates the same book again from the rating form — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it.
- *Steps:* 1. Open the rating form. 2. Enter `reader@example.com` and `9780000000017`, and choose 5 stars. 3. Select Submit.
- *Expected result:* The ratings service holds one rating for `reader@example.com` and `9780000000017`, with 5 stars.

**[TC02]: A repeat give after the order was cancelled is refused as a change — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while their order qualified; that order has since been cancelled.
- *Steps:* 1. Send a give for `lapsed@example.com`, `9780000000017` and 5 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal and still holds one rating for `lapsed@example.com` and `9780000000017`, with 3 stars.

**[TC03]: A give from another buyer of the same book is a rating of its own — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; `second@example.com` holds a qualifying order for `9780000000017` and no rating of it.
- *Steps:* 1. Send a give for `second@example.com`, `9780000000017` and 1 star.
- *Expected result:* The ratings service holds two ratings for `9780000000017`, one for each e-mail, and the rating of `reader@example.com` is unchanged.

**[TC04]: A repeat give typed in other capitals changes the same rating — Negative / boundary:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; the orders service holds a qualifying order for `9780000000017` under the spelling `Reader@Example.com` and none under `reader@example.com`.
- *Steps:* 1. Send a give for `Reader@Example.com`, `9780000000017` and 5 stars to the ratings write interface.
- *Expected result:* The ratings service holds one rating of `9780000000017` for that customer, with 5 stars.

#### [AC03]: Simultaneous gives leave one rating

If two gives for one e-mail and ISBN, the pair compared ignoring letter case, reach the ratings service at the same time ([AD#3]), then the ratings service shall leave exactly one rating for that e-mail and ISBN.

##### Test cases

**[TC01]: Two simultaneous gives from one buyer leave one rating — Happy path:**
- *Preconditions:* `racer@example.com` holds a qualifying order for ISBN `9780000000017` and no rating of it.
- *Steps:* 1. Send a give for `racer@example.com`, `9780000000017` and 3 stars and a give for the same e-mail and ISBN with 5 stars at the same moment.
- *Expected result:* The ratings service holds exactly one rating for `racer@example.com` and `9780000000017`, with 3 or 5 stars.

**[TC02]: Simultaneous gives from two buyers leave two ratings — Negative / boundary:**
- *Preconditions:* `racer1@example.com` and `racer2@example.com` each hold a qualifying order for ISBN `9780000000017` and no rating of it.
- *Steps:* 1. Send a give for `racer1@example.com` and a give for `racer2@example.com`, both for `9780000000017` and 4 stars, at the same moment.
- *Expected result:* The ratings service holds exactly one rating for each of the two e-mails for `9780000000017`.

**[TC03]: Many simultaneous gives from one buyer leave one rating — State / lifecycle:**
- *Preconditions:* `burst@example.com` holds a qualifying order for ISBN `9780000000017` and no rating of it.
- *Steps:* 1. Send ten gives for `burst@example.com` and `9780000000017`, with stars from 1 to 5, at the same moment.
- *Expected result:* The ratings service holds exactly one rating for `burst@example.com` and `9780000000017`.

#### [AC04]: An empty comment on a change keeps the stored comment

When a change with an empty or absent comment is saved, the ratings service shall keep the rating's stored comment.

##### Test cases

**[TC01]: Changing the stars without a comment keeps the comment — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it with the comment "Great plot".
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 4 stars and leave the comment field empty. 3. Select Submit.
- *Expected result:* The rating's stored comment is still "Great plot".

**[TC02]: A change sent with no comment field keeps the comment — Negative / boundary:**
- *Preconditions:* `direct@example.com` holds a qualifying order for ISBN `9780000000024` and a 5-star rating of it with the comment "Slow start".
- *Steps:* 1. Send a change of that rating to 4 stars with no comment field to the ratings write interface.
- *Expected result:* The rating's stored comment is still "Slow start".

**[TC03]: A repeat give from the rating form with no comment keeps the comment — State / lifecycle:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it with the comment "Great plot".
- *Steps:* 1. Open the rating form. 2. Enter `reader@example.com` and `9780000000017`, choose 4 stars and leave the comment field empty. 3. Select Submit.
- *Expected result:* The rating's stored comment is still "Great plot".

##### Open questions

- [ ] Re-review finding (MINOR): does a comment of spaces only count as empty on a change, keeping the stored comment under [AC04], or as a comment that replaces it under [AC05]? Read as a comment, it lets a customer clear a stored comment, which Scope rules out.

#### [AC05]: A comment on a change replaces the stored comment

When a change carrying a comment is saved, the ratings service shall replace the rating's stored comment with that comment.

##### Test cases

**[TC01]: A new comment through Update replaces the old one — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it with the comment "Great plot".
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 3 stars and enter the comment "Weaker second half". 3. Select Submit.
- *Expected result:* The rating's stored comment is "Weaker second half".

**[TC02]: A refused change leaves the stored comment — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017` with the comment "Good read", given while their order qualified; that order has since been cancelled.
- *Steps:* 1. Send a change of that rating to 5 stars with the comment "Changed my mind" to the ratings write interface.
- *Expected result:* The rating's stored comment is still "Good read".

**[TC03]: A repeat give from the rating form with a comment replaces it — State / lifecycle:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it with the comment "Great plot".
- *Steps:* 1. Open the rating form. 2. Enter `reader@example.com` and `9780000000017`, choose 3 stars and enter the comment "Weaker second half". 3. Select Submit.
- *Expected result:* The rating's stored comment is "Weaker second half".

#### [AC06]: A change updates Updated At and keeps Created At

When a change is saved, the ratings service shall set the rating's Updated At to the time of the change, leaving its Created At as the time the rating was first given.

##### Test cases

**[TC01]: A change through Update moves Updated At only — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it first given at time T1.
- *Steps:* 1. At a later time T2, change the rating's stars through Update. 2. Open the rating details screen for that rating.
- *Expected result:* The rating's Created At is T1 and its Updated At is T2.

**[TC02]: A refused change moves neither time — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017` given and last updated at time T1, while their order qualified; that order has since been cancelled.
- *Steps:* 1. At a later time T2, send a change of that rating to 5 stars to the ratings write interface. 2. Open the rating details screen for that rating.
- *Expected result:* The rating's Created At and Updated At are both still T1.

**[TC03]: A repeat give keeps Created At — State / lifecycle:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it first given at time T1.
- *Steps:* 1. At a later time T2, give a new rating for `reader@example.com` and `9780000000017` from the rating form. 2. Open the rating details screen for that rating.
- *Expected result:* The rating's Created At is T1 and its Updated At is T2.

##### Open questions

- [ ] Re-review finding (MINOR): does an identical repeated change move the rating's Updated At under [AC06]? [AD#4] says an identical repeat leaves the same values, and [AC01] [TC03] is titled as leaving the rating unchanged.

#### [AC07]: A change without a qualifying order is refused

If an otherwise valid change ([AC09], [AC10]) reaches the ratings service ([AD#4]) when orders reports no qualifying order for the rating's ISBN under the e-mail exactly as the change carries it, or under the stored rating's e-mail when the change carries none ([AD#2]), then the ratings service shall refuse it with the buyers-only refusal, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A change after the order was cancelled is refused — Happy path:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017`, given while their order qualified; that order has since been cancelled.
- *Steps:* 1. Send a change of that rating to 5 stars to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal, and the rating still has 3 stars.

**[TC02]: A change after the order was deleted is refused — Negative / boundary:**
- *Preconditions:* `gone@example.com` holds a 4-star rating of ISBN `9780000000017`, given while their order qualified; that order has since been deleted from the store.
- *Steps:* 1. Send a change of that rating to 1 star to the ratings write interface.
- *Expected result:* The ratings service answers with the buyers-only refusal, and the rating still has 4 stars.

#### [AC08]: The update form names the buyers-only rule

When the ratings service refuses a change from the update form with the buyers-only refusal ([AD#4]), the update form shall display exactly "Only customers who have bought this book can rate it."

##### Test cases

**[TC01]: A customer whose order was cancelled sees the rule — Happy path:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017`, given while their order qualified; that order has since been cancelled.
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 5 stars. 3. Select Submit.
- *Expected result:* The update form displays exactly "Only customers who have bought this book can rate it."

**[TC02]: A customer whose order was deleted sees the same text — Negative / boundary:**
- *Preconditions:* `gone@example.com` holds a rating of ISBN `9780000000017`, given while their order qualified; that order has since been deleted.
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 2 stars. 3. Select Submit.
- *Expected result:* The update form displays exactly "Only customers who have bought this book can rate it.", with no text naming the deletion.

#### [AC09]: A change is held to whole stars 1 to 5

If a change reaches the ratings service ([AD#4]) with no star value, or with a star value that is not a whole number from 1 to 5, then the ratings service shall refuse it as invalid with the reason "Invalid rating. Must be a whole number from 1 to 5.", leaving the rating exactly as it was.

##### Test cases

**[TC01]: A change to zero stars is refused — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 4-star rating of it.
- *Steps:* 1. Send a change of that rating to 0 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the change as invalid, and the rating still has 4 stars.

**[TC02]: A change with no star value is refused, not stored as zero — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 4-star rating of it.
- *Steps:* 1. Send a change of that rating with no star value to the ratings write interface.
- *Expected result:* The ratings service refuses the change as invalid, and the rating still has 4 stars.

**[TC03]: A change to six or to a fraction is refused — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 4-star rating of it.
- *Steps:* 1. Send a change of that rating to 6 stars. 2. Send a change of that rating to 2.5 stars.
- *Expected result:* The ratings service refuses both changes as invalid, and the rating still has 4 stars.

#### [AC10]: A change cannot move a rating

If a change reaches the ratings service ([AD#4]) carrying an e-mail different from the stored rating's, compared ignoring letter case, or an ISBN different from the stored rating's, then the ratings service shall refuse it as invalid, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A change to another book is refused — Happy path:**
- *Preconditions:* `reader@example.com` holds qualifying orders for ISBNs `9780000000017` and `9780000000024`, and a rating of `9780000000017`.
- *Steps:* 1. Send a change of that rating to 3 stars carrying ISBN `9780000000024` to the ratings write interface.
- *Expected result:* The ratings service refuses the change as invalid, and the rating is still for `9780000000017`.

**[TC02]: A change to another customer is refused — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it; `other@example.com` holds a qualifying order for the same ISBN.
- *Steps:* 1. Send a change of that rating to 3 stars carrying the e-mail `other@example.com`.
- *Expected result:* The ratings service refuses the change as invalid, and the rating is still for `reader@example.com`.

**[TC03]: A change that repeats or leaves out the e-mail and ISBN proceeds — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it.
- *Steps:* 1. Send a change of that rating to 3 stars carrying the same e-mail and ISBN. 2. Send a change of that rating to 4 stars carrying neither.
- *Expected result:* The ratings service refuses neither change as invalid, and the rating has 4 stars for `reader@example.com` and `9780000000017`.

#### [AC11]: A change keeps the stored e-mail as stored

When a change carrying the rating's e-mail in other capitals is saved, the ratings service shall keep the rating's e-mail exactly as stored ([AD#4]).

##### Test cases

**[TC01]: A change typed in other capitals keeps the stored e-mail — Happy path:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; the orders service holds a qualifying order for `9780000000017` under the spelling `Reader@Example.com` and none under `reader@example.com`.
- *Steps:* 1. Send a change of that rating to 4 stars carrying the e-mail `Reader@Example.com` to the ratings write interface.
- *Expected result:* The rating has 4 stars, and its e-mail is still `reader@example.com`.

**[TC02]: A repeat give typed in capitals keeps the stored e-mail — Negative / boundary:**
- *Preconditions:* The ratings service holds a 2-star rating of ISBN `9780000000017` stored under `reader@example.com`; the orders service holds a qualifying order for `9780000000017` under the spelling `READER@EXAMPLE.COM` and none under `reader@example.com`.
- *Steps:* 1. Send a give for `READER@EXAMPLE.COM`, `9780000000017` and 5 stars to the ratings write interface.
- *Expected result:* The rating has 5 stars, and its e-mail is still `reader@example.com`.

#### [AC12]: A change is refused while orders cannot answer

If orders does not answer the purchase check for an otherwise valid change within a bounded time, cannot be reached, or answers with an error other than not-found ([AD#2]), then the ratings service shall refuse the change as unavailable, leaving the rating exactly as it was.

##### Test cases

**[TC01]: A buyer's change sent to the ratings service is refused while orders is unreachable — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it; the orders service is unreachable from the ratings service.
- *Steps:* 1. Send a change of that rating to 4 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the change as unavailable, and the rating still has 2 stars.

**[TC02]: A repeat give while orders is unreachable leaves the rating — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it with the comment "Great plot"; the orders service is unreachable.
- *Steps:* 1. Send a give for `reader@example.com`, `9780000000017`, 5 stars and the comment "Even better" to the ratings write interface.
- *Expected result:* The ratings service refuses the give as unavailable, and the rating still has 2 stars and the comment "Great plot".

**[TC03]: A change is refused in bounded time while orders never answers — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it; the orders service accepts connections from the ratings service but never answers.
- *Steps:* 1. Send a change of that rating to 4 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the change as unavailable within its bounded time, and the rating still has 2 stars.

**[TC04]: An orders server error is not read as not bought — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017` whose order has since been cancelled; the orders service answers every request with a server error.
- *Steps:* 1. Send a change of that rating to 5 stars to the ratings write interface.
- *Expected result:* The ratings service refuses the change as unavailable, not with the buyers-only refusal, and the rating still has 3 stars.

#### [AC13]: The update form says the purchase check is unavailable

When the ratings service refuses a change from the update form as unavailable ([AD#4]), the update form shall display exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

##### Test cases

**[TC01]: A buyer sees the unavailable text on Update — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it; the orders service is unreachable from the ratings service.
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 5 stars. 3. Select Submit.
- *Expected result:* The update form displays exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

**[TC02]: A customer whose order was cancelled sees the unavailable text, not the rule — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017` whose order has since been cancelled; the orders service is unreachable from the ratings service.
- *Steps:* 1. Open the Ratings list and select Update on that rating. 2. Choose 5 stars. 3. Select Submit.
- *Expected result:* The update form displays exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." and not "Only customers who have bought this book can rate it."

#### [AC14]: The update form says the rating no longer exists

If a change from the update form reaches the ratings service for a rating that no longer exists ([AD#4]), then the update form shall display exactly "This rating no longer exists. Nothing was saved."

##### Test cases

**[TC01]: The rating is removed while the update form is open — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it.
- *Steps:* 1. Open the update form for that rating. 2. Remove that rating through the ratings service's per-rating removal ([AD#5]). 3. Choose 4 stars and select Submit.
- *Expected result:* The update form displays exactly "This rating no longer exists. Nothing was saved."

**[TC02]: The change does not bring the removed rating back — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it.
- *Steps:* 1. Open the update form for that rating. 2. Remove that rating through the per-rating removal. 3. Choose 4 stars and select Submit. 4. Read the ratings of `reader@example.com` from the ratings service.
- *Expected result:* The update form displays exactly "This rating no longer exists. Nothing was saved.", and the ratings service holds no rating of `9780000000017` for `reader@example.com`.

#### [AC15]: The update form opened for a removed rating says so

When the update form opens for a rating that no longer exists ([AD#7]), the update form shall display exactly "This rating no longer exists." in place of its fields.

##### Test cases

**[TC01]: Update selected on a stale Ratings list — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list. 2. Remove that rating through the ratings service's per-rating removal ([AD#5]), leaving the list as loaded. 3. Select Update on that rating.
- *Expected result:* The update form displays exactly "This rating no longer exists." and no e-mail, ISBN, star or comment field.

**[TC02]: An existing rating opens with its fields — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list. 2. Select Update on that rating.
- *Expected result:* The update form shows the rating's fields and not "This rating no longer exists."

#### [AC16]: The update form shows an invalid change's reason

When the ratings service refuses a change from the update form as invalid ([AD#4]), the update form shall display the reason the ratings service gives.

##### Test cases

**[TC01]: A star reason is shown — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; the ratings service the storefront calls refuses every change as invalid with the reason "Invalid rating. Must be a whole number from 1 to 5."
- *Steps:* 1. Open the update form for that rating. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form displays "Invalid rating. Must be a whole number from 1 to 5."

**[TC02]: Another reason is shown as given, not as the buyers-only text — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; the ratings service the storefront calls refuses every change as invalid with the reason "ISBN must be a 13-digits value".
- *Steps:* 1. Open the update form for that rating. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form displays "ISBN must be a 13-digits value" and not "Only customers who have bought this book can rate it."

#### [AC17]: The update form says when a change cannot be saved

If the update form cannot reach the ratings service, or the ratings service answers a change with anything [AC08], [AC13], [AC14], [AC16] and [AC20] do not cover ([AD#4]), then the update form shall display exactly "We can't save your rating right now. Please try again in a few minutes."

##### Test cases

**[TC01]: The ratings service becomes unreachable — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it.
- *Steps:* 1. Open the update form for that rating. 2. Make the ratings service unreachable from the storefront. 3. Choose 4 stars and select Submit.
- *Expected result:* The update form displays exactly "We can't save your rating right now. Please try again in a few minutes."

**[TC02]: An unexpected server error is not shown as a refusal — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; the ratings service answers every change with an unexpected server error.
- *Steps:* 1. Open the update form for that rating. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form displays exactly "We can't save your rating right now. Please try again in a few minutes.", and neither "Only customers who have bought this book can rate it." nor the purchase-check text of [AC13].

#### [AC18]: The update form shows the rating it changes

When the update form opens and loads a rating ([AD#7]), the update form shall show the heading "Update Rating", the rating's e-mail and ISBN as values that cannot be edited, and the five star choices 1 to 5 with the rating's current stars chosen.

##### Test cases

**[TC01]: The update form opens on the rating's values — Happy path:**
- *Preconditions:* `reader@example.com` holds a 3-star rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list and select Update on that rating.
- *Expected result:* The update form shows the heading "Update Rating", `reader@example.com` and `9780000000017`, and the star choices 1 to 5 with 3 chosen.

**[TC02]: The e-mail and ISBN cannot be edited — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the update form for that rating. 2. Try to change the e-mail and the ISBN.
- *Expected result:* The update form offers no way to change the e-mail or the ISBN.

**[TC03]: No value outside the scale can be chosen on the update form — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the update form for that rating. 2. Try to choose or enter 0, 6 and 3.5 as the star value.
- *Expected result:* The update form offers no way to choose any value other than 1, 2, 3, 4 or 5.

#### [AC19]: The update form keeps what was entered after a refusal

When the ratings service refuses a change from the update form ([AD#4]), or the update form cannot save it ([AC17]), the update form shall stay open with the star choice and comment as entered.

##### Test cases

**[TC01]: A buyers-only refusal keeps the entries — Happy path:**
- *Preconditions:* `lapsed@example.com` holds a 3-star rating of ISBN `9780000000017` whose order has since been cancelled.
- *Steps:* 1. Open the update form for that rating. 2. Choose 5 stars and enter the comment "Changed my mind". 3. Select Submit.
- *Expected result:* The update form stays open showing 5 stars and "Changed my mind".

**[TC02]: An unavailable refusal keeps the entries — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it; the orders service is unreachable from the ratings service.
- *Steps:* 1. Open the update form for that rating. 2. Choose 1 star and enter the comment "Not for me". 3. Select Submit.
- *Expected result:* The update form stays open showing 1 star and "Not for me", and the Ratings list is not opened.

**[TC03]: A save that cannot reach the ratings service keeps the entries — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it.
- *Steps:* 1. Open the update form for that rating. 2. Make the ratings service unreachable from the storefront. 3. Choose 2 stars, enter the comment "On reflection" and select Submit.
- *Expected result:* The update form stays open showing 2 stars and "On reflection".

#### [AC20]: A saved change opens the Ratings list

When the ratings service saves a change from the update form ([AD#4]), the update form shall open the Ratings list.

##### Test cases

**[TC01]: A saved change opens the Ratings list — Happy path:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a 2-star rating of it.
- *Steps:* 1. Open the update form for that rating. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form opens the Ratings list, which shows the rating of `reader@example.com` for `9780000000017` with 4 stars.

**[TC02]: A refused change does not open the Ratings list — Negative / boundary:**
- *Preconditions:* `lapsed@example.com` holds a rating of ISBN `9780000000017` whose order has since been cancelled.
- *Steps:* 1. Open the update form for that rating. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form does not open the Ratings list.

#### [AC21]: A rating outlives its order's cancellation

While a rating's qualifying order has been cancelled or deleted since the rating was last given or changed, the ratings service shall count the rating in its book's rating summary ([AD#6]).

##### Test cases

**[TC01]: A rating still counts after its order is cancelled — Happy path:**
- *Preconditions:* ISBN `9780000000031` holds one rating, 4 stars from `keeper@example.com`, given while their order qualified.
- *Steps:* 1. Cancel the order of `keeper@example.com` for `9780000000031`. 2. Load the book page of `9780000000031`.
- *Expected result:* The rating of `keeper@example.com` still counts: the book page shows "★ 4.0 (1 rating)".

**[TC02]: A rating still counts after its order is deleted — Negative / boundary:**
- *Preconditions:* ISBN `9780000000048` holds two ratings, 5 stars from `keeper2@example.com` and 3 stars from another buyer, each given while its order qualified.
- *Steps:* 1. Delete the order of `keeper2@example.com` for `9780000000048`. 2. Ask the ratings service for the rating summary of `9780000000048` ([AD#6]).
- *Expected result:* The rating of `keeper2@example.com` still counts: the summary has the count 2 and the average 4.0.

**[TC03]: A refused change leaves the rating counting as it was — State / lifecycle:**
- *Preconditions:* ISBN `9780000000055` holds one rating, 2 stars from `lapsed2@example.com`, whose order has since been cancelled.
- *Steps:* 1. Send a change of that rating to 5 stars. 2. Load the book page of `9780000000055`.
- *Expected result:* The rating still counts with its 2 stars: the book page shows "★ 2.0 (1 rating)".

#### [AC22]: The Ratings list keeps the rater and Update

The Ratings list shall show each rating's rater e-mail with an Update action that opens the update form for that rating.

##### Test cases

**[TC01]: A rating shows its e-mail and Update — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list. 2. Select Update on the rating of `reader@example.com`.
- *Expected result:* The Ratings list shows `reader@example.com` on that rating, and Update opens the update form for that rating.

**[TC02]: Each rating's Update opens its own rating — Negative / boundary:**
- *Preconditions:* `reader@example.com` and `second@example.com` each hold a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list. 2. Select Update on the rating of `second@example.com`.
- *Expected result:* The update form opens on the rating of `second@example.com`, not on that of `reader@example.com`.

#### [AC23]: The update form says when the rating cannot be loaded

If the update form cannot load the rating it opens for, because the ratings service cannot be reached or answers with anything other than the rating or not-found ([AD#7]), then the update form shall display exactly "We can't load this rating right now. Please try again in a few minutes." in place of its fields.

##### Test cases

**[TC01]: The ratings service is unreachable when Update is selected — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list. 2. Make the ratings service unreachable from the storefront. 3. Select Update on that rating.
- *Expected result:* The update form displays exactly "We can't load this rating right now. Please try again in a few minutes." and no e-mail, ISBN, star or comment field.

**[TC02]: An unexpected server error on loading is not shown as a removed rating — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`; the ratings service answers every read of a single rating with an unexpected server error.
- *Steps:* 1. Open the Ratings list. 2. Select Update on that rating.
- *Expected result:* The update form displays exactly "We can't load this rating right now. Please try again in a few minutes." and not "This rating no longer exists."

---

### [U03]: See how a book is rated on its book page

As a shopper looking at a book, I want to see its average rating and how many ratings it rests on, so that I can tell a broad opinion from a single one.

Done when the book page shows a rated book's average and count, and "Not yet rated" for a book with none.

Traces to: [US#3]; [AC#13]–[AC#16]; [VD#3]; [CG#6], [CG#7], [CG#8].

#### [AC01]: The book page shows the average rating

When the book page loads a book that holds at least one rating ([AD#6]), the book page shall show, as the book's rating line, "★" followed by the book's average rating, rounded half up to one decimal place and always shown with one decimal.

##### Test cases

**[TC01]: An average of 4.25 shows as 4.3 — Happy path:**
- *Preconditions:* ISBN `9780000000062` holds four ratings, of 5, 4, 4 and 4 stars.
- *Steps:* 1. Load the book page of `9780000000062`.
- *Expected result:* The book page shows "★ 4.3".

**[TC02]: A whole average shows one decimal — Negative / boundary:**
- *Preconditions:* ISBN `9780000000079` holds three ratings, each of 4 stars.
- *Steps:* 1. Load the book page of `9780000000079`.
- *Expected result:* The book page shows "★ 4.0".

**[TC03]: An average of exactly 1.05 rounds up — Negative / boundary:**
- *Preconditions:* ISBN `9780000000086` holds twenty ratings: nineteen of 1 star and one of 2 stars.
- *Steps:* 1. Load the book page of `9780000000086`.
- *Expected result:* The book page shows "★ 1.1".

#### [AC02]: The book page shows how many ratings the average rests on

When the book page shows a book's average rating ([AD#6]), the book page shall show beside it, on the rating line, the number of ratings the average is computed from, as "(1 rating)" for one rating and, for any other number, the number followed by "ratings" in parentheses, such as "(37 ratings)".

##### Test cases

**[TC01]: Thirty-seven ratings show their count — Happy path:**
- *Preconditions:* ISBN `9780000000093` holds thirty-seven ratings, each of 4 stars.
- *Steps:* 1. Load the book page of `9780000000093`.
- *Expected result:* The book page shows "★ 4.0 (37 ratings)".

**[TC02]: One rating is worded in the singular — Negative / boundary:**
- *Preconditions:* ISBN `9780000000109` holds one rating, of 5 stars.
- *Steps:* 1. Load the book page of `9780000000109`.
- *Expected result:* The book page shows "★ 5.0 (1 rating)".

**[TC03]: Two ratings are worded in the plural — Negative / boundary:**
- *Preconditions:* ISBN `9780000000116` holds two ratings, of 3 and 4 stars.
- *Steps:* 1. Load the book page of `9780000000116`.
- *Expected result:* The book page shows "★ 3.5 (2 ratings)".

#### [AC03]: An unrated book shows "Not yet rated"

When the book page loads a book that holds no rating ([AD#6]), the book page shall show "Not yet rated" in place of the rating line, with no count.

##### Test cases

**[TC01]: A book nobody has rated — Happy path:**
- *Preconditions:* ISBN `9780000000123` names a book that holds no rating.
- *Steps:* 1. Load the book page of `9780000000123`.
- *Expected result:* The book page shows "Not yet rated" and no count.

**[TC02]: A book whose only rating was removed — Negative / boundary:**
- *Preconditions:* ISBN `9780000000130` holds one rating.
- *Steps:* 1. Remove that rating through the ratings service's per-rating removal ([AD#5]). 2. Load the book page of `9780000000130`.
- *Expected result:* The book page shows "Not yet rated" and not "(0 ratings)".

#### [AC04]: The book page reflects each give, change and removal on its next load

When a rating of a book has been given, changed or removed since the book page last loaded, the book page shall show the average and the count including that give, change or removal the next time it loads ([AD#6]).

##### Test cases

**[TC01]: A new rating counts on the next load — Happy path:**
- *Preconditions:* ISBN `9780000000147` holds one rating, of 4 stars; `newbuyer@example.com` holds a qualifying order for it and no rating of it.
- *Steps:* 1. Load the book page of `9780000000147`. 2. Give a 2-star rating for `newbuyer@example.com` and `9780000000147`. 3. Load the book page again.
- *Expected result:* The book page shows "★ 3.0 (2 ratings)".

**[TC02]: A changed rating counts on the next load — State / lifecycle:**
- *Preconditions:* ISBN `9780000000154` holds one rating, 2 stars from `reader8@example.com`, who holds a qualifying order for it.
- *Steps:* 1. Load the book page of `9780000000154`. 2. Change that rating to 5 stars through Update. 3. Load the book page again.
- *Expected result:* The book page shows "★ 5.0 (1 rating)".

**[TC03]: A removed rating drops out on the next load — State / lifecycle:**
- *Preconditions:* ISBN `9780000000161` holds two ratings, of 5 and 1 stars.
- *Steps:* 1. Load the book page of `9780000000161`. 2. Remove the 1-star rating through the ratings service's per-rating removal. 3. Load the book page again.
- *Expected result:* The book page shows "★ 5.0 (1 rating)".

**[TC04]: A refused give changes nothing — Negative / boundary:**
- *Preconditions:* ISBN `9780000000178` holds one rating, of 4 stars; `browser@example.com` is a customer with no order for it.
- *Steps:* 1. Send a give for `browser@example.com`, `9780000000178` and 1 star. 2. Load the book page of `9780000000178`.
- *Expected result:* The book page shows "★ 4.0 (1 rating)".

#### [AC05]: A rating that cannot be loaded shows "Rating unavailable"

If the book's rating summary cannot be loaded ([AD#6]), then the book page shall show "Rating unavailable" in place of the rating line, with the book's other details shown as usual.

##### Test cases

**[TC01]: The ratings service is unreachable — Happy path:**
- *Preconditions:* ISBN `9780000000062` names a book holding ratings; the ratings service is unreachable from the storefront.
- *Steps:* 1. Load the book page of `9780000000062`.
- *Expected result:* The book page shows "Rating unavailable" in place of the rating line, and shows the book's ISBN, title, author, price and language.

**[TC02]: An unrated book is not called unrated when the load fails — Negative / boundary:**
- *Preconditions:* ISBN `9780000000123` names a book that holds no rating; the ratings service is unreachable from the storefront.
- *Steps:* 1. Load the book page of `9780000000123`.
- *Expected result:* The book page shows "Rating unavailable" and not "Not yet rated".

#### [AC06]: The ratings service answers a book's rating summary

When the ratings service is asked for one book's rating summary ([AD#6]), the ratings service shall answer the number of that book's stored ratings and their average, rounded half up to one decimal place from the exact quotient of the star sum by the count, with no average when the number is 0.

##### Test cases

**[TC01]: A rated book's summary — Happy path:**
- *Preconditions:* ISBN `9780000000062` holds four ratings, of 5, 4, 4 and 4 stars.
- *Steps:* 1. Ask the ratings service for the rating summary of `9780000000062`.
- *Expected result:* The ratings service answers the count 4 and the average 4.3.

**[TC02]: An unrated ISBN's summary has no average — Negative / boundary:**
- *Preconditions:* ISBN `9780000000123` holds no rating.
- *Steps:* 1. Ask the ratings service for the rating summary of `9780000000123`.
- *Expected result:* The ratings service answers the count 0 and no average.

**[TC03]: An exact 1.05 rounds up in the summary — Negative / boundary:**
- *Preconditions:* ISBN `9780000000086` holds nineteen ratings of 1 star and one of 2 stars.
- *Steps:* 1. Ask the ratings service for the rating summary of `9780000000086`.
- *Expected result:* The ratings service answers the count 20 and the average 1.1.

#### [AC07]: A summary request with no ISBN is refused

If a request for one book's rating summary carries no ISBN or a blank one ([AD#6]), then the ratings service shall refuse it as invalid.

##### Test cases

**[TC01]: A summary request with no ISBN — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Ask the ratings service for a rating summary with no ISBN.
- *Expected result:* The ratings service refuses the request as invalid.

**[TC02]: A summary request with a blank ISBN — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Ask the ratings service for a rating summary with an ISBN of spaces only.
- *Expected result:* The ratings service refuses the request as invalid rather than answering a count of 0.

#### [AC08]: Rating summaries name no rater

The ratings service shall leave every rater's e-mail out of each rating summary it answers ([AD#6]).

##### Test cases

**[TC01]: One book's summary carries no e-mail — Happy path:**
- *Preconditions:* ISBN `9780000000116` holds ratings from `reader@example.com` and `second@example.com`.
- *Steps:* 1. Ask the ratings service for the rating summary of `9780000000116`. 2. Read the whole response.
- *Expected result:* The response carries the ISBN, the count and the average, and no e-mail.

**[TC02]: A single-rating book's summary carries no e-mail — Negative / boundary:**
- *Preconditions:* ISBN `9780000000109` holds one rating, from `solo@example.com`.
- *Steps:* 1. Ask the ratings service for the rating summary of `9780000000109`. 2. Read the whole response.
- *Expected result:* The response does not carry `solo@example.com` or any other e-mail.

**[TC03]: The summaries of all books carry no e-mail — Security / privacy:**
- *Preconditions:* At least three books hold ratings from at least three different e-mails.
- *Steps:* 1. Ask the ratings service for the rating summaries of all books. 2. Read the whole response.
- *Expected result:* The response carries no e-mail.

---

### [U04]: Compare books by rating on the catalogue list

As a shopper browsing the catalogue list, I want to see each book's average rating, so that I can tell which books buyers rate highly before I open one.

Done when every book on the catalogue list shows the same average its book page shows, or "Not yet rated".

Traces to: [US#4]; [AC#17]–[AC#20]; [VD#3]; [CG#8], [CG#9].

#### [AC01]: The catalogue list shows each book's average

When the catalogue list loads ([AD#6]), the catalogue list shall show, in a "Rating" column after "OnVend", "★" followed by the average rating of each book that holds at least one rating, rounded half up to one decimal place, always shown with one decimal and with no rating count.

##### Test cases

**[TC01]: A rated book shows its average in the Rating column — Happy path:**
- *Preconditions:* ISBN `9780000000062` holds four ratings, of 5, 4, 4 and 4 stars.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.3" in the "Rating" column, after "OnVend", on the row of `9780000000062`.

**[TC02]: A whole average shows one decimal — Negative / boundary:**
- *Preconditions:* ISBN `9780000000109` holds one rating, of 5 stars.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 5.0" on the row of `9780000000109`.

**[TC03]: The list shows the average without a count — Negative / boundary:**
- *Preconditions:* ISBN `9780000000093` holds thirty-seven ratings, each of 4 stars.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.0" on the row of `9780000000093`, with no rating count.

#### [AC02]: An unrated book shows "Not yet rated" on the list

When the catalogue list loads ([AD#6]), the catalogue list shall show "Not yet rated" in the Rating column of each book that holds no rating.

##### Test cases

**[TC01]: An unrated book on the list — Happy path:**
- *Preconditions:* ISBN `9780000000123` names a book that holds no rating.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Not yet rated" on the row of `9780000000123`.

**[TC02]: Rated and unrated books side by side — Negative / boundary:**
- *Preconditions:* ISBN `9780000000062` holds ratings averaging 4.25; ISBN `9780000000123` holds none.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.3" on the row of `9780000000062` and "Not yet rated" on the row of `9780000000123`.

#### [AC03]: The list and the book page agree

While the catalogue list shows a book's average rating, the catalogue list shall show the same value the book's page shows at that moment ([AD#6]).

##### Test cases

**[TC01]: The list matches the book page — Happy path:**
- *Preconditions:* ISBN `9780000000062` holds four ratings, of 5, 4, 4 and 4 stars.
- *Steps:* 1. Open the catalogue list. 2. Open the book page of `9780000000062` with no rating given, changed or removed in between.
- *Expected result:* The catalogue list and the book page both show "★ 4.3" for `9780000000062`.

**[TC02]: The list matches the book page on an exact 1.05 — Negative / boundary:**
- *Preconditions:* ISBN `9780000000086` holds nineteen ratings of 1 star and one of 2 stars.
- *Steps:* 1. Open the catalogue list. 2. Open the book page of `9780000000086` with no rating given, changed or removed in between.
- *Expected result:* The catalogue list and the book page both show "★ 1.1" for `9780000000086`.

#### [AC04]: The catalogue list reflects each give, change and removal on its next load

When a rating of a book has been given, changed or removed since the catalogue list last loaded, the catalogue list shall show the book's average including that give, change or removal the next time it loads ([AD#6]).

##### Test cases

**[TC01]: A new rating counts on the next load — Happy path:**
- *Preconditions:* ISBN `9780000000123` holds no rating; `firstbuyer@example.com` holds a qualifying order for it.
- *Steps:* 1. Open the catalogue list. 2. Give a 4-star rating for `firstbuyer@example.com` and `9780000000123`. 3. Open the catalogue list again.
- *Expected result:* The catalogue list shows "★ 4.0" on the row of `9780000000123`.

**[TC02]: A removed rating drops out on the next load — State / lifecycle:**
- *Preconditions:* ISBN `9780000000130` holds one rating.
- *Steps:* 1. Open the catalogue list. 2. Remove that rating through the ratings service's per-rating removal ([AD#5]). 3. Open the catalogue list again.
- *Expected result:* The catalogue list shows "Not yet rated" on the row of `9780000000130`.

**[TC03]: A refused give changes nothing — Negative / boundary:**
- *Preconditions:* ISBN `9780000000178` holds one rating, of 4 stars; `browser@example.com` is a customer with no order for it.
- *Steps:* 1. Send a give for `browser@example.com`, `9780000000178` and 1 star. 2. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.0" on the row of `9780000000178`.

#### [AC05]: Ratings that cannot be loaded show "Rating unavailable" on the list

If the books' rating summaries cannot be loaded ([AD#6]), then the catalogue list shall show "Rating unavailable" in the Rating column of every book, with the books listed as usual.

##### Test cases

**[TC01]: The ratings service is unreachable — Happy path:**
- *Preconditions:* The catalogue holds rated and unrated books; the ratings service is unreachable from the storefront.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list lists every book with its usual columns and shows "Rating unavailable" in the Rating column of each.

**[TC02]: An unrated book is not called unrated when the load fails — Negative / boundary:**
- *Preconditions:* ISBN `9780000000123` names a book that holds no rating; the ratings service is unreachable from the storefront.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Rating unavailable", and not "Not yet rated", on the row of `9780000000123`.

#### [AC06]: The ratings service answers every rated book's summary

When the ratings service is asked for the rating summaries of all books ([AD#6]), the ratings service shall answer one summary, as [U03] [AC06] defines it, for every book that holds at least one rating.

##### Test cases

**[TC01]: Two rated books and one unrated — Happy path:**
- *Preconditions:* The ratings store holds ratings for ISBNs `9780000000062` (5, 4, 4 and 4 stars) and `9780000000109` (5 stars) only; ISBN `9780000000123` names a book with no rating.
- *Steps:* 1. Ask the ratings service for the rating summaries of all books.
- *Expected result:* The ratings service answers exactly two summaries: `9780000000062` with the count 4 and the average 4.3, and `9780000000109` with the count 1 and the average 5.0.

**[TC02]: No ratings at all — Negative / boundary:**
- *Preconditions:* The ratings store holds no rating.
- *Steps:* 1. Ask the ratings service for the rating summaries of all books.
- *Expected result:* The ratings service answers no summary.

---

### [U05]: Know that only Quillmark sees my comment

As a customer who rates a book, I want to know that only Quillmark sees my comment, so that I am not misled into thinking I am publishing a review.

Done when both rating forms carry the note, the comment is stored, and no shopper-reachable screen or ratings-service response shows it.

Traces to: [US#7]; [AC#5], [AC#6], [AC#12], [AC#28]; [CD#17]; [CG#10].

#### [AC01]: The forms say only Quillmark sees the comment

The rating form, and the update form once it has loaded a rating, shall each show at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

##### Test cases

**[TC01]: The rating form carries the note — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating.
- *Expected result:* The rating form shows at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

**[TC02]: The update form carries the same note — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list and select Update on that rating.
- *Expected result:* The update form shows at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

**[TC03]: The note stays while a refusal keeps the form open — Negative / boundary:**
- *Preconditions:* `browser@example.com` is a customer with no order for ISBN `9780000000017`.
- *Steps:* 1. Open the rating form. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The rating form, open with the refusal shown, still shows at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

#### [AC02]: The Ratings list shows no comment

The Ratings list shall show for each rating its e-mail ([U02] [AC22]), ISBN, stars and Updated At, with no comment ([AD#7]).

##### Test cases

**[TC01]: The Ratings list shows a rating's fields without its comment — Happy path:**
- *Preconditions:* `reader@example.com` holds a 4-star rating of ISBN `9780000000017` with the comment "Great plot".
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The Ratings list shows `reader@example.com`, `9780000000017`, 4 stars and Updated At for that rating, and no comment column.

**[TC02]: No comment text appears anywhere on the list — Security / privacy:**
- *Preconditions:* Three ratings hold the comments "Great plot", "Slow start" and "Gift for my mother".
- *Steps:* 1. Open the Ratings list. 2. Search the whole screen for the three comments.
- *Expected result:* The Ratings list shows none of the three comments.

**[TC03]: A rating with no comment shows the same columns — Negative / boundary:**
- *Preconditions:* `second@example.com` holds a 3-star rating of ISBN `9780000000017` with no comment.
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The Ratings list shows `second@example.com`, `9780000000017`, 3 stars and Updated At for that rating, and no comment column.

#### [AC03]: The rating details screen shows no comment

When the rating details screen opens and loads a rating, the rating details screen shall show its ISBN, e-mail, stars, Updated At and Created At, with no comment ([AD#7]).

##### Test cases

**[TC01]: The details of a rating with a comment — Happy path:**
- *Preconditions:* `reader@example.com` holds a 4-star rating of ISBN `9780000000017` with the comment "Great plot".
- *Steps:* 1. Open the Ratings list and select Details on that rating.
- *Expected result:* The rating details screen shows `9780000000017`, `reader@example.com`, 4 stars, Updated At and Created At, and not "Great plot".

**[TC02]: No comment label appears on the details screen — Negative / boundary:**
- *Preconditions:* `second@example.com` holds a rating of ISBN `9780000000017` with no comment.
- *Steps:* 1. Open the Ratings list and select Details on that rating.
- *Expected result:* The rating details screen shows no comment label or value.

#### [AC04]: The update form does not show the stored comment

When the update form opens and loads a rating, the update form shall show an empty comment field, whatever comment the rating holds.

##### Test cases

**[TC01]: A rating with a comment opens with an empty comment field — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017` with the comment "Great plot".
- *Steps:* 1. Open the Ratings list and select Update on that rating.
- *Expected result:* The update form shows an empty comment field, and not "Great plot".

**[TC02]: A rating with no comment opens with an empty comment field — Negative / boundary:**
- *Preconditions:* `second@example.com` holds a rating of ISBN `9780000000017` with no comment.
- *Steps:* 1. Open the Ratings list and select Update on that rating.
- *Expected result:* The update form shows an empty comment field.

#### [AC05]: No response of the ratings service carries a comment

The ratings service shall leave the comment out of every response it gives, its answers to a give and to a change included ([AD#7]).

##### Test cases

**[TC01]: Reading a rating returns no comment — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017` with the comment "Great plot".
- *Steps:* 1. Read that rating from the ratings service by its id. 2. Read the ratings of `reader@example.com`. 3. Read the ratings of `9780000000017`. 4. Read every rating. 5. Ask for the rating summary of `9780000000017` ([AD#6]). 6. Ask for the rating summaries of all books.
- *Expected result:* None of the six responses carries "Great plot" or any comment.

**[TC02]: The answer to a give carries no comment — Negative / boundary:**
- *Preconditions:* `commenter@example.com` holds a qualifying order for ISBN `9780000000017` and no rating of it.
- *Steps:* 1. Send a give for `commenter@example.com`, `9780000000017`, 4 stars and the comment "Loved the ending" to the ratings write interface.
- *Expected result:* The ratings service's answer to the give carries no comment.

**[TC03]: The answer to a change that keeps the stored comment does not return it — Security / privacy:**
- *Preconditions:* `reader@example.com` holds a qualifying order for ISBN `9780000000017` and a rating of it with the comment "Great plot".
- *Steps:* 1. Send a change of that rating to 5 stars with no comment to the ratings write interface.
- *Expected result:* The ratings service's answer to the change carries no comment, and not "Great plot".

#### [AC06]: A comment given with a rating is stored

When a give carrying a comment is stored, the ratings service shall store the comment with the rating.

##### Test cases

**[TC01]: A comment given through the rating form is kept — Happy path:**
- *Preconditions:* `commenter2@example.com` holds a qualifying order for ISBN `9780000000017` and no rating of it.
- *Steps:* 1. Give a 4-star rating for `commenter2@example.com` and `9780000000017` with the comment "Loved the ending" through the rating form. 2. Read the stored rating in the ratings store, where Quillmark reads comments.
- *Expected result:* The stored rating holds the comment "Loved the ending".

**[TC02]: A give with no comment is stored without one — Negative / boundary:**
- *Preconditions:* `quiet@example.com` holds a qualifying order for ISBN `9780000000017` and no rating of it.
- *Steps:* 1. Give a 3-star rating for `quiet@example.com` and `9780000000017` with no comment. 2. Read the stored rating in the ratings store.
- *Expected result:* The stored rating holds 3 stars and no comment.

---

### [U06]: Remove a single rating for an erasure request or fraud

As a Quillmark operator, I want to remove a single rating, so that I can meet a customer's erasure request or clean up a fraudulent rating.

Done when an operator can remove any one rating, and no shopper screen and no request removes ratings in bulk.

Traces to: [US#5]; [AC#21], [AC#23]; [VD#1]; [CG#4].

#### [AC01]: An operator removes one rating

When the ratings service is asked to remove a rating by its id ([AD#5]), the ratings service shall remove that rating and no other.

##### Test cases

**[TC01]: One rating is removed and the others stay — Happy path:**
- *Preconditions:* ISBN `9780000000017` holds ratings from `reader@example.com` and `second@example.com`.
- *Steps:* 1. Read the ratings of `reader@example.com` from the ratings service to find the rating's id. 2. Ask the ratings service to remove the rating with that id.
- *Expected result:* The rating of `reader@example.com` for `9780000000017` is gone, and the rating of `second@example.com` remains.

**[TC02]: Every rating of one customer is removed for an erasure request — State / lifecycle:**
- *Preconditions:* `erase@example.com` holds ratings of three books.
- *Steps:* 1. Read the ratings of `erase@example.com` from the ratings service. 2. Ask the ratings service to remove each of the three ratings by its id. 3. Read the ratings of `erase@example.com` again.
- *Expected result:* The ratings service holds no rating for `erase@example.com`.

**[TC03]: Removing the same id twice removes nothing more — Negative / boundary:**
- *Preconditions:* ISBN `9780000000017` holds ratings from `reader@example.com` and `second@example.com`.
- *Steps:* 1. Remove the rating of `reader@example.com` by its id. 2. Ask to remove the same id again.
- *Expected result:* The rating of `second@example.com` remains after both removals.

#### [AC02]: Removing an id that holds no rating removes nothing

If the ratings service is asked to remove a rating by an id that holds no rating ([AD#5]), then the ratings service shall answer as it answers a removal that succeeded, removing nothing.

##### Test cases

**[TC01]: An unused id — Happy path:**
- *Preconditions:* The ratings store holds two ratings; the id 999999 holds no rating.
- *Steps:* 1. Ask the ratings service to remove the rating with the id 999999. 2. Ask the ratings service to remove one of the two existing ratings by its id.
- *Expected result:* The ratings service answers both removals alike, and one rating remains.

**[TC02]: A repeated removal is answered alike — Negative / boundary:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Remove that rating by its id. 2. Ask to remove the same id again.
- *Expected result:* The ratings service answers the second removal as it answered the first.

#### [AC03]: No request removes every rating at once

If a request asks the ratings service to remove every rating at once ([AD#5]), then the ratings service shall refuse it as invalid, removing no rating.

##### Test cases

**[TC01]: The former remove-all request removes nothing — Happy path:**
- *Preconditions:* The ratings store holds five ratings.
- *Steps:* 1. Send the ratings service the request that removed every rating before this release ([AD#5]).
- *Expected result:* The ratings service refuses the request as invalid, and all five ratings remain.

**[TC02]: A removal with an id that is not a number removes nothing — Negative / boundary:**
- *Preconditions:* The ratings store holds five ratings.
- *Steps:* 1. Ask the ratings service to remove the rating with the id `all`.
- *Expected result:* The ratings service refuses the request as invalid, and all five ratings remain.

#### [AC04]: The Ratings list offers no removal

The Ratings list shall offer each rating only the Update and Details actions, and the list as a whole only New Rating ([AD#5]).

##### Test cases

**[TC01]: A rating's actions — Happy path:**
- *Preconditions:* `reader@example.com` holds a rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The Ratings list offers Update and Details on that rating and no Delete.

**[TC02]: No Delete All on the list — Negative / boundary:**
- *Preconditions:* The ratings store holds five ratings.
- *Steps:* 1. Open the Ratings list. 2. Look for any action that removes ratings.
- *Expected result:* The Ratings list offers New Rating and no Delete All, and no action that removes a rating.

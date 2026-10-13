---
kind: specification
key: BOOK-2-01-02
---

# Specification

- **Feature name**: Storefront ratings: give, change and see buyers' ratings
- **Version**: 1
- **Created**: 2026-10-13
- **Author**: ihudak
- **Published**: no
- **Open questions**: 0

## Problem statement

Shoppers on Quillmark's online store ignore its star ratings, and the storefront gives them nothing to trust. The rating form takes any number as a rating, zero and six included, and sends it without checking that an e-mail, an ISBN or a star value was entered. When the store refuses a rating, the form shows a bare pop-up with whatever the store sent, or nothing readable at all, so a customer cannot tell a rule from a fault. The form that changes a rating lets the customer edit the e-mail and the book, and shows them the comment they wrote. The Ratings list and the rating details screen show every customer's comment to any shopper, although Quillmark does not publish reviews, and the list lets anyone delete any rating, or every rating at once. Neither the book page nor the catalogue list shows how a book is rated, so a shopper cannot tell a broad opinion from a single one, or a well-rated book from an unrated one. In this release the store's rules change: only a buyer can rate a book, once, in whole stars from 1 to 5, and only Quillmark sees a comment. A storefront that kept behaving as it does today would contradict those rules on every screen a shopper sees.

## Scope

**In scope**

- The rating form offers only whole stars from 1 to 5, keeps Submit disabled until an e-mail, an ISBN and a star choice are entered, and says in words why a rating was refused or could not be saved, keeping what was entered.
- The update form shows the rating it changes, with its e-mail and book fixed, its current stars chosen and its comment field empty, and says in words why a change was refused or could not be saved, keeping what was entered.
- The book page shows a book's average rating to one decimal place with the number of ratings it rests on, "Not yet rated" for a book with none, and "Rating unavailable" when its rating cannot be loaded.
- The catalogue list shows each book's average rating to one decimal place, "Not yet rated" for a book with none, and "Rating unavailable" when the ratings cannot be loaded.
- Both forms say that only Quillmark sees the comment, and that an empty comment keeps any comment given before.
- Both forms have a screen reader announce their outcome text when it appears, and let the star choices be chosen with the keyboard alone.
- No comment on the Ratings list, the rating details screen or the update form.
- No removal of ratings anywhere on the storefront: the Ratings list loses Delete and Delete All.
- Each of these requests the storefront sends the ratings service — a book's rating summary, the summaries, a give, a change and the update form's load of a rating — gives up after a bounded time of at most 10 seconds, its exact value set by /design, and the screen then shows its failure text; an answer that arrives after the bounded time is disregarded.

**Out of scope**

- Enforcing any rating rule — buyers only, one rating per customer and book, whole stars from 1 to 5, a fixed e-mail and book — for any caller: the ratings service holds every rule ([[BOOK-2-01-01]], [AD#1]); the storefront shows the outcome.
- Rounding or averaging in the storefront: the figures are shown as the ratings service returns them ([AD#6]).
- Checking the format of an e-mail or an ISBN in the storefront: the ratings service's reason is shown instead.
- What the Ratings list and the rating details screen show when they cannot be loaded, and how long their loads wait, which stays as it is today.
- What the book page and the catalogue list show when the book or the books themselves cannot be loaded, the rating line and the Rating column included, which stays as it is today.
- Any storefront screen that removes a rating; an operator removes one through the ratings service ([[BOOK-2-01-01]], [AD#5]).
- Showing comments to anyone, an operator screen for comments, and a customer clearing a stored comment ([CD#17], [AD#7]).
- Hiding raters' e-mails: they stay visible on the Ratings list ([VD#6]).
- Opening a book's ratings by clicking its rating count ([CD#12]).
- Average ratings anywhere other than the book page and the catalogue list ([CD#10]; the sibling slice [BOOK-2-02](../../PRD-BOOK-2-02-book-discovery/)).
- Sign-in, notifications, any change to orders, books or the store's test-traffic generator, and response-time targets for any screen.

## User stories

### [U01]: Rate a book I bought

As a customer who bought a book, I want to give it a rating of whole stars from the rating form and be told plainly why the store refused it, so that my rating counts and I know what to do when it does not.

Done when the rating form sends only a complete rating of whole stars from 1 to 5 and shows, for each answer of the ratings service, the outcome text this specification names, keeping what was entered.

Traces to: [US#1]; [AC#2]–[AC#4], [AC#28]; the PRD-level specification's [U01]; [VD#2]; [AD#4]; [CG#5].

#### [AC01]: The rating form offers whole stars 1 to 5 only

When the rating form opens, the rating form shall offer exactly five star choices, the whole numbers 1, 2, 3, 4 and 5, with none chosen ([VD#2]).

##### Test cases

**[TC01]: The form opens with five star choices and none chosen — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating.
- *Expected result:* The rating form offers exactly the star choices 1, 2, 3, 4 and 5, with none chosen.

**[TC02]: No value outside the scale can be chosen — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Try to choose or enter 0, 6, -1 and 3.5 as the star value.
- *Expected result:* The rating form offers no way to choose any value other than 1, 2, 3, 4 or 5.

#### [AC02]: Submit stays disabled until an e-mail, an ISBN and a star choice are entered

While the e-mail field or the ISBN field is empty or holds only spaces, or no star choice is chosen, the rating form shall show Submit disabled.

##### Test cases

**[TC01]: A form just opened has Submit disabled — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating.
- *Expected result:* The rating form shows Submit disabled.

**[TC02]: With no star chosen, nothing is sent — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose no stars. 3. Try to select Submit.
- *Expected result:* The rating form shows Submit disabled, and the ratings service receives no request.

**[TC03]: With the e-mail or the ISBN empty, nothing is sent — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Leave the e-mail empty, enter `9780000000017`, choose 4 stars and try to select Submit. 3. Enter `reader@example.com`, clear the ISBN and try to select Submit.
- *Expected result:* The rating form shows Submit disabled each time, and the ratings service receives no request.

**[TC04]: A field of only spaces counts as empty — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter three spaces as the e-mail, enter `9780000000017`, choose 4 stars and try to select Submit. 3. Enter `reader@example.com`, replace the ISBN with three spaces and try to select Submit.
- *Expected result:* The rating form shows Submit disabled each time, and the ratings service receives no request.

#### [AC03]: The rating form sends the give as entered

When Submit sends a give, the rating form shall send the ratings service a `POST /api/v1/ratings` carrying the e-mail and the ISBN as typed, the star choice, and the comment as entered ([AD#4]).

##### Test cases

**[TC01]: The give carries what was entered — Happy path:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars, and enter the comment "Great plot". 3. Select Submit.
- *Expected result:* The ratings service receives one `POST /api/v1/ratings` carrying the e-mail `reader@example.com`, the ISBN `9780000000017`, 4 stars and the comment "Great plot".

**[TC02]: The e-mail and the ISBN are sent exactly as typed — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `400` and the reason "ISBN must be a 13-digits value".
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `Reader@Example.com` and ` 9780000000017` (with a leading space), and choose 1 star. 3. Select Submit.
- *Expected result:* The ratings service receives one `POST /api/v1/ratings` carrying the e-mail `Reader@Example.com` and the ISBN ` 9780000000017` exactly as typed, and 1 star.

**[TC03]: A give with no comment carries no comment text — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 5 stars. 3. Leave the comment empty and select Submit.
- *Expected result:* The ratings service receives one `POST /api/v1/ratings` carrying 5 stars and an empty or absent comment.

#### [AC04]: A stored give still opens the Ratings list

When the ratings service answers a give from the rating form with `200` ([AD#4]), the rating form shall open the Ratings list, as it does today.

##### Test cases

**[TC01]: A stored give opens the Ratings list — Happy path:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form opens the Ratings list.

**[TC02]: A refused give does not open the Ratings list — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `403`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form does not open the Ratings list.

#### [AC05]: The rating form names the buyers-only rule

When the ratings service answers a give from the rating form with `403` ([AD#4]), the rating form shall show beside Submit exactly "Only customers who have bought this book can rate it.", whatever the error body holds.

##### Test cases

**[TC01]: A buyers-only refusal shows the rule — Happy path:**
- *Preconditions:* The ratings service answers every give with `403` and the `message` "Only customers who have bought this book can rate it."
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "Only customers who have bought this book can rate it."

**[TC02]: The rule is shown whatever the refusal's body holds — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `403` and an empty body.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "Only customers who have bought this book can rate it."

#### [AC06]: The rating form shows an invalid give's reason

When the ratings service answers a give from the rating form with `400` and a reason in the `message` field of its error body ([AD#4]), the rating form shall show beside Submit that reason, exactly as given.

##### Test cases

**[TC01]: A malformed e-mail's reason is shown — Happy path:**
- *Preconditions:* The ratings service answers every give with `400` and the `message` "Invalid email address".
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader-at-example` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "Invalid email address".

**[TC02]: A short ISBN's reason is shown, not the buyers-only text — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `400` and the `message` "ISBN must be a 13-digits value".
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `978000000001`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "ISBN must be a 13-digits value" and not "Only customers who have bought this book can rate it."

**[TC03]: A reason the form has no text of its own for is shown as given — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `400` and the `message` "Invalid rating. Must be a whole number from 1 to 5."
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "Invalid rating. Must be a whole number from 1 to 5."

**[TC04]: A reason holding markup is shown as plain text — Security / privacy:**
- *Preconditions:* The ratings service answers every give with `400` and the `message` `<B>Invalid</B><img src=x onerror=alert(1)>`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly the characters `<B>Invalid</B><img src=x onerror=alert(1)>`, with no bold text, no image and no script run.

#### [AC07]: The rating form says the purchase check is unavailable

When the ratings service answers a give from the rating form with `503` ([AD#2], [AD#4]), the rating form shall show beside Submit exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

##### Test cases

**[TC01]: An unavailable purchase check is named — Happy path:**
- *Preconditions:* The ratings service answers every give with `503`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

**[TC02]: The unavailable text is shown, not the body's own words or the rule — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `503` and the `message` "Service Unavailable".
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes.", and neither "Service Unavailable" nor "Only customers who have bought this book can rate it."

#### [AC08]: The rating form says when a rating cannot be saved

If the rating form cannot reach the ratings service, gets no answer to a give within its bounded time, or gets any answer [AC04]–[AC07] do not cover, a `400` with no readable reason included, then the rating form shall show beside Submit exactly "We can't save your rating right now. Please try again in a few minutes."

##### Test cases

**[TC01]: The ratings service is unreachable — Happy path:**
- *Preconditions:* The ratings service is unreachable from the storefront.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes."

**[TC02]: An unexpected server error is not shown as a refusal — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `500`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and neither "Only customers who have bought this book can rate it." nor "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

**[TC03]: A `400` with no readable reason — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `400` and an empty body.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and no empty or "undefined" text.

**[TC04]: A `404` to a give — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `404`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and not "This rating no longer exists. Nothing was saved."

**[TC05]: A give that is never answered — State / lifecycle:**
- *Preconditions:* The ratings service accepts every give and never answers it.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Wait 10 seconds.
- *Expected result:* Within 10 seconds of Submit, the rating form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes."

**[TC06]: A give answered after the bounded time — State / lifecycle:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating, 15 seconds after receiving it.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Wait 20 seconds.
- *Expected result:* Within 10 seconds of Submit, the rating form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and it still shows it, with the Ratings list not opened, after the late `200` has arrived.

#### [AC09]: The rating form keeps what was entered

When a give from the rating form ends in any outcome other than `200` ([AC05]–[AC08]), the rating form shall stay open with the e-mail, the ISBN, the star choice and the comment as entered.

##### Test cases

**[TC01]: A buyers-only refusal keeps the entries — Happy path:**
- *Preconditions:* The ratings service answers every give with `403`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars, and enter the comment "Lovely cover". 3. Select Submit.
- *Expected result:* The rating form stays open showing `browser@example.com`, `9780000000017`, 3 stars and "Lovely cover".

**[TC02]: An unavailable refusal keeps the entries — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `503`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 5 stars, and enter the comment "Loved it". 3. Select Submit.
- *Expected result:* The rating form stays open showing `reader5@example.com`, `9780000000017`, 5 stars and "Loved it", and the Ratings list is not opened.

**[TC03]: A give that cannot reach the ratings service keeps the entries — Negative / boundary:**
- *Preconditions:* The ratings service is unreachable from the storefront.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 2 stars, and enter the comment "On reflection". 3. Select Submit.
- *Expected result:* The rating form stays open showing `reader5@example.com`, `9780000000017`, 2 stars and "On reflection".

**[TC04]: An invalid give keeps the entries — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `400` and the `message` "Invalid email address".
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader-at-example` and `9780000000017`, and choose 1 star, and enter the comment "Typo". 3. Select Submit.
- *Expected result:* The rating form stays open showing `reader-at-example`, `9780000000017`, 1 star and "Typo".

#### [AC10]: Submit is disabled while a give is being sent

While a give sent from the rating form has no outcome yet, the rating form shall show Submit disabled.

##### Test cases

**[TC01]: A second Submit sends nothing while the give is in flight — Happy path:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating, 3 seconds after receiving it.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Try to select Submit again before the answer arrives.
- *Expected result:* Submit is shown disabled until the answer arrives, and the ratings service receives exactly one `POST /api/v1/ratings`.

**[TC02]: Submit sends again once the give has its outcome — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `503`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Wait until the form shows "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." 5. Select Submit again.
- *Expected result:* The ratings service receives two `POST /api/v1/ratings`, the second sent after the first had its outcome.

#### [AC11]: Each Submit clears the earlier outcome text

When Submit sends a give, the rating form shall clear any outcome text an earlier Submit showed.

##### Test cases

**[TC01]: A new Submit clears the earlier text — Happy path:**
- *Preconditions:* The ratings service answers the first give with `503` and the second with `200`, 3 seconds after receiving it.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Once the form shows "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes.", select Submit again. 5. Look at the form before the second answer arrives.
- *Expected result:* While the second give has no outcome, the rating form shows no outcome text.

**[TC02]: The next outcome replaces the earlier text — Negative / boundary:**
- *Preconditions:* The ratings service answers the first give with `503` and the second with `403`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Once the first outcome is shown, select Submit again.
- *Expected result:* The rating form shows beside Submit exactly "Only customers who have bought this book can rate it." and not "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

#### [AC12]: A screen reader announces the outcome text

When the rating form shows an outcome text beside Submit, the rating form shall have a screen reader announce that text without the customer moving to it.

##### Test cases

**[TC01]: A refusal is announced — Happy path:**
- *Preconditions:* A screen reader is running; the ratings service answers every give with `403`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The screen reader announces "Only customers who have bought this book can rate it." without the customer moving to the text.

**[TC02]: A later outcome is announced too — Negative / boundary:**
- *Preconditions:* A screen reader is running; the ratings service answers the first give with `503` and the second with `403`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit. 4. Once the first outcome is announced, select Submit again.
- *Expected result:* The screen reader announces "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." after the first Submit and "Only customers who have bought this book can rate it." after the second.

#### [AC13]: The star choices can be chosen with the keyboard

The rating form shall let each of its five star choices be reached and chosen with the keyboard alone.

##### Test cases

**[TC01]: A rating given with the keyboard alone — Happy path:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Using only the keyboard, enter `reader@example.com` and `9780000000017`, move to the star choices, choose 4 stars and select Submit.
- *Expected result:* The ratings service receives one `POST /api/v1/ratings` carrying 4 stars.

**[TC02]: The keyboard reaches 1 and 5 and nothing beyond — Negative / boundary:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Using only the keyboard, move to the star choices and choose 1 star. 3. Try to move below 1. 4. Choose 5 stars. 5. Try to move above 5.
- *Expected result:* The rating form lets the keyboard choose 1 and 5, and offers no value below 1 or above 5.

#### [AC14]: Submit is enabled once the give can be sent

While the e-mail and the ISBN each hold a character other than a space, a star choice is chosen and no give is in flight, the rating form shall show Submit enabled.

##### Test cases

**[TC01]: A complete form enables Submit — Happy path:**
- *Preconditions:* The ratings service answers every give with `200` and the saved rating.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit.
- *Expected result:* The rating form shows Submit enabled once the star is chosen, and the ratings service receives one `POST /api/v1/ratings`.

**[TC02]: Submit is enabled again after an outcome — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `503`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `reader5@example.com` and `9780000000017`, and choose 4 stars. 3. Select Submit. 4. Wait until the outcome text is shown.
- *Expected result:* Once "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." is shown, the rating form shows Submit enabled.

---

### [U02]: Change my rating of a book

As a customer who rated a book, I want to change its stars and comment from the Ratings list, with no way to move the rating to another e-mail or book, so that my rating says what I think now.

Done when the update form shows the rating with its e-mail and ISBN fixed and its stars chosen, sends a change, and shows, for each answer of the ratings service, the outcome text this specification names, keeping what was entered.

Traces to: [US#2]; [AC#6], [AC#8], [AC#9], [AC#11], [AC#24], [AC#29]; the PRD-level specification's [U02]; [VD#2]; [AD#4]; [CG#10].

#### [AC01]: The Ratings list keeps the rater and Update

The Ratings list shall show each rating's e-mail with an Update action that opens the update form for that rating.

##### Test cases

**[TC01]: A rating shows its e-mail, and Update opens it — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7, `reader@example.com`'s rating of ISBN `9780000000017`, and answers the read of rating 7 with it.
- *Steps:* 1. Open the Ratings list. 2. Select Update on the rating of `reader@example.com`.
- *Expected result:* The Ratings list shows `reader@example.com` on that rating, and Update opens the update form for rating 7.

**[TC02]: Each rating's Update opens its own rating — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7 of `reader@example.com` and rating 8 of `second@example.com`, both of ISBN `9780000000017`, and answers the read of each.
- *Steps:* 1. Open the Ratings list. 2. Select Update on the rating of `second@example.com`.
- *Expected result:* The update form opens on rating 8 of `second@example.com`, not on rating 7.

#### [AC02]: The update form shows the rating it changes

When the update form opens and loads a rating ([AD#7]), the update form shall show the heading "Update Rating", the rating's e-mail and ISBN as values that cannot be edited, and the five star choices 1 to 5 with the rating's current stars chosen.

##### Test cases

**[TC01]: The update form opens on the rating's values — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7.
- *Expected result:* The update form shows the heading "Update Rating", `reader@example.com` and `9780000000017`, and the star choices 1 to 5 with 3 chosen.

**[TC02]: The e-mail and the ISBN cannot be edited — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Try to change the e-mail and the ISBN.
- *Expected result:* The update form offers no way to change the e-mail or the ISBN.

**[TC03]: No value outside the scale can be chosen on the update form — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Try to choose or enter 0, 6 and 3.5 as the star value.
- *Expected result:* The update form offers no way to choose any value other than 1, 2, 3, 4 or 5.

#### [AC03]: The update form sends the change for the loaded rating

When Submit sends a change, the update form shall send the ratings service a `PUT /api/v1/ratings/{id}` for the loaded rating's id, carrying that id, the rating's e-mail and ISBN as loaded, the star choice, and the comment as entered ([AD#4]).

##### Test cases

**[TC01]: The change carries the loaded rating — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 5 stars and enter the comment "Changed my mind". 3. Select Submit.
- *Expected result:* The ratings service receives one `PUT /api/v1/ratings/7` carrying the id 7, the e-mail `reader@example.com`, the ISBN `9780000000017`, 5 stars and the comment "Changed my mind".

**[TC02]: The e-mail is sent as loaded, capitals included — Negative / boundary:**
- *Preconditions:* Rating 7 is a rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `Reader@Example.com`, the ISBN `9780000000017` and 3 stars, and answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The ratings service receives one `PUT /api/v1/ratings/7` carrying the id 7, the e-mail `Reader@Example.com` exactly as loaded, the ISBN `9780000000017` and 4 stars.

**[TC03]: An empty comment field sends no comment text — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and leave the comment empty. 3. Select Submit.
- *Expected result:* The ratings service receives one `PUT /api/v1/ratings/7` carrying the id 7, 4 stars and an empty or absent comment.

#### [AC04]: A saved change opens the Ratings list

When the ratings service answers a change from the update form with `200` ([AD#4]), the update form shall open the Ratings list.

##### Test cases

**[TC01]: A saved change opens the Ratings list — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form opens the Ratings list.

**[TC02]: A refused change does not open the Ratings list — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `403`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form does not open the Ratings list.

#### [AC05]: The update form names the buyers-only rule

When the ratings service answers a change from the update form with `403` ([AD#4]), the update form shall show beside Submit exactly "Only customers who have bought this book can rate it.", whatever the error body holds.

##### Test cases

**[TC01]: A buyers-only refusal shows the rule — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `403` and the `message` "Only customers who have bought this book can rate it."
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 5 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "Only customers who have bought this book can rate it."

**[TC02]: The rule is shown whatever the refusal's body holds — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `403` and an empty body.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 2 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "Only customers who have bought this book can rate it."

#### [AC06]: The update form shows an invalid change's reason

When the ratings service answers a change from the update form with `400` and a reason in the `message` field of its error body ([AD#4]), the update form shall show beside Submit that reason, exactly as given.

##### Test cases

**[TC01]: A star reason is shown — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `400` and the `message` "Invalid rating. Must be a whole number from 1 to 5."
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "Invalid rating. Must be a whole number from 1 to 5."

**[TC02]: Another reason is shown as given, not as the buyers-only text — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `400` and the `message` "A rating's e-mail and ISBN cannot be changed."
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "A rating's e-mail and ISBN cannot be changed." and not "Only customers who have bought this book can rate it."

**[TC03]: A reason holding markup is shown as plain text — Security / privacy:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `400` and the `message` `<B>Invalid</B><img src=x onerror=alert(1)>`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly the characters `<B>Invalid</B><img src=x onerror=alert(1)>`, with no bold text, no image and no script run.

#### [AC07]: The update form says the purchase check is unavailable

When the ratings service answers a change from the update form with `503` ([AD#2], [AD#4]), the update form shall show beside Submit exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

##### Test cases

**[TC01]: An unavailable purchase check is named — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `503`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 5 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

**[TC02]: The unavailable text is shown, not the body's own words or the rule — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `503` and the `message` "Service Unavailable".
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 5 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes.", and neither "Service Unavailable" nor "Only customers who have bought this book can rate it."

#### [AC08]: The update form says the rating no longer exists

When the ratings service answers a change from the update form with `404` ([AD#4]), the update form shall show beside Submit exactly "This rating no longer exists. Nothing was saved."

##### Test cases

**[TC01]: The rating was removed while the update form was open — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `404`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "This rating no longer exists. Nothing was saved."

**[TC02]: A `404` on Submit is not shown as a failure to save — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `404` and the `message` "Rating not found".
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "This rating no longer exists. Nothing was saved.", and neither "We can't save your rating right now. Please try again in a few minutes." nor "Rating not found".

#### [AC09]: The update form says when a change cannot be saved

If the update form cannot reach the ratings service, gets no answer to a change within its bounded time, or gets any answer [AC04]–[AC08] do not cover, a `400` with no readable reason included, then the update form shall show beside Submit exactly "We can't save your rating right now. Please try again in a few minutes."

##### Test cases

**[TC01]: The ratings service becomes unreachable — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Make the ratings service unreachable from the storefront. 3. Choose 4 stars and select Submit.
- *Expected result:* The update form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes."

**[TC02]: An unexpected server error is not shown as a refusal — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `500`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and neither "Only customers who have bought this book can rate it." nor "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

**[TC03]: A `400` with no readable reason — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `400` and an empty body.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars. 3. Select Submit.
- *Expected result:* The update form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and no empty or "undefined" text.

**[TC04]: A change that is never answered — State / lifecycle:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it accepts every change and never answers it.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Wait 10 seconds.
- *Expected result:* Within 10 seconds of Submit, the update form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes."

**[TC05]: A change answered after the bounded time — State / lifecycle:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating, 15 seconds after receiving it.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Wait 20 seconds.
- *Expected result:* Within 10 seconds of Submit, the update form shows beside Submit exactly "We can't save your rating right now. Please try again in a few minutes.", and it still shows it, with the Ratings list not opened, after the late `200` has arrived.

#### [AC10]: The update form keeps what was entered

When a change from the update form ends in any outcome other than `200` ([AC05]–[AC09]), the update form shall stay open with the star choice and the comment as entered.

##### Test cases

**[TC01]: A buyers-only refusal keeps the entries — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `403`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 5 stars and enter the comment "Changed my mind". 3. Select Submit.
- *Expected result:* The update form stays open showing 5 stars and "Changed my mind".

**[TC02]: An unavailable refusal keeps the entries — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `503`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 1 star and enter the comment "Not for me". 3. Select Submit.
- *Expected result:* The update form stays open showing 1 star and "Not for me", and the Ratings list is not opened.

**[TC03]: A change to a removed rating keeps the entries — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `404`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and enter the comment "Still good". 3. Select Submit.
- *Expected result:* The update form stays open showing 4 stars and "Still good".

**[TC04]: A change that cannot reach the ratings service keeps the entries — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Make the ratings service unreachable from the storefront. 3. Choose 2 stars, enter the comment "On reflection" and select Submit.
- *Expected result:* The update form stays open showing 2 stars and "On reflection".

#### [AC11]: Submit is disabled while a change is being sent

While a change sent from the update form has no outcome yet, the update form shall show Submit disabled.

##### Test cases

**[TC01]: A second Submit sends nothing while the change is in flight — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating, 3 seconds after receiving it.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Try to select Submit again before the answer arrives.
- *Expected result:* Submit is shown disabled until the answer arrives, and the ratings service receives exactly one `PUT /api/v1/ratings/7`.

**[TC02]: Submit sends again once the change has its outcome — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `503`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Wait until the outcome text is shown. 4. Select Submit again.
- *Expected result:* The ratings service receives two `PUT /api/v1/ratings/7`, the second sent after the first had its outcome.

#### [AC12]: Each Submit clears the earlier outcome text

When Submit sends a change, the update form shall clear any outcome text an earlier Submit showed.

##### Test cases

**[TC01]: A new Submit clears the earlier text — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers the first change with `503` and the second with `200`, 3 seconds after receiving it.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Once the outcome text is shown, select Submit again. 4. Look at the form before the second answer arrives.
- *Expected result:* While the second change has no outcome, the update form shows no outcome text.

**[TC02]: The next outcome replaces the earlier text — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers the first change with `503` and the second with `404`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Once the first outcome is shown, select Submit again.
- *Expected result:* The update form shows beside Submit exactly "This rating no longer exists. Nothing was saved." and not "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes."

#### [AC13]: The update form opened for a removed rating says so

When the ratings service answers the update form's load of a rating with `404` ([AD#7]), the update form shall show exactly "This rating no longer exists." in place of its fields and Submit.

##### Test cases

**[TC01]: Update selected on a removed rating — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7, `reader@example.com`'s rating of ISBN `9780000000017`, and answers the read of rating 7 with `404`.
- *Steps:* 1. Open the Ratings list. 2. Select Update on rating 7.
- *Expected result:* The update form shows exactly "This rating no longer exists." and no e-mail, ISBN, star or comment field and no Submit.

**[TC02]: An existing rating opens with its fields — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7.
- *Expected result:* The update form shows the rating's fields and Submit, and not "This rating no longer exists."

#### [AC14]: The update form says when the rating cannot be loaded

If the update form cannot reach the ratings service to load the rating, gets no answer within its bounded time, or gets any answer other than the rating or `404` ([AD#7]), then the update form shall show exactly "We can't load this rating right now. Please try again in a few minutes." in place of its fields and Submit.

##### Test cases

**[TC01]: The ratings service is unreachable when Update is selected — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7, `reader@example.com`'s rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list. 2. Make the ratings service unreachable from the storefront. 3. Select Update on rating 7.
- *Expected result:* The update form shows exactly "We can't load this rating right now. Please try again in a few minutes." and no e-mail, ISBN, star or comment field and no Submit.

**[TC02]: A server error on loading is not shown as a removed rating — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7 and answers the read of rating 7 with `500`.
- *Steps:* 1. Open the Ratings list. 2. Select Update on rating 7.
- *Expected result:* The update form shows exactly "We can't load this rating right now. Please try again in a few minutes." and not "This rating no longer exists."

**[TC03]: An answer that is not a rating — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7 and answers the read of rating 7 with `200` and an empty body.
- *Steps:* 1. Open the Ratings list. 2. Select Update on rating 7.
- *Expected result:* The update form shows exactly "We can't load this rating right now. Please try again in a few minutes." and no e-mail, ISBN, star or comment field and no Submit.

**[TC04]: A load that is never answered — State / lifecycle:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7, and accepts the read of rating 7 and never answers it.
- *Steps:* 1. Open the Ratings list. 2. Select Update on rating 7. 3. Wait 10 seconds.
- *Expected result:* Within 10 seconds of selecting Update, the update form shows exactly "We can't load this rating right now. Please try again in a few minutes." in place of its fields and Submit.

**[TC05]: A rating answered after the bounded time — State / lifecycle:**
- *Preconditions:* The ratings service answers the Ratings list with rating 7, and answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, 15 seconds after receiving it.
- *Steps:* 1. Open the Ratings list. 2. Select Update on rating 7. 3. Wait 20 seconds.
- *Expected result:* Within 10 seconds of selecting Update, the update form shows exactly "We can't load this rating right now. Please try again in a few minutes." in place of its fields and Submit, and it still shows that text, with no e-mail, ISBN, star or comment field and no Submit, after the late answer has arrived.

#### [AC15]: The update form shows only its heading until the load has an outcome

While the update form's load of the rating it opened for has no outcome yet, the update form shall show only its heading "Update Rating", with no e-mail, ISBN, star choice, comment field, text or Submit.

##### Test cases

**[TC01]: Only the heading shows while the rating loads — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7, 3 seconds after receiving it, with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Look at the update form before the rating has loaded.
- *Expected result:* The update form shows the heading "Update Rating" and no e-mail, ISBN, star choice, comment field, text or Submit.

**[TC02]: The fields and Submit appear once the rating has loaded — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7, 3 seconds after receiving it, with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Wait until the rating has loaded. 3. Choose 4 stars and select Submit.
- *Expected result:* Once the rating has loaded, the update form shows `reader@example.com`, `9780000000017`, the star choices with 3 chosen and Submit, and the ratings service receives one `PUT /api/v1/ratings/7` carrying 4 stars.

#### [AC16]: A screen reader announces the outcome text

When the update form shows an outcome text beside Submit, the update form shall have a screen reader announce that text without the customer moving to it.

##### Test cases

**[TC01]: A refusal is announced — Happy path:**
- *Preconditions:* A screen reader is running; rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `403`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 5 stars and select Submit.
- *Expected result:* The screen reader announces "Only customers who have bought this book can rate it." without the customer moving to the text.

**[TC02]: A later outcome is announced too — Negative / boundary:**
- *Preconditions:* A screen reader is running; rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers the first change with `503` and the second with `404`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Once the first outcome is announced, select Submit again.
- *Expected result:* The screen reader announces "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." after the first Submit and "This rating no longer exists. Nothing was saved." after the second.

#### [AC17]: The star choices can be chosen with the keyboard

The update form, once it has loaded a rating, shall let each of its five star choices be reached and chosen with the keyboard alone.

##### Test cases

**[TC01]: A change made with the keyboard alone — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Using only the keyboard, move to the star choices, choose 5 stars and select Submit.
- *Expected result:* The ratings service receives one `PUT /api/v1/ratings/7` carrying 5 stars.

**[TC02]: The keyboard reaches 1 and 5 and nothing beyond — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Using only the keyboard, move to the star choices and choose 1 star. 3. Try to move below 1. 4. Choose 5 stars. 5. Try to move above 5.
- *Expected result:* The update form lets the keyboard choose 1 and 5, and offers no value below 1 or above 5.

#### [AC18]: Submit is enabled once the change can be sent

While the update form has loaded the rating and no change is in flight, the update form shall show Submit enabled.

##### Test cases

**[TC01]: A loaded rating enables Submit — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Select Submit without changing anything.
- *Expected result:* The update form shows Submit enabled once the rating has loaded, and the ratings service receives one `PUT /api/v1/ratings/7` carrying 3 stars.

**[TC02]: Submit is enabled again after an outcome — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `503`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit. 3. Wait until the outcome text is shown.
- *Expected result:* Once "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." is shown, the update form shows Submit enabled.

---

### [U03]: See how a book is rated on its book page

As a shopper looking at a book, I want to see its average rating and how many ratings it rests on, so that I can tell a broad opinion from a single one.

Done when the book page shows a rated book's average and count, "Not yet rated" for a book with none, and "Rating unavailable" when the book's rating cannot be loaded.

Traces to: [US#3]; [AC#13]–[AC#16]; the PRD-level specification's [U03]; [VD#3]; [AD#6]; [CG#6], [CG#7], [CG#8].

#### [AC01]: The book page shows the average rating

When the book page loads a book whose rating summary is readable and has a count above 0 ([AD#6]), the book page shall show, below the book's other details, the rating line: "★" followed by the summary's average, always with one decimal place.

##### Test cases

**[TC01]: An average of 4.3 shows on the rating line — Happy path:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062`; the ratings service answers its rating summary with the count 4 and the average 4.3.
- *Steps:* 1. Load the book page of `9780000000062`.
- *Expected result:* The book page shows "★ 4.3" on the rating line, below the book's other details.

**[TC02]: A whole average shows one decimal — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000079`; the ratings service answers its rating summary with the count 3 and the average 4.
- *Steps:* 1. Load the book page of `9780000000079`.
- *Expected result:* The book page shows "★ 4.0".

#### [AC02]: The book page shows how many ratings the average rests on

When the book page shows a book's average rating, the book page shall show on the rating line, after the average, the summary's count as "(1 rating)" for a count of 1 and as the count followed by "ratings" in parentheses for any other count, such as "(37 ratings)".

##### Test cases

**[TC01]: Thirty-seven ratings show their count — Happy path:**
- *Preconditions:* The books service answers the book of ISBN `9780000000093`; the ratings service answers its rating summary with the count 37 and the average 4.
- *Steps:* 1. Load the book page of `9780000000093`.
- *Expected result:* The book page shows "★ 4.0 (37 ratings)".

**[TC02]: One rating is worded in the singular — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000109`; the ratings service answers its rating summary with the count 1 and the average 5.
- *Steps:* 1. Load the book page of `9780000000109`.
- *Expected result:* The book page shows "★ 5.0 (1 rating)".

**[TC03]: Two ratings are worded in the plural — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000116`; the ratings service answers its rating summary with the count 2 and the average 3.5.
- *Steps:* 1. Load the book page of `9780000000116`.
- *Expected result:* The book page shows "★ 3.5 (2 ratings)".

#### [AC03]: An unrated book shows "Not yet rated"

When the book page loads a book whose rating summary is readable and has a count of 0 ([AD#6]), the book page shall show "Not yet rated" in place of the rating line, with no count.

##### Test cases

**[TC01]: A book nobody has rated — Happy path:**
- *Preconditions:* The books service answers the book of ISBN `9780000000123`; the ratings service answers its rating summary with the count 0 and the average `null`.
- *Steps:* 1. Load the book page of `9780000000123`.
- *Expected result:* The book page shows "Not yet rated" in place of the rating line, with no count.

**[TC02]: An unrated book shows no star and no zero count — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000130`; the ratings service answers its rating summary with the count 0 and the average `null`.
- *Steps:* 1. Load the book page of `9780000000130`.
- *Expected result:* The book page shows "Not yet rated", and neither "★" nor "(0 ratings)".

#### [AC04]: A rating that cannot be loaded shows "Rating unavailable"

If the book page cannot reach the ratings service for the book's rating summary, gets no answer within its bounded time, or gets any answer other than a readable summary ([AD#6]), then the book page shall show "Rating unavailable" in place of the rating line, with the book's other details shown as usual.

##### Test cases

**[TC01]: The ratings service is unreachable — Happy path:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062`; the ratings service is unreachable from the storefront.
- *Steps:* 1. Load the book page of `9780000000062`.
- *Expected result:* The book page shows "Rating unavailable" in place of the rating line, and shows the book's ISBN, Title, Author, Price, Lang and Vend as usual.

**[TC02]: An unrated book is not called unrated when the load fails — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000123`; the ratings service answers its rating summary with `500`.
- *Steps:* 1. Load the book page of `9780000000123`.
- *Expected result:* The book page shows "Rating unavailable" and not "Not yet rated".

**[TC03]: A summary that cannot be read — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062`; the ratings service answers its rating summary with `200`, the count 4 and no average.
- *Steps:* 1. Load the book page of `9780000000062`.
- *Expected result:* The book page shows "Rating unavailable", and no "★".

**[TC04]: A summary that is never answered — State / lifecycle:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062`; the ratings service accepts the summary request and never answers it.
- *Steps:* 1. Load the book page of `9780000000062`. 2. Wait 10 seconds.
- *Expected result:* Within 10 seconds of loading, the book page shows "Rating unavailable" in place of the rating line, and the book's other details as usual.

**[TC05]: A summary answered after the bounded time — State / lifecycle:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062`; the ratings service answers its rating summary with the count 4 and the average 4.3, 15 seconds after receiving the request.
- *Steps:* 1. Load the book page of `9780000000062`. 2. Wait 20 seconds.
- *Expected result:* Within 10 seconds of loading, the book page shows "Rating unavailable" in place of the rating line, and it still shows "Rating unavailable", not "★ 4.3", after the late summary has arrived.

#### [AC05]: Nothing stands in for the rating while it loads

While the book's rating summary has not yet answered and the book page's bounded time has not passed, the book page shall show nothing in place of the rating line.

##### Test cases

**[TC01]: Nothing stands in for the rating while it loads — Happy path:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062` at once; the ratings service answers its rating summary with the count 4 and the average 4.3, 3 seconds after receiving the request.
- *Steps:* 1. Load the book page of `9780000000062`. 2. Look at the book page before the summary answers.
- *Expected result:* The book page shows the book's other details and nothing in place of the rating line: no "★", no "Not yet rated" and no "Rating unavailable".

**[TC02]: An unrated book is not called unrated before its summary answers — Negative / boundary:**
- *Preconditions:* The books service answers the book of ISBN `9780000000123` at once; the ratings service answers its rating summary with the count 0, 3 seconds after receiving the request.
- *Steps:* 1. Load the book page of `9780000000123`. 2. Look at the book page before the summary answers. 3. Look again once it has answered.
- *Expected result:* Before the summary answers, the book page shows nothing in place of the rating line; afterwards it shows "Not yet rated".

#### [AC06]: The book page asks for the rating on every load

When the book page loads, the book page shall request `GET /api/v1/ratings/summary` for the book's ISBN on that load, never reusing an answer from an earlier load ([AD#6]).

##### Test cases

**[TC01]: The book page asks for the book's summary — Happy path:**
- *Preconditions:* The books service answers the book of ISBN `9780000000062`; the ratings service answers its rating summary with the count 4 and the average 4.3.
- *Steps:* 1. Load the book page of `9780000000062`.
- *Expected result:* The ratings service receives one `GET /api/v1/ratings/summary` for the ISBN `9780000000062`.

**[TC02]: A second load asks again and shows the new answer — State / lifecycle:**
- *Preconditions:* The books service answers the book of ISBN `9780000000147`; the ratings service answers its first rating summary request with the count 1 and the average 4, and every later one with the count 2 and the average 3.
- *Steps:* 1. Load the book page of `9780000000147`. 2. Load the book page of `9780000000147` again.
- *Expected result:* The ratings service receives two `GET /api/v1/ratings/summary` requests, and the second load shows "★ 3.0 (2 ratings)".

**[TC03]: Coming back to the book page asks again — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list and the book of ISBN `9780000000062`; the ratings service answers the summaries and the rating summary of `9780000000062` with the count 4 and the average 4.3.
- *Steps:* 1. Load the book page of `9780000000062`. 2. Open the catalogue list. 3. Select Details on the row of `9780000000062`.
- *Expected result:* The ratings service receives two `GET /api/v1/ratings/summary` requests for `9780000000062`, one for each time the book page loaded.

---

### [U04]: Compare books by rating on the catalogue list

As a shopper browsing the catalogue list, I want to see each book's average rating, so that I can tell which books buyers rate highly before I open one.

Done when every book on the catalogue list shows the average its book page shows, "Not yet rated", or "Rating unavailable".

Traces to: [US#4]; [AC#17]–[AC#20]; the PRD-level specification's [U04]; [VD#3]; [AD#6]; [CG#8], [CG#9].

#### [AC01]: The catalogue list shows each book's average

When the catalogue list loads ([AD#6]), the catalogue list shall show a "Rating" column after "OnVend" holding, for each book a readable summaries response lists with a count above 0, "★" followed by the book's average, always with one decimal place and with no count.

##### Test cases

**[TC01]: A rated book shows its average in the Rating column — Happy path:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000062`; the ratings service answers the summaries with `9780000000062`, the count 4 and the average 4.3.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.3" in a "Rating" column, after "OnVend", on the row of `9780000000062`.

**[TC02]: A whole average shows one decimal — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000109`; the ratings service answers the summaries with `9780000000109`, the count 1 and the average 5.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 5.0" on the row of `9780000000109`.

**[TC03]: The list shows the average without a count — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000093`; the ratings service answers the summaries with `9780000000093`, the count 37 and the average 4.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.0" on the row of `9780000000093`, with no rating count.

#### [AC02]: An unrated book shows "Not yet rated" on the list

When the catalogue list loads ([AD#6]), the catalogue list shall show "Not yet rated" in the Rating column of each book a readable summaries response does not list, or lists with a count of 0.

##### Test cases

**[TC01]: A book the summaries do not list — Happy path:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000123`; the ratings service answers the summaries with no entry for it.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Not yet rated" on the row of `9780000000123`.

**[TC02]: Rated and unrated books side by side — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the books of ISBN `9780000000062` and `9780000000123`; the ratings service answers the summaries with `9780000000062`, the count 4 and the average 4.3, and no entry for `9780000000123`.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "★ 4.3" on the row of `9780000000062` and "Not yet rated" on the row of `9780000000123`.

**[TC03]: A book listed with a count of 0 — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000130`; the ratings service answers the summaries with `9780000000130`, the count 0 and the average `null`.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Not yet rated" on the row of `9780000000130`, and no "★".

#### [AC03]: The list and the book page show one answer alike

When the catalogue list's summaries and the book page's summary agree on a book, with the same count and average or with no rating (absent from the summaries, a count of 0 in the summary), the catalogue list shall show the same value the book page shows: the same average in the same format, or the same "Not yet rated" ([AD#6]).

##### Test cases

**[TC01]: The list matches the book page — Happy path:**
- *Preconditions:* The books service answers the catalogue list and the book of ISBN `9780000000062`; the ratings service answers the summaries with `9780000000062`, the count 4 and the average 4.3, and the rating summary of `9780000000062` with the same.
- *Steps:* 1. Open the catalogue list. 2. Open the book page of `9780000000062`.
- *Expected result:* The catalogue list and the book page both show "★ 4.3" for `9780000000062`.

**[TC02]: The list matches the book page for an unrated book — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list and the book of ISBN `9780000000123`; the ratings service answers the summaries with no entry for `9780000000123`, and its rating summary with the count 0.
- *Steps:* 1. Open the catalogue list. 2. Open the book page of `9780000000123`.
- *Expected result:* The catalogue list and the book page both show "Not yet rated" for `9780000000123`.

#### [AC04]: Ratings that cannot be loaded show "Rating unavailable" on the list

If the catalogue list cannot reach the ratings service for the summaries, gets no answer within its bounded time, or gets any answer other than a readable summaries response ([AD#6]), then the catalogue list shall show "Rating unavailable" in the Rating column of every book, with the books listed as usual.

##### Test cases

**[TC01]: The ratings service is unreachable — Happy path:**
- *Preconditions:* The books service answers the catalogue list with the books of ISBN `9780000000062` and `9780000000123`; the ratings service is unreachable from the storefront.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list lists both books with their usual columns and shows "Rating unavailable" in the Rating column of each.

**[TC02]: An unrated book is not called unrated when the load fails — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000123`; the ratings service answers the summaries with `500`.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Rating unavailable", and not "Not yet rated", on the row of `9780000000123`.

**[TC03]: Summaries that cannot be read — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000062`; the ratings service answers the summaries with `200` and a body that is not a list of summaries.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Rating unavailable" on the row of `9780000000062`.

**[TC04]: Summaries that are never answered — State / lifecycle:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000062`; the ratings service accepts the summaries request and never answers it.
- *Steps:* 1. Open the catalogue list. 2. Wait 10 seconds.
- *Expected result:* Within 10 seconds of opening, the catalogue list lists the book as usual and shows "Rating unavailable" on its row.

**[TC05]: Summaries answered after the bounded time — State / lifecycle:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000062`; the ratings service answers the summaries with `9780000000062`, the count 4 and the average 4.3, 15 seconds after receiving the request.
- *Steps:* 1. Open the catalogue list. 2. Wait 20 seconds.
- *Expected result:* Within 10 seconds of opening, the catalogue list shows "Rating unavailable" on the row of `9780000000062`, and it still shows "Rating unavailable", not "★ 4.3", after the late summaries have arrived.

**[TC06]: One unreadable entry makes the whole response unreadable — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the books of ISBN `9780000000062` and `9780000000123`; the ratings service answers the summaries with `200`, an entry for `9780000000062` with the count 4 and the average 4.3, and an entry for `9780000000123` with the count 2 and no average.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The catalogue list shows "Rating unavailable" on both rows, and neither "Not yet rated" nor "★" on either.

#### [AC05]: Nothing stands in for the ratings while they load

While the summaries have not yet answered and the catalogue list's bounded time has not passed, the catalogue list shall show nothing in any book's Rating cell.

##### Test cases

**[TC01]: Nothing stands in for the ratings while they load — Happy path:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000062` at once; the ratings service answers the summaries with `9780000000062`, the count 4 and the average 4.3, 3 seconds after receiving the request.
- *Steps:* 1. Open the catalogue list. 2. Look at the list before the summaries answer.
- *Expected result:* The catalogue list lists the book and shows nothing in its Rating column: no "★", no "Not yet rated" and no "Rating unavailable".

**[TC02]: An unrated book is not called unrated before the summaries answer — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000123` at once; the ratings service answers the summaries with no entry for it, 3 seconds after receiving the request.
- *Steps:* 1. Open the catalogue list. 2. Look at the list before the summaries answer. 3. Look again once they have answered.
- *Expected result:* Before the summaries answer, the Rating column of `9780000000123` shows nothing; afterwards it shows "Not yet rated".

#### [AC06]: The catalogue list asks for every book's rating once per load

When the catalogue list loads, the catalogue list shall send exactly one `GET /api/v1/ratings/summaries` request on that load, never one request per book and never reusing an answer from an earlier load ([AD#6]).

##### Test cases

**[TC01]: One request for the whole list — Happy path:**
- *Preconditions:* The books service answers the catalogue list with three books; the ratings service answers the summaries with an entry for one of them.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The ratings service receives exactly one `GET /api/v1/ratings/summaries` and no `GET /api/v1/ratings/summary`.

**[TC02]: A second load asks again and shows the new answer — State / lifecycle:**
- *Preconditions:* The books service answers the catalogue list with the book of ISBN `9780000000123`; the ratings service answers its first summaries request with no entry for it, and every later one with `9780000000123`, the count 1 and the average 4.
- *Steps:* 1. Open the catalogue list. 2. Open the catalogue list again.
- *Expected result:* The ratings service receives two `GET /api/v1/ratings/summaries` requests, and the second load shows "★ 4.0" on the row of `9780000000123`.

**[TC03]: Fifty books still take one request — Negative / boundary:**
- *Preconditions:* The books service answers the catalogue list with fifty books; the ratings service answers the summaries with an entry for each.
- *Steps:* 1. Open the catalogue list.
- *Expected result:* The ratings service receives exactly one `GET /api/v1/ratings/summaries`.

---

### [U05]: Know that only Quillmark sees my comment

As a customer who rates a book, I want the forms to tell me that only Quillmark sees my comment, and no screen to show it, so that I am not misled into thinking I am publishing a review.

Done when both forms carry the comment note and the keep-comment text, and neither the Ratings list, the rating details screen nor the update form shows a comment.

Traces to: [US#7]; [AC#5], [AC#12], [AC#30]; the PRD-level specification's [U05]; [CD#17]; [AD#7]; [CG#10].

#### [AC01]: The forms say only Quillmark sees the comment

The rating form, and the update form once it has loaded a rating, shall each show at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

##### Test cases

**[TC01]: The rating form carries the note — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating.
- *Expected result:* The rating form shows at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

**[TC02]: The update form carries the same note — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7.
- *Expected result:* The update form shows at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

**[TC03]: The note stays while a refusal keeps the form open — Negative / boundary:**
- *Preconditions:* The ratings service answers every give with `403`.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating. 2. Enter `browser@example.com` and `9780000000017`, and choose 3 stars. 3. Select Submit.
- *Expected result:* The rating form, open with the refusal shown, still shows at the comment field exactly "Only Quillmark sees your comment. It is not shown to other shoppers."

#### [AC02]: The forms say an empty comment keeps the earlier one

The rating form, and the update form once it has loaded a rating, shall each show at the comment field exactly "Leave this empty to keep any comment you gave before."

##### Test cases

**[TC01]: The rating form says an empty comment keeps the earlier one — Happy path:**
- *Preconditions:* None.
- *Steps:* 1. Open the rating form from the Ratings list's New Rating.
- *Expected result:* The rating form shows at the comment field exactly "Leave this empty to keep any comment you gave before."

**[TC02]: The update form says the same — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7.
- *Expected result:* The update form shows at the comment field exactly "Leave this empty to keep any comment you gave before."

**[TC03]: The text stays while a refusal keeps the update form open — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars, and it answers every change with `503`.
- *Steps:* 1. Open the Ratings list and select Update on rating 7. 2. Choose 4 stars and select Submit.
- *Expected result:* The update form, open with the refusal shown, still shows at the comment field exactly "Leave this empty to keep any comment you gave before."

#### [AC03]: The update form does not show the stored comment

When the update form opens and loads a rating, the update form shall show an empty comment field, whatever comment the rating holds.

##### Test cases

**[TC01]: A rating read with a comment opens with an empty comment field — Happy path:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017`, 3 stars and the comment "Great plot", as the ratings service did before this release.
- *Steps:* 1. Open the Ratings list and select Update on rating 7.
- *Expected result:* The update form shows an empty comment field, and "Great plot" appears nowhere on it.

**[TC02]: A rating with no comment opens with an empty comment field — Negative / boundary:**
- *Preconditions:* Rating 7 is `reader@example.com`'s rating of ISBN `9780000000017`; the ratings service answers the read of rating 7 with the e-mail `reader@example.com`, the ISBN `9780000000017` and 3 stars.
- *Steps:* 1. Open the Ratings list and select Update on rating 7.
- *Expected result:* The update form shows an empty comment field.

#### [AC04]: The Ratings list shows no comment

The Ratings list shall show for each rating its e-mail, ISBN, stars and Updated At, under the heading "LastUpdated", with no comment column ([AD#7]).

##### Test cases

**[TC01]: The Ratings list shows a rating's fields without a comment — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list with `reader@example.com`'s 4-star rating of ISBN `9780000000017`, updated at `2026-10-13T10:00:00`.
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The Ratings list shows `reader@example.com`, `9780000000017`, 4 and `2026-10-13T10:00:00` under the heading "LastUpdated" for that rating, and no comment column.

**[TC02]: No comment text appears even when a read carries one — Security / privacy:**
- *Preconditions:* The ratings service answers the Ratings list with three ratings carrying the comments "Great plot", "Slow start" and "Gift for my mother", as the ratings service did before this release.
- *Steps:* 1. Open the Ratings list. 2. Search the whole screen for the three comments.
- *Expected result:* The Ratings list shows none of the three comments.

**[TC03]: A rating with no comment shows the same columns — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list with `second@example.com`'s 3-star rating of ISBN `9780000000017`, carrying no comment.
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The Ratings list shows `second@example.com`, `9780000000017`, 3 and its Updated At for that rating, and no comment column.

#### [AC05]: The rating details screen shows no comment

When the rating details screen opens and loads a rating, the rating details screen shall show its ISBN, e-mail, stars, Updated At and Created At, with no comment ([AD#7]).

##### Test cases

**[TC01]: The details of a rating — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list and the read of rating 7 with `reader@example.com`'s 4-star rating of ISBN `9780000000017`, updated at `2026-10-13T10:00:00` and created at `2026-10-12T09:00:00`.
- *Steps:* 1. Open the Ratings list and select Details on rating 7.
- *Expected result:* The rating details screen shows `9780000000017`, `reader@example.com`, 4, `2026-10-13T10:00:00` as Updated At and `2026-10-12T09:00:00` as Created At, and no comment.

**[TC02]: No comment appears even when the read carries one — Security / privacy:**
- *Preconditions:* The ratings service answers the Ratings list and the read of rating 7 with `reader@example.com`'s 4-star rating of ISBN `9780000000017` and the comment "Great plot", as the ratings service did before this release.
- *Steps:* 1. Open the Ratings list and select Details on rating 7.
- *Expected result:* The rating details screen shows no comment label and not "Great plot".

**[TC03]: No comment label for a rating with no comment — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list and the read of rating 8 with `second@example.com`'s 3-star rating of ISBN `9780000000017`, carrying no comment.
- *Steps:* 1. Open the Ratings list and select Details on rating 8.
- *Expected result:* The rating details screen shows no comment label or value.

---

### [U06]: Trust that ratings are not removed from the storefront

As a shopper reading the store's ratings, I want no storefront screen to remove a rating, so that a book's average moves only when a customer rates it or an operator removes a single rating.

Done when the Ratings list offers no removal and the storefront sends no removal request to the ratings service.

Traces to: [US#5]; [AC#23]; the PRD-level specification's [U06]; [VD#1]; [AD#5]; [CG#4].

#### [AC01]: The Ratings list offers no removal

The Ratings list shall offer each rating only the Update and Details actions, and the list as a whole only New Rating ([AD#5]).

##### Test cases

**[TC01]: A rating's actions — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list with `reader@example.com`'s rating of ISBN `9780000000017`.
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The Ratings list offers Update and Details on that rating, and no Delete.

**[TC02]: No Delete All on the list — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list with five ratings.
- *Steps:* 1. Open the Ratings list. 2. Look for any action that removes ratings.
- *Expected result:* The Ratings list offers New Rating, no Delete All, and no action that removes a rating.

#### [AC02]: The storefront sends no removal request

The storefront shall send the ratings service no removal request, neither `DELETE /api/v1/ratings/{id}` nor `DELETE /api/v1/ratings/delete-all`, from any screen ([AD#5]).

##### Test cases

**[TC01]: Using every ratings screen sends no removal — Happy path:**
- *Preconditions:* The ratings service answers the Ratings list and the read of rating 7 with `reader@example.com`'s rating of ISBN `9780000000017`, and answers every give and change with `200` and the saved rating.
- *Steps:* 1. Open the Ratings list and select Details on rating 7. 2. Open the Ratings list and select Update on rating 7, choose 4 stars and select Submit. 3. From the Ratings list, select New Rating, enter `reader@example.com` and `9780000000017`, choose 5 stars and select Submit.
- *Expected result:* The ratings service receives no `DELETE` request of any kind.

**[TC02]: An empty Ratings list sends no removal — Negative / boundary:**
- *Preconditions:* The ratings service answers the Ratings list with no ratings.
- *Steps:* 1. Open the Ratings list.
- *Expected result:* The ratings service receives no `DELETE` request of any kind.

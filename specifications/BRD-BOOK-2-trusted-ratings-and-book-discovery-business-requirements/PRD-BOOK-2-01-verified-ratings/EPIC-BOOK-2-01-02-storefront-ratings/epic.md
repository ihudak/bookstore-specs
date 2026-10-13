---
kind: epic
key: BOOK-2-01-02
target: bookstore:web
kept_whole: "It's a small change, and that's my decision to keep one Epic per module"
---

# Shoppers see trustworthy ratings and rate only what they bought

## Goal
The storefront lets a buyer give or change a 1 to 5 star rating and see why a rating was refused, and shows each book's average rating with its count on the book page and its average on the catalogue list, so that shoppers can trust the stars (PRD [[BOOK-2-01]], "Verified ratings").

## Business value
Shoppers can tell a broad opinion from a single one before opening or buying a book, and customers who rate are told plainly that only buyers can rate and that only Quillmark sees their comment, which keeps the stars credible and the form honest.

## Scope

### In scope
- The rating form: exactly five star choices 1 to 5 with none chosen on open; Submit held back while the e-mail, the ISBN or the star choice is missing; the comment note "Only Quillmark sees your comment. It is not shown to other shoppers."; the entered values kept after a refusal; the Ratings list opened after a stored give.
- The update form: heading "Update Rating", the rating's e-mail and ISBN shown and not editable, the five star choices with the current stars chosen, an empty comment field with the same comment note, the entered stars and comment kept after a refusal, the Ratings list opened after a saved change.
- The texts shown for each outcome, verbatim from the specification: the buyers-only refusal "Only customers who have bought this book can rate it.", the unavailable text "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes.", the cannot-save text "We can't save your rating right now. Please try again in a few minutes.", "This rating no longer exists. Nothing was saved." on a change, "This rating no longer exists." and "We can't load this rating right now. Please try again in a few minutes." in place of the update form's fields, and, for an invalid write, the reason from the `message` field of the error body.
- The Ratings list without Delete, Delete All and the comment column, keeping each rating's e-mail, ISBN, stars, Updated At, the Update and Details actions, and New Rating; the service methods that call per-rating delete and `delete-all` removed.
- The rating details screen without the comment, showing ISBN, e-mail, stars, Updated At and Created At.
- The book page rating line "★" and the average to one decimal with the count in parentheses ("(1 rating)", "(37 ratings)"), "Not yet rated" for an unrated book, and "Rating unavailable" when the summary cannot be loaded, the book's other details shown as usual.
- The catalogue list "Rating" column after "OnVend": "★" and the average to one decimal with no count, "Not yet rated" for an unrated book, "Rating unavailable" in every row when the summaries cannot be loaded, and the books listed as usual.
- The web model of a rating without its comment, a consumer-side summary model, and unit tests with the ratings service stubbed (for example with `HttpClientTestingModule`).

### Out of scope
- Any rating rule enforced for non-storefront callers; those live in bookstore:ratings ([[BOOK-2-01-01]]).
- Showing comments to anyone, an operator screen for comments, sign-in and notifications.
- Averages anywhere other than the book page and the catalogue list ([[BOOK-2-02]]).
- Load-failure texts for the Ratings list and the rating details screen.
- Any storefront screen that removes a rating, and any change to the load generator or to bookstore:orders.
- Rounding or averaging in the browser; the figures come from the ratings summary endpoints as returned.

## Acceptance criteria
- Given the rating form opens, then it offers exactly the star choices 1, 2, 3, 4 and 5 with none chosen.
- Given the rating form opens, then it shows the comment note.
- Given the rating form is open, when the e-mail, the ISBN or the star choice is missing, then it keeps Submit from sending.
- Given the ratings service refuses a give or a change from a form as buyers-only, unavailable or invalid, or the form cannot reach the ratings service or gets any other answer, then the form shows the matching text from this Epic's scope (for an invalid write, the reason from the `message` field of the error body) and stays open with the entered values (e-mail, ISBN, stars and comment on the rating form; stars and comment on the update form).
- Given the ratings service stores a give or saves a change, then the form opens the Ratings list.
- Given the update form opens for a rating that holds a comment, then it shows the heading "Update Rating", the rating's e-mail and ISBN as values that cannot be edited, the five star choices with the rating's current stars chosen, the comment note, and an empty comment field.
- Given the update form opens for a rating that no longer exists, then it shows "This rating no longer exists." in place of its fields.
- Given the update form opens for a rating that cannot be loaded, then it shows "We can't load this rating right now. Please try again in a few minutes." in place of its fields.
- Given a change submitted from the update form reaches a rating that no longer exists, then it shows "This rating no longer exists. Nothing was saved." and the form stays open with its entries.
- Given the Ratings list and the rating details screen, then neither shows a comment.
- Given the Ratings list, then it offers each rating only Update and Details and the list as a whole only New Rating.
- Given the web storefront, then it never calls `delete-all` or per-rating delete.
- Guard: from the Ratings list a customer can still select Update on a rating and change its stars and comment.
- Guard: the Ratings list keeps each rating's rater e-mail and its Update action.
- Guard: the rating form still accepts a comment.
- Given the book page loads a book whose summary holds a count above 0, then its rating line shows "★", the average with one decimal and the count as "(1 rating)" for one rating or "(N ratings)" otherwise; and given the count is 0 then it shows "Not yet rated" alone, with no count.
- Given the summary for the book page cannot be loaded, then the book page shows "Rating unavailable" in place of the rating line, never "Not yet rated", with the book's other details shown as usual.
- Given the catalogue list loads, then a "Rating" column after "OnVend" shows "★" and the average with one decimal and no count for each rated book, and "Not yet rated" for a book absent from a successful summaries response, the same value as that book's page shows at that moment.
- Given the summaries cannot be loaded, then the catalogue list shows "Rating unavailable" in the Rating column of every book, never "Not yet rated", with the books listed as usual.
- Given a rating has been given, changed or removed since a screen last loaded, then the book page and the catalogue list show the figures including it the next time they load.

## Independent Test
This Epic is verifiable standalone by running the storefront against a stub of the ratings write, delete, summary and read interfaces (answering success, each refusal, each failure, rated and unrated books) and delivers the forms, lists and rating figures shoppers see without the ratings service changes being built.

## Dependencies
- [[BOOK-2-01-01]] produces the ratings interfaces ([AD#4], [AD#5], [AD#6], [AD#7]) this Epic consumes. This Epic does not wait for it: its Independent Test runs against a stub of them, so it may be built before [[BOOK-2-01-01]] lands, and it is released after it.
- The existing books service for the book page and the catalogue list (unchanged).
- Release order: ratings lands before web, so the new web never calls a summary endpoint that does not exist yet; between the two, the old web's Delete All fails and its Comment column is empty, which is acceptable for the length of a deploy.

## Contract
- Consumes: [AD#4] — the stub of the ratings write API (`200` without comment, `400` with the reason in the `message` field of the error body, identical `403`, `404`, `503`) this Epic's Independent Test runs against.
- Consumes: [AD#5] — the stub of per-id delete and the absent `delete-all`; web calls neither.
- Consumes: [AD#6] — the stub of `summary` and `summaries` answering `{isbn, count, average}`, a failure, and a book absent from `summaries`.
- Consumes: [AD#7] — the stub of rating reads that carry no comment.

## Covers
- Stories, storefront side: [US#1], [US#2], [US#3], [US#4], [US#5], [US#7]
- [AC#2], [AC#3], [AC#5], [AC#6], [AC#9], [AC#11], [AC#12], [AC#13], [AC#14], [AC#15], [AC#16], [AC#17], [AC#18], [AC#19], [AC#20], [AC#23], [AC#24], [AC#28], [AC#29], [AC#30]
- [UC#1], [UC#2], [UC#4]
- [U01], [U01/AC03], [U01/AC05], [U01/AC06], [U01/AC09], [U01/AC11], [U01/AC12], [U01/AC13], [U01/AC14]
- [U02], [U02/AC08], [U02/AC13], [U02/AC14], [U02/AC15], [U02/AC16], [U02/AC17], [U02/AC18], [U02/AC19], [U02/AC20], [U02/AC22], [U02/AC23]
- [U03], [U03/AC01], [U03/AC02], [U03/AC03], [U03/AC04], [U03/AC05]
- [U04], [U04/AC01], [U04/AC02], [U04/AC03], [U04/AC04], [U04/AC05]
- [U05], [U05/AC01], [U05/AC02], [U05/AC03], [U05/AC04]
- [U06], [U06/AC04]

## Suggested stories
- Give a rating with the rating form: five star choices with none chosen, required fields, the comment note, and the refusal, unavailable, cannot-save and invalid texts with entries kept.
- Change a rating with the update form: the "Update Rating" heading, fixed e-mail and ISBN, current stars chosen, empty comment field, and the changed, no-longer-exists and cannot-load texts.
- Keep the Ratings list and rating details free of comments and deletion: remove Delete, Delete All and the comment column, and the service methods behind them.
- Show how books are rated: the book page rating line and the catalogue list Rating column, with the "Not yet rated" and "Rating unavailable" states.

## References
- Parent PRD: [[BOOK-2-01]]
- [Source: specs/specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/ard.md#Architecture decisions] — [AD#4] to [AD#7] as consumed by web
- [Source: specs/specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/PRD-BOOK-2-01-verified-ratings/specification.md] — storefront texts, verbatim
- [Source: web/src/app/ratings/create-rating/create-rating.component.html] — rating form
- [Source: web/src/app/ratings/update-rating/update-rating.component.html] — update form (editable e-mail and ISBN, "Create Rating" heading)
- [Source: web/src/app/ratings/rating-list/rating-list.component.html] — Delete, Delete All, comment column
- [Source: web/src/app/ratings/rating-details/rating-details.component.html] — comment on the details screen
- [Source: web/src/app/ratings/rating.service.ts] — `deleteRating`, `deleteAllRatings`, ratings base URL
- [Source: web/src/app/books/book-details/book-details.component.ts] — book page loads only the book
- [Source: web/src/app/books/book-list/book-list.component.ts] — catalogue list loads only the books list

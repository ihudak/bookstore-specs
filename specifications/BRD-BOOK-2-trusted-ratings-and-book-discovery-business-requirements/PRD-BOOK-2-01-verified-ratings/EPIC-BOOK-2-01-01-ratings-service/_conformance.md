---
kind: conformance-record
key: BOOK-2-01-01
---

# Conformance: BOOK-2-01-01

- **Run:** Epic — role `dev` — 2026-10-13
- **Code checked:** bookstore at feat/BOOK-2-01-01-buyers-only-ratings-one-per-book @ 7410896
- **Verdicts:** 71 satisfied · 4 partial · 0 missing · 2 contradicts · 0 not-checkable · 0 unverified

## Satisfied

- [U01] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-90@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:82-90@7410896
- [U01]/[AC01] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-67@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:77-79@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:15-16@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-10@7410896
- [U01]/[AC02] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:82-90@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:72-126@7410896
- [U01]/[AC03] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:40-42@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:57-71@7410896
- [U01]/[AC04] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:35@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:132@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingsErrorAdvice.java:21-30@7410896
- [U01]/[AC05] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:34@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:175-181@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingsErrorAdvice.java:28@7410896
- [U01]/[AC06] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:32-33@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:155-172@7410896
- [U01]/[AC07] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:61-63@7410896
- [U01]/[AC08] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:61-66@7410896
- [U01]/[AC09] — ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:37-62@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersUnavailableException.java:9-11@7410896; ratings/src/main/java/com/dynatrace/ratings/config/OrdersClientConfig.java:16-20@7410896; ratings/src/main/resources/application.properties:23-24@7410896
- [U01]/[AC10] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-90@7410896; ratings/src/main/java/com/dynatrace/ratings/model/RatingRequest.java:7@7410896
- [U01]/[AC11] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/Eligibility.java:3@7410896; ratings/src/main/resources/application.properties:27@7410896
- [U02] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:93-113@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-66@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-10@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:103-106@7410896 — the tracer's `partial` dropped at the re-check: the story's "Done when" (stars replaced, one rating per pair, no move) holds, and its comment rule is [U02]/[AC05]'s, recorded as [DD#2]
- [U02]/[AC01] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:108-112@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-66@7410896
- [U02]/[AC02] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:93-97@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:136-139@7410896
- [U02]/[AC04] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-10@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:77-89@7410896
- [U02]/[AC06] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896
- [U02]/[AC07] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:136-143@7410896
- [U02]/[AC08] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:103-112@7410896
- [U02]/[AC09] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:93-94@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:175-181@7410896
- [U02]/[AC10] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:95-96@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:155-172@7410896
- [U02]/[AC11] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:36@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:103-106@7410896
- [U02]/[AC12] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:98@7410896
- [U02]/[AC13] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:37@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:99-102@7410896
- [U02]/[AC14] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:93-108@7410896
- [U02]/[AC15] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:93-108@7410896
- [U02]/[AC16] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:108@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896
- [U02]/[AC17] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:108@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:37-62@7410896
- [U02]/[AC18] — ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:18-22@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-126@7410896
- [U03] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:55-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-126@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:145-149@7410896
- [U03]/[AC01] — ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:18-19@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:145-149@7410896
- [U03]/[AC02] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:119-121@7410896; ratings/src/main/java/com/dynatrace/ratings/model/RatingSummary.java:6@7410896
- [U03]/[AC03] — ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:21-22@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:124-126@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:62-66@7410896
- [U03]/[AC04] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-126@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:18-22@7410896
- [U03]/[AC05] — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-118@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:57@7410896
- [U03]/[AC06] — ratings/src/main/java/com/dynatrace/ratings/model/RatingSummary.java:6@7410896
- [U04] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:52-59@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:104-108@7410896
- [U04]/[AC01] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:52-59@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:68-70@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:77-79@7410896
- [U04]/[AC02] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:56@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:68-70@7410896
- [U04]/[AC03] — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:105-106@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:33-97@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingsErrorAdvice.java:23-29@7410896
- [U04]/[AC04] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:33-79@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:104-108@7410896
- [U05] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:99-104@7410896
- [U05]/[AC01] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:100-104@7410896
- [U05]/[AC02] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:102-104@7410896
- [U05]/[AC03] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:100-104@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingsErrorAdvice.java:21@7410896
- [U05]/[AC04] — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:100-104@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:12-22@7410896
- epic.md "Given an e-mail for which orders reports no completed order for the ISBN (includ…" — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:65-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:108@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:40-42@7410896
- epic.md "Given an otherwise valid give or change for an e-mail with a completed order for…" — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-90@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:108-112@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:105-106@7410896
- epic.md "Given orders times out, cannot be reached or answers with an error other than no…" — ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:37-62@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingsErrorAdvice.java:21-30@7410896
- epic.md "Given a give or a change with no star value or one that is not a whole number fr…" — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:32-34@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:93-108@7410896; ratings/src/main/java/com/dynatrace/ratings/model/RatingRequest.java:7@7410896
- epic.md "Given an e-mail and ISBN that hold no rating or already hold one (compared ignor…" — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-10@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:77-89@7410896; ratings/src/main/resources/application-prod.properties:9@7410896
- epic.md "Given a change, when it carries an e-mail other than the stored rating's (compar…" — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:103-106@7410896
- epic.md "Given a change, when it carries the stored e-mail in other capitals, then the st…" — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:103-112@7410896
- epic.md "Given a saved change, then the rating's Updated At is set to the time of the cha…" — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:136-143@7410896
- epic.md "Given a rating, when a request to `DELETE /api/v1/ratings/delete-all` arrives, t…" — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:100-104@7410896
- epic.md "Guard: when `DELETE /api/v1/ratings/{id}` is requested, then the ratings service…" — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:100-104@7410896
- epic.md "Given stored ratings, when a rating summary is requested for one ISBN or for all…" — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-126@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:145-149@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:18-22@7410896
- epic.md "Given a summary request for an ISBN that is missing or blank, then the ratings s…" — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-118@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:57@7410896
- epic.md "Given any ratings response, when it is a read, a `POST` answer or a `PUT` answer…" — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:104-108@7410896
- epic.md "Given a give that carries a comment, when the rating is stored, then the comment…" — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:52-59@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:77-79@7410896
- epic.md "Guard: a rating given or changed stays and keeps counting in its book's summary…" — ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:18-22@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896
- epic.md Contract "Produces: [AD#5] — `delete-all` removed (refused `400`), per…" — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:99-104@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:12-22@7410896
- epic.md Contract "Produces: [AD#6] — the new `summary` and `summaries` endpoin…" — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:54-66@7410896
- epic.md Contract "Produces: [AD#7] — no ratings response carries the comment.…" — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:104-108@7410896; ratings/src/main/java/com/dynatrace/ratings/model/RatingSummary.java:6@7410896
- epic.md Contract "Consumes: [AD#2] — exists — used as it runs (orders `findByE…" — ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:29-39@7410896; ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:74-77@7410896; ratings/src/main/resources/application-prod.properties:17@7410896
- [AD#1] Rule — ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-134@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-10@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:99-104@7410896
- [AD#2] Rule — ratings/src/main/java/com/dynatrace/ratings/purchase/OrdersPurchaseCheck.java:29-71@7410896; ratings/src/main/java/com/dynatrace/ratings/config/OrdersClientConfig.java:14-20@7410896; ratings/src/main/resources/application.properties:23-24@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:128-134@7410896
- [AD#3] Rule — ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-10@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:65-89@7410896; ratings/src/main/resources/application-prod.properties:9@7410896
- [AD#5] Rule — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:99-104@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:12-22@7410896
- [AD#6] Rule — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:54-66@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:115-126@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:145-149@7410896; ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:18-22@7410896
- [AD#7] Rule — ratings/src/main/java/com/dynatrace/ratings/model/RatingRequest.java:7@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:104-108@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:33-79@7410896

## Unverified

None.

### [DD#1] [U02]/[AC03]

- **document:** specification.md, in BOOK-2-01-01
- **verdict:** partial
- **shipped:** A give for a pair that already holds a rating is applied to it through `applyChange` (stars, Updated At and the no-move rules hold), but not under the comment rule of a change: a blank comment on it clears the stored comment.
- **evidence:** ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:68-74@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896
- **decision:** fix-code — **status:** proposed
- **rationale:** deliberate demo gap, see [ER8]
- **effect:** follow-up proposed: Run `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`: [U02]/[AC03] a repeat give is applied as a change under every rule of a change — a blank comment on a repeat give clears the stored comment instead of keeping it (`Rating.applyChange`)

### [DD#2] [U02]/[AC05]

- **document:** specification.md, in BOOK-2-01-01
- **verdict:** contradicts
- **shipped:** `Rating.applyChange` sets the comment through `commentOrNull`, so an empty, absent or all-space comment on a change — a `PUT`, or a give applied as one — sets the stored comment to null instead of keeping it.
- **evidence:** ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:110-112@7410896; ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:68-74@7410896
- **decision:** fix-code — **status:** proposed
- **rationale:** deliberate demo gap, see [ER8]
- **effect:** follow-up proposed: Run `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`: [U02]/[AC05] an empty, absent or all-space comment on a change keeps the stored comment — it clears it to null (`Rating.applyChange` via `commentOrNull`)

### [DD#3] "Given an e-mail and ISBN that already hold a rating (compared ignoring letter case), when a give reaches the ratings service, then it is applied to that rating as a change under every rule of a change."

- **document:** epic.md, in BOOK-2-01-01
- **verdict:** partial
- **shipped:** The repeat give is applied to the pair's rating through `applyChange`, but not under the comment rule of a change: a blank comment clears the stored comment.
- **evidence:** ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:68-74@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896
- **decision:** fix-code — **status:** proposed
- **rationale:** deliberate demo gap, see [ER8]
- **effect:** follow-up proposed: Run `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`: epic.md acceptance criterion "a give for a pair that already holds a rating is applied as a change under every rule of a change" — a blank comment on that give clears the stored comment (`Rating.applyChange`)

### [DD#4] "Given a saved change, when its comment is empty, absent or only spaces, then the stored comment is kept; when it holds a character other than a space, then it replaces the stored comment."

- **document:** epic.md, in BOOK-2-01-01
- **verdict:** contradicts
- **shipped:** A comment holding another character replaces the stored one as given; an empty, absent or all-space comment sets the stored comment to null instead of keeping it.
- **evidence:** ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896
- **decision:** fix-code — **status:** proposed
- **rationale:** deliberate demo gap, see [ER8]
- **effect:** follow-up proposed: Run `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`: epic.md acceptance criterion "a saved change with an empty, absent or all-space comment keeps the stored comment" — it clears it to null (`Rating.applyChange`)

### [DD#5] "Produces: [AD#4] — the changed write API: statuses, one identical `403`, `503`, fixed e-mail and ISBN on a change, comment kept when empty, the saved rating returned without its comment."

- **document:** epic.md (## Contract), in BOOK-2-01-01
- **verdict:** partial
- **shipped:** Every clause of the produced write API holds except "comment kept when empty": a blank comment on a change clears the stored comment.
- **evidence:** ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-113@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896; ratings/src/main/java/com/dynatrace/ratings/controller/RatingsErrorAdvice.java:21-30@7410896
- **decision:** fix-code — **status:** proposed
- **rationale:** deliberate demo gap, see [ER8]
- **effect:** follow-up proposed: Run `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`: epic.md Contract "Produces: [AD#4]" — the write API's "comment kept when empty" clause: a blank comment on a change clears the stored comment (`Rating.applyChange`)

### [DD#6] [AD#4] Rule

- **document:** ard.md, in BOOK-2-01
- **verdict:** partial
- **shipped:** The give-or-change flow, the statuses, the identical `403`, the `503`, the checks decided before orders and the fixed e-mail and ISBN on a change hold; the clause "an absent or empty comment keeps the stored comment, and a non-empty one replaces it ([AC#30])" does not — `applyChange` writes null.
- **evidence:** ratings/src/main/java/com/dynatrace/ratings/service/RatingService.java:60-113@7410896; ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:62-70@7410896
- **decision:** fix-code — **status:** proposed
- **rationale:** deliberate demo gap, see [ER8]
- **effect:** follow-up proposed: Run `/dev-workflows:implement BOOK-2-01-01` from repository `bookstore`: [AD#4] Rule "on a change … an absent or empty comment keeps the stored comment" — a blank comment on a change, through `PUT` or a give applied as one, clears the stored comment (`Rating.applyChange`)

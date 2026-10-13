---
kind: proposal
key: BOOK-2-01
title: Verified ratings — effort proposal
date: 2026-10-12
readiness_tier: 3 · Architected
engagement_model: time-and-material
revision_of: none
---

# Effort proposal: Verified ratings (BOOK-2-01)

## 1. Header

| | |
|---|---|
| Prepared by | USoft |
| Prepared for | Quillmark |
| Requirement set | BOOK-2-01 Verified ratings: the verified-ratings part of Quillmark's requirements *Trusted ratings and book discovery* (BOOK-2), as Quillmark's review of 2026-10-12 and the follow-up meeting of the same day settled it |
| Date | 2026-10-12 |
| Readiness tier | **3 · Architected.** The requirement set has a product requirements document, code grounding whose every finding was independently re-derived, a decision register whose interview rounds are all settled, and an architecture document. It has no specification yet, which is what keeps it below tier 4. |
| Highest confidence a package may carry at this tier | High for a package an architecture decision covers; Medium for one no decision covers |
| Engagement model | time-and-material |
| Revision | First revision |

This document states **human delivery hours only**. It carries no rate and no monetary figure of any kind; commercial terms are agreed separately. Every figure below is hours of USoft's delivery team.

## 2. Scope

This proposal estimates the work to deliver Quillmark's requirements 3.1 to 3.7 of section 3 (*Verified ratings*), as Quillmark's review settled them, together with the three additions Quillmark made in section 4 of that review:

- **3.1** — only a customer who has bought a book can rate it, a purchase being an order for that book that is completed and paid and not since cancelled ([CD#1]).
- **3.2** — one rating per customer per book. Its sentence "A rating, once given, is final" is withdrawn ([CD#5]).
- **3.3** — a rating is a whole number of stars from 1 to 5 ([CD#7]).
- **3.4** and **3.5** — the book page shows the book's average rating, rounded half up to one decimal place, and the number of ratings it rests on; a book with no ratings shows "Not yet rated" alone ([CD#8], [CD#9], [VD#3]).
- **3.6** — the catalogue list shows each book's average rating, or "Not yet rated" ([CD#10]: the catalogue list only).
- **3.7** — a customer can change their rating at any time; the change replaces the old one and may alter the stars and the comment ([VD#4]).
- **Review section 4, three additions** — a rating's eligibility is checked when it is given and when it is changed, and a rating stays if its order is later cancelled ([VD#5]); operators, never shoppers, can remove a single rating, and so every rating of one customer, for an erasure request or fraud ([CD#6], [VD#1]). No storefront screen offers removal, but with no sign-in the removal path itself can be reached by anyone who calls the API (risk R8); rating comments are kept but never shown to shoppers, and the rating forms say so ([CD#17]).

The work changes two parts of the store, the ratings service and the storefront (`web/`), and reads the orders service as it is today. The store starts this release with no ratings: Quillmark removed every existing rating from its live store on 2026-10-12 ([CD#19], [CD#20]).

Requirement 3.8 (synthetic monitoring) and the Figure 1 note (a list of one book's ratings opened from its count) are withdrawn from this release ([CD#11], [CD#12]) and are not estimated. Section 20 lists everything else outside scope.

## 3. The naive baseline

What Quillmark's requirements would take if each were read at face value, with no correctness, completeness or enforcement obligation behind it, using the same team and productivity basis as the estimate. It is printed first because the real figure in section 6 means something only against it.

| Requirement | Face-value reading | Hours |
|---|---|---|
| 3.1 | "The store works this way today, and it must keep doing so" — nothing to build | 0 |
| 3.2 | Look for an existing rating before saving one, and refuse a second | 4 |
| 3.3 | The scale is already checked; make the form's input 1 to 5 | 1 |
| 3.4 and 3.5 | The book page fetches the book's ratings, averages and counts them, and shows "Not yet rated" when there are none | 6 |
| 3.6 | The catalogue list does the same for each book | 4 |
| 3.7 | A rating can already be updated — nothing to build | 0 |
| Review section 4 | Operators already have a per-rating removal — nothing to build; hide the comment column on the ratings list | 1 |
| Testing | Check the screens above | 4 |
| **Total** | | **20** |

The estimate in section 6 is **215 hours**. Section 4 explains why it is not 20.

## 4. Cost drivers

Each driver below cites evidence an independent reader can open in this folder: a code-grounding finding that a second agent re-derived ([grounding/code-grounding.md](grounding/code-grounding.md)), a decision frozen in the register ([decisions.md](decisions.md)), or a code defect confirmed for repair. A driver that states a fact about the code cites a grounding finding. The hours column is the share of the expected estimate that each driver accounts for, by package.

| Driver | What it is | Evidence | Packages and hours | Hours |
|---|---|---|---|---|
| [ED#1] | **Buyers only is new work, not existing behaviour.** Creating a rating checks only that the book and the customer exist, then saves; nothing in the ratings service consults orders, and it is configured with no orders address at all (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:72-80`, `120-132`; `ratings/src/main/resources/application-dev.properties:17-18`). The endpoint's own description of its preconditions is also wrong (`RatingController.java:73`), a confirmed defect. | [CG#2] | [WP#2] 22, [WP#6] 5, [WP#9] 2 | 29 |
| [ED#2] | **What counts as a purchase, and when it is checked.** A qualifying order is one for that book that is completed and paid and not since cancelled, checked when a rating is given or changed; a rating stays, and keeps counting, if its order is later cancelled or deleted, and its customer then cannot change it. | [CD#1], [VD#5] | [WP#2] 4 | 4 |
| [ED#3] | **One rating per customer and book is enforced nowhere.** The ratings table carries no unique constraint, the lookup by e-mail and ISBN is declared and never called, and a repeat create inserts a second row; the schema is generated by Hibernate with `ddl-auto=update` (`ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:8-9`; `ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java:16`; `RatingController.java:74-80`; `ratings/src/main/resources/application-prod.properties:9`). | [CG#3] | [WP#3] 8 | 8 |
| [ED#4] | **A repeat rating changes the existing one, and a rating can be changed at any time.** One rating per customer and book holds even though ratings change, so a second add must be applied to the first rating rather than refused, with its comment kept or replaced. | [CD#4], [CD#5] | [WP#3] 6, [WP#6] 4 | 10 |
| [ED#5] | **An update rewrites the whole rating today.** The update endpoint saves the request body under the rating's id, so it can change the customer e-mail and the ISBN, which the update form leaves editable, and it overwrites the rating's creation time, a confirmed defect (`RatingController.java:84-97`; `web/src/app/ratings/update-rating/update-rating.component.html:6`, `11`; `Rating.java:37`, `114-115`). Quillmark's decision fixes the e-mail and the ISBN on every path. | [CG#10], [VD#4] | [WP#2] 6, [WP#6] 2, [WP#9] 3 | 11 |
| [ED#6] | **The star scale is checked only where an omitted value skips it.** The range check sits in the entity setter; a request that leaves out the stars stores 0; the setter's message says "between 0 and 5" while rejecting 0; and the form's number input has no minimum, maximum or step (`Rating.java:36-38`, `86-90`; `RatingController.java:74`, `80`, `86`, `97`; `web/src/app/ratings/create-rating/create-rating.component.html:16`). A confirmed defect. | [CG#5] | [WP#6] 3, [WP#9] 9 | 12 |
| [ED#7] | **No average, count or empty state exists anywhere.** The ratings API lists, finds, creates, updates and deletes, and computes nothing; the book page and the catalogue list load books only and call nothing in ratings; the only rating count in the store is a table-wide total; "Not yet rated" appears nowhere (`RatingController.java:38-108`; `web/src/app/books/book-details/book-details.component.ts:18-21`; `web/src/app/books/book-list/book-list.component.ts:18-20`; `ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32`). The average is rounded half up, so 4.25 shows as 4.3. | [CG#6], [CG#7], [CG#8], [CG#9], [VD#3] | [WP#4] 12, [WP#7] 5, [WP#8] 4 | 21 |
| [ED#8] | **Comments are kept and shown to no shopper.** Quillmark decided that comments are kept, that the rating forms keep taking them and say that only Quillmark sees them, and that no shopper sees one ([CD#17]). The rater is whoever's e-mail is typed, with no sign-in ([CD#14]), so USoft's architecture ([AD#7]) keeps the comment out of every ratings response as well as off every screen. | [CD#17], [CD#14] | [WP#5] 5, [WP#6] 5 | 10 |
| [ED#9] | **Shopper deletion goes; operators keep one-at-a-time removal.** The API offers deletion of one rating and of every rating, and the storefront's ratings list offers Delete on every rating and Delete All (`RatingController.java:101-111`; `web/src/app/ratings/rating-list/rating-list.component.html:22-23`, `33`). Quillmark decided that no shopper removes a rating ([CD#6]) and that delete-all goes. USoft proposes, under [VD#6], that the list keeps each rater's e-mail and its Update action; that answer is on its way to Quillmark (D3). | [CG#4], [VD#1], [CD#6], [VD#6] | [WP#5] 4, [WP#6] 4 | 8 |
| [ED#10] | **Every environment's deploy starts from an empty ratings table.** The ratings schema is generated by Hibernate with `ddl-auto=update`, and the database's init script only creates the database. Nothing in the database holds one rating per customer and book, and the create path accepts the duplicate customer-and-book pairs the load generator posts (`application-prod.properties:9`; `database/mysql/init/01-create-db.sql:5`; `ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:338-340`). Quillmark decided the release starts with no ratings. The architecture reads that as an empty ratings table at each environment's deploy, so the new constraint is created on an empty table and confirmed before anything relies on it (architecture, *Edge cases & risks*). In the live store this empties the table a second time and removes any rating given between 2026-10-12 and the deploy, which needs Quillmark's agreement (D7). | [CG#3], [CD#19] | [WP#10] 7 | 7 |
| [ED#11] | **The unchanged load generator meets the new rules in test environments.** It deletes every rating at the start of each cycle and posts ratings for random customer-and-book pairs (`IngestController.java:335-337`; `ingest/src/main/java/com/dynatrace/ingest/model/Rating.java:37-38`). It stays unchanged and never runs against the live store, so test environments must be checked with it running against the new rules. | [CG#4], [CG#11], [VD#1], [CD#16] | [WP#10] 5 | 5 |
| | **All drivers** | | | **125** |

**From the naive baseline to the estimate:**

| | Hours |
|---|---|
| Naive baseline (section 3) | 20 |
| The eleven drivers above | 125 |
| The two packages every engagement carries, beyond what any driver accounts for: discovery and design ([WP#1], all 38 hours) and the part of test, UAT and release ([WP#10]) no driver or baseline line accounts for (32 of its 48 hours) | 70 |
| **Expected estimate** | **215** |

The largest single driver is [ED#1], buyers only: 29 hours, 13.5 % of the estimate. The two fixed packages together account for 70 hours, 32.6 %.

## 5. Work packages

Each package can be built, tested and accepted on its own; section 19 states how.

| Package | Title | Seam | Requirements (PRD) | Architecture decisions | Confirmed defects |
|---|---|---|---|---|---|
| [WP#1] | Discovery and design | The specification (test cases for every live acceptance criterion, and the texts the architecture left to it) and the engineering design (the seven items the architecture deferred), each approved on its own | All | — | — |
| [WP#2] | Buyers-only rating writes (ratings service) | The ratings API gives and changes a rating only against a qualifying order read from orders, refuses everything else with one identical answer, and keeps a rating's customer and book fixed | [AC#1], [AC#2], [AC#8], [AC#10], [AC#24] | [AD#1], [AD#2], [AD#4] | — |
| [WP#3] | One rating per customer and book (ratings service) | The ratings API holds one rating per customer and book, and applies a repeat add as a change | [AC#7], [AC#30] | [AD#3], [AD#4] | — |
| [WP#4] | Rating summaries (ratings service) | Two new read endpoints return a book's count and average, and every rated book's | [AC#13], [AC#14], [AC#17], [AC#19] (the figures) | [AD#6] | — |
| [WP#5] | No bulk deletion and comment privacy (ratings service) | The ratings API drops delete-all, keeps the per-rating removal, and returns a comment in no response | [AC#12], [AC#21], [AC#23] (the API) | [AD#5], [AD#7] | — |
| [WP#6] | Storefront rating screens (web) | The add and update forms, the ratings list and the rating details screen show and send what the new API expects | [AC#2], [AC#3], [AC#5], [AC#6], [AC#7], [AC#11], [AC#12], [AC#23], [AC#24], [AC#28], [AC#29] (the screens) | [AD#4], [AD#5], [AD#7] | — |
| [WP#7] | Book page rating (web) | The book page shows the average and count, or "Not yet rated" | [AC#13], [AC#14], [AC#15], [AC#16] | [AD#6] | — |
| [WP#8] | Catalogue list averages (web) | The catalogue list shows each book's average, or "Not yet rated" | [AC#17], [AC#18], [AC#19], [AC#20] | [AD#6] | — |
| [WP#9] | Defect remediation (ratings service) | Three confirmed defects repaired: a missing star value stored as 0 and the misleading range message; an update overwriting the creation time; the create endpoint's wrong description of its preconditions | [AC#4], [AC#9] | [AD#4] | [CG#5], [CG#10], [CG#2] |
| [WP#10] | Test, UAT and release | The whole release tested end to end, accepted by Quillmark, and deployed to each environment with the ratings table emptied and the constraint confirmed | [SM#1], [SM#2]; every acceptance criterion end to end | — | — |

[WP#9] repairs defects USoft's own grounding found. Its hours are part of the estimate and it is not offered as an option (section 18).

**The defect sweep confirmed three defects.** It searched the code-grounding findings and USoft's self-review for code defects. There is no code-defect log. It confirmed [CG#5], [CG#10] and [CG#2], all repaired in [WP#9]. It did not confirm [SR#6], the self-review finding that deleting an order erases the only record of the purchase, because [VD#5] keeps orders unchanged in this release. That answer is on its way to Quillmark (D3, risk R9).

## 6. Hours by package and role

Roles from the profile: TL = Tech Lead / Architect (part-time), BE = Backend Engineer, FE = Frontend Engineer, QA = QA Engineer (part-time). The role columns are expected hours. Low and High are each package's range (section 7), to one decimal place.

| Package | TL | BE | FE | QA | Expected | Low | High |
|---|---|---|---|---|---|---|---|
| [WP#1] Discovery and design | 18 | 8 | 6 | 6 | 38 | 30.4 | 53.2 |
| [WP#2] Buyers-only rating writes | 3 | 23 | 0 | 6 | 32 | 25.6 | 44.8 |
| [WP#3] One rating per customer and book | 2 | 13 | 0 | 3 | 18 | 14.4 | 25.2 |
| [WP#4] Rating summaries | 1 | 9 | 0 | 2 | 12 | 10.2 | 15.6 |
| [WP#5] No bulk deletion and comment privacy | 1 | 6 | 0 | 2 | 9 | 7.7 | 11.7 |
| [WP#6] Storefront rating screens | 2 | 0 | 18 | 5 | 25 | 20.0 | 35.0 |
| [WP#7] Book page rating | 1 | 0 | 8 | 2 | 11 | 8.8 | 15.4 |
| [WP#8] Catalogue list averages | 0 | 0 | 6 | 2 | 8 | 6.4 | 11.2 |
| [WP#9] Defect remediation | 1 | 10 | 0 | 3 | 14 | 11.9 | 18.2 |
| [WP#10] Test, UAT and release | 9 | 13 | 7 | 19 | 48 | 38.4 | 67.2 |
| **Total** | **38** | **82** | **45** | **50** | **215** | **173.8** | **297.5** |

At 6 productive hours a developer-day, the expected 215 hours are about 35.8 developer-days.

**How QA was sized.** There is no specification yet, so there is no authored test-case count to size QA from. For the build packages, [WP#2] to [WP#9], QA hours are a stated ratio: **25 % of the package's engineering (BE + FE) hours**, rounded half up to the whole hour. QA hours in [WP#1] (writing the specification's test cases) and [WP#10] (end-to-end testing, test-environment checks and UAT) are sized by the activity. When the specification exists, QA will be re-sized from its test-case count (section 8).

## 7. Confidence and range

**The low and high figures describe a credible range, not best and worst cases.** Each package's band is the default for its confidence grade: High is −15 % / +30 %, Medium is −20 % / +40 %. No band deviates from its grade's default. [WP#5]'s low, 7.65 hours, is shown as 7.7.

| Package | Confidence | Band | Range (hours) | Reason |
|---|---|---|---|---|
| [WP#1] | Medium | −20 % / +40 % | 30.4 – 53.2 | No architecture decision covers discovery itself, so Medium is this package's ceiling. Its scope is bounded by the architecture's list of deferred items: two for the specification and seven for the design. There is no exported design of any screen, so the screens are specified from Quillmark's sketch and today's storefront. |
| [WP#2] | Medium | −20 % / +40 % | 25.6 – 44.8 | [AD#1], [AD#2] and [AD#4] would allow High. It is held at Medium because two design questions that shape it are still open: the orders call's timeout and client, and how the ratings service tells orders' "unknown customer" from a wrong orders address. This is also the ratings service's first call to orders, and no response time for that call has been measured. |
| [WP#3] | Medium | −20 % / +40 % | 14.4 – 25.2 | [AD#3] would allow High. It is held at Medium because the design has not yet settled how an add that loses a race on the new constraint is applied as a change, or how the tests exercise the case-insensitive pair on the in-memory test database. |
| [WP#4] | High | −15 % / +30 % | 10.2 – 15.6 | [AD#6] fixes the endpoints, their shapes, the rounding, and the empty and error cases. Only the aggregate query is left to the design. |
| [WP#5] | High | −15 % / +30 % | 7.7 – 11.7 | [AD#5] and [AD#7] fix both changes completely: one endpoint goes and one field leaves every response. Nothing is deferred. |
| [WP#6] | Medium | −20 % / +40 % | 20.0 – 35.0 | [AD#4], [AD#5] and [AD#7] would allow High. It is held at Medium because two things are left to the specification: what the storefront says when the ratings service cannot reach orders, and the star input's detail. There is also no exported design of these screens. |
| [WP#7] | Medium | −20 % / +40 % | 8.8 – 15.4 | [AD#6] would allow High. It is held at Medium because the wording when a summary request fails is left to the specification, and the only picture of the book page is Quillmark's sketch, which Quillmark called an example rather than a required format. |
| [WP#8] | Medium | −20 % / +40 % | 6.4 – 11.2 | [AD#6] would allow High. It is held at Medium because how the summaries are joined onto the list is left to the design, and there is no exported design of the list. |
| [WP#9] | High | −15 % / +30 % | 11.9 – 18.2 | [AD#4] covers all three repairs. It fixes the star refusal. It fixes that a change alters only the stars and the comment, so the creation time is kept. And it fixes every response the create endpoint gives, which settles what the corrected endpoint description must say. The design still has to decide whether ratings keeps its own client and book existence calls, but that changes no response [AD#4] fixes. A verified finding locates each repair to a file and line ([CG#5], [CG#10], [CG#2]). |
| [WP#10] | Medium | −20 % / +40 % | 38.4 – 67.2 | No architecture decision covers testing and release, so Medium is the ceiling. The release hours assume two environments, and each deploy depends on Quillmark's operators emptying the ratings table first. |
| **Total** | | | **173.8 – 297.5** | Each total is the sum of its column. |

## 8. Re-estimate gates

**No package carries a re-estimate gate.** A gate is mandatory only for a package graded Low, and no package here is Low. The document-level gate applies only to a tier-1 proposal, and this one is tier 3. Nothing is left deliberately unpriced.

**A planned revision, not a gate.** Once the specification exists ([WP#1]), the requirement set reaches tier 4, and USoft will re-run this proposal. It will size QA from the specification's authored test-case count instead of section 6's ratio, and grade each package against the specification. Every figure that moves will be listed, with its cause, in that revision's changelog.

## 9. Delivery approach

The packages are delivered in three increments. The order follows the architecture's landing order: the orders service, whose existing interface is read and which changes nothing, then the ratings service, then the storefront.

1. **Increment 1, week 1: design settled, and the work that needs no further design.** [WP#1] produces the specification and the design. Alongside it, the backend engineer repairs the confirmed defects ([WP#9]) and removes bulk deletion and comments from the API's responses ([WP#5]). The architecture fixes both of those completely. The corrected endpoint description is written to [AD#4]'s responses, which [WP#2] implements in week 2. Nothing ships before week 4.
2. **Increment 2, weeks 2 and 3: the rating rules and the screens.** In the ratings service: buyers-only writes ([WP#2]), then one rating per customer and book ([WP#3]) and the summaries ([WP#4]). In parallel, in the storefront: the rating screens ([WP#6]), the book page ([WP#7]) and the catalogue list ([WP#8]), built against the interfaces the architecture fixes ([AD#4] to [AD#7]).
3. **Increment 3, week 4: one release.** [WP#10] tests the whole release end to end, including with the load generator running in a test environment. Quillmark then runs UAT, and USoft deploys to each environment in landing order, ratings before storefront, after Quillmark's operators have emptied the ratings table.

Each package can be demonstrated when it is finished. Only Increment 3 reaches shoppers.

## 10. Team composition

| Role | Count | Engagement | Hours | Average over the 4-week schedule |
|---|---|---|---|---|
| Tech Lead / Architect | 1 | part-time | 38 | 0.32 FTE |
| Backend Engineer | 1 | | 82 | 0.68 FTE |
| Frontend Engineer | 1 | | 45 | 0.38 FTE |
| QA Engineer | 1 | part-time | 50 | 0.42 FTE |

Each average is the role's hours ÷ 4 weeks ÷ 30 delivery hours a week, rounded to two decimals. The team works with AI coding assistants, and the hours assume 6 productive hours a developer-day and 30 a week.

## 11. Indicative schedule

| Week | Packages | TL | BE | FE | QA | Hours |
|---|---|---|---|---|---|---|
| 1 | [WP#1], [WP#9], [WP#5] | 20 (0.67 FTE) | 24 (0.80 FTE) | 6 (0.20 FTE) | 11 (0.37 FTE) | 61 |
| 2 | [WP#2], [WP#6] | 5 (0.17 FTE) | 23 (0.77 FTE) | 18 (0.60 FTE) | 11 (0.37 FTE) | 57 |
| 3 | [WP#3], [WP#4], [WP#7], [WP#8] | 4 (0.13 FTE) | 22 (0.73 FTE) | 14 (0.47 FTE) | 9 (0.30 FTE) | 49 |
| 4 | [WP#10] | 9 (0.30 FTE) | 13 (0.43 FTE) | 7 (0.23 FTE) | 19 (0.63 FTE) | 48 |
| **Total** | | **38** | **82** | **45** | **50** | **215** |

**Peak concurrency is four people**, one in each role, and it holds in every week. No week needs more than one person in any role, and no role exceeds 30 hours in any week. Each week's FTE is that role's hours that week ÷ 30.

The schedule is built on the expected figures. The high end of the range is 297.5 hours, 38.4 % above expected. If the extra hours fell in proportion across the roles, the backend engineer's hours in weeks 1 to 3 would rise from 69 to about 95.5. Those weeks hold 90 hours, so the backend engineer would be about 5.5 hours over, roughly one developer-day. That would push the release week into a fifth calendar week. No other role would exceed 30 hours in any week. Quillmark's availability for UAT in week 4 is a dependency (section 13).

## 12. Assumptions

1. The productivity basis is engineers working with AI coding assistants, at 6 productive hours a developer-day and 30 delivery hours a week.
2. Quillmark's shoppers use the storefront in this repository, `web/` ([CD#14]).
3. The store has no sign-in in this release, and the rater is whoever's e-mail is typed ([CD#14]). No authentication work is estimated.
4. The orders service is unchanged. The ratings service reads purchases through the orders interface that exists today ([VD#5]).
5. The load generator never runs against the live store ([CD#16]) and is not changed ([VD#1]).
6. The release is deployed to **two environments**: one test environment and Quillmark's live store. Each further environment adds about 3 hours to [WP#10] for its deploy and constraint check.
7. Quillmark's operators empty the ratings table in each environment at the deploy, following USoft's runbook. USoft writes and supports the runbook, and the hours for that are in [WP#10]. Quillmark's operators' own time is not in this estimate. In the live store this depends on Quillmark's agreement under D7.
8. A test environment running the orders, ratings, books, clients and storefront services is available to USoft from week 1.
9. The hours are USoft's alone. Quillmark's time for reviews, specification approval and UAT is not estimated here.
10. The documentation the PRD lists as new (shopper help, an operator guide for removing ratings, and a reference note on the ratings operations) is written in [WP#10], 6 hours, as draft pages for Quillmark to publish.

## 13. Dependencies

**The register leaves nothing open.** The sweep found:

- no open assumption record in the register;
- no customer question left unanswered in any interview round: the three questions that were not frozen (round 1, questions 26, 27 and 30) were closed as moot when their requirements were withdrawn;
- no code defect recorded as blocking.

So nothing from the register is outstanding before week 1.

The work depends on:

| # | Dependency | Who | Needed by |
|---|---|---|---|
| D1 | Access to the `bookstore` repository, and a test environment running the orders, ratings, books, clients and storefront services | Quillmark | **Before week 1** |
| D2 | Quillmark's Head of E-commerce available to approve the specification: its test cases, the text shown when ratings cannot reach orders, and the text shown when a rating summary cannot load | Quillmark, Head of E-commerce | **Week 1** |
| D3 | Quillmark's owners receive USoft's answers to the three points section 10 of the review left with USoft: how the operators' removal path is kept from shoppers ([VD#1]), how purchase records relate to deleted orders ([VD#5]), and what remains of the storefront's ratings list ([VD#6]). The review says none of them stops work from starting. | USoft sends; Head of E-commerce and Operations Lead receive | Before UAT, week 4 |
| D4 | Quillmark's operators stop the old ratings service and, where it runs, the load generator; empty the ratings table; and start the new ratings service, in each environment | Quillmark, Operations Lead | Week 4, at each deploy |
| D5 | The orders service's address is reachable from the ratings service in each environment | Quillmark, Operations Lead | Week 4 |
| D6 | UAT participants from Quillmark's e-commerce team | Quillmark, Head of E-commerce | Week 4 |
| D7 | Quillmark agrees that the live store's ratings table is emptied again at the deploy. This is the architecture's reading of [CD#19], and it removes any rating given between 2026-10-12 and the deploy. | Quillmark, Head of E-commerce and Operations Lead | Before the live deploy, week 4 |

## 14. Risks

| # | Risk | Effect | Mitigation |
|---|---|---|---|
| R1 | Every give and change now calls the orders service, which in turn calls the clients service twice (architecture, *Edge cases & risks*). No response time has been measured. | Slow rating writes; more load in test environments, where most of the load generator's ratings are refused | A bounded timeout ([AD#2]), with the value set in [WP#1]'s design. Writes are measured in [WP#10]. |
| R2 | A wrong orders address would make every lookup look like an unknown customer, so every buyer would be refused | No rating can be given in that environment | The design ([WP#1]) settles how the ratings service tells the two apart. The release checks it in each environment ([WP#10]). |
| R3 | If any row remains in the ratings table at deploy, Hibernate cannot create the unique constraint, and it logs the failure and starts anyway | One rating per customer and book would not be guaranteed | The runbook empties the table and confirms the constraint from the table's indexes before the release is relied on (D4). |
| R4 | The typed-e-mail identity lets anyone rate, or change a rating, under a buyer's e-mail ([CD#14]). USoft reads raters' e-mails staying visible on the ratings list as part of that identity ([VD#6]). | The rules hold, but not against impersonation | Quillmark accepted the typed-e-mail identity in its review ([CD#14]). USoft's reading of the visible e-mails reaches Quillmark under D3. Not estimated. |
| R5 | A paid order moved to another customer or book through the order update still reads as completed (architecture, *Edge cases & risks*) | A rating for a book the customer did not pay for counts against [SM#1] | Accepted under [VD#5], which keeps orders unchanged. Closing it is an orders change for a later release. |
| R6 | Orders matches an e-mail exactly as stored, so a buyer who types their e-mail with different capitals is refused | Refusals counted in [SMC#1] | Accepted under [VD#5]. |
| R7 | The all-books summary returns every rated book in one response | Its response grows with the catalogue | Paging is left for when the catalogue needs it, and is not estimated (section 20). |
| R8 | The operators' per-rating removal can be reached by anyone who calls the ratings API. With no sign-in, nothing tells an operator's request from a shopper's (architecture, [AD#5]). Quillmark's review requires that operators, never shoppers, remove ratings, and it asks how that path is kept from shoppers. | A shopper outside the storefront's screens could remove a rating | USoft's answer ([VD#1]) accepts the risk until the store has sign-in. Quillmark has not yet received it (D3). If Quillmark does not accept it, section 15 applies. |
| R9 | The storefront's order Delete and Delete All erase the only record that a purchase happened. Orders stay unchanged in this release ([VD#5]). Quillmark's review asks how purchase records survive them. | A rating already given stays and keeps counting, but its customer can no longer change it. A shopper using Delete All would end that right for every buyer at once. | USoft's answer ([VD#5]) keeps orders unchanged and accepts this. Quillmark has not yet received it (D3). If Quillmark does not accept it, section 15 applies. |

## 15. What the range does not cover

The range in section 7 is computed under the conditions below. If one of them fails, the actual figure can fall outside the range: **the high figure is not a ceiling.** Section 20 is a different list: it holds the work that was never estimated.

- **A frozen decision reversed.** For example, Quillmark asks for the orders service to change after all ([VD#5]), for sign-in ([CD#14]), for ratings to become final again ([CD#5]), or for comments to be shown ([CD#17]).
- **Quillmark not accepting an answer still on its way to it.** That could be one of USoft's three answers under D3: the removal path ([VD#1], R8), deleted orders ([VD#5], R9), or the ratings list ([VD#6]). Or it could be the emptying of the live ratings table at the deploy (D7).
- **More callers of the ratings API than the evidence records.** The architecture finds every caller in this repository: the storefront and the load generator. Another system writing ratings, or reading their comments, in Quillmark's live store would add work.
- **A deferred design item resolved in the widening direction.** For example, telling an unknown customer from a wrong orders address turns out to need a change to the orders service, or the summaries need paging now.
- **Customer-side delay** on a named dependency, D1 to D6 in section 13: specification approval, the table emptying at deploy, or UAT.
- **The orders call proving too slow** under measured load (R1), so that the orders service needs a lighter lookup.
- **More than two environments** to deploy into (assumption 6), beyond about 3 hours for each further environment.

## 16. Engagement governance

Under **time and material**, Quillmark is invoiced on the hours actually worked, on commercial terms agreed separately. The range in section 7 is USoft's forecast, and governance tracks the actual hours against it:

- **Cadence.** A weekly status report to Quillmark's Head of E-commerce. A short review at the end of each increment (section 9).
- **Reporting.** Each week's report gives the hours worked against each package's expected and high figures, the forecast to complete, and any package that has passed its expected figure. USoft raises a package as soon as its forecast passes its high figure, before more hours are spent on it.
- **Who accepts.** For Quillmark, the Head of E-commerce accepts the rating rules and how they are shown, [WP#1] to [WP#9]. The Operations Lead accepts the operating side: the deploy runbook and the operator guide in [WP#10]. That split matches the ownership in section 9 of Quillmark's review. For USoft, the Tech Lead / Architect is accountable for every package.

## 17. Change control

Quillmark raises a change through its account manager, quoting the requirement number, as section 6 of its requirements asks.

Under time and material, a change does not need a priced change order before the work starts. It does need a re-forecast:

- USoft states which packages the change touches and how their expected and high figures move.
- If the change touches a decision frozen in the register, such as a reversal from section 15, the decision is reopened and settled first.
- After that, this proposal is re-run, and its changelog names the change as the cause of every figure that moved.

Work on a change starts once Quillmark accepts the re-forecast.

## 18. Scope levers and priced options

Under time and material, a lever moves the forecast, not a price.

| Lever | Effect on the range | What Quillmark gives up |
|---|---|---|
| **Defer [WP#8]**, catalogue list averages, to a later release | Removes 8 hours (6.4 – 11.2) | Requirement 3.6 in this release. The all-books summary that [WP#4] builds still ships, so the later release needs only the list itself. |
| **Defer [WP#7] and [WP#8]**, both storefront displays | Removes 19 hours (15.2 – 26.6) | Requirements 3.4, 3.5 and 3.6 in this release. Ratings become trustworthy, but no shopper sees an average yet. |

Packages that are not levers:

- [WP#1] and [WP#10] are part of every delivery.
- [WP#2], [WP#3], [WP#5] and [WP#6] carry rules Quillmark itself decided ([CD#1], [CD#4], [CD#6], [CD#17]).
- [WP#4] serves both displays.
- [WP#9] repairs defects USoft's own grounding found, so it is not an option to decline.

**Priced options:** none. The items Quillmark may want later are outside this requirement set, so none is priced here. These include sign-in, an orders change that keeps purchase records when orders are deleted, e-mail matching that ignores capitals in orders, and paging of summaries. Each would be estimated when asked for.

## 19. Acceptance

Each package is accepted on its own, against the criteria below. The specification ([WP#1]) turns each criterion into test cases.

| Package | Accepted when |
|---|---|
| [WP#1] | Quillmark's Head of E-commerce approves the specification, and USoft's Tech Lead / Architect approves the design. |
| [WP#2] | Through the ratings API, a give or a change is saved only with a qualifying order ([AC#1], [AC#8]). Every refusal gives one identical answer ([AC#2]). A rating whose order is later cancelled or deleted stays and counts ([AC#10]). No path changes a rating's customer e-mail or book ([AC#24]). When orders cannot answer, nothing is saved. |
| [WP#3] | A repeat add changes the existing rating ([AC#7]). An empty comment keeps the stored one, and a new comment replaces it ([AC#30]). Two concurrent adds for one customer and book leave exactly one rating. |
| [WP#4] | The two summary endpoints return the count and the average rounded half up (4.25 as 4.3), return no average for a book with no ratings, and carry no e-mail and no comment. |
| [WP#5] | Bulk deletion no longer exists ([AC#23]). The per-rating removal behaves exactly as before ([AC#21]). No ratings response carries a comment ([AC#12]). |
| [WP#6] | The storefront's screens meet [AC#2]'s text, [AC#3], [AC#5], [AC#6], [AC#11], [AC#12], [AC#23], [AC#28] and [AC#29], and the add form applies a repeat as a change ([AC#7]). |
| [WP#7] | The book page meets [AC#13], [AC#14], [AC#15] and [AC#16]. |
| [WP#8] | The catalogue list meets [AC#17], [AC#18], [AC#19] and [AC#20]. |
| [WP#9] | A missing or out-of-range star value is refused on every write ([AC#4], [AC#9]). A change keeps the rating's creation time. The create endpoint's description states its real preconditions. |
| [WP#10] | The release passes end to end, including with the load generator running in a test environment, and Quillmark's Head of E-commerce signs off UAT. Each environment is deployed with the unique constraint confirmed. The go-live checks for [SM#1] and [SM#2] return 100 % and 0. The Operations Lead accepts the runbook and the operator guide. |

## 20. Exclusions from scope

None of the following is estimated:

- Synthetic monitoring that buys a book and then rates it: requirement 3.8, withdrawn ([CD#11]).
- A list of one book's ratings opened from its count: the Figure 1 note, withdrawn ([CD#12]).
- Written reviews, their moderation, and showing comments to shoppers ([CD#17]). Also any operator screen or endpoint for reading comments: Quillmark reads them in the ratings database ([AD#7]).
- A customer removing their own rating ([CD#6]), and a single action that removes every rating of one customer, which is the per-rating removal repeated ([VD#1]).
- Moving a rating to another customer or book ([VD#4]).
- Sign-in, or any check that the person typing an e-mail owns it ([CD#14]).
- Average ratings anywhere other than the book page and the catalogue list: search results, book suggestions and the Top rated shelf belong to the sibling slice BOOK-2-02 ([CD#10]).
- Any change to orders, checkout, payment or cancellation ([VD#5]). This includes keeping a purchase record when an order is deleted (R9). USoft's own self-review raised that ([SR#6]), and the defect sweep did not confirm it as a defect this engagement repairs (section 5).
- Notifications, and a mobile app.
- Changes to the load generator ([VD#1], [CD#16]).
- Any clean-up of existing ratings: Quillmark removed them before go-live ([CD#19], [CD#20]). The table emptying at each deploy (D4, and for the live store D7) is Quillmark's operators' work, supported by USoft's runbook.
- A database migration tool, or a start-up check on the constraint (architecture, *Edge cases & risks*).
- Paging of the all-books summary (R7).

## 21. Traceability

Quillmark's requirements are cited by their own numbers in [Quillmark's requirements](../brd/source/quillmark-trusted-ratings-brd.md) and [its review](customer-review-20261012.md), and USoft's requirements by their identifiers in [prd.md](prd.md). Architecture decisions are in [ard.md](ard.md), decisions in [decisions.md](decisions.md), and code-grounding findings in [grounding/code-grounding.md](grounding/code-grounding.md).

| Quillmark's requirement | PRD | Packages |
|---|---|---|
| 3.1 — only a buyer can rate a book | [US#1]: [AC#1], [AC#2]; [US#2]: [AC#8] | [WP#2], [WP#6], [WP#10] |
| 3.2 — one rating per customer per book ("final" withdrawn, [CD#5]) | [US#2]: [AC#7] | [WP#3], [WP#6] |
| 3.3 — whole stars from 1 to 5 | [US#1]: [AC#3], [AC#4]; [US#2]: [AC#9] | [WP#6], [WP#9] |
| 3.4 — average and count on the book page | [US#3]: [AC#13], [AC#14], [AC#16] | [WP#4], [WP#7] |
| 3.5 — "Not yet rated" | [US#3]: [AC#15]; [US#4]: [AC#18] | [WP#7], [WP#8] |
| 3.6 — average on the catalogue list | [US#4]: [AC#17], [AC#19], [AC#20] | [WP#4], [WP#8] |
| 3.7 — change a rating at any time | [US#2]: [AC#6], [AC#7], [AC#11], [AC#29], [AC#30] | [WP#3], [WP#6] |
| 3.8 — synthetic monitoring | withdrawn ([CD#11]) | not estimated |
| Figure 1 note — a book's ratings list | withdrawn ([CD#12]) | not estimated |
| Review section 4 — eligibility checked when a rating is given or changed | [US#2]: [AC#8], [AC#10] | [WP#2] |
| Review section 4 — operators remove a single rating, or every rating of one customer | [US#5]: [AC#21], [AC#23], [AC#24] | [WP#5], [WP#6], [WP#2] |
| Review section 4 — comments stored, never shown to shoppers | [US#7]: [AC#5], [AC#12], [AC#28] | [WP#5], [WP#6] |
| The rules hold however a rating arrives (PRD, *Non-functional requirements*) | — | [WP#2], [WP#3], [WP#5], [WP#9] |
| Success measures | [SM#1], [SM#2] | [WP#10] |

---

*This estimate is indicative, valid for 30 days, and states human delivery hours only; commercial terms are agreed separately.*

<!-- priced-against
$SPECS_PATH/.dev-workflows/proposal-profile.yml 124b2d7f9d3054f6498a8e6cf504a3b60558d96d
ard.md 2895dfd5934307ded2d1aa97160613a36bf67a44
decisions.md d390fd150014e9d5e3b7c479435cd88835db4979
grounding/baselines.md 40d9aa7eb4dd49ff4d9d4ac8e3a0d07206ad744e
grounding/code-grounding.md 11dd20735bd718cb4a22331ea1fb924baf93612b
grounding/design-grounding.md 6bfdabd22a31c23ff6e51d33d99f9538b637b797
interview/customer-questions.md 7fabbf23ac5d47dd100032595b542c811ff7f782
interview/round-1.md e4f47b8194b06cc1c7ca5638b5a3f56a540b3c12
interview/round-2.md 639cde6bdec2992d31a45c42e4c19dd5c42aae54
interview/round-3.md 285ed2880335817d196387de7c09c4086ffc39f6
prd.md a24ea747b75185216c1120dd72e1e2106fb17e2a
self-review-20261012.md 7c8ca4b0a31371ea66ade902020539b4130c499b
-->

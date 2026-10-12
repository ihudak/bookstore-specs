# Decision register: BOOK-2-01

id: [VD#1]
statement: Shoppers cannot delete ratings: the storefront offers no rating Delete or Delete All, and the ratings API drops its delete-all. Operators remove a single rating through the API's existing per-rating delete, which no storefront screen calls. No clean-up of existing ratings is part of this release ([CD#19]), and the load generator is unchanged.
options_considered: ["Remove shopper deletion: drop the storefront's Delete and Delete All and the API's delete-all; keep the API's per-rating delete as the operators' removal path, called by no storefront screen; leave the load generator unchanged", "As the first, and also build an operator removal of every rating of one customer and change the load generator's four rating-deletion paths", "Keep bulk delete as an operator-only maintenance action outside the storefront", "Leave deletion as it is today"]
chosen: "Remove shopper deletion: drop the storefront's Delete and Delete All and the API's delete-all; keep the API's per-rating delete as the operators' removal path, called by no storefront screen; leave the load generator unchanged"
argumentation: |
  Requirement 3.2 makes a given rating final, and 3.4 and 3.6 publish averages and counts over all
  ratings. Any path that deletes ratings in bulk — the storefront's Delete All, or the load
  generator's per-cycle wipe — silently resets every average a shopper sees, which is the very
  distrust section 3 sets out to fix. Removing bulk deletion leaves no route by which ratings
  disappear, except individual removal if the customer allows it (round 1, question 9). The answer
  would change if Quillmark needs a way to purge ratings (legal erasure, fraud clean-up), which
  would favour an operator-only path.

  Reopened 20261012: by [CD#2], [CD#3], [CD#5], [CD#6] and [CD#16], from the customer's returned review (customer-review-20261012.md). [CD#5] withdraws 3.2's finality, on which this reasoning opens; [CD#2] and [CD#3] call for one-off removals at go-live (ratings from customers with no qualifying order, and every repeat but the latest), which a blanket ban on bulk deletion forbids; [CD#6] settles that a customer cannot remove their own rating, so the exception for individual removal "if the customer allows it" no longer stands; and [CD#16] establishes that the load generator never runs against Quillmark's live store, so the claim that its wipe resets the averages shoppers see does not hold and the decision must say what it means for test environments instead.

  Re-decided 20261012, round 2: the causes move the answer, and the review's section 6, points (a) to (f), is the position it is argued against. With finality withdrawn ([CD#5]), the reason is no longer that a given rating is final: it is that an average shoppers see moves only when a customer gives or changes their own rating. So no shopper can remove a rating ([CD#6]); the storefront's per-rating Delete goes with its Delete All, and the API's delete-all goes too (points (a), (b), (f)). Operators still need to remove a rating for an erasure request or fraud (point (c)). The API's existing per-rating delete serves that with no new code, and once both buttons go no storefront screen calls it. Removing every rating of one customer is that delete repeated: no requirement row asks for a dedicated endpoint, since the review's addition carries no [BR#n] (reconciliation-20261012.md), so none is built. The storefront has no sign-in ([CD#14]), so nothing can tell an operator's API call from a shopper's. Keeping the API out of shoppers' reach needs sign-in, which this release does not have. That risk is accepted and stated, which answers the open point the review left with the delivery team. The one-off go-live clean-up in [CD#2] and [CD#3] is run once by operators at go-live and offered on no storefront screen, so it is not the deletion this decision forbids (point (d)). The load generator never runs against Quillmark's live store ([CD#16]), so it is left unchanged (point (e)). In test environments its delete-all calls fail once the endpoint is gone, and [BR#1] and [BR#2] refuse most of its random ratings. The generator already catches and logs both failures (ingest `IngestRepository.java` lines 15-19 and `RatingRepository.java` lines 40-51), so it keeps running, and the four deletion paths the review cites no longer bear on this decision. The answer would change if Quillmark adds sign-in, which would let the operator path move behind it, or if test environments need the generator to keep resetting ratings.

  Reopened 20261012: by [CD#19] and [CD#20], from Quillmark's follow-up meeting with the delivery team on 20261012. They replace the go-live clean-up of [CD#2] and [CD#3], on which point (d) of this decision rests, with the removal of every existing rating before go-live, which has already been done in the live store.

  Re-decided 20261012, round 3: the change does not move the answer; it removes point (d). No clean-up runs at go-live. Every existing rating was removed before the release, through the store as it is today, so the release starts with no ratings and needs no removal of its own beyond the operators' single-rating path. The rest of the round 2 reasoning stands: no shopper deletion, the API's delete-all dropped, the per-rating API delete as the operators' path, and the load generator unchanged.
evidence: [[CG#4]]
altitude: architecture
status: decided
consumed_by: ARD
round: 3

id: [VD#2]
statement: Every path that writes a rating refuses a missing star value or one outside the whole numbers 1 to 5, and the rating input offers only whole stars 1 to 5.
options_considered: ["Enforce on every write path and in the UI: refuse a missing or out-of-range value, and limit the input to whole stars 1 to 5", "Enforce on every write path only, leaving the UI input unrestricted", "Leave enforcement as it is today"]
chosen: "Enforce on every write path and in the UI: refuse a missing or out-of-range value, and limit the input to whole stars 1 to 5"
argumentation: |
  [CG#5] shows the setter is the only gate and is bypassed when the value is omitted, so a rating
  with no stars is stored as 0 — outside the scale requirement 3.3 states, and pulling every average
  in 3.4 and 3.6 down. Rejecting missing and out-of-range values on every write path makes the
  server the guarantee; limiting the UI input to whole stars 1 to 5 keeps shoppers from meeting
  errors the form could have prevented. The answer would change only if the customer said 3.3 does
  not bind (round 1, question 12) and wanted a different scale — the server-side guarantee would
  then follow that scale instead.
evidence: [[CG#5]]
altitude: implementation
status: decided
consumed_by: none
round: 1

id: [VD#3]
statement: A displayed average rating is rounded half up to one decimal place, so 4.25 shows as 4.3.
options_considered: ["Round half up (4.25 shows as 4.3)", "Round half to even (4.25 shows as 4.2)", "Truncate to one decimal place (4.29 shows as 4.2)"]
chosen: "Round half up (4.25 shows as 4.3)"
argumentation: |
  Requirements 3.4 and 3.6 ask for one decimal place but name no rule, and averages of whole stars
  routinely land on x.x5. Half up is the rule shoppers and spreadsheets apply, so a shopper who
  checks the arithmetic gets the number the page shows; the bias it introduces is confined to exact
  ties and is at most 0.05, below the precision displayed. The answer would change if Quillmark
  already rounds ratings a different way elsewhere (for instance in its shops' systems) and wants
  the online store to match.
evidence: [[CG#6], [CG#9]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [VD#4]
statement: A change to a rating may alter its stars and its comment; the customer e-mail and the ISBN never change, on any path, operator paths included. A rating recorded against the wrong book is removed through the operators' removal path ([VD#1]), never moved to another book.
options_considered: ["Stars and comment may change; the customer e-mail and ISBN are fixed on every path, and a rating against the wrong book is removed, never reassigned", "Only the stars may change; the comment, the customer e-mail and the ISBN are fixed on every path", "Keep today's edit, which can change the stars, the comment, the customer e-mail and the ISBN"]
chosen: "Stars and comment may change; the customer e-mail and ISBN are fixed on every path, and a rating against the wrong book is removed, never reassigned"
argumentation: |
  Requirement 3.7 speaks of a customer changing their rating of a book, so the rater and the book
  are the rating's identity, not editable data. An edit that could move a rating to another customer
  or another book would undo 3.1's buyer check and 3.2's one-rating-per-book rule after the fact, on
  records those rules already passed. Fixing them holds whichever way the customer answers round 1,
  question 8 — if ratings are final, nothing changes at all. The answer would change only if
  Quillmark wants operators to correct a rating recorded against the wrong book, which would still
  be a separate correction path, not the shopper's edit.

  Reopened 20261012: by [CD#17], from the customer's returned review (customer-review-20261012.md). The customer decided that rating comments are kept, hidden from shoppers, still taken by the forms, and that a change to a rating may alter its comment, which contradicts "only its stars" may change.

  Re-decided 20261012, round 2: [CD#17] keeps comments and lets a change alter them, so "only its stars" no longer holds and the comment joins the stars. The reason the customer e-mail and the ISBN stay fixed is unchanged: they are the rating's identity, and moving either would undo [BR#1]'s buyer check and [BR#2]'s one rating per customer and book after the fact. The customer's correction (customer-review-20261012.md, section 6) also settles the operator case this reasoning left open. A rating recorded against the wrong book is removed through [VD#1]'s per-rating delete and never reassigned, so no separate correction path is built. The update form still shows the e-mail and the ISBN, but no longer lets them be edited. The answer would change only if Quillmark wanted a rating's comment fixed once given, which its answer on comments rules out.
evidence: [[CG#10], [CG#4]]
altitude: product
status: decided
consumed_by: PRD
round: 2

id: [CD#1]
statement: "A qualifying order for rating a book is one for that book that is completed and paid, and not since cancelled."
options_considered: ["An order for that book that is completed and paid, and not since cancelled (the reading we propose)", "Any order placed for that book, paid or not", "An order for that book that has been delivered"]
chosen: "An order for that book that is completed and paid, and not since cancelled (the reading we propose)"
argumentation: |
  A customer has bought a book when they have paid for it and kept it. An unpaid order is not a purchase. "Delivered" is a state the store does not record ([SR#11]), and adding it would change the order flow, which our section 5 excludes. Checked when the rating is given or changed (section 4).
evidence: [[CG#2]]
settles: [[DEF#1]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#2]
statement: "When the buyers-only rule takes effect, every rating from a customer with no qualifying order is removed in a one-off clean-up at go-live."
options_considered: ["Remove them", "Keep them, but leave them out of every average and count", "Keep them and count them, as today"]
chosen: "Remove them"
argumentation: |
  Only buyers' ratings are trustworthy, which is the point of section 3. A rating kept but excluded would hold a customer's e-mail address for no purpose. The code has allowed any customer to rate any book, so such ratings may well exist in our live data. We hold no count of them, and the answer applies to whatever exists.

  Superseded 20261012: by [CD#19]
evidence: [[CG#2]]
altitude: product
status: superseded
consumed_by: PRD
round: 1

id: [CD#3]
statement: "At go-live, only the latest rating each customer gave each book is kept, and every earlier repeat is removed."
options_considered: ["Keep only the latest rating each customer gave each book, and remove the rest (the reading we propose)", "Keep only the first rating each customer gave each book, and remove the rest", "Keep all of them, and apply the once-only rule to new ratings only"]
chosen: "Keep only the latest rating each customer gave each book, and remove the rest (the reading we propose)"
argumentation: |
  It matches 3.7: a customer's newest rating replaces their earlier one. Keeping the first would contradict that, and keeping all would leave the repeat ratings that section 3 blames for shoppers' distrust.

  Superseded 20261012: by [CD#20]
evidence: [[CG#3]]
settles: [[DEF#3]]
altitude: product
status: superseded
consumed_by: PRD
round: 1

id: [CD#4]
statement: "3.2's one-rating-per-customer-per-book rule ([BR#2]) and its finality clause ([BR#3]) are independent requirements, and one rating per customer per book holds whichever rule governs finality."
options_considered: ["Yes: two independent requirements, and one rating per customer per book holds either way (the reading we propose)", "No: 3.2 is one rule, and it stands or falls as a whole"]
chosen: "Yes: two independent requirements, and one rating per customer per book holds either way (the reading we propose)"
argumentation: |
  Repeat ratings are what we named as the problem, not changed ones. One rating per customer per book must hold even though ratings can be changed.
evidence: [[CG#3], [CG#4]]
settles: [[DEF#4]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#5]
statement: "Requirement 3.7 governs, so a customer can change their rating at any time, and the word \"final\" in 3.2 ([BR#3]) is withdrawn."
options_considered: ["3.7 governs: a customer can change their rating at any time, and \"final\" in 3.2 is withdrawn", "3.2 governs: a rating is final once given, and 3.7 is withdrawn", "A rating can be changed for a limited period after it is given, then becomes final (please give the period)"]
chosen: "3.7 governs: a customer can change their rating at any time, and \"final\" in 3.2 is withdrawn"
argumentation: |
  Shoppers change their minds. A changed rating still counts once, so allowing changes does not bring back the repeat-rating problem. A time limit would add a rule shoppers cannot see.
evidence: [[CG#4], [CG#10]]
settles: [[DEF#5]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#6]
statement: "A customer can change their rating of a book but cannot remove it."
options_considered: ["No: once given, a rating can be changed (where 3.7 allows it) but not removed", "Yes: a customer can remove their own rating, and the book's average and count update accordingly"]
chosen: "No: once given, a rating can be changed (where 3.7 allows it) but not removed"
argumentation: |
  A change already covers a customer who has changed their mind. Letting customers remove ratings would make counts swing for no reason shoppers can see. Removal exists, but only for operators handling erasure requests and fraud (section 4).
evidence: [[CG#4]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#7]
statement: "The whole-star 1-to-5 scale in 3.3 is a requirement of this release, and every rating is a whole number of stars from 1 to 5."
options_considered: ["A requirement: every rating is a whole number of stars from 1 to 5 (the reading we propose)", "A description of today's scale, not a requirement of this release"]
chosen: "A requirement: every rating is a whole number of stars from 1 to 5 (the reading we propose)"
argumentation: |
  It is the scale shoppers see on the page. A 0 or a fraction would distort every average in 3.4 and 3.6.
evidence: [[CG#5]]
settles: [[DEF#6]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#8]
statement: "3.4 is two requirements, the average ([BR#5]) and the count ([BR#6]), and the count shown is always the count of exactly the ratings the average is computed from."
options_considered: ["Yes: two requirements, and the number is always the count of the ratings in the average (the reading we propose)", "No: 3.4 is one requirement (please say how it differs)"]
chosen: "Yes: two requirements, and the number is always the count of the ratings in the average (the reading we propose)"
argumentation: |
  The count is there to show how many people the average rests on (section 3's introduction). Any other number would mislead.
evidence: [[CG#6], [CG#7]]
settles: [[DEF#7]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#9]
statement: "For a book with no ratings, the book page shows \"Not yet rated\" alone, with no count."
options_considered: ["\"Not yet rated\" alone, with no count (the reading we propose)", "\"Not yet rated\" together with \"(0 ratings)\""]
chosen: "\"Not yet rated\" alone, with no count (the reading we propose)"
argumentation: |
  "(0 ratings)" says the same thing a second time and draws the eye to an empty state.
evidence: [[CG#8], [CG#7]]
settles: [[DEF#8]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#10]
statement: "Requirement 3.6 covers the catalogue list of books only."
options_considered: ["The catalogue list only", "Every list of books in the store: the catalogue list, search results wherever the store shows them, the book-page suggestions and the Top rated shelf"]
chosen: "The catalogue list only"
argumentation: |
  3.6 is about the list of books we have today. The suggestions and the Top rated shelf belong to section 4 and the other BOOK-2 package, and 3.6 should not widen them.
evidence: [[CG#9]]
settles: [[DEF#9]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#11]
statement: "No synthetic monitor is built or supported in this release: requirement 3.8 ([BR#10]) is withdrawn, and Quillmark will set up third-party monitoring itself later."
options_considered: ["Build: we build and run a new synthetic monitor that buys a book and then rates it", "Support: Quillmark already runs synthetic monitoring (please name the tool and who operates it), and the store must let it buy and rate"]
chosen: "outside the options: **Neither, in this release:** 3.8 is withdrawn. Quillmark will set up third-party monitoring itself later *(our own knowledge)*."
argumentation: |
  The choice of monitoring tool is ours, and it is not made yet. We will bring 3.8 back, with the tool and who operates it, in a later version of our requirements.
evidence: [[CG#11]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#12]
statement: "The Figure 1 note (\"Clicking the count opens the list of this book's ratings.\") is not a requirement of this release, so [BR#11] is withdrawn."
options_considered: ["Yes: clicking the count on the book page opens the list of that book's ratings", "No: the note illustrates an idea, not a requirement of this release"]
chosen: "No: the note illustrates an idea, not a requirement of this release"
argumentation: |
  We drew it to show what the count is for. We do not need a list of a book's ratings now.
evidence: [[CG#12]]
settles: [[DEF#13]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#13]
statement: "Answering [SR#1] of self-review-20261012.md: when 3.8 returns, the monitor test-data question (round 1, question 27) is split as [SR#1] proposes; for this release it is moot, because 3.8 is withdrawn."
options_considered: ["as the package states", "as the finding argues"]
chosen: "as the finding argues"
argumentation: |
  See round 1, question 27.

  When 3.8 returns, please split the question as [SR#1] proposes. Ask us only whether the monitor's purchases may be real paid orders, and whether its test customers and books may be visible to shoppers. The test-data design is the supplier's to argue.
evidence: [[CG#3], [CG#11]]
altitude: product
status: decided
consumed_by: none
round: 1

id: [CD#14]
statement: "Answering [SR#2] of self-review-20261012.md: Quillmark's shoppers use the `web/` application in the repository as their storefront, and for this release the rater is the customer whose e-mail is entered."
options_considered: ["as the package states", "as the finding argues"]
chosen: "as the package states"
argumentation: |
  The package was right to depend on this, and wrong not to say so. Every decision that depends on it now does (section 6).
evidence: [[CG#2], [CG#6], [CG#9], [CG#10]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#15]
statement: "Answering [SR#5] of self-review-20261012.md: round 1, question 5 is answered in the reframed form [SR#5] asks for, about any repeat ratings Quillmark's live store holds, not as an established fact about its data."
options_considered: ["as the package states", "as the finding argues"]
chosen: "as the finding argues"
argumentation: |
  The load generator never runs in our live store *(our own knowledge)*, so any repeat ratings there are real shoppers'. We hold no count of them.
evidence: [[CG#3], [CG#4]]
altitude: product
status: decided
consumed_by: none
round: 1

id: [CD#16]
statement: "Answering [SR#8] of self-review-20261012.md: none of the load generator's consequences reaches Quillmark's live store, because the generator never runs there; its behaviour in test environments is the supplier's call, and [VD#1] must say so rather than claim to protect shoppers' averages."
options_considered: ["as the package states", "as the finding argues"]
chosen: "outside the options: Settled: none of these consequences reaches our live store, because the load generator never runs there *(our own knowledge)*."
argumentation: |
  The generator's behaviour in test environments is the supplier's call. The decision must say so rather than claim to protect shoppers' averages ([VD#1] in section 6).
evidence: [[CG#2], [CG#3], [CG#4], [CG#5], [CG#11]]
altitude: architecture
status: decided
consumed_by: ARD
round: 1

id: [CD#17]
statement: "Answering [SR#10] of self-review-20261012.md: rating comments are kept but never shown to shoppers, the rating forms keep taking them, and a change to a rating may alter its comment; the date question is moot with round 1, question 30."
options_considered: ["as the package states", "as the finding argues"]
chosen: "outside the options: **Settled:** comments are kept but hidden from shoppers, the forms keep taking them, and a change may alter them (section 4 and [VD#4]). The date question is moot with question 30."
argumentation: |
  We had not known ratings carry comments. Hiding them keeps written reviews out of scope without throwing away what customers wrote.
evidence: [[CG#10], [CG#12]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#18]
statement: "Answering [SR#11] of self-review-20261012.md: the customer answered round 1, question 2 knowing \"delivered\" is not recorded and would need order-flow work their section 5 excludes."
options_considered: ["as the package states", "as the finding argues"]
chosen: "as the finding argues"
argumentation: |
  We chose knowing that "delivered" is not recorded and would mean order-flow work our section 5 excludes.
evidence: [[CG#2]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [VD#5]
statement: Orders are unchanged in this release. A customer's eligibility is checked against the orders the store holds when a rating is given or changed ([CD#1]); a rating stays if its order is later cancelled or deleted, and a customer whose qualifying order no longer exists cannot change their rating.
options_considered: ["Leave orders unchanged: deleting an order removes the right to change a rating, never a rating already given", "Keep a copy of the purchase in the ratings service when a rating is given", "Stop the storefront deleting orders: drop the order Delete and Delete All"]
chosen: "Leave orders unchanged: deleting an order removes the right to change a rating, never a rating already given"
argumentation: |
  The review's challenge to the derivation matrix (customer-review-20261012.md, section 5,
  challenge 2) is right that the storefront's order Delete and Delete All erase the only record that
  a purchase happened. That does not reach a rating already given: [CD#1] checks eligibility when a
  rating is given or changed, and the review's own section 4 says a rating stays if its order is
  later cancelled. What it reaches is a later change by a customer whose order was deleted, and that
  change is refused. Changing how the store keeps orders would reopen order handling, which the
  requirements' section 5 keeps out of this release, and copying purchases into the ratings service
  would duplicate the orders service's data for that one edge case. The ratings service reads
  orders through the orders API that exists today, so the orders service itself does not change.
  This settles the second open point the review left with the delivery team (its section 10). The
  answer would change if Quillmark wants a customer's right to change their rating to outlive a
  deleted order, which would favour keeping a copy of the purchase in the ratings service.
evidence: [[CG#2]]
altitude: product
status: decided
consumed_by: PRD
round: 2

id: [VD#6]
statement: The storefront's Ratings list stays, without the comment column ([CD#17]) and without its Delete and Delete All ([VD#1], [CD#6]). It keeps each rater's e-mail and the Update action, because the typed e-mail is the rater's identity in this release ([CD#14]).
options_considered: ["Keep the list without comments and without deletion; the e-mail and Update stay", "Also hide the e-mail column and remove Update from the list", "Remove the Ratings list from the storefront"]
chosen: "Keep the list without comments and without deletion; the e-mail and Update stay"
argumentation: |
  The review leaves this screen to the delivery team (its sections 6 and 10). Comments must not be
  shown ([CD#17]) and shoppers may not delete ([CD#6], [VD#1]), so both go. The e-mail and Update
  stay. With no sign-in ([CD#14]), a customer changes their rating by finding it in this list under
  their e-mail, which is the change requirement 3.7 asks the store to allow. Removing the list or
  the action would leave no way to make that change. Showing raters' e-mails is the exposure the
  typed-e-mail identity already accepts. The answer would change once the store has sign-in, when
  the list can show a customer only their own ratings.
evidence: [[CG#10], [CG#12]]
altitude: product
status: decided
consumed_by: PRD
round: 2

id: [CD#19]
statement: "Every rating the store held before this release is removed before go-live, instead of the selective go-live clean-up of [CD#2]; in Quillmark's live store this was done on 2026-10-12, so the release starts with no ratings and carries no clean-up of existing ratings."
options_considered: ["Remove every existing rating before go-live", "Remove them: every rating from a customer with no qualifying order, at go-live (as [CD#2])", "Keep them, but leave them out of every average and count", "Keep them and count them, as today"]
chosen: "Remove every existing rating before go-live"
argumentation: |
  "The existing ratings can't be trusted, which is why shoppers ignore them. The store starts fresh with verified ratings only."

  Recorded 20261012 by the delivery team from Quillmark's follow-up meeting after its review, at which the Head of E-commerce and the Operations Lead took this position and the Operations Lead confirmed the live store's ratings were removed the same day. There is no returned review file for it. It answers round 1, question 3 again and replaces [CD#2]. The comments customers had written went with their ratings.
evidence: [[CG#2]]
altitude: product
status: decided
consumed_by: PRD
round: 1

id: [CD#20]
statement: "No repeat ratings remain to reduce at go-live: every existing rating was removed before go-live under [CD#19], so the go-live reduction to each customer's latest rating in [CD#3] is not needed."
options_considered: ["Nothing to reduce: every existing rating is removed before go-live ([CD#19])", "Keep only the latest rating each customer gave each book, and remove the rest (as [CD#3])"]
chosen: "Nothing to reduce: every existing rating is removed before go-live ([CD#19])"
argumentation: |
  "With every existing rating removed, there are no repeat ratings left to reduce."

  Recorded 20261012 by the delivery team from Quillmark's follow-up meeting after its review (see [CD#19]). It answers round 1, question 5 again and replaces [CD#3]. One rating per customer and book ([CD#4]) still holds for every rating given from go-live on.
evidence: [[CG#3]]
settles: [[DEF#3]]
altitude: product
status: decided
consumed_by: PRD
round: 1

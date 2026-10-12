> **Banner, 20261012 — superseded in part by `/brd-reconcile` (`reconciliation-20261012.md`).** This file records the self-review of the BOOK-2-01 package as it stood on 20261012, and nothing beneath this banner has changed. The six findings escalated to the customer are answered: [SR#1] by [CD#13], [SR#2] by [CD#14], [SR#5] by [CD#15], [SR#8] by [CD#16], [SR#10] by [CD#17] and [SR#11] by [CD#18]. [VD#1] (the target of [SR#3], [SR#7] and [SR#8]) and [VD#4] (the target of [SR#9]) are reopened in `decisions.md`. The current position is recorded in `decisions.md` and `reconciliation-20261012.md`.

# Self-review 20261012: BOOK-2-01

Adversarial review of the BOOK-2-01 package before it goes to the customer. Internal to the delivery
organisation: this file is never copied into the bundle.

- **Reviewed:** 2026-10-12, first package for this slice (no prior self-review)
- **Pass ran on:** `claude-opus-5-5` (review model, Opus chain; no degradation)
- **Classification:** SIGNIFICANT
- **Inputs handed to the pass:** `decisions.md`, `grounding/code-grounding.md`,
  `grounding/design-grounding.md`, `coverage-ledger.md`, `interview/customer-questions.md`. No seeds,
  no `code-defect-log.md` and no prior self-review exist.
- **Status returned:** OK — 11 findings, per-pass account present

```yaml
model_routing:
  classification: SIGNIFICANT
  reason: adversarial self-review gates the run, and the rendered prompt leaves the organisation
  current_model: claude-opus-5-5
  detection_model: claude-sonnet-5-5
  review_model: claude-opus-5-5
  opus_available: true
  notes: none
```

Ids below are assigned by this dated review. Each record keeps the pass-local number the agent gave it
(`pass id`); in this first pass the two coincide.

## Findings

### [SR#1]

- **pass id:** SR#1
- **class:** mis-tagged-question
- **target:** The questions held for the customer: round 1, question 27 (which customer and which books the monitoring uses)
- **attack:**

  This question asks Quillmark to choose between two designs for the monitor's test data. That choice is a technical trade-off the delivery team owns and should decide and argue itself. What separates the two options is how the store behaves, and Quillmark has no way to weigh that.

  - **The first option cannot run as written.** The store keeps one order per customer per book: the orders table carries a uniqueness rule on customer e-mail and ISBN (orders/src/main/java/com/dynatrace/orders/model/Order.java:11, cited in [CG#3]). Submitting an order that is already completed returns it without buying again (orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:151-153). So one customer cannot buy the same book "with a paid order ... on each run". The monitor would have to cancel and re-buy every minute, moving stock each time. The question does not say so.
  - **The second option has an unmentioned cost.** It creates a new customer account every minute, more than half a million a year.
  - **Which option is possible depends on two other open questions.** One is whether requirement 3.7 governs (round 1, question 8). The other is whether Quillmark's own tool runs the monitor (round 1, question 25). If Quillmark runs it, Quillmark configures it. If the supplier builds it, choosing its test data is the supplier's job.

  Only two parts of this are Quillmark's to decide. First, whether the monitor's purchases may be real paid orders, with money moving through the payment service. Second, whether test customers or a test book may appear anywhere shoppers can see them. Neither is offered as a choice: both options assume "a paid order".

  Put as it stands, Quillmark's answer would carry its sign-off on a design whose consequences it was never shown.
- **rests_on:** [CG#3], [CG#11]
- **what_would_settle_it:**

  Split the question. The supplier decides and argues the monitor's test-data design, once Quillmark has said it wants the supplier to build the monitor (round 1, question 25). Quillmark is asked only what it alone can settle:
  - whether the monitor's purchases may be real paid orders;
  - whether its test customers and books may be visible to shoppers.

  Alternatively, the package shows that both options work with the store as it stands, and that the choice between them turns on something only Quillmark knows.
- **disposition:** escalated-to-customer
- **reason:** The two parts only Quillmark can settle — whether the monitor's purchases may be real paid orders, and whether its test customers and books may be visible to shoppers — are rules the business sets, so this finding is put to Quillmark beside question 27 in the decisions part. The test-data design itself stays the supplier's to decide and argue once question 25 is answered; Quillmark's answer to question 27's options is read as a preference, not as sign-off on that design.

### [SR#2]

- **pass id:** SR#2
- **class:** assumption-as-fact
- **target:** The decision register and the questions held for the customer: the unstated premise that the web application in the repository is the storefront Quillmark's shoppers use
- **attack:**

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
- **rests_on:** [CG#2], [CG#6], [CG#9], [CG#10], [VD#1], [VD#2]
- **what_would_settle_it:**

  Either Quillmark confirms which front end its shoppers use, or the package records, as an assumption put to Quillmark, that it is this web application. In both cases, every decision and question that depends on the answer says that it does.
- **disposition:** escalated-to-customer
- **reason:** Which front end Quillmark's shoppers actually use is a fact about Quillmark's own operation that only Quillmark holds, and one sentence from them settles it. Put to them in the decisions part; the user-interface halves of [VD#1] and [VD#2], and the pages questions 3 and 21 describe, are read against their answer.

### [SR#3]

- **pass id:** SR#3
- **class:** overclaimed-grounding
- **target:** [VD#1]
- **attack:**

  The reasoning says the load generator's per-cycle wipe "silently resets every average a shopper sees". The finding it cites, [CG#4], says something narrower: every generator cycle deletes all ratings, "so while the load generator runs, ratings are not even durable". The decision drops "while the load generator runs" and widens the claim to the store Quillmark's shoppers use.

  Nothing in the package establishes that the load generator runs against Quillmark's live store. [CG#11] shows that the repository's own Kubernetes manifest starts it automatically (k8s/ingest.yaml:113). That is the repository's deployment, not a finding about Quillmark's. No average exists today ([CG#6]), so the sentence also describes a harm that does not yet occur.

  This sentence is the only justification for half of the chosen option: stopping the load generator from deleting ratings. Quillmark's operations team will know at once whether a load generator runs against its live store.
- **rests_on:** [CG#4], [CG#11], [CG#6]
- **what_would_settle_it:**

  Quillmark states that its live store runs the load generator. Failing that, the reasoning keeps the condition its finding carries ("while the load generator runs"), and the package records, as an assumption put to Quillmark, that the generator runs against the live store.
- **disposition:** accepted-risk
- **fixed-by:** /product-workflows:brd-interview — [VD#1] is a register record that command writes; a re-run here after it would show [VD#1]'s argumentation keeping [CG#4]'s "while the load generator runs" condition, or an [AS#n] recording that the generator runs against Quillmark's live store.
- **reason:** The team agrees: the argumentation dropped the condition its finding carries. The register record is not this command's to change, so the finding ships under *where to attack us hardest* as agreed and recorded for repair outside this package.

### [SR#4]

- **pass id:** SR#4
- **class:** overclaimed-grounding
- **target:** The questions held for the customer: round 1, question 3, the sentence saying that ratings from customers who never bought the book are on file
- **attack:**

  [CG#2] establishes two things about the code:
  - it accepts a rating from any existing customer for any existing book;
  - the load generator routinely writes ratings for random customer-and-book pairs.

  It establishes nothing about what Quillmark's live ratings data holds: grounding reads code at a commit, not a live database. Where the load generator runs, every rating on file is its own output, deleted and recreated every cycle ([CG#4]). None of it was "given by customers".

  The question still asks Quillmark what should happen to "the ratings already given by customers who never bought the book". Its first option is to remove them, as though real shoppers' ratings of that kind were known to exist. Quillmark's own requirement 3.1 says its store already limits ratings to buyers. Its reviewer will ask for evidence that such ratings exist, and the package has none.
- **rests_on:** [CG#2], [CG#4]
- **what_would_settle_it:**

  Either evidence from Quillmark's live data, which only Quillmark holds, that such ratings exist. Or the sentence is restated to say what the code permits and what the load generator writes wherever it runs, and the question asks what should happen to any such ratings Quillmark's store holds.
- **disposition:** accepted-risk
- **fixed-by:** /product-workflows:brd-interview — question 3 is an entry in the question set that command writes; a re-run here after it would show the question restated as what the code permits and what the load generator writes, asking what should happen to any such ratings Quillmark's store holds.
- **reason:** The team agrees: the sentence states as a fact about Quillmark's data what [CG#2] establishes only about the code. (First put to the operator before the finding was shown; re-put with the finding in full, and this disposition is the one that stands.) The question-set entry is not this command's to change, so the finding ships under *where to attack us hardest* as agreed and recorded for repair outside this package.

### [SR#5]

- **pass id:** SR#5
- **class:** overclaimed-grounding
- **target:** The questions held for the customer: round 1, question 5, the sentence saying that repeat ratings are on file
- **attack:**

  [CG#3] establishes that nothing in the code stops a repeat rating. It also establishes that the load generator posts ratings for random customer-and-book pairs, and that the store accepts the repeats this produces. It establishes nothing about Quillmark's live ratings data.

  Quillmark's introduction to section 3 says a customer can rate the same book again and again, which matches what the code shows. It does not say the store holds shoppers' repeat ratings.

  Where the load generator runs, every repeat on file is a random repeat of its own, deleted and replaced every cycle ([CG#4]). The package's proposed reading is to keep only the latest rating each customer gave each book and remove the rest. In that case it would be a clean-up of generated data, presented as a decision about shoppers' ratings. The question puts that choice to Quillmark as a fact about its customers, and the package does not hold that fact.
- **rests_on:** [CG#3], [CG#4]
- **what_would_settle_it:**

  Either evidence from Quillmark's live data that shoppers' repeat ratings exist, or the sentence is restated as what the code permits, and the question asks what should happen to any repeat ratings Quillmark's store holds.
- **disposition:** escalated-to-customer
- **reason:** Whether Quillmark's live store holds shoppers' repeat ratings is evidence only Quillmark holds. Put to them beside question 5 in the decisions part, so their answer to question 5 is given knowing whether it settles shoppers' ratings or only generated ones.

### [SR#6]

- **pass id:** SR#6
- **class:** overclaimed-grounding
- **target:** The code-grounding findings, derivation matrix: the row "Whether the rater bought the book", classed as already captured and stored, unchanged
- **attack:**

  [CG#2] establishes where the fact of a purchase lives: in an order that is completed and paid, held only by the orders service. It does not establish that the fact is kept, and the code at the pinned commit shows it is not:
  - An order can be deleted on its own or all at once (orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:206-219).
  - Every load generator cycle begins by deleting every order (ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:256-266, called at line 309).
  - Cancelling clears the same completed flag in place (OrderController.java:173-203 and 302-313). The order record therefore keeps no trace that a cancelled order was ever completed.

  The note under the matrix warns that the load generator empties the ratings table. It says nothing about the generator emptying the orders table, which the buyers-only rule depends on. Requirement 3.1, and requirement 4.4 in the sibling package, both need to know who bought a book after the purchase. "Unchanged" asserts that nothing has to be built for that. A reviewer with the repository will contradict the row by reading those lines.
- **rests_on:** [CG#2], [CG#4]
- **what_would_settle_it:**

  Either a finding that order records are kept on every path that runs in Quillmark's live store, or the row is classed as needing new work, with the retention gap stated beside it.
- **disposition:** accepted-risk
- **fixed-by:** /product-workflows:prd-ground — the derivation matrix is grounding output that command writes, each row resting on a verified finding; a re-run here after `/product-workflows:prd-ground BOOK-2-01 --rebaseline` would show the purchase row reclassed as needing new work, with the order-deletion and cancellation retention gap verified and stated beside it.
- **reason:** The team agrees: [CG#2] locates the purchase fact but does not establish that it is kept. `fixed` was picked first and is inadmissible — the matrix is grounding output, and rewriting the row in the bundle's copy would put an unverified claim in front of the customer; `escalated-to-customer` was picked next and is inadmissible — the code settles it, so asking the customer would return their belief about the system. Ships under *where to attack us hardest* as agreed and recorded for repair outside this package.

### [SR#7]

- **pass id:** SR#7
- **class:** will-not-survive-review
- **target:** [VD#1]
- **attack:**

  The decision says ratings cannot be deleted in bulk. Its reasoning says this "leaves no route by which ratings disappear, except individual removal if the customer allows it (round 1, question 9)". The package contradicts that in three places:
  - **Round 1, question 5.** The package's own proposed reading is to keep only the latest rating each customer gave each book "and remove the rest". That is a bulk removal.
  - **Round 1, question 3.** It offers "Remove them" for every rating from a non-buyer.
  - **Round 1, question 9.** If Quillmark picks its first option ("changed ... but not removed"), this decision leaves no way to remove any rating, for anyone, for any reason.

  The reasoning itself names the case that would change the answer: Quillmark needing to purge ratings for legal erasure or fraud clean-up. Only Quillmark knows whether it does, and the package never asks. Every rating stores the customer's e-mail address (ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:16-17), so a request to erase a customer's data reaches this table.

  The reasoning also opens on requirement 3.2's finality. Round 1, question 8 offers Quillmark the option of withdrawing that finality, and the reasoning does not say whether the decision still stands if Quillmark does.
- **rests_on:** [CG#4], [VD#1]
- **what_would_settle_it:**

  - The decision says whether the one-off removals that questions 3 and 5 may call for fall outside "bulk deletion".
  - Quillmark is asked whether it must be able to remove ratings, for erasure requests or fraud.
  - The reasoning shows that the decision holds under every answer to questions 8 and 9.
- **disposition:** accepted-risk
- **fixed-by:** /product-workflows:brd-interview — [VD#1] is a register record that command writes, and the erasure-or-fraud purge question is one it would hold as a [C]; a re-run here after it would show [VD#1] stating whether the one-off removals questions 3 and 5 may call for are outside "bulk deletion", holding under every answer to questions 8 and 9, and a held question asking Quillmark whether ratings must be removable for erasure requests or fraud.
- **reason:** The team agrees: the argumentation's "no route by which ratings disappear" is contradicted by the package's own proposals, and its own flip condition is never put to Quillmark. The register record and the question set are not this command's to change, so the finding ships under *where to attack us hardest* as agreed and recorded for repair outside this package.

### [SR#8]

- **pass id:** SR#8
- **class:** will-not-survive-review
- **target:** [VD#1]
- **attack:**

  The decision changes one thing the load generator does, deleting ratings, and says nothing about the rest. Take the decision's own premise, that the generator runs against the live store:
  - **Its ratings would pile up.** Each cycle writes about 700 ratings (k8s/ingest.yaml:132, per [CG#3]). They are for random customer-and-book pairs unrelated to any purchase, and never a 5 ([CG#2], [CG#5], [CG#11]). Cycles run back to back ([CG#11]). Today each cycle's ratings are wiped. Under this decision they stay, and every average and count shoppers will see is made of them.
  - **It would keep erasing the records the buyers-only rule needs.** It still deletes every order at the start of every cycle, and on cycles that regenerate them, every customer and book (ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java:256-274, called at line 309).
  - **Its rating step would start failing.** Once requirements 3.1 and 3.2 are built, the store will refuse the generator's random ratings, as coming from non-buyers or as repeats. The decision does not say what the generator does then.

  In round 1, question 26, the package treats the monitor's single rating a minute as needing Quillmark's decision on whether it should count. Here it lets hundreds of generated ratings per cycle count, without saying so.
- **rests_on:** [CG#2], [CG#3], [CG#4], [CG#5], [CG#11]
- **what_would_settle_it:**

  The decision states what the load generator does with ratings and orders after the change, and how its ratings are kept out of what shoppers see. Alternatively, it shows that none of these consequences reaches Quillmark's live store.
- **disposition:** escalated-to-customer
- **reason:** Two parts of what settles it are Quillmark's alone: whether a load generator runs against its live store at all, and — if one does — whether generated ratings may count toward the averages and counts shoppers see, the same kind of business rule question 26 puts for the monitor's ratings. Put to them in the decisions part. What the load generator does with ratings and orders after the change stays the delivery team's to decide in [VD#1], read against their answer.

### [SR#9]

- **pass id:** SR#9
- **class:** will-not-survive-review
- **target:** [VD#4]
- **attack:**

  The decision says a change to a rating may alter "only its stars". But every rating also carries a free-text comment:
  - it is stored with the rating (ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:25-26);
  - the create and edit forms both take it (web/src/app/ratings/create-rating/create-rating.component.html:20-21 and web/src/app/ratings/update-rating/update-rating.component.html:20-21);
  - the ratings list and the rating details page show it.

  "Only its stars" silently forbids changing the comment, and the decision never names that field. Its second option describes today's edit as changing "the stars, the customer e-mail and the ISBN", but today's edit also changes the comment. No finding records the comment at all.

  The decision is also inconsistent about which paths it binds. The chosen option fixes the customer and the book "on every path". The reasoning then says that an operator correcting a rating recorded against the wrong book "would still be a separate correction path, not the shopper's edit". A reader cannot tell whether such a correction is allowed, which is exactly what reasoning for a decision has to make clear. Quillmark's reviewer will raise both points by opening the edit form.
- **rests_on:** [CG#10], [CG#4]
- **what_would_settle_it:**

  The decision names every field a change may alter, and says what happens to the comment. It states whether "every path" includes an operator correction path, with reasoning that matches.
- **disposition:** accepted-risk
- **fixed-by:** /product-workflows:brd-interview — [VD#4] is a register record that command writes (the comment field itself first needs a verified finding, which a `/product-workflows:prd-ground BOOK-2-01 --rebaseline` run would add); a re-run here after both would show [VD#4] naming every field a change may alter, the comment included, and stating whether "every path" covers an operator correction path, with matching argumentation.
- **reason:** The team agrees: the decision silently covers a field no finding records, and its option and argumentation disagree on which paths it binds. The register record is not this command's to change, so the finding ships under *where to attack us hardest* as agreed and recorded for repair outside this package.

### [SR#10]

- **pass id:** SR#10
- **class:** will-not-survive-review
- **target:** The questions held for the customer: round 1, question 30 (what the list of a book's ratings shows for each rating)
- **attack:**

  The question tells Quillmark that written reviews are out of scope and proposes showing "the stars and the date only". It leaves out two facts about the store as it stands:
  - **Ratings already carry comments.** Every rating has an optional free-text comment, taken by the create and edit forms and shown in the ratings list and on the rating details page (ratings/src/main/java/com/dynatrace/ratings/model/Rating.java:25-26; web/src/app/ratings/rating-list/rating-list.component.html:8 and 19). In practice that is a short written review. Quillmark's section 5 may have been written in the belief that none exist.
  - **The original date is not kept.** Every change to a rating today overwrites the time it was first given ([CG#10]). "The date" could mean when the rating was given or when it was last changed, and the store does not keep the first.

  If Quillmark accepts the proposed reading, it will have decided to hide comments its customers have already written, without being told they exist. It will also have chosen a date the store may not be able to show.
- **rests_on:** [CG#10], [CG#12]
- **what_would_settle_it:**

  The question:
  - tells Quillmark that ratings already carry comments, and asks what happens to them (shown, hidden or removed);
  - says which date it means;
  - says that today the original date is lost whenever a rating changes.
- **disposition:** escalated-to-customer
- **reason:** Whether ratings' existing comments are shown, hidden or removed, and which date the list shows, are Quillmark's product calls; the finding's own words carry the two facts question 30 left out, so it is put to them beside question 30 in the decisions part. `fixed` was picked first and is inadmissible: question 30 is a question-set entry the interview writes, and patching its rendering in the prompt would carry a fact no verified finding records.

### [SR#11]

- **pass id:** SR#11
- **class:** will-not-survive-review
- **target:** The questions held for the customer: round 1, question 2 (which orders entitle a customer to rate a book)
- **attack:**

  The third option, "an order for that book that has been delivered", names a state the store does not record. An order holds a customer, a book, a quantity, a price, a completed flag and two timestamps, and nothing about delivery (orders/src/main/java/com/dynatrace/orders/model/Order.java:18-39). No finding examined delivery at all.

  The question does not tell Quillmark this. If Quillmark chooses that option, it commits to recording deliveries in the order flow without knowing it. Its own section 5 puts changes to checkout, payment and order cancellation out of scope, so it may be choosing an option its own scope forbids. The package could then only ask the question again.
- **rests_on:** [CG#2]
- **what_would_settle_it:**

  The question says which options the store can evaluate from what it records today, and that choosing "delivered" means new work in the order flow. Alternatively, a finding shows that the store does record delivery.
- **disposition:** escalated-to-customer
- **reason:** Which order state entitles a customer to rate, and whether Quillmark accepts new order-flow work to reach "delivered" given its own out-of-scope list, are Quillmark's business rules. The finding's own words carry the fact question 2 left out, so it is put to them beside question 2 in the decisions part.

## Per-pass account

**unsupported-decision.** Opened every finding in every evidence list and read its verdict and claim.
- [VD#1] cites [CG#4] (REWRITTEN, agree). It supports the premise that bulk deletion exists. The decision itself is argued from the requirements, not passed off as settled by the finding. Its gaps are filed under will-not-survive-review (SR#7, SR#8).
- [VD#2] cites [CG#5] (AMENDED; contradict was forced by the missing control, verdict unchanged). It carries the premises that an omitted value stores 0 and that the setter is the only gate.
- [VD#3] cites [CG#6] and [CG#9] (REWRITTEN). They establish that nothing in the store computes or rounds an average. That does bear on the premise that there is no in-store convention to match. The reasoning's flip condition names a convention outside the store, and it does not claim the findings settle that. I considered a "decorative citation" attack and did not file it.
- [VD#4] cites [CG#10] (AMENDED) and [CG#4]. They support the premise that today's edit can change the e-mail and ISBN. The comment it misses is filed as SR#9.
- No FALSE-FRIEND or NOT-PROVABLE finding is cited as support by any decision.
- Will-change rule: every finding is horizon current and no record carries conditional_on, so nothing fires and no decision was closed against the rule.

**mis-tagged-question.** Applied the "what would settle it" test to all 15 held questions: 2, 3, 5, 6, 8, 9, 12, 15, 19, 21, 25, 26, 27, 29 and 30.
- Filed question 27 (SR#1).
- Considered questions 6 and 15. These are split confirmations raised by the duplicate-class defects [DEF#4] and [DEF#7], and their substantive clauses restate the customer's own text. But how the customer's own document is to be read is customer authority, so they are low-value rather than misrouted. Not filed.
- Question 19 is empty-state copy, and requirement 3.5 itself dictates the copy.
- Questions 2, 3, 5 and 9 are business rules. Questions 8, 12 and 29 are readings of the customer's document. Question 21 is scope. Questions 25 and 26 are operating and business decisions. Question 30 is privacy and display.
- Also checked the reverse direction on the three delivery-team questions. [VD#1]'s flip condition, Quillmark's need to purge ratings, is customer-held; it is carried into SR#7. [VD#3]'s flip condition, Quillmark's rounding elsewhere, is customer-held, but the position is visible in the register and recoverable. Not filed.

**assumption-as-fact.** The register holds no [AS#n], and the round 1 record says none was needed. Searched every argumentation, every question preamble, the grounding prose and derivation matrix, the ledger and the design-grounding note for flat assertions with neither a finding nor an assumption behind them.
- Found the web-application-as-storefront premise (SR#2).
- Found the load-generator-runs-live premise. It is filed under overclaimed-grounding (SR#3) because it enters the package as a dropped hedge on [CG#4].
- [VD#3]'s "half up is the rule shoppers and spreadsheets apply" is a general argument, not a claim about the store. Not filed.
- Question 26's "more than half a million" is arithmetic.

**overclaimed-grounding.** Read every customer-facing assertion about today's store against the finding it cites.
- Question 2 against [CG#2]: holds.
- Question 3: filed (SR#4). Question 5: filed (SR#5).
- Question 8 against [CG#4] and [CG#10]: holds. Question 9 against [CG#4]: holds.
- Question 12 against [CG#5]: the 0 caveat is preserved.
- Question 21: the "catalogue list" wording falls under SR#2.
- Question 25 against [CG#11]: the hedge on monitors held outside the store is preserved, and "on no schedule" is literally accurate.
- Question 29 against [CG#12]: holds.
- [VD#1]: filed (SR#3). [VD#2]: holds. [VD#4]: holds, and respects [CG#10]'s verifier note not to claim "any caller can rewrite any rating".
- Derivation matrix: filed the purchase row (SR#6). The rating-value row carries the 0 caveat. The list-of-ratings row's "exists" matches [CG#12]'s backend endpoint. The DERIVED rows match [CG#6] to [CG#9].
- Ledger: dispositions only, nothing asserted. Design-grounding note: an honest skip.

**will-not-survive-review.** Read the package as Quillmark's reviewer would, with the repository at the pinned commit. /workspace/bookstore HEAD is e439178 and the tree was clean at session start.
Applied the sufficiency test to all four argumentations:
- [VD#1] names its flip condition, but contradicts the package's own proposals (SR#7) and leaves out the load generator's other effects (SR#8).
- [VD#2] passes: it names question 12 and holds under either answer, since today's scale is the same.
- [VD#3] passes: it names its flip condition.
- [VD#4] fails on "every path" against "a separate correction path", and on the comment (SR#9).
Questions: filed 2 (SR#11) and 30 (SR#10). Checked question 8's limited-period option against [CG#10]'s overwritten creation time. That is an implementation repair rather than a customer-facing position, so not filed.

**unrecorded-defect-claim.** Read all four [VD#n] argumentations for any claim that a code defect is recorded, raised or known. No [CD#n] or [AS#n] exists. None makes such a claim, no record carries a defects field, no code-defect log exists, and round 1 records "code defects: none". These are consistent.
For the caller, though not this class: the package describes code behaviour that reads as defective without raising any [CDF#n]. Examples: an omitted value stored as 0, which [VD#2] repairs; the creation time overwritten on every edit ([CG#10]); and the setter's error text saying "between 0 and 5". None is claimed to be recorded, so nothing falls in this class.

## Reviewer notes

Inputs:
- Present: decisions, both grounding files, the ledger and the question set.
- Absent, as the caller stated: seeds, a code-defect log and prior reviews.
- Also read for context: interview/round-1.md, the customer's brd/source/quillmark-trusted-ratings-brd.md, the parent brd/brd-figures.md and brd/brd-defect-log.md, grounding/baselines.md, and the slice brd/brd-inventory.md, all under /workspace/specs/specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/.

References: bundle-packaging.md was read through line 756 of 958. The parts this review relies on (sections 1, 1.1, 3 and the plugin-free constraints) were read in full.

Facts not in any finding: SR#1, SR#2, SR#6, SR#7, SR#8, SR#9, SR#10 and SR#11 cite repository lines that no finding covers. They were read directly at /workspace/bookstore, which is checked out at the pinned commit:
- IngestController clearData deletes orders, carts and storage, and sometimes clients and books, every cycle.
- Order has no delivery field.
- OrderController has single and bulk order deletes.
- Ratings carry a comment field on the model and on the create, edit, list and details screens.
- The web menu and Delete All buttons are as described in SR#2.
These are attacks, not evidence. A re-grounding is the route to making any of them a finding.

Dependency between findings: SR#3 and SR#8 together cover both possibilities for [VD#1]'s load-generator half. Either the generator does not run live, in which case the premise is unsupported (SR#3), or it does, in which case its consequences are unaddressed (SR#8).

## Dispositions

Recorded one at a time by the operator on 2026-10-12; each finding's disposition, reason and, where
it applies, `fixed-by:` marker is on its own record above.

| Disposition | Findings |
|---|---|
| fixed | none |
| accepted-risk (all agreed, each carrying `fixed-by:`) | [SR#3], [SR#4], [SR#6], [SR#7], [SR#9] |
| escalated-to-customer | [SR#1], [SR#2], [SR#5], [SR#8], [SR#10], [SR#11] |
| rejected-with-reason | none |

No finding was disposed `fixed`, so no second reviewer pass ran. This is the slice's first package,
so no finding restates or re-escalates an earlier one.

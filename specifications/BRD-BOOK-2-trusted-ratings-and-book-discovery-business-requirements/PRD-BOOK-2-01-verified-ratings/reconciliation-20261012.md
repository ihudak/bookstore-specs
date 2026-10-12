# Reconciliation 20261012: BOOK-2-01

## Pass 1 — `customer-review-20261012.md`

### What the review was reconciled against

- **A package `/product-workflows:brd-package` built and handed off.**
  - `customer-review-prompt-20261012.md` is on `origin/main`, and the working copy matches it (`require-on-main` row A, pass).
  - Its siblings were verified on `origin/main`, not inferred: `self-review-20261012.md`, `customer-delivery-note-20261012.md` and `bundle-20261012/` (manifest present).
- **Which package the review answers:** BOOK-2-01 20261012. This was determined by the `Package reviewed: BOOK-2-01 20261012` line in the review's section 1.
- **Which self-review its `[SR#n]` ids resolve against:** `self-review-20261012.md`.

### The review

- **Canonicalised copy:** `customer-review-20261012.md`.
  - It arrived as `/workspace/customer-returns/BOOK-2-01 Customer Review 20261012.md`.
  - Its date comes from the review's own section 1 ("Review finished: 2026-10-12").
  - It was handed off before anything read it, on `brd/BOOK-2-01-verified-ratings` (PR #18).
- **Mode: schema.** `customer-review-reader` found all twelve section headings, in schema order. Section 7 is a table of rows, and section 4 uses only the four closed values. The reader was dispatched with `mode: auto`.
- **Verdict:** `approved-with-required-changes`.
- **Readiness statement (verbatim):** "The delivery team is clear to write the PRD and design for requirements 3.1, 3.2 (one rating per customer per book), 3.3, 3.4, 3.5, 3.6 (catalogue list only) and 3.7, together with the three additions in section 4, on the answers in this review and with [VD#1] and [VD#4] as amended in section 6, and is not clear to build anything for 3.8 or for a list of a book's ratings, both withdrawn from this release."
- **Section states:** sections 1–9, 11 and 12 are `present`. Section 10 is `stated-none` ("None", followed by three items for the delivery team to propose).
- **Evidence limitations, as stated:**
  - The review claims the Full tier. `bookstore` was at `e439178550371f2bc4f5a9acfee9a1c4f2289ae4`, and the three pin commands ran clean.
  - Nothing failed to open.
  - The store was not built or run.
  - Some claims were taken on the package's word, and section 5 names them.
  - Four facts rest on the customer's own knowledge: which front end shoppers use, that the load generator never runs live, the plans for third-party monitoring, and that the shops use no other rounding rule.
- **Anomalies (unrepaired):**
  1. **The reasons are not the customer's own words.** Section 1 says the reasons in sections 7 and 8 were written up by the customer's review agent from the interview, while the answers are the customer's. Each `[CD#n]` below quotes the reason as the review states it.
  2. **Round 1, question 25 was answered outside the options put** ("Neither, in this release").
  3. **Round 1, questions 26, 27 and 30 read "Not answered: moot".** These are declared refusals to answer.
  4. **Section 10 conflicts with its own content.** It says "None" but lists three items for the delivery team to propose. Section 6 leaves the storefront Ratings list for the Head of E-commerce to decide.
  5. **Some answers address findings the package never put to the customer.** [SR#3], [SR#4], [SR#6], [SR#7] and [SR#9] were disposed `accepted-risk`, not escalated, yet sections 4, 6, 8 and 9 answer them.
  6. **The section 8 assumption rows match no `[AS#n]`.** The register holds none.
  7. **Section 1 says "twelve" documents** and also mentions reading the delivery note, which sits outside the bundle.
  8. **No reviewer names are given**, only roles.
  9. **The go-live clean-up and the "cancel later" rule apply at different moments.** They do not contradict each other, but the delivery team should state this.
  10. **The review minted no identifiers.**

### What changed

**Customer decisions frozen.** Every row below was parsed in schema mode and confirmed by the operator one at a time. None was inferred.

| `[CD#n]` | Answers | `chosen` | Status | Customer's words |
|---|---|---|---|---|
| [CD#1] | Round 1, question 2 | completed and paid, not since cancelled (option 1) | decided | "An order for that book that is completed and paid, and not since cancelled (the reading proposed)." |
| [CD#2] | Round 1, question 3 | Remove them | decided | "**Remove them** at go-live: every rating from a customer with no qualifying order. Asked as [SR#4] reframes it, about any such ratings our live store holds." |
| [CD#3] | Round 1, question 5 | keep only the latest (option 1) | decided | "**Keep only the latest rating each customer gave each book, and remove the rest** (the reading proposed), at go-live. …" |
| [CD#4] | Round 1, question 6 | two independent requirements (option 1) | decided | "Yes: two independent requirements, and one rating per customer per book holds either way (the reading proposed)." |
| [CD#5] | Round 1, question 8 | 3.7 governs (option 1) | decided | "**3.7 governs:** a customer can change their rating at any time, and \"final\" in 3.2 is withdrawn." |
| [CD#6] | Round 1, question 9 | No: changed but not removed (option 1) | decided | "**No:** a customer can change their rating but not remove it." |
| [CD#7] | Round 1, question 12 | a requirement (option 1) | decided | "A requirement: every rating is a whole number of stars from 1 to 5 (the reading proposed)." |
| [CD#8] | Round 1, question 15 | two requirements, count matches average (option 1) | decided | "Yes: two requirements, and the number is always the count of the ratings in the average (the reading proposed)." |
| [CD#9] | Round 1, question 19 | "Not yet rated" alone (option 1) | decided | "\"Not yet rated\" alone, with no count (the reading proposed)." |
| [CD#10] | Round 1, question 21 | The catalogue list only | decided | "**The catalogue list only.**" |
| [CD#11] | Round 1, question 25 | outside the options | decided | "**Neither, in this release:** 3.8 is withdrawn. Quillmark will set up third-party monitoring itself later *(our own knowledge)*." |
| [CD#12] | Round 1, question 29 | No: illustrates an idea (option 2) | decided | "**No:** the note illustrates an idea, not a requirement of this release." |
| [CD#13] | [SR#1] of `self-review-20261012.md` | as the finding argues | decided | "Moot for this release (3.8 is withdrawn). We agree the question must be split when it returns." |
| [CD#14] | [SR#2] of `self-review-20261012.md` | as the package states | decided | "**Settled:** our shoppers use the `web/` application in the repository *(our own knowledge)*. For this release we accept that the rater is the customer whose e-mail is entered (section 8)." |
| [CD#15] | [SR#5] of `self-review-20261012.md` | as the finding argues | decided | "Settled by our answer to round 1, question 5, asked in the reframed form." |
| [CD#16] | [SR#8] of `self-review-20261012.md` | outside the options | decided | "Settled: none of these consequences reaches our live store, because the load generator never runs there *(our own knowledge)*." |
| [CD#17] | [SR#10] of `self-review-20261012.md` | outside the options | decided | "**Settled:** comments are kept but hidden from shoppers, the forms keep taking them, and a change may alter them (section 4 and [VD#4]). The date question is moot with question 30." |
| [CD#18] | [SR#11] of `self-review-20261012.md` | as the finding argues | decided | "Settled by our answer to round 1, question 2." |

- **Altitude.** Copied for [CD#1] to [CD#12] from each question's `- **Altitude:**` line. For [CD#13], [CD#15], [CD#17] and [CD#18] it was copied from the held question each finding targets. For [CD#16] it was copied from [VD#1] (`architecture`). For [CD#14], **this run decided it**: its finding targets the register and the question set together, so there was no single target to copy. It is `product`, because which front end shoppers use is the PRD's to state.
- **`evidence` for the `[SR#n]` answers** is the `[CG#n]` the finding's `rests_on` names.
- **[CD#13]'s `argumentation`** quotes its section 7 reason ("See round 1, question 27.") and the reason that row gives.
- **No `[CD#n]` was bound by the will-change rule.** Every finding cited carries `horizon: current`.

**Candidates not frozen**

| Candidate | Disposition | Why |
|---|---|---|
| Round 1, question 26 | Reject — declared refusal | "**Not answered: moot**, because 3.8 is withdrawn." Reason: "When 3.8 returns, we will need to know how our chosen tool's purchases and ratings can be told apart from shoppers'. Please re-ask then." |
| Round 1, question 27 | Reject — declared refusal | "**Not answered: moot**, because 3.8 is withdrawn." Reason: "When 3.8 returns, please split the question as [SR#1] proposes. …" |
| Round 1, question 30 | Reject — declared refusal | "**Not answered: moot**, because no list of a book's ratings is required (question 29)." Reason: "If the list returns, ask us again as [SR#10] proposes: which date, and what of the comments, which stay hidden from shoppers (section 4)." |
| Section 8 row: `web/` is the storefront (matched to [SR#2]) | Reject | It duplicates the [SR#2] answer frozen as [CD#14] |
| Section 8 row: the generator runs live, marked "corrected" | Recorded for a human | It answers no question the package put |
| Section 8 row: typed-e-mail identity, marked "accepted" | Recorded for a human | It answers no question the package put |
| Section 8 row: non-buyer and repeat ratings in live data, marked "cannot-say" | Recorded for a human | It answers no question the package put |

- No candidate was skipped as already reconciled. None re-affirmed a live record, and none was re-pointed. No conflicting-answer pick arose, and no missing-reason pick arose.

**Register moves (`decisions.md`)**

- **[VD#1]** went from `decided` to **`reopened`**. Causes: [CD#2], [CD#3], [CD#5], [CD#6] and [CD#16]. Its `Reopened 20261012:` paragraph says what each one changes.
- **[VD#4]** went from `decided` to **`reopened`**. Cause: [CD#17], which says comments are editable.
- **[VD#2] and [VD#3] are unchanged.** The review confirmed both.
- **Nothing else moved.** No `[AS#n]` was superseded (the register holds none). No record was superseded, re-decided in place or completed from `open`. No record was left at its status with a constraining `[CD#n]`.

**Questions closed.** Each of these was appended *answered by the customer* in `interview/round-1.md` and `interview/customer-questions.md`:
- Round 1, questions 2, 3, 5, 6, 8, 9, 12, 15, 19, 21, 25 and 29.
- They map to [CD#1] to [CD#12], in that order.

**Still held for the customer:** round 1, questions 26, 27 and 30. **Round 1 stays open**, so no `Status:` line was appended.

**Corrections (review section 12)**

| Review row | Disposition | Detail |
|---|---|---|
| `decisions.md` [VD#1]: restate per (a)–(f) | refused-with-reason | A direct edit of a register field is refused. The substance landed through the register: [VD#1] is reopened by [CD#2], [CD#3], [CD#5], [CD#6] and [CD#16]. It is re-decided by the next `/product-workflows:brd-interview` round, where the review's (a)–(f) is the customer's input. |
| `decisions.md` [VD#4]: restate | refused-with-reason | Same channel: [VD#4] is reopened by [CD#17] and re-decided in the next interview round. |
| `decisions.md`: record the four section 8 assumptions | refused-with-reason | This command never creates an `[AS#n]`. The substance landed in two places. [CD#14] covers the storefront and typed-e-mail identity, and [CD#16] covers the generator never running live. The cannot-say row is recorded for a human below. |
| defect log [DEF#1]–[DEF#15]: record resolutions | applied-with-deviation | Resolutions were written from each `[CD#n]`'s `settles` and from the withdrawals, not from the review's wording. [DEF#2] stays `open`: no customer decision settles it, and an `unsourced` defect closes on a grounding outcome, which this command never writes. |
| `brd/brd-inventory.md`: add three rows; mark [BR#3], [BR#10], [BR#11] withdrawn | refused-with-reason | Inventory rows mirror the immutable source, and no `[BR#n]` is minted here. The withdrawals landed in `coverage-ledger.md` through [CD#5], [CD#11] and [CD#12]. The three new requirements are recorded for a human below. |
| `grounding/code-grounding.md` [CG#4]: add further deletion paths | deferred-to-next-round | A `[CG#n]` changes only by independent re-derivation: `/product-workflows:prd-ground BOOK-2-01`. |
| `grounding/code-grounding.md`: derivation matrix row "Whether the rater bought the book" | deferred-to-next-round | Same: `/product-workflows:prd-ground BOOK-2-01`. |
| `grounding/code-grounding.md`: note under the matrix | deferred-to-next-round | Same: `/product-workflows:prd-ground BOOK-2-01`. |
| `interview/customer-questions.md`: questions 3 and 5 answered in reframed form | applied-with-deviation | The question text, which records what was asked, is not rewritten. The reframing is recorded in [CD#2], [CD#3] and [CD#15] and in this record, and both entries carry their *answered by the customer* line. |
| `interview/customer-questions.md`: mark questions 26, 27 and 30 moot | deferred-to-next-round | A declined question keeps its holding state, and nothing writes "moot" or "declined" into the question set. The requirements behind them are now withdrawn ([CD#11], [CD#12]). How those held questions are disposed is a later `/product-workflows:brd-interview` round's to settle. Until then they ship in the next package. |

**Banners added:** `customer-review-prompt-20261012.md` and `self-review-20261012.md`.

**Overturned bundle documents.** These are named rather than bannered. The copies under `bundle-20261012/` stay byte-identical to what the customer received, and are the ones to quote from:
- `BOOK-2-01-customer-review-prompt-20261012.md`
- `BOOK-2-01-decisions.md`: [VD#1] and [VD#4] are now `reopened`.
- `BOOK-2-01-customer-questions.md`: 12 of its questions are now answered.
- `BOOK-2-01-coverage-ledger.md`: [BR#3], [BR#10] and [BR#11] are now `rejected`.
- `BOOK-2-01-brd-defect-log.md`: 14 entries are now resolved.

**Defect resolutions**, written into the parent's log `specifications/BRD-BOOK-2-trusted-ratings-and-book-discovery-business-requirements/brd/brd-defect-log.md`. The cross-BRD guard ran first: pass, row A.
- `resolved-by` a customer decision:
  - [DEF#1]: `resolved-by: BOOK-2-01/[CD#1]`
  - [DEF#3]: `resolved-by: BOOK-2-01/[CD#3]`
  - [DEF#4]: `resolved-by: BOOK-2-01/[CD#4]`
  - [DEF#6]: `resolved-by: BOOK-2-01/[CD#7]`
  - [DEF#7]: `resolved-by: BOOK-2-01/[CD#8]`
  - [DEF#8]: `resolved-by: BOOK-2-01/[CD#9]`
  - [DEF#9]: `resolved-by: BOOK-2-01/[CD#10]`
- `withdrawn`:
  - [DEF#5]: a conflict whose listed row, [BR#3], is the one dropped by [CD#5]. The pair is [BR#3] and [BR#9]. The listed-row branch was taken because [BR#3] is the lower-numbered row, which holds the entry.
  - [DEF#10], [DEF#11] and [DEF#12]: [BR#10] was withdrawn by [CD#11].
  - [DEF#13], [DEF#14] and [DEF#15]: [BR#11] was withdrawn by [CD#12].
- [DEF#2] stays `open`.

**Ledger rows moved (`coverage-ledger.md`)**
- [BR#3]: from `covered-here` to `rejected: [DEF#5]`.
- [BR#10]: from `covered-here` to `rejected: [DEF#10]`.
- [BR#11]: from `covered-here` to `rejected: [DEF#13]`.

### The sweeps

**Changed-id set, as fixed:**
- [CD#1] to [CD#18], frozen.
- [VD#1] and [VD#4], reopened.
- [SR#1], [SR#2], [SR#5], [SR#8], [SR#10] and [SR#11], answered.
- [BR#3], [BR#10] and [BR#11], whose ledger rows moved.
- [DEF#1], [DEF#3]–[DEF#15], which gained a resolution.

**Propagation sweep:** none.
- No `brd-link.md` under `specifications/` declares a `depends-on:` naming BOOK-2-01.
- This ledger holds no `covered-by` row.
- No `conditional_on: BOOK-2-01/…` exists anywhere.

Nothing was disposed and nothing was recorded-not-written.

**Stale cross-reference sweep** (root: the parent folder; literal ids matched whitespace-tolerantly, plus a prose search for each changed position):

| File | Found | Outcome |
|---|---|---|
| `PRD-BOOK-2-01-verified-ratings/slices.md` | Says all 11 rows are `covered-here`. The [BR#3]/[BR#9] conflict is "the slice's open question to decide". [BR#10]'s monitor "cannot be settled". | **updated**, naming [CD#5], [CD#11] and [CD#12] |
| `PRD-BOOK-2-02-book-discovery/slices.md` | "Sibling BOOK-2-01's [BR#10] is in the same position." | **updated**, naming BOOK-2-01 [CD#11]. The cross-BRD guard passed. |
| `slices.md` (parent) | The claims list and the `covered-by: BOOK-2-01` allocation of [BR#1]–[BR#11] | still-true: allocation is unchanged, and the slice's ledger records the rows' fate |
| `coverage-ledger.md` (parent) | [BR#3], [BR#10], [BR#11] read `covered-by: BOOK-2-01` | still-true: delegation is unchanged, and the parent's line resolves them one hop to `rejected` |
| `brd/brd-inventory.md`, parent and slice | Rows [BR#3], [BR#10], [BR#11] with their defect lists | still-true: the text is the customer's verbatim, and withdrawal is recorded in the ledger |
| `brd/brd-figures.md` (parent) | *Rows* line "yields [BR#11]" | still-true: the image still yields that row, which is now `rejected`. The *Rows* line belongs to `/brd-intake`. |
| `brd/brd-defect-log.md` (parent) | Entries [DEF#1], [DEF#3]–[DEF#15] | still-true: this run's own resolutions |
| `brd-link.md` | `claims:` lists [BR#3], [BR#10], [BR#11] | still-true: the slice still claims them, and their fate is in the ledger |
| `coverage-ledger.md` (slice) | The three rows, and their defect lists | still-true: this run's own writes |
| `decisions.md` | [VD#1]'s argumentation opens on 3.2's finality. [VD#4]'s says "if ratings are final, nothing changes". | needs-a-human: both are reopened, and are re-decided in the next interview round. `argumentation` is appended to, never rewritten. |
| `decisions.md` | [VD#2] and [VD#3] argumentation | still-true: [CD#7] confirms 3.3 binds, and the rounding rule was confirmed |
| `grounding/code-grounding.md`, `grounding/design-grounding.md` | Findings on [BR#3], [BR#10], [BR#11] | still-true as findings of the pinned commit. The customer's challenges go to `/prd-ground`. |
| `interview/customer-questions.md` | Held questions 26, 27, 30, which concern now-withdrawn [BR#10]/[BR#11] | needs-a-human (see below) |
| `interview/round-1.md` | Question entries naming the changed ids | needs-a-human: a dated, append-only record. The closures were appended. |
| `customer-review-prompt-20261012.md`, `self-review-20261012.md` | Positions this run changed | needs-a-human: dated snapshots, now bannered |
| `customer-review-20261012.md` | The customer's own review | needs-a-human: the customer's document, never edited |
| `bundle-20261012/*` | Every bundle copy naming a changed id | needs-a-human: the byte-identical record of what was sent, never edited or bannered (named above) |

### What still needs a human

**Answers that settle no question the package put.** Each is quoted from the review's section 8:
- "The load generator runs against Quillmark's live store ([SR#3], [VD#1]'s reasoning) | corrected | It never runs against our live store, only in test and demonstration environments *(our own knowledge)*." Its substance is in [CD#16].
- "The store may identify the rater by the e-mail address entered, with no sign-in, for this release | accepted | We accept that someone could type a buyer's e-mail. Sign-in is not part of this release." It bears on [BR#1]. Its substance is in [CD#14].
- "Our live data holds ratings from non-buyers, and repeat ratings ([SR#4], [SR#5]) | cannot-say | We hold no count. Our answers to round 1, questions 3 and 5 apply to whatever exists." It bears on [BR#1] and [BR#2].

**Requirements the customer asked for that no `[BR#n]` covers.** These come from section 4's three `missing` rows:
- "**A rating's eligibility is checked when it is given and when it is changed.** If the customer later cancels the qualifying order, the rating stays and keeps counting. …"
- "**Operators, never shoppers, must be able to remove a single rating, or every rating of one customer, to meet an erasure request or to clean up fraud.** …"
- "**Rating comments are stored but never shown to shoppers.** The rating forms keep taking a comment, and a customer may change their comment when they change their rating. …"

The third is also frozen in substance as [CD#17]. The route that adds any of them as a `[BR#n]` is a revised source document from Quillmark, intaken over the root with `/product-workflows:brd-intake BOOK-2 @<revised-file>`. That re-run puts every row of the root's ledger back to `unallocated`. Short of that, a person takes them up with the customer.

**Questions still held, whose requirement is now withdrawn.** Round 1, questions 26 and 27 belong to [BR#10], which [CD#11] withdrew. Question 30 belongs to [BR#11], which [CD#12] withdrew. The customer declined them as moot and asked for them to be re-asked when 3.8 or the list returns. Until a later `/product-workflows:brd-interview` round disposes them, they keep round 1 open and ship in the next package.

**Code and design challenges** (review section 5). Each is quoted, and each is fixed by `/product-workflows:prd-ground BOOK-2-01`. The repository has not moved from the pin, so no `--rebaseline` is needed.
- [CG#4] and [VD#1]: "Starting the generator first clears every service's data, ratings included (`IngestController.java` line 55 …). `DELETE /api/v1/ingest`, when the generator is idle, does the same (lines 193–202). The ingest pod's startup script deletes all ratings directly … (`k8s/ingest.yaml` line 124)."
- The derivation matrix row "Whether the rater bought the book": "A shopper who deletes an order, or all of them, erases the only record that a purchase happened, and with it the right to rate or to change a rating under 3.1."
- [SR#3]'s citation: "line 113 is `sleep_period=360`. The POST that starts the generator is on line 132." The review needs no document change for this one.

**Reopened decisions.** [VD#1] and [VD#4] are re-decided in the next `/product-workflows:brd-interview` round, once round 1 is closed. The review's section 6, points (a) to (f) for [VD#1] and the correction for [VD#4], is the customer's stated position to argue against.

**Items the review leaves for the delivery team to propose** (section 6 and section 10). Each goes to the owner the review's section 9 names:
- how the operator removal path is kept from shoppers in a storefront with no sign-in;
- how purchase records survive the storefront's order Delete and Delete All;
- what of the storefront Ratings list remains for shoppers. It shows raters' e-mails and comments, and offers Update and Delete on every row.

**[DEF#2] stays `open`.** It is `unsourced`: it asserts the store already allows only buyers to rate. It closes on a grounding outcome, not on a customer decision.

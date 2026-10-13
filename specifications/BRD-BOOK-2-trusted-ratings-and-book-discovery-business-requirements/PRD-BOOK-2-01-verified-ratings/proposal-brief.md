---
kind: proposal-brief
key: BOOK-2-01
title: Verified ratings — proposal brief
date: 2026-10-12
readiness_tier: 3 · Architected
engagement_model: time-and-material
revision_of: none
---

# Proposal brief: Verified ratings (BOOK-2-01)

| | |
|---|---|
| Prepared by | USoft, for Quillmark |
| Date | 2026-10-12 |
| Readiness tier | **3 · Architected.** The requirement set has a PRD, verified code grounding, a settled decision register and an architecture document. It has no specification yet. |
| Engagement model | time-and-material |
| Revision | First revision |

This is a two-minute pre-read for the effort proposal in [proposal.md](proposal.md). Every figure here is taken from that document, and every figure is **human delivery hours**. Neither document carries a rate or any monetary figure.

## 1. Why the number is 215 hours and not 20

**Read at face value, your requirements 3.1 to 3.7, the three additions in your review, and testing them would take about 20 hours.** At face value, 3.1 and 3.7 describe how the store already works, 3.3's scale is already checked, and the averages could be computed in the browser. The proposal's estimate is **215 hours expected**, with a credible range of **173.8 to 297.5 hours**, about 35.8 developer-days.

The gap is not padding. Eleven cost drivers account for 125 hours of it. Each cites a record in this folder: a code finding that a second agent re-derived independently, a decision you made, or a code defect USoft confirmed for repair.

- **The largest single driver is buyers only: 29 hours, 13.5 % of the estimate ([ED#1]).** Requirement 3.1 says "the store works this way today". It does not. Creating a rating checks only that the book and the customer exist, and nothing in the ratings service consults orders ([CG#2]). You confirmed this in your review. Buyers only is therefore new work: the ratings service reads purchases from orders, refuses everyone else with one identical answer, and the storefront shows that answer.
- **The other drivers come from the code as it stands, and from your decisions.** Nothing enforces one rating per customer and book ([ED#3], 8 hours). An update can rewrite a rating's customer and book ([ED#5], 11 hours). A rating with no stars is stored as 0 ([ED#6], 12 hours). No average, count or "Not yet rated" exists anywhere, and a server-side summary keeps raters' e-mails and comments out of the browser ([ED#7], 21 hours). Your decisions on repeat ratings ([ED#4], 10 hours), comments ([ED#8], 10 hours) and deletion add work. The deletion driver includes USoft's proposal for the ratings list ([ED#9], 8 hours). So do the empty ratings table each deploy needs ([ED#10], 7 hours) and the unchanged load generator in test environments ([ED#11], 5 hours). Your definition of a qualifying order ([ED#2]) adds 4 hours.
- **The two packages every engagement carries account for the remaining 70 hours, 32.6 %:** discovery and design ([WP#1]), and the part of test, UAT and release ([WP#10]) that no driver accounts for.

The bridge is 20 (face value) + 125 (drivers) + 70 (the fixed packages) = 215 hours.

## 2. Corrections this revision owes you

None. This is the first revision of the proposal, so nothing earlier is corrected or withdrawn.

## 3. Reconciliation to your prior estimate

None. No prior estimate was given to reconcile against.

## 4. Work deliberately left unpriced

None. No package is graded Low, so no package waits on a re-estimate gate to be priced. Once the specification exists, USoft will re-run the proposal and size testing from the specification's test cases. That is a planned revision, not a gate.

## 5. What is needed before week 1

Your decision register leaves nothing open: no open assumption, no unanswered question, and no blocking defect. Two things from the proposal's dependencies (section 13) are needed at the start:

- **Before week 1 (D1):** access to the `bookstore` repository, and a test environment running the orders, ratings, books, clients and storefront services.
- **In week 1 (D2):** your Head of E-commerce, available to approve the specification. That covers its test cases, the text shown when ratings cannot reach orders, and the text shown when a rating summary cannot load.

Later dependencies, D3 to D7, are listed in section 13 of the proposal:

- USoft's three answers to the points your review left with it reach your owners (D3).
- The ratings table is emptied at each deploy, and the orders address is reachable (D4, D5).
- UAT participants are available (D6).
- You agree to emptying the live ratings table again at the deploy (D7).

## 6. Where USoft's architecture and your own text still disagree

Four points are still open. Three are USoft's answers to the points your review left with it, and they are on their way to your owners (D3). The fourth is an agreement USoft needs from you (D7). On the first two and the fourth, USoft's architecture goes beyond what you have recorded.

- **Who can remove a rating.** Your review requires that "Operators, never shoppers, must be able to remove a single rating", and asks how that path is kept from shoppers. The store has no sign-in ([CD#14]), so USoft's architecture leaves the operators' per-rating removal reachable by anyone who calls the ratings API. USoft's answer ([VD#1]) accepts that risk until the store has sign-in. It has not yet reached your owners (D3, and risk R8 in the proposal).
- **Purchase records and deleted orders.** Your review asks how purchase records survive the storefront's order Delete and Delete All. USoft's answer ([VD#5]) is that they don't, because orders stay unchanged in this release. A rating already given stays and keeps counting, but its customer can no longer change it, and a shopper using Delete All would end that right for every buyer. USoft's grounding record still describes the purchase record as kept, which [VD#5] concedes overstates it. This answer has not yet reached your owners either (D3, and risk R9).
- **What remains of the storefront's ratings list.** Your review left this screen for USoft to propose. USoft's answer ([VD#6]) keeps each rater's e-mail and the Update action, and drops the comment column, Delete and Delete All. The store has no sign-in, so the list is how a customer finds their rating to change it. This answer has not yet reached your owners either (D3).
- **Emptying the live ratings table at the deploy (needs your agreement).** Your follow-up decision ([CD#19]) says the release starts with no ratings and carries no clean-up of existing ratings. USoft's architecture reads that as emptying the ratings table again at each deploy, the live store included. That removes any rating given between 2026-10-12 and the deploy, so it needs your agreement (D7).

Two of your sentences disagreed with the code, and both are now settled:

- **3.1's "The store works this way today".** The code does not limit ratings to buyers ([CG#2]). Your review confirmed that sentence was mistaken, and the proposal prices buyers only as new work.
- **3.2's "A rating, once given, is final".** It conflicted with 3.7, and you withdrew it ([CD#5]).

There is also a tension between the architecture's accepted risks and the 100 % target in [SM#1]. An order already paid for can be moved to another customer or book and still read as completed, so a rating could count for a book its customer did not pay for. That is accepted under [VD#5] (risk R5).

---

*This estimate is indicative, valid for 30 days, and states human delivery hours only; commercial terms are agreed separately.*

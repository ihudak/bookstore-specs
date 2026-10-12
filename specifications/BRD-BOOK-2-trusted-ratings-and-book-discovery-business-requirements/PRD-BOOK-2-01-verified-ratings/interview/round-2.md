# Interview round 2: BOOK-2-01

Opened 2026-10-12, after round 1 closed. Recorded by the delivery team (PM) by hand, in the shape
`/product-workflows:brd-interview BOOK-2-01` writes, to settle what the customer's returned review
(`customer-review-20261012.md`, reconciled in `reconciliation-20261012.md`) left with the delivery
team. The grounding is unchanged since round 1 (`bookstore` pinned at `e439178`), and no design
grounding exists for this slice.

generated against: [CG#1]=CONFIRMED/agree, [CG#2]=REWRITTEN/extend, [CG#3]=FALSE-FRIEND/contradict, [CG#4]=REWRITTEN/agree, [CG#5]=AMENDED/contradict, [CG#6]=REWRITTEN/agree, [CG#7]=REWRITTEN/agree, [CG#8]=REWRITTEN/agree, [CG#9]=REWRITTEN/agree, [CG#10]=AMENDED/contradict, [CG#11]=FALSE-FRIEND/extend, [CG#12]=FALSE-FRIEND/contradict

## Scope

This round puts again the two records the review reopened, [VD#1] and [VD#4], each argued against
the review's section 6. It also settles the three points the review's section 10 left for the
delivery team to propose:
- how the operators' removal path is kept from shoppers (question 1);
- how purchase records survive the storefront's order Delete and Delete All (question 3);
- what remains of the storefront's Ratings list for shoppers (question 4).

Round 1's last held questions, 26, 27 and 30, were closed with nothing frozen before this round
opened. Their requirements are withdrawn ([CD#11], [CD#12]) and their requirement defects
[DEF#10], [DEF#11] and [DEF#14] are `withdrawn`. The customer asked for them to be put again only
if 3.8 or the list of a book's ratings returns. No question in this round is held for the customer.

The review's code challenges are answered by these decisions rather than by re-grounding. The load
generator's further rating-deletion paths (its section 5, challenge 1) no longer bear on [VD#1],
which leaves the generator unchanged. The purchase-record gap in the derivation matrix (challenge 2)
is decided in [VD#5].

## Questions

### 1. [V] — [BR#1], [BR#2], [BR#9]

Now that a rating is no longer final, who may remove a rating, and what happens to bulk deletion,
the go-live clean-up and the load generator?

- **Re-puts:** [VD#1]
- **State:** decided — [VD#1]

### 2. [V] — [BR#9]

What may a change to a rating alter, now that comments are kept and may change?

- **Re-puts:** [VD#4]
- **State:** decided — [VD#4]

### 3. [V] — [BR#1]

Does this release change how orders are kept, so that the purchase behind a rating survives the
storefront's order Delete and Delete All?

- **Re-puts:** none
- **State:** decided — [VD#5]

### 4. [V] — [BR#9], [BR#2]

What of the storefront's Ratings list remains for shoppers?

- **Re-puts:** none
- **State:** decided — [VD#6]

## Account

requirement defects: none this BRD asks
code defects: none
Status: closed 20261012 — every question decided: [VD#1] and [VD#4] re-decided, [VD#5] and [VD#6] recorded

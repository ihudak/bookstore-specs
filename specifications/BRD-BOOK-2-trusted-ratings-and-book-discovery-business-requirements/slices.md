---
kind: slices
key: BOOK-2
---

# Slices: BOOK-2

Written by `/brd-split` (split_mode: full). Two slices were carved, and every one of the 19
requirements in this BRD's coverage ledger was given to one of them (`covered-by`). Nothing was
deferred, rejected or superseded.

## Slice BOOK-2-01 — Verified ratings

- **Folder:** `PRD-BOOK-2-01-verified-ratings/`
- **Claims:** [BR#1], [BR#2], [BR#3], [BR#4], [BR#5], [BR#6], [BR#7], [BR#8], [BR#9], [BR#10], [BR#11]
- **Rationale:** the instruction puts every requirement in section 3 into this slice. That is
  [BR#1]–[BR#10], anchored at `3. Verified ratings`, and [BR#11], the requirement drawn from the
  book-page sketch (`source/assets/book-page-wireframe.svg`). The instruction cuts this slice first
  because the Top rated shelf in section 4 counts only verified ratings.

## Slice BOOK-2-02 — Book discovery

- **Folder:** `PRD-BOOK-2-02-book-discovery/`
- **Claims:** [BR#12], [BR#13], [BR#14], [BR#15], [BR#16], [BR#17], [BR#18], [BR#19]
- **Rationale:** the instruction puts every requirement in section 4 into this slice, which is
  [BR#12]–[BR#19], all anchored at `4. Helping readers find their next book`.
- **Depends on BOOK-2-01:** [BR#16] (highest average rating) and [BR#17] (count only ratings from
  customers who bought the book) both read the verified ratings BOOK-2-01 builds.

## The slicing instruction

The instruction, verbatim:

> Two slices. First, Verified ratings: every requirement in section 3, including the one drawn from the book-page sketch (BR#1 to BR#11). It comes first because the Top rated shelf in section 4 counts only verified ratings. Second, Book discovery: every requirement in section 4 (BR#12 to BR#19).

How it was read:

- **Placed directly (Step A): all 19 rows.** The instruction names both groups by section and by an
  explicit `[BR#n]` range, and each row's `source_anchor` agrees with that range. [BR#11] is
  anchored on the wireframe rather than on a section heading, and it is the "one drawn from the
  book-page sketch" the instruction names.
- **Settled by grill (Step B): none.** Nothing was left to settle, so no question was asked.
- **Left unplaced: none.**

## The ledger walk

All 19 rows were walked one at a time. The bulk offer (Step 1) did not fire, because two slices
stand and it fires only when exactly one does. On every row the operator took the instruction's
recommendation:

| Disposition | Rows |
|---|---|
| `covered-by: BOOK-2-01` | [BR#1] … [BR#11] (11) |
| `covered-by: BOOK-2-02` | [BR#12] … [BR#19] (8) |

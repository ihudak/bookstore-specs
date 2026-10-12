---
kind: slices
key: BOOK-2-01
---

# Slices: BOOK-2-01

Written by `/brd-split` (split_mode: allocate-only). No slice was proposed. BOOK-2-01 is a slice of
BOOK-2. Nesting is capped at one level, so no child BRD can be created below it. This run allocated
BOOK-2-01's own ledger and created nothing. All 11 claimed requirements are now `covered-here`.
Nothing was deferred, rejected or superseded. Since reconciled (20261012): [BR#3], [BR#10] and [BR#11]
now read `rejected` in the ledger, withdrawn by the customer in [CD#5], [CD#11] and [CD#12].

## The ledger walk — bulk resolution

The Step 1 uniform-answer offer fired, because 11 rows were `unallocated` and no instruction placed
any of them elsewhere. The operator took it in one confirmation. It wrote:

| Disposition | Rows |
|---|---|
| `covered-here` | [BR#1], [BR#2], [BR#3], [BR#4], [BR#5], [BR#6], [BR#7], [BR#8], [BR#9], [BR#10], [BR#11] (11) |

- **Held back: none.** The operator named no row to walk one at a time, and no instruction placed a
  row on another disposition.
- **Left open for `/brd-interview`.** The offer named three rows that could be walked individually,
  and the operator chose not to. They now read `covered-here` like the rest:
  - [BR#3] ("a rating, once given, is final") and [BR#9] ("a customer must be able to change their
    rating at any time") conflict, per the parent's [DEF#5], and the code matches [BR#9] ([CG#4],
    [CG#10]). Building both rows here does not settle which one stands. That is the slice's open
    question to decide. Settled 20261012 by [CD#5]: 3.7 governs, and [BR#3] is withdrawn.
  - [BR#10]'s synthetic monitor is a FALSE-FRIEND at the pinned commit ([CG#11]). Whether a
    Dynatrace synthetic monitor exists in the tenant cannot be settled from the repository
    ([DEF#12]). Settled 20261012 by [CD#11]: 3.8 is withdrawn from this release, and [DEF#12]
    with it.

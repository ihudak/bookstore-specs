# Grounding baselines: BOOK-2-01

> This copy omits the block of filing fields at the top of the working document, which means nothing outside the delivery team's own records. It recorded that these are the grounding baselines of BOOK-2-01.

One dated entry per repository per grounding run, appended and never overwritten. A reviewer can
re-run the three commands against their own checkout to confirm the pin.

## 2026-10-12 — bookstore

- repo: bookstore
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  verification:
    - git -C "<repo>" rev-parse HEAD → e439178550371f2bc4f5a9acfee9a1c4f2289ae4
    - git -C "<repo>" diff --ignore-cr-at-eol --stat → (empty)
    - git -C "<repo>" status --porcelain → (empty; no entry to line-count compare)
  result: clean — pinned at a verified, unmodified commit
  finding: [CG#1]

# Design glossary: BOOK-2-01-02

Terms the design settles beyond the specification's `_glossary.md` (this folder) and the PRD-level `../_glossary.md`.

- **Write outcome** — the one result the storefront takes from a give or a change, from which the form picks its outcome text: *stored* (`200`), *buyers-only* (`403`), *invalid* (`400` with a readable reason), *unavailable* (`503`), *gone* (`404` to a change only) and *cannot-save* (everything else — a `400` with no readable reason, a `404` to a give, any other status, no answer within the bounded time, the ratings service unreachable).
- **Load outcome** — the one result the update form takes from its load of a rating: *loaded* (a readable rating), *gone* (`404`) or *cannot-load* (everything else, the bounded time included).
- **Summary outcome** — the one result the book page takes from a book's rating summary, and the catalogue list from the summaries, for each book: *rated* (a readable count above 0 and its average), *unrated* (a readable count of 0, or a book a readable summaries response does not list) or *unavailable* (anything else, the bounded time included).
- **Late answer** — an answer that arrives after the bounded time; the storefront has already cancelled the request and shown its failure text, and the answer changes nothing.

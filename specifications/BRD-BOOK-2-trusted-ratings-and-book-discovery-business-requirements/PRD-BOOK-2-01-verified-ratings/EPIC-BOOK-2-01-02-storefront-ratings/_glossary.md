# Glossary: BOOK-2-01-02 specification

Terms follow the PRD-level specification's glossary (`../_glossary.md`), narrowed to what the storefront shows and sends.

- **Rating form** — the storefront's "Create Rating" screen, opened from the Ratings list's New Rating; the PRD's "add form".
- **Update form** — the screen the Ratings list's Update opens for one rating, headed "Update Rating".
- **Give** — a `POST /api/v1/ratings` the rating form sends. The ratings service applies a give for an e-mail and ISBN that already hold a rating as a change ([AD#3], [AD#4]); the rating form does not know which it was.
- **Change** — a `PUT /api/v1/ratings/{id}` the update form sends for the rating it loaded.
- **Outcome text** — the text a form shows beside Submit after a give or change that was not stored: the buyers-only text, a `400` reason, the unavailable text, "This rating no longer exists. Nothing was saved." (update form only) or the cannot-save text. Never a browser alert; each Submit clears it.
- **Readable reason** — a `message` field in the JSON error body of a `400` that holds at least one character other than a space. A `400` without one is answered with the cannot-save text.
- **Cannot be saved** — a give or change that could not reach the ratings service, got no answer within the bounded time, or got an answer no other criterion covers; the form shows "We can't save your rating right now. Please try again in a few minutes." The text does not say whether the rating was saved, and repeating the give or change is safe ([AD#4]).
- **Cannot be loaded** — the update form could not reach the ratings service to read the rating, got no answer within the bounded time, or got an answer other than the rating or `404`; the form shows "We can't load this rating right now. Please try again in a few minutes." in place of its fields and Submit.
- **Bounded time** — the limit the storefront puts on waiting for an answer of the ratings service to each of these requests: a book's rating summary, the summaries, a give, a change and the update form's load of a rating. At most 10 seconds; the exact value is set by /design. An answer that arrives after the bounded time is disregarded, as if it had never come.
- **Rating summary** — the ratings service's answer to `GET /api/v1/ratings/summary?isbn=<ISBN>`: the book's ISBN, `count` of ratings and `average`, rounded half up to one decimal place by the ratings service, `null` when the count is 0 ([AD#6]).
- **Readable summary** — a `200` whose body has a whole-number `count`, and, whenever the count is above 0, a numeric `average` with at most one decimal place. Anything else counts as a summary that could not be loaded.
- **Readable summaries response** — a `200` whose body is a list of entries, each a readable summary for one ISBN; a response holding any entry that is not one is not readable. A book a readable response does not name holds no rating ([AD#6]).
- **Rating line** — the book page's line below the book's other details, "★ 4.3 (4 ratings)"; "Not yet rated" or "Rating unavailable" stands in its place, and nothing stands there while the summary loads.
- **Updated At** — the time a rating was last given or changed; the Ratings list's column for it is headed "LastUpdated". **Created At** — the time the rating was first given.

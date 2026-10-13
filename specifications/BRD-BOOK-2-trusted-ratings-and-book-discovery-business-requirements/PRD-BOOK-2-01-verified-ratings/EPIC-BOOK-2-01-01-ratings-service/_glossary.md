# Glossary: BOOK-2-01-01 specification

Terms follow the PRD-level specification's glossary (`../_glossary.md`), narrowed to what the ratings service answers.

- **Give** — a `POST /api/v1/ratings` request to rate a book, from the storefront's rating form or from any other caller. A give for an e-mail and ISBN that already hold a rating is applied to it as a change ([AD#3], [AD#4]).
- **Change** — a `PUT /api/v1/ratings/{id}` request that alters an existing rating's stars or comment, or a give applied to an existing rating as one.
- **Pair** — a rating's e-mail and ISBN, compared ignoring letter case, as the ratings store compares text ([AD#3]); each pair holds at most one rating.
- **Qualifying order** — an order for the book, under the e-mail as the give or change carries it (or the stored e-mail, for a change that carries none), that orders reports as completed, which means paid and not since cancelled ([CD#1], [AD#2]).
- **Buyers-only refusal** — the `403` the ratings service answers for every reason a give or change lacks a qualifying order, with a body identical in every field except the time of the answer and the request path, and the `message` "Only customers who have bought this book can rate it." ([AD#4]).
- **Refused as invalid** — a `400` for a fault in the request itself, with its reason in the `message` field of the error body; decided before any call to orders ([AD#4]).
- **Refused as unavailable** — a `503` because orders did not answer the purchase check within the ratings service's bounded time, could not be reached, answered with any status other than a success or not-found, or answered a success the ratings service cannot read ([AD#2]).
- **Bounded time** — the limit the ratings service puts on waiting for orders' answer; its value is a design decision (`ard.md`, Deferred).
- **Rating summary** — a book's ISBN, `count` of stored ratings and `average`, their mean rounded half up to one decimal place from the exact quotient of the star sum by the count, or `null` when the count is 0 ([AD#6]).
- **Ratings store** — where the ratings service keeps ratings, and where Quillmark reads comments, since no answer of the service carries one ([AD#7]).
- **Updated At** — the time a rating was last given or changed. **Created At** — the time the rating was first given.
- **Empty comment** — a comment that is empty, absent or only spaces. [AD#4]'s "an absent or empty comment keeps the stored comment" is read this way, as the PRD-level specification and the Epic read it: on a change it keeps the stored comment ([U02] [AC05]), and on a give that creates a rating it stores none ([U04] [AC02]).
- **Same values** — [AD#4]'s "repeating an identical give or change leaves the same single rating with the same values" means the same stars, comment, e-mail and ISBN; Updated At still moves to the time of the repeat ([U02] [AC07]), as the PRD-level specification reads it.

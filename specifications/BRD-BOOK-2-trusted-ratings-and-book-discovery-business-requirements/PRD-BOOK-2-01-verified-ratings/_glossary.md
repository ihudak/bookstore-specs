# Glossary: BOOK-2-01 specification

- **Qualifying order** — an order for the book, under the e-mail entered, that is completed and paid and not since cancelled, as the orders service reports it at the moment of the give or change ([CD#1], [AD#2]).
- **Give** — a request to the ratings service to rate a book: from the rating form (the storefront's "Create Rating" screen, the PRD's "add form"), or sent straight to the ratings write interface ([AD#4]). A give for an e-mail and ISBN that already hold a rating is applied as a change.
- **Change** — a request that alters an existing rating's stars or comment: through Update on the Ratings list (the update form), a change sent straight to the ratings write interface, or a give applied as one.
- **Ratings write interface** — the ratings service's give and change interface that the storefront and any other caller use ([AD#4]).
- **Buyers-only refusal** — the one identical refusal the ratings service gives for every reason a give or change lacks a qualifying order; the storefront shows it as "Only customers who have bought this book can rate it." ([AC#2], [AD#4]).
- **Refused as invalid** — a write refused for a missing or out-of-scale star value, a missing or malformed e-mail or ISBN, or a change that would move the rating; decided before the purchase check ([AD#4]).
- **Refused as unavailable** — a write refused because orders could not answer the purchase check; the storefront shows "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes." ([AD#2]).
- **Rating summary** — a book's rating count and its average rounded half up to one decimal place, computed by the ratings service on each request ([AD#6]).
- **Rating line** — the book page's "★ 4.2 (37 ratings)" line; "Not yet rated" or "Rating unavailable" stands in its place.
- **Updated At** — the time a rating was last given or changed; the Ratings list's column for it is headed "LastUpdated". **Created At** is the time the rating was first given.
- **Cannot be saved** — the rating form or the update form could not reach the ratings service, or got an answer no other criterion covers; the form shows "We can't save your rating right now. Please try again in a few minutes."

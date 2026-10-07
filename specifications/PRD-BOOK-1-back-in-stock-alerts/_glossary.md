# Glossary — BOOK-1 specification

- **Current client** — the client the person using the store has chosen in the store, identified by that client's email ([AD#2]). The store has no sign-in; this replaces the PRD's "signed-in client".
- **No current client selected** — the state in which nobody has been chosen; replaces the PRD's "visitor who is not signed in as a client".
- **Available copies / no available copies** — a book has no available copies when the store holds no stock for it or its stock is zero ([AD#3]); it *returns to stock* (comes back into stock) when a change takes it from none to one or more.
- **Removed from the catalogue (single removal)** — one book deleted from the catalogue on its own. Unpublishing is not removal ([AD#6]).
- **Bulk catalogue reset** — deleting every book at once (the catalogue's Delete All), whoever runs it: the synthetic traffic's bulk loop, the web store's *Delete All* action or the ingest service's start-up reset ([AD#7]).
- **The synthetic journey's own books, clients, subscriptions and alerts** — the reserved data of [AD#11], which the ingest service's back-in-stock journey uses and which no bulk reset removes. A subscription or alert is the journey's own when its book or its client is the journey's, so another client's subscription to a journey book is kept too.
- **Told of a reset / missed a reset** — whether the books service's single call to the alerts service on a bulk catalogue reset ([AD#7]) reached it. A missed reset's leftover subscriptions are withdrawn later by the alerts service's own check ([AD#5]).

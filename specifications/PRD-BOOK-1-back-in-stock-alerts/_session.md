# /specify session — BOOK-1

- **Command:** /product-workflows:specify BOOK-1
- **Feature folder:** specifications/PRD-BOOK-1-back-in-stock-alerts (PRD-level, focus_key: none)
- **Granularity:** broad PRD-level spec — the PRD has no Epics and is multi-component (ARD components: alerts, storage, books, clients, web, ingest; k8s deploy)
- **Refresh policy:** fetch + pull default branch
- **Repos:** bookstore @ /workspace/bookstore (master)
- **ARD:** specifications/PRD-BOOK-1-back-in-stock-alerts/ard.md — status: found, [AD#1]–[AD#11] live
- **Grounding:** docs OFF, architecture OFF, team decisions OFF; no grounding/ folder

## Stages

- [x] 1. Header + Problem statement — written, confirmed by the user
- [x] 2. Scope — written, confirmed by the user (inferred out-of-scope items confirmed)
- [x] 3. User stories — [U01]–[U08] written, confirmed by the user
- [x] 4. Acceptance criteria — 52 written (the play-back said 50, a miscount of the same listed set), confirmed by the user
- [x] 5. Test cases — 128 written on the confirmed conventions (≥1 happy path and ≥1 negative/boundary per criterion)

## Decisions

1. **Client term.** The spec says *the current client* — a client chosen in the store, per [AD#2] — and *no current client selected* replaces the PRD's "visitor not signed in as a client". The PRD's "signed-in" wording is left for `/product-workflows:update-prd`, as the ARD's first open question already routes it. (user)
2. **Problem statement** confirmed as played back, including "or leaving the catalogue" (inferred from [US#5], confirmed). (user)
3. **Bulk catalogue reset follows [AD#7].** A bulk reset clears every non-reserved subscription and alert and sends no alert; withdrawal alerts apply to removing a single book. The PRD's [AC#12]/[AC#14] exception is left to `/product-workflows:update-prd`, per the ARD's second open question. No open question in the spec. (user)
4. **Current-client choice is remembered in the browser** — across reloads, new tabs and browser restarts, until changed or cleared. (user)
5. **The current client is chosen from the Clients pages** — a "make current client" action on each client in the Clients list and on its details page; every page shows the current client's email with a way to clear it. (user)
6. **Book page shows nothing new when the book has copies** — settled by the PRD's [AC#5] guard: only the subscription offer (on a book with no available copies, for a current client) is added; with no current client selected, no offer. (fact)
7. **Stage 3 stories** [U01]–[U08] confirmed: [U03] (current client) and [U07] (silent bulk reset) added from decisions 3–5; the unread count sits in [U02]; journey data surviving resets sits in [U08]. (user)
8. **Pre-release performance guard added.** Under the same synthetic load and the same store configuration, the p95 of the browse, cart and order journeys for clients holding no subscription stays within 10% of a baseline run without the feature; [SMC#1] stays the production measure. (user)
9. **Stage 4 criteria confirmed**, inferred ones included: subscription failure reasons from [AD#8]'s 409/404/503; a stock change, a removal and a reset each complete when alerts does not answer ([AD#4], [AD#6], [AD#7]); a missed reset's leftovers are withdrawn with withdrawal alerts ([AD#5], [AD#7]) — the one case where a reset produces alerts; the pending list links each title to its book's page and shows the date subscribed. (user)
10. **Performance comparison runs last 1 hour each** (baseline without the feature, then with it, same synthetic load and store configuration). (user)
11. **Test-case conventions** confirmed: self-contained setup with the bulk loop off unless stated; one-minute limits checked at 60 seconds, with a lost-notice case per recovery criterion; the 30-day boundary at ±1 minute; 1,000 subscribers on one book; privacy cases for another client's data; journey cases over 5-minute windows. (user)
12. **Review 1 — BLOCK** (7 BLOCKER, 3 MAJOR, 13 MINOR, 5 NIT). Fix cycle scope: blockers, majors and the mechanical minors and nits; the four product-input minors (observability, a bound on books waited on, stopping the journey, the link of a back-in-stock alert whose book was later removed) stay open. (user)
13. **Out of scope: marking an alert unread again, and restoring a dismissed alert** — matching [AD#8], which has neither operation. (user)
14. **With no current client the store shows no way into the alerts list or the pending-subscriptions list**; the choose-a-client prompt appears only when a list is opened by its address. (user)
15. **When the alerts service (or storage, on a book page) does not answer**: the book page says subscriptions are unavailable in place of the action and the subscribed state; no unread count; the two lists say they are unavailable and to try again later; a cancel or dismissal with no answer says it could not be done and leaves the item as it was; one rejected because the current client changed in another tab shows the current client's list with a notice that the client changed. (user)
16. **Subscription failure reasons merged**: "the book or the current client no longer exists", since [AD#8] answers both with one 404; no contract change. (user)
17. **Fix cycle applied**: 13 criteria added and [U06] AC04 split into four — 65 criteria, 161 test cases; [U07] limited to a reset the alerts service is told of, with the missed-reset withdrawal stated in Scope; alert times defined as when the store recorded the change; [AD#N] tags added to every cross-component criterion. (fix)
18. **Re-review — BLOCK** (2 BLOCKER, 2 MAJOR, 13 MINOR, 5 NIT). Both BLOCKERs were contradictions the fix cycle introduced. Resolved by manual fix notes — the reviewer's own fixes, applied in one bounded pass with no further review: [U01] AC07 TC04 and AC08 TC02 now open the book page while storage answers and stop it before subscribing; [U02] AC11 and [U04] AC06 now trigger only when the web store reads the list. The re-review's MAJOR, MINOR and NIT findings are deferred to the final report. (user)

## Open questions

None in the specification. Review findings left open are listed in the run's final report.


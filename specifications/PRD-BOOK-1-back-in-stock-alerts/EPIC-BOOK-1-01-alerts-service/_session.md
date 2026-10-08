# /specify session — BOOK-1-01

- **Command:** /product-workflows:specify BOOK-1-01
- **Feature folder:** specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01-alerts-service (Epic-level, focus_key: BOOK-1-01)
- **prd_dir:** specifications/PRD-BOOK-1-back-in-stock-alerts (idea route, no brd-link.md)
- **Gates:** PRD `prd.md` pass; Epic `epic.md` pass; register gate n/a (idea route)
- **Refresh policy:** fetch + pull default branch
- **Repos:** bookstore @ /workspace/bookstore — the Epic's target `bookstore:alerts` (paths: alerts) narrows the set to this repository
- **ARD:** specifications/PRD-BOOK-1-back-in-stock-alerts/ard.md — status: found, [AD#1]–[AD#11] live; no Epic-level ARD
- **Grounding:** docs OFF (/workspace/bookstore-docs/ holds no markdown files), architecture OFF (/workspace/architecture holds no catalog, radar or ADR folder), team decisions OFF (no knowledge base yet); no grounding/ folder
- **Model routing:** SIGNIFICANT — new service with its own schema and public REST API; authoring on claude-opus-5-5

## Stages

- [x] 1. Header + Problem statement — written, confirmed by the user
- [x] 2. Scope — written, confirmed by the user (inferred items confirmed: failure reasons, version/config operations, unpublished books subscribable, no list cap)
- [x] 3. User stories — [U01]–[U07] written, confirmed by the user
- [x] 4. Acceptance criteria — 49 written, confirmed by the user (inferred: unpublished subscribable, title stored at subscribe, all-or-nothing recording, books-first in the sweep, mark-read covers every alert, version/config operations)
- [x] 5. Test cases — 136 written on the confirmed conventions (≥1 happy path and ≥1 negative/boundary per criterion)

## Decisions

1. **Problem statement** confirmed as played back, inferred from epic.md, the PRD and the PRD-level spec, including the framing that the rest of the feature has nothing to act on until this record exists. (user)
2. **Email matching ignores letter case** in every operation — subscribe's check, every read, cancel, dismiss and delete-all's reserved-email rule; a subscription's `email` is returned as given at subscribe; an email with `+` is carried intact, including to clients' `find`. Grounds: clients' `find` is case-insensitive only through MySQL collation (scan). (user)
3. **Alerts must come up on an existing deployment** whose PostgreSQL volume predates this feature, without resetting other services' data; how is design's. Grounds: `01-create-db.sql` runs only on a fresh volume and `restart.sh -nodb` exists to avoid data resets. (user)
4. **Sweep bound: the 30-second cycle is guaranteed for up to 100 distinct books with pending subscriptions**; above that every book is still checked but a pass may run longer; notices that arrive are unaffected. Grounds: demo catalogue ≈2,150 books (`k8s/ingest.yaml:132`), ~3 requests per checked ISBN; the PRD-level review left this bound open. (user)
5. **No cap or paging on the alerts list** — settled by [AD#8] ("the client's alerts, newest first", within 30 days). (fact)
6. **Scope** confirmed as played back, inferred items included: subscribe answers its failure reason; alerts answers the version and configuration operations every service offers; an unpublished book can be subscribed to; capping or paging the alerts list is out of scope. (user)
7. **Stories [U01]–[U07]** confirmed: subscribe; back-in-stock; withdrawal; alerts list/count/read/dismiss; pending list/cancel; reset (demo operator); deployment (demo operator). Role "client" for operations the web store and the journey call on a client's behalf, as in the PRD-level spec. (user)
8. **"Not answering"**: a call from alerts to storage, books or clients that gets no reply within 3 seconds, is refused, or replies with anything other than its found or not-found answer (5xx, 429 included). Subscribe then answers 503 and always answers within 10 seconds; the sweep leaves that book's subscriptions pending until its next pass. Grounds: no RestTemplate timeout exists in the repo (scan). (user)
9. **Subscribe precedence**: an existing pending subscription of the email (ignoring case) to the ISBN answers 200 without calling any service; otherwise clients (404 unknown / 503) → books (404 / 503) → storage (409 copies / 201 none / 503); the first failing check decides and nothing is recorded. (user)
10. **occurredAt** is when the alerts service learned of the change — a notice's arrival or the sweep's observation (up to 30 s late for a lost notice); createdAt is when the alert was recorded; lists order by occurredAt, then createdAt, newest first. Grounds: notices carry no time; storage's updatedAt moves on any later write. (user)
11. **Answerable window**: an ended subscription answers cancel with 204 for 30 days after it ended, and an alert answers dismiss with 204 for 30 days after it was recorded; afterwards the id answers 404 and the record may be removed (purge or filter is design's). Grounds: reserved journey data survives every reset and would otherwise grow without bound. (user)
12. **Missing or blank email or isbn** answers 400 on every operation and changes nothing; no ISBN or email format check beyond that. (user)
13. **Stage 4 criteria confirmed** — 49, inferred ones included: an unpublished book is subscribed to like any other; an alert carries the title stored at subscribe; recording for an ISBN is all or none ([AD#4]'s one transaction); the sweep asks books first, so a removed book is withdrawn whatever storage reports; mark-read marks every alert of the email, one recorded after the list was read included; alerts answers the version and configuration operations. (user)
14. **Test-case conventions confirmed**: tests call alerts' operations directly beside storage, books and clients as they run today, sending the notices and the reset themselves; a lost notice is a stock or catalogue change with no notice; a dependency not answering is stopped, holding requests 5 s, or replying with an error; limits checked at 10 s / 40 s / 10 s; 30-day boundaries at ±1 minute with fixture times; reserved fixtures per [AD#11]; bulk loop off. (user)
15. **Pre-lint** (advisory): one finding — [U04] AC06 had no negative/boundary test; added TC03 (a dismissed alert recorded 30 days less 1 minute ago). Otherwise clean: no placeholders, IDs contiguous, Open questions 0 = `- [ ]` count. 137 test cases. (fix)
16. **Whole-artifact gate** passed: test-case details and `_glossary.md` played back; sent to spec-reviewer. (user)
17. **Review 1 — BLOCK** (6 BLOCKER, 3 MAJOR, 6 MINOR, 4 NIT). (review)
18. **Sweep follows [AD#5] unconditionally** — every pending book is checked every 30 seconds whatever their number; 100 books is the tested size, for restocks ([U02] AC04) and now removals ([U03] AC04); the load of a large pass goes to design. Supersedes decision 4's "above 100 a pass may run longer". (user)
19. **Sweep rule per book: books first** — not-found withdraws the book's subscriptions whatever storage holds; no answer leaves them for the next check without asking storage; found (published or not) then storage: ≥1 copy alerts them back in stock, no row / 0 / no answer leaves them pending. [U02] AC03 applies while books answers that it knows the book; [U02] AC05 and [U03] AC05/AC06 now say "records no … alert". (user)
20. **Reserved-email rule follows [AD#11] literally** — an email ending with `@bis-journey.invalid` exactly as written; client matching elsewhere still ignores letter case. Supersedes decision 2's "delete-all's reserved-email rule" clause. (user)
21. **Removal criterion added** — [U07] AC05: `delete.sh` removes alerts' deployment and service; `delete.sh -all` also removes its data. (user)
22. **Fix-cycle scope**: the 6 BLOCKERs, the withdrawal all-or-none MAJOR ([U03] AC07), the agent MAJOR ([U07] AC06), and the mechanical MINORs and NITs (400 in Scope, 40 seconds in Scope, re-subscribe tests, race-free setups, encoded `+` email, pending list newest createdAt first, "fetched their alerts list", the duplicate [U01] AC03 TC03 dropped). Deferred to the final report: the polled reads' response-time limit (MAJOR), the status every operation answers while alerts' own database is down (MINOR), and whether `occurredAt` = when alerts learned needs an ARD note (MINOR). (user)
23. **Fix cycle applied**; mark-read added to [U01] AC05 with a test; U03 renumbered (AC01–AC08) and U07 (AC01–AC07); 53 criteria, 148 test cases; glossary's sweep and reserved entries updated. Read against overlaps: books-first conditions keep [U02] AC03/AC05 and [U03] AC03/AC05/AC06 to one outcome per book. (fix)
24. **Re-review — PASS WITH RECOMMENDATIONS** (0 BLOCKER, 1 MAJOR, 8 MINOR, 2 NIT); the cap's fix cycle and re-review are spent. (review)
25. **Edit-only recommendations applied after the verdict**, unreviewed, at the user's choice: the lost-notice clocks of [U02] AC03/AC04 and [U03] AC03/AC04 start at the first moment the storage service holds copies, or the books service holds no such book, and answers; [U02] AC05 and [U03] AC05 scoped to the alerts service's own check; 400 first in [U01] AC10's order (+ TC05), and its TC01 made race-free; [U04] AC05/AC06 limited to alerts recorded within the last 30 days; [U04] story "all marked read at once"; [U03] AC07 tests state a quantity of 0; [U03] AC03 TC03 made race-free (storage writes call books, so the book is deleted after its quantity is set, under a 5-second hold); [U07] AC01 TC02 relabelled Happy path, TC03 added; "published or not" moved into Scope's first item. 53 criteria, 150 test cases. (user)
26. **Four product-ruling findings recorded as open questions** (Open questions: 4): the status every operation answers while alerts' own database is down (Scope); an ARD deviation on [AD#8] — occurredAt = when alerts learned, and the 30-day answerable window — flag: architect (Scope); a response-time limit for the polled reads ([U04]); whether alerts' configuration entries change its behaviour and time limits ([U07]). (fix)

## Open questions

- [ ] Status of every operation while the alerts service cannot read or write its own records — Scope.
- [ ] ARD deviation on [AD#8]: occurredAt meaning and the 30-day answerable window — Scope, flag: architect.
- [ ] Response-time limit for the polled reads at a stated number of polling clients — [U04].
- [ ] Effect of alerts' configuration entries on its behaviour and time limits — [U07].

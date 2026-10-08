# /design session — BOOK-1-01

- **Command:** /dev-workflows:design BOOK-1-01
- **Feature folder:** specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01-alerts-service (Epic-level; PRD BOOK-1, Epic BOOK-1-01)
- **Spec gate:** `specification.md` on `origin/main` @ 7892c6c, worktree matches (row A); spec `Published: no`, 4 spec-level open questions inherited
- **Specs preflight:** B2 — PR #6 merged; switched to `main`, pulled, deleted `spec/BOOK-1-01-alerts-service`
- **Run flags:** all default
- **Refresh policy:** fetch + pull default branch
- **Repos search base:** /workspace
- **Architecture grounding:** OFF (ARCHITECTURE_REPO_PATH=/workspace/architecture holds no catalog, radar or ADR folder); team decisions OFF (no knowledge base yet)
- **Model routing:** SIGNIFICANT (provisional at Phase 1.5) — new service with its own schema, a public REST API, inter-service contracts and scheduled fan-out; authoring on claude-opus-5-5 (Opus session — no gate); detection_model claude-sonnet-5-5; review_model claude-opus-5-5
- **ARD:** specifications/PRD-BOOK-1-back-in-stock-alerts/ard.md — status: found; [AD#1]–[AD#11] live; no Epic-level ARD; `## Contracts` present
- **Target:** bookstore:alerts (paths: alerts); ride-along `bookstore:k8s` (kind: deploy)
- **Confirmed repo set:** bookstore @ /workspace/bookstore (origin ihudak/bookstore) — strict mount gate passed

## Stages

- [x] Phase 4 — code scan — bookstore OK @ master af46545 (refreshed, up to date); architecture-grounder not dispatched (grounding OFF)
- [x] Phase 5 — grill (challenge + design) — 14 decisions + 2 confirmation gates; design.md written; spec Engineering review [ER1]–[ER11]; spec OQs 4 → 2
- [x] Phase 5.5 — pre-lint — all core + scaled headings present; design `- [ ]` 0 = header 0; spec header 2 = 2 `- [ ]`; [ER1]–[ER11] consistent; 3 placeholder-shaped tokens (`<subscriptions>`, `<n>`, `<service>`) fixed inline
- [x] Phase 6 — review gate — review 1 PASS WITH RECOMMENDATIONS (2 MAJOR, 6 MINOR, 2 NIT); findings 1–6, 8, 9 applied; re-review (cap spent) PASS WITH RECOMMENDATIONS (1 MAJOR, 8 MINOR, 2 NIT) — deferred to the final report
- [ ] Phase 7 — handoff

## Decisions

1. **Repo set confirmed:** bookstore only (target's repository). (user)
2. **Classification SIGNIFICANT** (not HIGH-RISK): demo store, no auth/money/PII, additive; new schema + public contract + concurrency + scheduled fan-out. (inferred, stated)
3. **Q1 Test DB:** alerts' integration tests run on Testcontainers PostgreSQL (testcontainers postgresql + junit-jupiter as alerts test deps); CI `gradle.yml` runs `./gradlew build` on ubuntu-latest with Docker. PostgreSQL-specific SQL is allowed. (user)
4. **Q2 Schema:** Flyway (`flyway-core` + `flyway-database-postgresql`, alerts only) owns the schema in `V1__alerts_schema.sql`; `spring.jpa.hibernate.ddl-auto=validate`. Departs from the repo's ddl-auto=update convention for alerts only. (user)
5. **Q3 Data model:** `subscriptions` (id identity, email as given, email_key = lower(email), isbn varchar(13), title varchar(255) stored at subscribe, status PENDING|CANCELLED|ALERTED|WITHDRAWN, created_at, ended_at; CHECK pending⇔ended_at null; partial UNIQUE (email_key,isbn) WHERE PENDING; partial INDEX (isbn) WHERE PENDING; INDEX (email_key, created_at)); `alerts` (id identity, subscription_id UNIQUE FK ON DELETE CASCADE, email/email_key/isbn/title copied, type BACK_IN_STOCK|WITHDRAWN, occurred_at, created_at, is_read default false, dismissed_at; INDEX (email_key, created_at)); `configs` (the shared 6-column entity). Timestamps timestamptz ↔ java.time.Instant. (user)

6. **Q4 Notice recorder:** contested (signal: ≥3 callers share the shape — /available, /withdrawn, sweep); user chose the three-take fan-out (A minimise / B flexibility / C common caller). (user)
7. **Q5 Retention:** every list/count/dismiss/cancel query filters on `cutoff = clock.instant() − 30 days` from an injected `java.time.Clock` (exact ±1-minute boundaries); an hourly `@Scheduled` purge deletes subscriptions with `ended_at < cutoff` in batches (alerts cascade) — housekeeping, never correctness. (user)

8. **Q6 DB bootstrap:** a `create-db` initContainer in `k8s/alerts.yaml` (postgres:16-alpine, creds from bookstore-secret) waits for postgres-service and creates `dt_books_alerts` owned by pguser only where `pg_database` lacks it — idempotent on every rollout; plus the `create database dt_books_alerts owner pguser;` line in `database/pg/init/01-create-db.sql` for fresh volumes and the local docker-compose. (user)
9. **Q4 result — notice recorder:** fan-out ran, 3 of 3 takes returned; chose **hybrid** — take A's `int record(String isbn, AlertType type, Instant occurredAt)` + take B's `@Transactional(propagation = REQUIRES_NEW, timeout = 10)` + an injected `Clock` for `recordedAt` (= `alerts.created_at` = `subscriptions.ended_at`); `DataAccessException` mapped to 503 once in an alerts `@RestControllerAdvice`; one ordered-lock data-modifying CTE (`ORDER BY id FOR UPDATE`), `UNIQUE(subscription_id)` backstop. Lost: B (six types, receipt/event/tryRecordAll extension surface no requirement asks for; executor in the port), C (HTTP status inside the service exception; sugar methods add no depth), A as-is (joins an outer transaction; per-controller 503 mapping). (user)

10. **Q7 Store lookups:** contested (signal: spans a network boundary); user chose the fan-out; 3 of 3 takes returned; chose **hybrid A+C** — one `StoreLookups` port (`Lookup<Void> client(email)`, `Lookup<String> book(isbn)` → title, `Lookup<Integer> copies(isbn)`), sealed `Lookup<T>` = `Found(value)` | `NotFound` | `NoAnswer(reason)` (reason for logs only); `HttpStoreLookups` on `RestClient` + `JdkClientHttpRequestFactory` (connect 1 s, request timeout 3 s — the JDK request timer covers connect, so ≤ 3 s per call, ≤ ~9 s for subscribe's three), values as URI-template variables (`+` → `%2B`), `.exchange()` classification (200+body → Found; storage empty 200 → Found(0); clients/books empty 200 → NoAnswer; 404 → NotFound; anything else/exception → NoAnswer); storage `NotFound` → 404 in subscribe, skip in the sweep; `InMemoryStoreLookups` fake for subscribe/sweep tests, WireMock for the adapter. Lost: B (generic RemoteLookup/Deadline/Cause/Endpoint/decorators — speculative surface; raw HttpClient; storage 404 → 503), A as-is (no reason for logs; 3 s connect), C as-is (ClientRef/BookRef types no caller needs). (user)
11. **Q8 Sweep cadence:** `@EnableScheduling` (first in repo); `@Scheduled(fixedDelayString = "${alerts.sweep.delay:20s}")`; a pass reads the distinct pending ISBNs at its start, runs each on a dedicated `sweepExecutor` (`alerts.sweep.workers:8`, unbounded queue sized by the pass), waits for all, logs ISBN count / alerted / withdrawn / no-answer / duration; scheduler pool size 2 (sweep + purge), no overlap; worst case D + 2P + record ≈ 23 s for 100 ISBNs (≤ 30 s period, ≤ 40 s criteria); replicas: 1 (a second replica double-checks harmlessly — recorder is at-most-once). (user)

12. **Q9 Own DB unavailable (spec OQ 1):** every operation answers 503 Service Unavailable, changing nothing, when its database work fails — `DataAccessException`, `CannotCreateTransactionException`, `TransactionTimedOutException` mapped once in an alerts `@RestControllerAdvice`; alerts' `spring.datasource.hikari.connection-timeout=3000` (repo convention 30 s) so the 503 is prompt; a notice answered 503 is a lost notice, recovered by the sweep once the DB is back. Spec OQ to be closed with this answer in `## Engineering review`. (user)
13. **Q10 Config hooks (spec [U07] OQ):** `runThreatScan()` + `applySecurityPolicy()` are called first by the client-facing operations only — subscribe, pending-subscriptions read, alerts read, unread count, mark-read, cancel, dismiss; never by `/available`, `/withdrawn`, `/delete-all`, the sweep or the purge. So [U02] AC02–AC04 and [U03] AC02–AC04 hold whatever is configured; [U01] AC09's 10 s holds while no entry is turned on. Spec OQ to be closed with this answer. (user)

14. **Q11 Read latency (spec [U04] OQ):** design target p95 ≤ 200 ms for the unread count, the alerts read and the pending-subscriptions read at 50 req/s sustained (≈ 1,000 open pages polling every 20 s, [AD#9]), no config entry turned on; proposed in the spec's `## Engineering review` as a new [U04] criterion for the PM to adopt; the spec OQ stays open until adopted; verified post-release in Dynatrace. (user)
15. **Q12 [AD#8] deviation (spec OQ 2):** design implements the spec (occurredAt = when alerts learned; cancel/dismiss 404 past the 30-day window) and records `- ARD deviation: [AD#8] … — flag: architect` under `## ARD deviations` in design.md; the spec's open question stays open for the architect (`/product-workflows:create-ard BOOK-1` refine); no design.md `- [ ]`. (user)

16. **Q13 HTTP stub:** WireMock (`org.wiremock:wiremock-standalone` 3.x, alerts test dependency, version pinned at implementation) proves `HttpStoreLookups` — 5-s hold → NoAnswer within 3 s, refused connection, 500, 429, 404, storage empty 200 → Found(0), books empty 200 → NoAnswer, `+` arriving as `%2B`. (user)
17. **Q14 Signals:** one key=value line per business event on a dedicated `com.dynatrace.alerts.events` logger pinned to INFO in every profile — `notice type= isbn= ended= ms=`, `sweep isbns= alerted= withdrawn= noAnswer= failed= ms=`, `purge deleted=`, `reset deleted=`; read in Dynatrace logs; no Micrometer. (user)

18. **Gate 1 confirmed** — Architecture & components, Interfaces / contracts, Seams, Data flow, Error handling as played back (inferred: component/package layout; version message `Count: <subscriptions>`; subscribe validation before hooks; concurrent-insert re-read → 200, one retry, then 503; cancel/dismiss/mark-read/read SQL; reset predicate case-sensitive with cascade; purge batches of 1,000; subscribe-vs-notice race closed by the next sweep pass; Hikari pool 20; JSON Instants ISO-8601 UTC; default Spring error body). (user)
19. **Gate 2 confirmed** — Requirements coverage, Test strategy, Risks, Migration/rollout, Observability & release verification, ARD deviations, Out of scope, spec Engineering review as played back (inferred: AbstractAlertsIT harness on postgres:16-alpine re-pinned to the cluster major; DatabaseDownIT via container pause; in-statement failure via pre-inserted conflicting alert; SweepScheduleIT at 1 s; deployment criteria as scripted post-deploy checks; risks list; rollout via restart.sh -nodb + initContainer; rollback = delete alerts.yaml; baselines read in Dynatrace stage over 7 days before deploy; rollback signal books /find p95 > 2× baseline or failure +2 pp, or alerts 5xx > 5 %, 15 min, no config entry on; out-of-scope additions: ingest's config fan-out/version polling, other services' RestTemplates, Version.setVerDocker, Micrometer). (user)
20. **Spec edits:** `## Engineering review` [ER1]–[ER11] appended; Scope OQ 1 and [U07] OQ ticked with [ER1] / [ER2]; header Open questions 4 → 2 ([AD#8] deviation — architect; read latency — PM, proposed [U04] AC09 in [ER3]); no new spec `- [ ]`. (fix)

21. **Phase 6 review 1 — PASS WITH RECOMMENDATIONS** (0 BLOCKER, 2 MAJOR, 6 MINOR, 2 NIT). (review)
22. **Findings 1–6, 8, 9 applied at the user's choice** (the two [AD#8] MINORs stay with the architect): sweep re-paced — `@Scheduled(fixedRate = alerts.sweep.period 20s)`, W = clamp(⌈N/10⌉, 8, 32) lanes on a 32-thread `sweepExecutor`, circuit stops after 8 consecutive `timedOut` answers from storage (stop asking storage; books' not-found still withdraws) or books (skip the rest); `Lookup.NoAnswer` gains `boolean timedOut` — supersedes decision 11's fixedDelay / 8 workers and amends decision 10; spec [ER12] (proposed change: the check's cadence holds to ≈ 300 pending books) + a Scope open question (spec Open questions 2 → 3); `ConfigHooksIT`, `VersionConfigIT`, `DatabaseDownIT` widened (reset, cancel, dismiss, mark-read; waits 1 s after pause; 503 within 7 s); `socketTimeout=5` on the JDBC URL, [ER1] reworded to "within about 5 seconds"; rollout adds `kubectl rollout restart deployment -n bookstore`; the in-pod check wrapped in `sh -c`; alerts.yaml env `SERVICE_NAME: Alerts`, `DT_SERVICE_NAME: AlertController`, `DT_TAGS: "BookStore MicroService=BookStore.Alerts"`, selectors by `tag("MicroService=BookStore.Alerts")`; ports 91/30010, debug 5010/32010; dt-postgres rebuild note; version check serviceId `alerts`; coverage labels aligned ([U04] AC03 proposed-change; [U02] AC08 / [U03] AC07 validated; [U02]/[U03] AC03–AC04 questioned [ER12]). Read against overlaps: ER11/ER6/ARD-deviation "≈ 23 s" still hold under the fixed rate; spec glossary's sweep entry covered by the new Scope OQ. (user)

23. **Phase 6 re-review — PASS WITH RECOMMENDATIONS** (0 BLOCKER, 1 MAJOR, 8 MINOR, 2 NIT); review cap spent; findings deferred to the final report. MAJOR: the namespace-wide `kubectl rollout restart deployment -n bookstore` added by decision 22 also restarts postgres, mysql and ingest (whose populate-configs sidecar calls delete-all on six services) — restart `deployment/storage` only. (review)

## Interface candidates

_(live candidate shapes recorded as they arise; struck when eliminated)_

### Notice recorder (the fan-out: /available, /withdrawn, the sweep — 3 callers)

- **R-A** `int endAll(String isbn, AlertType type, Instant occurredAt)` — one method, type as parameter; one data-modifying CTE (UPDATE … RETURNING → INSERT) per call. → **settled** (as `record`, hybrid, decision 9)
- ~~**R-B** two methods `int alertBackInStock(String isbn, Instant occurredAt)` / `int withdraw(String isbn, Instant occurredAt)` over one private statement.~~ — eliminated (no depth over R-A; take C's form)
- ~~**R-C** a batch form `Map<String,Integer> endAll(Map<String, AlertType> byIsbn, Instant occurredAt)` so one sweep pass records many ISBNs in one call.~~ — eliminated (couples one ISBN's failure to another's; spec's all-or-none unit is per ISBN)
- ~~**Take B** `NoticeReceipt record(Notice)` + `tryRecord`/`tryRecordAll(Collection<Notice>, Executor)` + after-commit `NoticeRecorded` event~~ — eliminated (decision 9)

### Store lookups (clients / books / storage — network boundary)

- ~~**L-A** three ports in the repo's one-`*Repository`-per-remote-service convention, each throwing `StoreUnavailableException` where the service does not answer.~~ — eliminated (an exception for an expected outcome; a caller can forget the third case)
- **L-B** a sealed `Lookup<T>` = `Found(T)` | `NotFound` | `Unavailable(reason)` — no exceptions for a remote outcome. → **settled** (one port, three methods; decision 10)
- ~~**L-C** use-shaped ports hiding the call order — `SubscribeChecks.check(email, isbn)`, `SweepProbe.probe(isbn)`.~~ — eliminated (buries the spec's fixed check order inside the adapter)
- ~~**Take B (lookups)** generic `RemoteLookup<K,V>` + `Deadline` + `Cause` + `Endpoint` + decorators~~ — eliminated (decision 10)

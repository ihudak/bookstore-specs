---
kind: design
key: BOOK-1-01
---

# Design

- **Feature name**: Back-in-stock alerts — subscriptions and alerts service
- **Spec**: specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01-alerts-service/specification.md (BOOK-1-01)
- **Classification**: SIGNIFICANT
- **Version**: 1
- **Created**: 2026-10-08
- **Author**: ivan.gudak
- **Repos**: bookstore (/workspace/bookstore, origin ihudak/bookstore) @ master af46545
- **Target**: bookstore:alerts
- **Open questions**: 0

## Context & problem

A client who finds a book with no available copies has nowhere to leave their interest, so the store cannot tell them when it returns or is removed ([specification.md § Problem statement](specification.md#problem-statement)). This Epic delivers the new `alerts` service that every other back-in-stock Epic builds on: it records one pending subscription per client and book, turns a restock or removal — told by a notice or found by its own sweep — into one alert per waiting client within the spec's limits, serves the client-facing reads and actions, clears non-reserved data on a catalogue reset, and runs in the store's Kubernetes deployment, an existing one included. It lands first in the ARD's landing order, and until [[BOOK-1-02]] and [[BOOK-1-03]] land its sweep alone delivers restock and removal alerts.

## Requirements coverage

Challenge notes cite the spec's `## Engineering review` entries ([ER1]–[ER12]). Every in-scope Scope item is covered by the rows below; none is deferred.

| Requirement | How this design addresses it | Challenge |
|---|---|---|
| [U01] AC01 Record the subscription | `SubscriptionService`: clients → books (title kept) → storage `Found(0)` → insert `PENDING` → `201` + Subscription | validated |
| [U01] AC02 One pending per client and book | Pending lookup on (`email_key`, isbn) first → `200`, no remote call; partial unique index + re-read turns a concurrent insert into `200` (§ Data flow) | validated |
| [U01] AC03 Never lapse | No status change on time; purge touches only ended rows; the sweep alerts only on ≥ 1 copy or books' not-found | validated |
| [U01] AC04 Accept an unpublished book | `book(isbn)` returns `Found(title)` published or not; `published` is never read | validated |
| [U01] AC05 Email ignoring letter case | `email_key` on every lookup, uniqueness and ownership check | validated |
| [U01] AC06 Keep a plus sign | Stored as given; sent to clients as a URI-template variable (`%2B`) | validated [ER9] |
| [U01] AC07 Refuse a book with copies | `copies(isbn)` `Found(n ≥ 1)` → `409`, nothing recorded | questioned [ER6] |
| [U01] AC08 Refuse an unknown book or client | `NotFound` from clients or books → `404`; storage `404` after books found the book → `404` | validated [ER5] |
| [U01] AC09 Refuse when the store cannot check | `NoAnswer` → `503`; each call ≤ 3 s (1-s connect, 3-s request timeout), ≤ ≈ 9 s for three | validated [ER8] |
| [U01] AC10 Fixed order | 400 → pending → clients → books → storage in `SubscriptionService`; the hooks run after validation | validated |
| [U01] AC11 Missing or blank email/ISBN | `hasText` check → `BadRequestException` (`400`) on every operation, before any change | validated |
| [U02] AC01 Alert every subscriber on a notice | `/available` → `NoticeRecorder.record(isbn, BACK_IN_STOCK, now)` → `204` | validated |
| [U02] AC02 1,000 within 10 s | One set-based statement on the partial `(isbn)` index; 10-s transaction timeout; tested at 1,000 + 1,000 | validated |
| [U02] AC03 Recover a lost notice | Sweep: books found → storage ≥ 1 → record; passes start every 20 s (at once after one that overran) in a stable ISBN order, so an ISBN is re-checked within ≈ max(20 s, P) + spread ≈ 21–23 s, plus < 1 s recording — inside 40 s while P ≤ ≈ 20 s | questioned [ER12] |
| [U02] AC04 Recover 100 books at once | One pass covers every pending ISBN in W = clamp(⌈N/10⌉, 8, 32) lanes — 10 lanes for 100, ≈ 1 s at normal latency, ≈ 20 s with books answering in ≈ 1 s | questioned [ER12] |
| [U02] AC05 Nothing while there are no copies | `Found(0)`, storage `404` and `NoAnswer` all skip | validated |
| [U02] AC06 Alert each subscription once | Row locks + re-check of `PENDING` + `UNIQUE(subscription_id)` | validated |
| [U02] AC07 Never alert an ended subscription | The statement touches `PENDING` rows only | validated |
| [U02] AC08 All or none | One statement in one `REQUIRES_NEW` transaction; failure → rollback, `503` | validated [ER7], [ER10] |
| [U02] AC09 Fill the back-in-stock alert | Alert row copies isbn and stored title; `occurred_at` = arrival or observation; `created_at` = `recordedAt`; `is_read` false | validated [ER11]; ARD deviation |
| [U02] AC10 Synthetic clients by the same rules | No reserved-data branch anywhere but the reset | validated |
| [U03] AC01 Withdraw on a notice | `/withdrawn` → `record(isbn, WITHDRAWN, now)` → `204`; a repeat ends nothing | validated |
| [U03] AC02 1,000 within 10 s | As [U02] AC02 | validated |
| [U03] AC03 Recover a lost removal | Sweep asks books first; `NotFound` withdraws whatever storage holds — even while storage holds requests, since storage is never asked for a book books does not know; cadence as [U02] AC03 | questioned [ER12] |
| [U03] AC04 Recover 100 removed books | As [U02] AC04 | questioned [ER12] |
| [U03] AC05 Nothing for an unpublished book | `Found(title)` for an unpublished book is never a withdrawal | validated |
| [U03] AC06 Nothing while the catalogue cannot answer | Books `NoAnswer` → skip, storage not asked | validated |
| [U03] AC07 All or none | As [U02] AC08 | validated [ER7], [ER10] |
| [U03] AC08 Fill the withdrawal alert | As [U02] AC09 with `WITHDRAWN`; title stored at subscribe | validated; ARD deviation |
| [U04] AC01 List newest first | `ORDER BY occurred_at DESC, created_at DESC` on `email_key` | validated |
| [U04] AC02 Leave out dismissed and old | `dismissed_at IS NULL AND created_at >= cutoff` on every list and count | validated |
| [U04] AC03 Count unread | Same predicate `AND NOT is_read` → `{"count": n}`; read latency target p95 ≤ 200 ms at 50 req/s | proposed-change [ER3] |
| [U04] AC04 Mark every alert read | `UPDATE … SET is_read = true WHERE email_key = ?` → `204`, idempotent | validated |
| [U04] AC05 Dismiss | `dismissed_at` set where owned and `created_at >= cutoff` → `204` | validated |
| [U04] AC06 Dismiss a dismissed alert | `COALESCE` keeps the first `dismissed_at`; `204` | validated |
| [U04] AC07 Refuse an alert not the email's | 0 rows updated → `404` (other email, unknown id, or before the cutoff) | validated; ARD deviation |
| [U04] AC08 Empty for an email with no alert | Empty list and `{"count": 0}` | validated |
| [U05] AC01 List pending subscriptions | `status = 'PENDING' ORDER BY created_at DESC`; `email` as given | validated |
| [U05] AC02 Cancel | `PENDING` → `CANCELLED`, `ended_at = now` → `204` | validated |
| [U05] AC03 Cancel an ended one | Owned and `ended_at >= cutoff` → `204`, no change | validated |
| [U05] AC04 Refuse one not the email's | Otherwise `404` | validated; ARD deviation |
| [U05] AC05 Drop ended subscriptions | Pending list reads `PENDING` only | validated |
| [U05] AC06 Empty for an email with none | Empty list | validated |
| [U06] AC01 Remove what is not reserved | One `DELETE … WHERE isbn NOT LIKE '00000000%' AND email NOT LIKE '%@bis-journey.invalid'`, cascade | validated |
| [U06] AC02 Keep what is reserved | The same predicate, case-sensitive, either side reserved keeps the row | validated |
| [U06] AC03 No alert on a reset | The reset never calls the recorder; deleted rows are no longer `PENDING` for the sweep | validated |
| [U07] AC01 The store's addresses | `/api/alerts/?(.*)` ingress path; `DT_ALERTS_SERVER: alerts-svc:91` | validated |
| [U07] AC02 Existing routes unchanged | New path inserted before the web catch-all; no other line changes | validated |
| [U07] AC03 Existing deployment | `create-db` initContainer; Flyway V1 on the new database only | validated |
| [U07] AC04 Data across a restart | PostgreSQL on `postgres-pvc`; nothing held in memory | validated |
| [U07] AC05 Removed with the store | `alerts.yaml` in `delete.sh` / `delete.bat`; `-all` deletes `databases.yaml`'s PVC | validated |
| [U07] AC06 The agent the store selects | `alerts_agent` key, `AGENT` env from `bookstore-agents-configmap`, `docker.agent.vendor=${AGENT:NONE}` in the version | validated |
| [U07] AC07 Version and configuration | Copies of storage's `VersionController` / `ConfigController`; `configs` created empty by Flyway | validated [ER2] |
| Scope — own records unreadable (spec open question) | `503` on every operation, nothing changed; a notice so answered is recovered by the sweep | resolved [ER1] |
| Scope — [AD#8] deviation (spec open question) | Implemented as the spec states; recorded under § ARD deviations | left to the architect [ER4] |
| Scope — the check "whatever the number of books waited on" (spec open question) | Fixed-rate passes, lanes growing with the pending count, circuit stops; the cadence holds to ≈ 320 pending ISBNs at ≈ 1-s answers | proposed-change [ER12] |

## Architecture & components

All code is new and lives in the target's path, `alerts/` (Gradle module `alerts`, package `com.dynatrace.alerts`), modelled on `storage/` and `orders/`, the repository's PostgreSQL-backed services. The deploy files are `bookstore:k8s` ride-alongs; `settings.gradle`, the root `build.gradle`, `database/pg/init/01-create-db.sql` and `CLAUDE.md` are the repository's shared ground (`workflows-core:components` §2). No other module changes.

```
alerts/
  build.gradle, Dockerfile, push_docker.sh                       (copies of storage/'s; Dockerfile SERVICE_FULL_NAME=BookStore-Alerts, which k8s/alerts.yaml overrides as every manifest does)
  src/main/resources/
    application.properties, application-{dev,test,stage,prod}.properties, banner-*.txt
    db/migration/V1__alerts_schema.sql                           (Flyway — the only schema source)
  src/main/java/com/dynatrace/alerts/
    AlertsApplication            @SpringBootApplication @EnableScheduling
    config/  WebConfig (CORS copy), ClockConfig (Clock.systemUTC()), LookupConfig (RestClient),
             SweepConfig (sweepExecutor: 32 threads; a pass uses W = clamp(⌈N/10⌉, 8, 32) lanes)
    controller/
      SubscriptionController     /api/v1/alerts/subscriptions — subscribe, pending list, cancel
      AlertController            /api/v1/alerts — list, unread-count, read, dismiss; /available, /withdrawn, /delete-all
      ConfigController, VersionController                        (copies of storage/'s; version message "Count: " + subscriptionRepository.count())
      ApiExceptionHandler        @RestControllerAdvice — 400 / 503 mapping (§ Error handling)
    service/
      SubscriptionService        subscribe flow (lookups in the fixed order), cancel
      AlertQueryService          alerts list, unread count, mark-read, dismiss, pending list
      NoticeRecorder             int record(isbn, AlertType, occurredAt) — the one fan-out
      Sweep                      @Scheduled(fixedRate) pass over pending ISBNs in W lanes, with timeout circuit stops
      Purge                      @Scheduled hourly delete of subscriptions ended > 30 days ago
      ResetService               delete-all keeping [AD#11]'s reserved rows
    lookup/
      StoreLookups, Lookup<T>    the port to clients / books / storage
      HttpStoreLookups           RestClient adapter
    model/      Subscription, Alert, Config (JPA entities); AlertType; SubscriptionStatus; SubscriptionView, AlertView, Count (JSON)
    repository/ SubscriptionRepository, AlertRepository, ConfigRepository (Spring Data JPA);
                NoticeSql, MaintenanceSql (JdbcClient native statements: the fan-out CTE, purge, reset)
  src/test/java/…                                                 (§ Test strategy)
```

Responsibilities, deepest first:

- **`NoticeRecorder`** — one method, `int record(String isbn, AlertType type, Instant occurredAt)`, hiding the locking, the status mapping, the alert row and atomicity behind it (§ Interfaces). It is a concrete `@Service`, not an interface: its only adapter is PostgreSQL, which the tests run for real, so a port would be a one-adapter seam.
- **`StoreLookups`** — the one port to the three remote services, returning a sealed three-outcome `Lookup<T>`; the HTTP adapter owns URL encoding, timeouts and status classification so neither caller ever catches a remote exception.
- **`SubscriptionService`** — [U01]'s fixed order, the 200 short-cut, the 201 insert and the concurrent-insert recovery.
- **`Sweep`** — the books-first decision per ISBN; it calls `NoticeRecorder` exactly as the notice endpoints do. Its pace does not depend on how long a pass takes: passes start at a fixed rate, the lanes grow with the number of pending ISBNs, and consecutive timeouts from one service stop the pass from waiting on it (§ Data flow).
- **Controllers** — validation, the configuration hooks for client-facing operations only, and status codes; no business logic.
- **`ApiExceptionHandler`** — the repository's first `@RestControllerAdvice`, the single place where a database failure becomes 503.

Shared-ground and ride-along edits:

- `settings.gradle` — `include 'alerts'`; root `build.gradle` — a `project("alerts")` block: `spring-boot-starter-data-jpa`, `runtimeOnly 'org.postgresql:postgresql'`, `implementation 'org.flywaydb:flyway-core'` and `runtimeOnly 'org.flywaydb:flyway-database-postgresql'` (versions from the Spring Boot BOM), `:common`, `:exceptions`; tests: `spring-boot-testcontainers`, `org.testcontainers:postgresql`, `org.testcontainers:junit-jupiter` (BOM-managed) and `org.wiremock:wiremock-standalone` 3.x (not BOM-managed; pinned at implementation).
- `database/pg/init/01-create-db.sql` — `create database dt_books_alerts owner pguser;`. It reaches a cluster only once `ghcr.io/ihudak/dt-postgres` is rebuilt, since the script is baked into that image (`database/pg/Dockerfile:2`); the initContainer creates the database meanwhile, and on every existing volume.
- `bookstore:k8s` — new `k8s/alerts.yaml`; `/api/alerts/?(.*)` in `k8s/ingress.yaml` before the web catch-all; `DT_ALERTS_SERVER: "alerts-svc:91"` in `k8s/configmap.yaml`; `alerts_agent: oneAgent` in `k8s/config_agents.yaml`; `alerts.yaml` in `restart.sh`, `restart.bat`, `delete.sh`, `delete.bat`, `redeploy.cmd`; `alerts` in `build_docker_all.sh`'s `dt_projects` and `build_docker_all.bat`'s indexed list (web moves to index 10). `preset_deployment.sh` rewrites `*.yaml` by glob and needs no edit. The Windows twins (`restart.bat`, `delete.bat`, `redeploy.cmd`, `build_docker_all.bat`) sit in `k8s/`, inside the `bookstore:k8s` component the Epic's `Also touches:` line names.
- `k8s/alerts.yaml` — a copy of `k8s/storage.yaml` with: Deployment `alerts`, image `ghcr.io/ihudak/alerts-{AGENT}:{FLAVOR}` (rewritten by `preset_deployment.sh`); env `DT_PG_DBNAME: dt_books_alerts`, `DT_MYSQL_DBNAME: none`, `AGENT` from `alerts_agent`, `SERVICE_NAME: Alerts`, `SERVICE_FULL_NAME: "$(APP_NAME).$(SERVICE_NAME).$(SVC_SUFFIX)"`, `DT_TAGS: "BookStore MicroService=BookStore.Alerts"`, `DT_CUSTOM_PROP: "custom.app.name=BookStore,custom.microservice.name=alerts"`, `DT_SERVICE_NAME: AlertController`, `DT_HOST_NAME_PREFIX: alerts`; storage's probes and 400m CPU / 768Mi; the `create-db` initContainer (§ Migration); Service `alerts-svc`, type LoadBalancer, port `91` → `8080` (nodePort `30010`) and debug `5010` → `5005` (nodePort `32010`) — the next free numbers after web's `30000` and ingest's `30009` / `32009` / `5009`.
- `CLAUDE.md` — alerts in the module list, the PostgreSQL row of the database table and the port list (alerts=91).

### Alternatives considered

- **Notice recorder — take B (maximise flexibility)**, `NoticeReceipt record(Notice)` plus `tryRecord` / `tryRecordAll(Collection<Notice>, Executor)`, an after-commit `NoticeRecorded` event and an HTTP-free `NoticeStoreException`: six types and an executor in the port for extensions (email, push, replay) no requirement asks for; lost on depth.
- **Notice recorder — take C (optimise for the common caller)**, `recordAvailable(isbn)` / `recordWithdrawn(isbn)` sugar plus `record(...)`, failing with a `ResponseStatusException` (503): the sugar adds no depth, and an HTTP status inside the service makes a sweep failure an HTTP exception; lost on locality.
- **Notice recorder — take A as proposed** (joining any caller's transaction, `Instant.now()`, each controller mapping the failure): kept as the base of the chosen hybrid, with take B's `REQUIRES_NEW`, an injected `Clock`, and the 503 mapping moved into one advice.
- **Store lookups — take B (maximise flexibility)**, a generic `RemoteLookup<K,V>.find(key, Deadline)` with a `Cause` taxonomy, `Endpoint` descriptors and a decorator seam on a raw JDK `HttpClient`: speculative surface for two callers; lost on depth. Its observation that a JDK request timeout bounds only time to headers is accepted as a residual (bodies are a few hundred bytes).
- **Store lookups — take A / take C as proposed**: the chosen design is their hybrid — take A's three-method port, take C's `NoAnswer(reason)` and `.exchange()` classification — without take C's `ClientRef` / `BookRef` types, which no caller reads. After review, `NoAnswer` also carries `timedOut`, which the sweep's circuit stop reads.
- **Sweep paced by `fixedDelay` on a fixed 8 workers** (the first version of this design): the gap between passes then grows by the whole pass, so a storage service holding requests (3 s per ISBN) or a books service answering in ≈ 1 s stretches the 30-s cadence past 40 s at a few dozen ISBNs; replaced by a fixed rate, per-pass lanes and circuit stops after review.
- **Store lookups — exceptions per service (the repository's convention)**: an expected outcome thrown as an exception lets a caller forget the third case; lost to the sealed result.
- **Schema by `ddl-auto=update` (the repository's convention)**: cannot create the partial unique index that holds one pending subscription per client and book; Flyway with `validate` chosen.
- **H2 for alerts' tests (the repository's convention)**: cannot prove row-lock re-evaluation, the partial index or the data-modifying CTE that the concurrency and atomicity criteria rest on; Testcontainers PostgreSQL chosen.
- **Creating `dt_books_alerts` from the application at startup, or by a documented manual step**: early-startup bootstrap code in the service, or an existing deployment that fails [U07] AC03 until someone runs a command; an idempotent initContainer chosen.

## Interfaces / contracts

### Client-facing and notice API (boundary — produced)

Every path is under `/api/v1/alerts`; the browser reaches it as `/api/alerts/api/v1/alerts/…` through the ingress (`rewrite-target: /$1`, `k8s/ingress.yaml:9`), another service at `http://$DT_ALERTS_SERVER/api/v1/alerts/…`. `{id}` is `{id:\d+}`, so `/delete-all` never matches it.

| Operation | Request | Success | Other answers | Hooks |
|---|---|---|---|---|
| Subscribe | `POST /subscriptions?email=&isbn=` | `201` + Subscription (new); `200` + Subscription (an existing pending one) | `400` email or isbn missing/blank; `404` clients or books knows no such email/ISBN, or storage answers 404; `409` storage holds ≥ 1 copy; `503` a service does not answer, or alerts' database fails | yes |
| Pending subscriptions | `GET /subscriptions?email=` | `200` + Subscription[] newest `createdAt` first | `400`; `503` | yes |
| Cancel | `DELETE /subscriptions/{id}?email=` | `204` | `400`; `404` not the email's, or ended before the cutoff; `503` | yes |
| Alerts | `GET ?email=` | `200` + Alert[] newest `occurredAt`, then `createdAt`, first | `400`; `503` | yes |
| Unread count | `GET /unread-count?email=` | `200` + `{"count": 2}` (the number of listed unread alerts) | `400`; `503` | yes |
| Mark read | `POST /read?email=` | `204` | `400`; `503` | yes |
| Dismiss | `DELETE /{id}?email=` | `204` | `400`; `404` not the email's, or recorded before the cutoff; `503` | yes |
| Available notice | `POST /available?isbn=` | `204` | `400`; `503` | no |
| Withdrawn notice | `POST /withdrawn?isbn=` | `204` | `400`; `503` | no |
| Reset | `DELETE /delete-all` | `204` | `503` | no |
| Version, config | `GET /api/v1/version`; `/api/v1/config` CRUD | as every BookStore service (`storage/…/VersionController.java`, `ConfigController.java`) | — | as storage's |

Bodies (JSON; `java.time.Instant` as ISO-8601 UTC, Spring Boot's default):

```json
Subscription: {"id": 17, "email": "First.Last+Books@Example.com", "isbn": "9783161484100", "title": "The Long Wait", "createdAt": "2026-10-08T09:00:00Z"}
Alert:        {"id": 42, "type": "BACK_IN_STOCK", "isbn": "9783161484100", "title": "The Long Wait", "occurredAt": "2026-10-09T07:15:02Z", "createdAt": "2026-10-09T07:15:02.31Z", "read": false}
```

`email` is returned exactly as given at subscribe; `type` is `BACK_IN_STOCK` or `WITHDRAWN`. Error bodies are Spring's default `{timestamp, status, error, path}`, as every BookStore service answers today.

**Behaviour a schema does not carry:**

- **Matching.** Every operation matches the client on `email_key = email.toLowerCase(Locale.ROOT)`; [AD#11]'s reserved-email test reads the stored `email` as written. Parameters are taken as given — no trimming, no format check beyond blank ([U01] AC11).
- **Subscribe.** Validation, then the hooks, then: an existing pending subscription for (`email_key`, isbn) answers `200` with it and calls no service; otherwise clients, books, storage in that order, the first failing check deciding, nothing recorded on any answer but `201`. Side effect: one `subscriptions` row on `201` only. A repeat while pending answers `200` with the same subscription; two concurrent calls answer one `201` and one `200` with the same subscription (§ Data flow). Worst case ≤ 3 s per remote call, so ≤ ≈ 9 s for the three.
- **Cancel, dismiss, mark-read.** Idempotent: a repeat answers `204` and changes nothing; cancel of a subscription that already ended, and dismiss of an alert already dismissed, answer `204` while inside the 30-day window ([U05] AC03, [U04] AC06). Mark-read marks every alert of the email read, listed or not.
- **Available / withdrawn notices** ([AD#4], [AD#6]). Side effect: every pending subscription to the ISBN ends (`ALERTED` / `WITHDRAWN`) with one alert each, in one transaction, all or none, within the 10-second transaction timeout. `204` whether or not any existed. A repeat, or a concurrent duplicate, records nothing new (at most one alert per subscription). A database failure answers `503` and changes nothing; the caller never retries ([AD#4], [AD#6]), so the notice counts as lost and the sweep recovers it. No ordering across ISBNs.
- **Reset** ([AD#7], [AD#11]). Side effect: deletes every subscription whose `isbn` does not start with `00000000` and whose `email` does not end with `@bis-journey.invalid` (both case-sensitive), its alerts with it; records no alert. A repeat deletes nothing new and answers `204`. A failure answers `503` and deletes nothing (one statement).
- **Timeouts.** Alerts' own requests answer within the remote-call bound above. Its database work fails within 3 s when no connection can be had (`hikari.connection-timeout=3000`) and within 5 s when a statement's database goes silent (`socketTimeout=5` on the JDBC URL), so every operation answers `503` within about 5 s of the database failing ([ER1]); the recorder's transaction is also bounded at 10 s.

### Internal interfaces

```java
// service — the fan-out; three callers: /available, /withdrawn, Sweep
@Service public class NoticeRecorder {
    /** Ends every PENDING subscription to isbn with type at occurredAt and records one alert each.
     *  Returns how many ended (0 is success). Throws DataAccessException on any database failure;
     *  the transaction has rolled back, so every subscription is still PENDING and no alert exists. */
    @Transactional(propagation = Propagation.REQUIRES_NEW, timeout = 10)
    public int record(String isbn, AlertType type, Instant occurredAt);  // recordedAt = clock.instant()
}

// lookup — the port to the three remote services; never throws for a remote outcome
public interface StoreLookups {
    Lookup<Void>    client(String email);  // clients GET /find?email=
    Lookup<String>  book(String isbn);     // books   GET /find?isbn=        → Found(title)
    Lookup<Integer> copies(String isbn);   // storage GET /findByISBN?isbn=  → Found(quantity), Found(0) for an empty 200
}
public sealed interface Lookup<T> {
    record Found<T>(T value)          implements Lookup<T> {}
    record NotFound<T>()              implements Lookup<T> {}
    record NoAnswer<T>(String reason, boolean timedOut) implements Lookup<T> {}   // reason: the log line only; timedOut: the call hit its connect or request timeout
}
```

The recorder's statement, run through `JdbcClient` (`ended_at` and `created_at` are the same `recordedAt`, so the CHECK holds and the 30-day windows of an alert and the subscription it ended coincide):

```sql
WITH ended AS (
  UPDATE subscriptions SET status = :endStatus, ended_at = :recordedAt
   WHERE id IN (SELECT id FROM subscriptions
                 WHERE isbn = :isbn AND status = 'PENDING'
                 ORDER BY id FOR UPDATE)
     AND status = 'PENDING'
  RETURNING id, email, email_key, isbn, title)
INSERT INTO alerts (subscription_id, email, email_key, isbn, title, type, occurred_at, created_at, is_read)
SELECT id, email, email_key, isbn, title, :type, :occurredAt, :recordedAt, false FROM ended
```

`HttpStoreLookups` classification (one private function over `RestClient.exchange(...)`, so no status handler throws): a `200` with a parseable body → `Found`; storage's empty `200` → `Found(0)`; an empty or unparseable `200` from clients or books, or a found book with a null title → `NoAnswer`; `404` → `NotFound`; any other status (`429`, `5xx`, `3xx`, other `4xx`) and any exception (timeout, refused connection, I/O, JSON, interrupt — the interrupt flag restored) → `NoAnswer`. `timedOut` is true only where the 1-s connect or the 3-s request timeout fired; a refused connection, a status and a parse failure answer at once and carry `false`. A storage quantity below 1 is out of stock. Values travel as URI-template variables (`"/find?email={email}"`), which `RestClient`'s default `TEMPLATE_AND_VALUES` encoding sends `+` as `%2B`; the repository's string concatenation (`orders/…/ClientRepository.java:25`) is not copied. Transport: `JdkClientHttpRequestFactory` over a `HttpClient` with a 1-second connect timeout and a 3-second request timeout, which the JDK starts before connecting, so each call is bounded at 3 s.

### Schema — `V1__alerts_schema.sql`

```sql
CREATE TABLE subscriptions (
  id         BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  email      VARCHAR(255) NOT NULL,           -- as given at subscribe
  email_key  VARCHAR(255) NOT NULL,           -- lower(email), Locale.ROOT
  isbn       VARCHAR(13)  NOT NULL,
  title      VARCHAR(255) NOT NULL,           -- the book's title when subscribed
  status     VARCHAR(16)  NOT NULL CHECK (status IN ('PENDING','CANCELLED','ALERTED','WITHDRAWN')),
  created_at TIMESTAMPTZ  NOT NULL,
  ended_at   TIMESTAMPTZ,
  CHECK ((status = 'PENDING') = (ended_at IS NULL)));
CREATE UNIQUE INDEX subscriptions_one_pending ON subscriptions (email_key, isbn) WHERE status = 'PENDING';
CREATE INDEX subscriptions_pending_isbn ON subscriptions (isbn) WHERE status = 'PENDING';
CREATE INDEX subscriptions_email ON subscriptions (email_key, created_at);
CREATE INDEX subscriptions_ended ON subscriptions (ended_at) WHERE ended_at IS NOT NULL;

CREATE TABLE alerts (
  id              BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  subscription_id BIGINT NOT NULL UNIQUE REFERENCES subscriptions (id) ON DELETE CASCADE,
  email           VARCHAR(255) NOT NULL,
  email_key       VARCHAR(255) NOT NULL,
  isbn            VARCHAR(13)  NOT NULL,
  title           VARCHAR(255) NOT NULL,
  type            VARCHAR(16)  NOT NULL CHECK (type IN ('BACK_IN_STOCK','WITHDRAWN')),
  occurred_at     TIMESTAMPTZ  NOT NULL,
  created_at      TIMESTAMPTZ  NOT NULL,
  is_read         BOOLEAN      NOT NULL DEFAULT FALSE,
  dismissed_at    TIMESTAMPTZ);
CREATE INDEX alerts_email ON alerts (email_key, created_at);

CREATE TABLE configs (                       -- the shared entity SecurityController reads (storage/…/model/Config.java)
  id VARCHAR(255) PRIMARY KEY, load_cpu BIGINT, load_ram BIGINT, probab_fail DOUBLE PRECISION,
  property_str VARCHAR(255), turn_on BOOLEAN);
```

### Configuration keys (new)

| Key | Value | Where |
|---|---|---|
| `http.service.clients` / `.books` / `.storage` | `http://${DT_CLIENTS_SERVER:localhost:8081}/api/v1/clients`, `…${DT_BOOKS_SERVER:localhost:8082}/api/v1/books`, `…${DT_STORAGE_SERVER:localhost:8084}/api/v1/storage` | every profile |
| `spring.datasource.url` | `jdbc:postgresql://${DT_PG_SERVER:localhost}:${DT_PG_PORT:5432}/${DT_PG_DBNAME:dt_books_alerts}?socketTimeout=5` | dev / stage / prod |
| `spring.jpa.hibernate.ddl-auto` | `validate` (Flyway owns the schema; no `spring.sql.init.*` copied) | every profile |
| `spring.datasource.hikari.maximum-pool-size` / `connection-timeout` | `20` / `3000` | `application.properties` |
| `spring.task.scheduling.pool.size` | `2` (sweep + purge) | `application.properties` |
| `alerts.sweep.enabled` / `alerts.sweep.period` / `alerts.sweep.max-lanes` / `alerts.sweep.circuit-timeouts` | `true` (`false` in `application-test.properties`) / `20s` / `32` / `8` | `application.properties` |
| `alerts.purge.cron` | `0 0 * * * *` | `application.properties` |
| `alerts.lookup.connect-timeout` / `alerts.lookup.request-timeout` | `1s` / `3s` | `application.properties` |
| `logging.level.com.dynatrace.alerts.events` | `INFO` | every profile |
| `server.port` | `${DT_SERVER_PORT:8091}` dev; `${DT_SERVER_PORT:80}` stage / prod | per profile |
| `DT_ALERTS_SERVER` | `alerts-svc:91` | `k8s/configmap.yaml` |

### Contract rows ([AD#N])

- **Produces [AD#4]** `POST /available` — meets the Rule through `NoticeRecorder` (one transaction, `REQUIRES_NEW`, 10-second timeout, at-most-once by row locks and `UNIQUE(subscription_id)`, `204` either way); producer-side tests in § Test strategy (notice tests, 1,000 in 10 s).
- **Produces [AD#6]** `POST /withdrawn` — the same recorder with `WITHDRAWN`; the alert carries the title stored at subscribe; producer-side tests as [AD#4]'s.
- **Produces [AD#7]** `DELETE /delete-all` — one `DELETE … WHERE NOT reserved` with cascade, no alert; producer-side reset tests on reserved fixtures ([AD#11]).
- **Produces [AD#8]** the client-facing API above — producer-side API tests per operation; two departures are recorded under § ARD deviations.
- **Consumes [AD#5]** books `GET /api/v1/books/find` and storage `GET /api/v1/storage/findByISBN` — `exists`, used as they run (`BookController.java:52`, `StorageController.java:51`); tests use `InMemoryStoreLookups`, whose `Found` / `NotFound` / `NoAnswer` model the endpoints' answers and failures, and WireMock models the HTTP shapes.
- **Consumes [AD#2]** clients `GET /api/v1/clients/find?email=` — `exists`, used as it runs (`ClientController.java:50`); the same doubles.
- **Consumes [AD#11]** the reserved-data definition — applied literally by the reset; tests use fixtures built to it, standing in for [[BOOK-1-06]]'s data.

## Seams

| Seam | Category | How it is exercised |
|---|---|---|
| HTTP API (controllers, `ApiExceptionHandler`) | in-process | `MockMvc` against the full Spring context — real services, real PostgreSQL (Testcontainers), `InMemoryStoreLookups` in place of the HTTP adapter. The highest seam that still isolates alerts from the other services. |
| `StoreLookups` → clients / books / storage | remote-but-owned | Port with two real adapters: `InMemoryStoreLookups` (tests: per-key `Found` / `NotFound` / `NoAnswer`, call log for "storage was not asked") and `HttpStoreLookups` (production), the latter proven against WireMock. |
| `NoticeRecorder`, repositories, `MaintenanceSql` → PostgreSQL | local-substitutable | Testcontainers PostgreSQL; no port at the database — the recorder is a concrete class with one adapter (two-adapters heuristic). |
| `Clock` | in-process | A fixed or offset `Clock` bean in tests sets `recordedAt`, `occurredAt` and the 30-day cutoff. |
| `Sweep` scheduling | in-process | `Sweep.runPass()` is public and called directly by tests; `@Scheduled` is a one-line wrapper switched off by `alerts.sweep.enabled=false` in the test profile, so no test waits on a timer. |

Deep-module check: `NoticeRecorder` (one method over locking, mapping and atomicity) and `StoreLookups` (three methods over encoding, timeouts and seven HTTP outcomes) each concentrate complexity that would otherwise be spread over three callers; deleting either would duplicate it, not remove it.

## Data flow

**Subscription lifecycle.**

```
            subscribe (201)
                 │
             PENDING ──cancel──────────────▶ CANCELLED ─┐
                 │──available / sweep ≥1 copy▶ ALERTED  ─┤  + one alert (BACK_IN_STOCK)
                 │──withdrawn / sweep 404 ──▶ WITHDRAWN ─┤  + one alert (WITHDRAWN)
                 │                                      │
                 └──reset (not reserved)──▶ deleted ◀───┴── purge (ended_at < now − 30 d) / reset (not reserved)
```

An alert is created with the subscription it ends, in the same statement, `created_at = ended_at`; it becomes read by mark-read, dismissed by dismiss (`dismissed_at`), and is deleted only with its subscription (cascade).

**Subscribe.** Controller validates → hooks → `SubscriptionService`: `SELECT … WHERE email_key = ? AND isbn = ? AND status = 'PENDING'` → hit: `200`. Miss: `client(email)` → `book(isbn)` (keeps the title) → `copies(isbn)` → `0`: `INSERT` (own transaction) → `201`. A unique-index violation on the insert means a concurrent subscribe won: re-read the pending row and answer `200` with it; if the winner was already cancelled, retry the insert once, and a second violation answers `503`.

**Notice.** `/available` or `/withdrawn` → `occurredAt = clock.instant()` → `NoticeRecorder.record` (its own transaction) → `204`; one `notice` event line.

**Sweep pass** (`@Scheduled(fixedRateString = "${alerts.sweep.period:20s}")` on its own scheduler thread: a pass starts 20 s after the previous one started, or as soon as it ends where it overran; passes never overlap). `SELECT DISTINCT isbn FROM subscriptions WHERE status = 'PENDING' ORDER BY isbn` → N ISBNs in that stable order → W = clamp(⌈N/10⌉, 8, 32) lanes on `sweepExecutor`, each taking the next ISBN from the list → per ISBN: `book(isbn)`: `NotFound` → `record(isbn, WITHDRAWN, clock.instant())`; `NoAnswer` → skip, storage not asked; `Found` → `copies(isbn)`: `Found(n ≥ 1)` → `record(isbn, BACK_IN_STOCK, clock.instant())`; anything else → skip. **Circuit stops**, counted across the lanes and reset by any other answer from that service: after 8 consecutive storage answers with `timedOut`, the pass stops asking storage — books is still asked, so a not-found still withdraws, and a found book counts as `noAnswer`; after 8 consecutive books answers with `timedOut`, the remaining ISBNs are skipped as `noAnswer`. A fast failure (`5xx`, `429`, a refused connection — what an injected fault produces) never trips them. The pass waits for every lane, then logs one `sweep` event line. ISBNs subscribed during a pass are checked by the next one.

**Cadence.** The stable order keeps each ISBN at about the same offset in every pass, so it is re-checked within ≈ max(20 s, P) + the spread of its offset, P being the pass's length: ≈ 21–23 s at normal latency (≈ 60 ms per ISBN). W lanes at ≈ 2 s per ISBN — books answering in ≈ 1 s, asked twice through storage's `verifyBook` — keep P ≈ 20 s up to ≈ 320 pending ISBNs ([ER12]); a storage service holding requests costs the pass 8 timeouts (≈ 24 s of lane time spread over the lanes) before it stops waiting on storage.

**Reads.** Alerts: `WHERE email_key = ? AND dismissed_at IS NULL AND created_at >= :cutoff ORDER BY occurred_at DESC, created_at DESC`; unread count: the same predicate `AND NOT is_read`; pending subscriptions: `WHERE email_key = ? AND status = 'PENDING' ORDER BY created_at DESC`. `:cutoff = clock.instant() − 30 days`, so the ±1-minute boundaries of [U04] AC02 are exact whatever the purge has done.

**Cancel / dismiss.** Cancel: `UPDATE … SET status = 'CANCELLED', ended_at = :now WHERE id = ? AND email_key = ? AND status = 'PENDING'` → 1 row: `204`; 0 rows: the row exists for that email with `ended_at >= :cutoff` → `204`, else `404`. Dismiss: `UPDATE alerts SET dismissed_at = COALESCE(dismissed_at, :now) WHERE id = ? AND email_key = ? AND created_at >= :cutoff` → 1 row: `204`, 0: `404`.

**Reset.** `DELETE FROM subscriptions WHERE isbn NOT LIKE '00000000%' AND email NOT LIKE '%@bis-journey.invalid'` (alerts cascade) → `204`; one `reset` event line.

**Purge** (hourly). `DELETE FROM subscriptions WHERE id IN (SELECT id FROM subscriptions WHERE ended_at < :cutoff LIMIT 1000)`, repeated until a batch deletes fewer than 1,000 rows; alerts cascade; one `purge` event line.

## Error handling & edge cases

| Condition | Behaviour |
|---|---|
| `email` or `isbn` missing or blank | `400`, nothing changed — an explicit `StringUtils.hasText` check throwing the `exceptions` module's `BadRequestException`, checked before the hooks so [U01] AC10's order holds. |
| `{id}` not numeric | No handler matches `{id:\d+}` → Spring's `404`. |
| A remote service does not answer (subscribe) | `503`, nothing recorded; the first failing check decides ([U01] AC10). |
| Storage `404` after books found the book (subscribe) | `404` — storage's `verifyBook` relays books' not-found (`StorageController.java:146`), so the book was removed between the two calls ([U01] AC08 TC03). |
| A remote service does not answer (sweep) | That ISBN is skipped this pass; books not answering means storage is not asked ([AD#5]). |
| Storage `404` in the sweep | Skip — only books' own not-found withdraws ([U03] AC03). |
| Alerts' database fails (any operation) | `ApiExceptionHandler` maps `DataAccessException`, `CannotCreateTransactionException` and `TransactionTimedOutException` to `503`; the failed transaction changed nothing. A connection is waited for at most 3 s and a statement in flight at most 5 s (`socketTimeout=5`). A notice so answered is lost and the sweep recovers it. |
| Recording exceeds 10 s or fails part-way | The recorder's transaction rolls back; every subscription to that ISBN stays pending with no alert ([U02] AC08, [U03] AC07). |
| A configuration entry turned on | `SecurityException` (`503`, `exceptions/…/SecurityException.java`) or CPU/RAM load on client-facing operations only; notices, reset, sweep and purge unaffected. |
| Two notices, or a notice and the sweep, for one ISBN at once | Row locks serialise them; the second re-checks `status = 'PENDING'` and records nothing; `UNIQUE(subscription_id)` is the backstop. Lock order `ORDER BY id` prevents deadlock. |
| A subscribe racing a restock or removal notice | The checks see 0 copies, the notice ends the existing subscriptions, then the insert lands: the new subscription stays pending while copies exist (or the book is gone) until the next sweep pass alerts or withdraws it — within ≈ 23 s. Recorded in the spec's `## Engineering review`. |
| A notice or the sweep during a reset | Row locks serialise them; the recorder only touches rows still `PENDING`, so a deleted subscription is never alerted. |
| A sweep task throws | Caught per ISBN, counted as `failed=` in the pass line; other ISBNs are unaffected. Reading the pending ISBNs fails → the pass logs and ends; the next starts on schedule. |
| A pass longer than 20 s (many ISBNs, slow services) | The next pass starts as soon as it ends — no overlap; the pass line's `ms=` shows it (§ Risks, [ER12]). |
| Storage holding requests during a pass | 8 consecutive `timedOut` answers stop the pass asking storage; withdrawals from books' not-found go on; restocks wait for a pass in which storage answers ([U02] AC05 TC04, [U03] AC03 TC03). |
| Books holding requests during a pass | 8 consecutive `timedOut` answers end the pass's remaining ISBNs as `noAnswer`; nothing is decidable without books ([U03] AC06). |
| Purge fails | Logged; the next hour retries. Reads never depend on it. |
| Unknown email on a read | Empty list / `{"count": 0}` ([U04] AC08, [U05] AC06). |
| A storage quantity below 1 | Out of stock. |
| Database `dt_books_alerts` missing at startup | In Kubernetes the initContainer has created it; elsewhere Flyway / Hikari fail the start with the JDBC error (local setup in § Migration). |

## Test strategy

Prior art is thin: every module has one H2 `contextLoads` (`storage/src/test/java/com/dynatrace/storage/StorageApplicationTests.java`) and nothing else, so alerts brings its own harness. All of it runs in `./gradlew :alerts:test`, and so in CI's `./gradlew build` (`.github/workflows/gradle.yml`, `ubuntu-latest`, Docker available); `docker-images.yml` builds with `-x test` and is unaffected.

**Harness.** `AbstractAlertsIT`: `@SpringBootTest`, `@ActiveProfiles("test")`, one static `PostgreSQLContainer` (`postgres:16-alpine`, re-pinned at implementation to the cluster's PostgreSQL major — § Risks) with `@ServiceConnection`, Flyway applying `V1`; a `@TestConfiguration` with a `@Primary` `InMemoryStoreLookups` and a `MutableClock`; `alerts.sweep.enabled=false` and the purge cron off (`-`) in `application-test.properties`; tables truncated between tests. `AlertsApplicationTests.contextLoads` extends it, replacing the H2 convention for this module.

**Producer-side tests of the boundary interfaces** (`MockMvc` on the full context, keyed to the spec's test cases):

| Seam / interface | Category | Tests |
|---|---|---|
| [AD#8] subscribe | in-process over local-substitutable | `SubscribeApiIT` — [U01] AC01–AC11 TCs: every outcome and the fixed order by scripting `InMemoryStoreLookups`; the 200 short-cut asserts no lookup was called; two concurrent subscribes (latch-synchronised threads) answer one `201` and one `200` with the same id; case and `+` emails |
| [AD#8] reads, mark-read, dismiss, cancel | in-process over local-substitutable | `AlertsApiIT`, `SubscriptionsApiIT` — [U04] AC01–AC08 and [U05] AC01–AC06 TCs; ±1-minute 30-day boundaries by moving `MutableClock`; other-email ids answer `404` |
| [AD#4] / [AD#6] notices | in-process over local-substitutable | `NoticeApiIT` — [U02] AC01, AC06, AC07, AC09, AC10 and [U03] AC01, AC08 TCs: `204` with and without subscribers, repeats record nothing, titles stored at subscribe, `occurredAt` = the clock at arrival |
| [AD#7] reset | in-process over local-substitutable | `ResetApiIT` — [U06] AC01–AC03 TCs on fixtures built to [AD#11]: seven zeros, an email only containing the domain, the domain in capitals, another client's subscription to a reserved book, no alert recorded |
| Database unavailable | local-substitutable | `DatabaseDownIT` — pauses the container (Docker pause) and waits 1 s, past Hikari's 500-ms validation window: a notice, a subscribe, a read, a cancel, a dismiss, a mark-read and the reset each answer `503` within 7 s; after unpause nothing changed — every subscription still pending, no alert read, dismissed or deleted ([U02] AC08 TC01, [U03] AC07 TC01, [ER1]) |
| Configuration hooks | in-process over local-substitutable | `ConfigHooksIT` — inserts `dt.simulate.crash` turned on at probability 100 with no time window: subscribe, both reads, the unread count, mark-read, cancel and dismiss answer `503` and change nothing; `/available`, `/withdrawn` and `/delete-all` answer `204` and do their work; `Sweep.runPass()` still records ([ER2]) |
| Version and configuration | in-process over local-substitutable | `VersionConfigIT` — `GET /api/v1/version` answers with serviceId `alerts`; `GET /api/v1/config` answers an empty list after `V1` ([U07] AC07 TC01–TC02) |

**`NoticeRecorder` on PostgreSQL** (`NoticeRecorderIT`): 1,000 pending to ISBN B beside 1,000 to B2 — `record(B, …)` returns 1,000, finishes in under 10 s (expected well under 1 s), and B2's stay pending ([U02] AC02, [U03] AC02); two threads recording B at the same moment → exactly 1,000 alerts in total ([U02] AC06 TC03); a record racing a cancel on one subscription → that subscription is either cancelled with no alert or alerted, never both; a conflicting alert row pre-inserted for one of 1,000 pending subscriptions makes the statement fail on `UNIQUE(subscription_id)` → `DataAccessException`, all 1,000 still pending, no new alert — the in-statement failure standing in for "the connection cut part-way" ([U02] AC08 TC02, [U03] AC07 TC02), since the recording is one statement.

**Sweep** (`SweepIT`, `runPass()` called directly, real recorder, scripted lookups): restock → `BACK_IN_STOCK` with `occurredAt` = the pass's clock; books `NotFound` → `WITHDRAWN` whatever storage would say; books `NoAnswer` → nothing, and the lookup log shows storage was not asked; storage `NotFound`, `Found(0)` and `NoAnswer` → nothing; unpublished-but-found with copies → alerted ([U02] AC03 TC04); 100 ISBNs restocked together → 100 alerts in one pass, and one restocked of 100 → 99 still pending ([U02] AC04, [U03] AC04); one ISBN whose recording fails (pre-inserted conflict) → the others committed and the pass line counts `failed=1`; lanes W = 8 for 10 ISBNs, 10 for 100, 32 for 500; 8 consecutive storage `timedOut` answers → storage no longer asked while a books `NotFound` later in the same pass still withdraws; 8 consecutive books `timedOut` answers → the rest skipped; 8 fast storage `NoAnswer`s (`500`) → no circuit stop. `SweepScheduleIT` turns the schedule on with `alerts.sweep.period=1s` and asserts a restock is alerted within 5 s, and that a pass made to last 2 s is followed at once by the next — the wiring, not the 40-s budget, which § Data flow derives.

**Retention and purge** (`RetentionIT`, `PurgeIT`): rows at 30 days ± 1 minute against `MutableClock` for every read, count, dismiss and cancel ([U04] AC02, AC05–AC07; [U05] AC03, AC04); the purge deletes 2,500 subscriptions ended 30 days and 1 minute ago in three batches, with their alerts, and keeps those ended 30 days less 1 minute ago and every pending one ([U01] AC03).

**`HttpStoreLookups` against WireMock** (`HttpStoreLookupsTest`, no Spring context): a 5-s fixed delay → `NoAnswer` in under 3.5 s; a closed port → `NoAnswer`; `500`, `429` → `NoAnswer`; `404` → `NotFound`; storage empty `200` → `Found(0)`, `{"quantity": 2}` → `Found(2)`; books empty `200` → `NoAnswer`, a book JSON → `Found(title)`; `first.last+books@example.com` arrives with `%2B` in the raw query. `SubscribeHttpIT` runs the full context with the real adapter against WireMock for [U01] AC09 TC02 and TC05 (all three services holding 5 s → `503` within 10 s) and [U01] AC06 end to end.

**Consumed interfaces.** [AD#2] and [AD#5] rows are `exists`: `InMemoryStoreLookups` models their success shapes and the failures their Rules name — not-found, and every "does not answer" case as `NoAnswer` — and WireMock models the HTTP forms of the same.

**Deployment criteria** ([U07] AC01–AC06) have no cluster in CI; they are the scripted post-deploy checks of § Observability, run on the stage cluster, plus `kubectl apply --dry-run=client -f k8s/alerts.yaml -f k8s/ingress.yaml` in the implementation's verification.

## Risks & mitigations

- **Sweep load on books and storage.** Per pending ISBN every 20 s the sweep calls books once and storage once, and storage's `verifyBook` calls books again (`StorageController.java:146`): for 100 pending ISBNs ≈ 10 req/s on books `/find` (each running its two `configs` lookups) and ≈ 5 req/s on storage, from at most 32 concurrent lanes. Mitigation: only pending ISBNs are swept ([AD#5]); the lanes are bounded; the pass line shows the size; `alerts.sweep.period` and `alerts.sweep.max-lanes` are configurable; the rollback signal watches books' response time.
- **The sweep's scale limit** ([ER12]). With services answering in ≈ 1 s (books' `/find` runs `runThreatScan()`, `BookController.java:55`), an ISBN costs ≈ 2 s and 32 lanes keep a pass within 20 s up to ≈ 320 pending ISBNs; at normal latency (≈ 60 ms) far more. Beyond that a pass outlasts 20 s, the next starts when it ends, and the 30-s cadence and the 40-s recovery criteria no longer hold — proposed to the PM as a bound on the spec's "whatever the number of books waited on". Mitigation: lanes grow with the pending count; circuit stops keep a holding service from consuming the pass; the pass line's `ms=` is the signal.
- **Recording contends for connections.** Up to 32 lanes may record at once against a pool of 20; a lane that waits more than 3 s for a connection fails that ISBN for this pass (`failed=`) and the next pass retries it. Accepted; recordings take milliseconds.
- **Row locks during a fan-out.** Recording locks up to 1,000 rows of one ISBN; a concurrent cancel of one of them waits for the statement (milliseconds). Accepted.
- **PostgreSQL connections.** Alerts' pool is 20; with books, storage, orders and ingest at Hikari's default 10 the server carries ≈ 60 of its default 100. Accepted; the blast-radius signal watches it.
- **PostgreSQL major mismatch.** Production runs `ghcr.io/ihudak/dt-postgres:latest`, built `FROM postgres:latest` (`database/pg/Dockerfile:1`) — major unknown — while the tests pin `postgres:16-alpine`; Flyway 11 (Spring Boot 3.5's) may log a warning on a newer major. Mitigation: the implementation reads `SELECT version()` on the stage cluster and pins the test image to that major. Every feature used (partial indexes, data-modifying CTEs, `FOR UPDATE` re-check) exists since PostgreSQL 9.5.
- **Docker becomes a build prerequisite.** `./gradlew build` now needs Docker for alerts' tests. CI has it; a developer without it runs `./gradlew build -x :alerts:test`. Accepted.
- **Departures from repository conventions** — Flyway with `validate`, Testcontainers, `RestClient` with timeouts, a service layer, a `@RestControllerAdvice` — confined to `alerts/`; each is justified in § Architecture's alternatives.
- **No authentication.** Any caller can act as any client by naming its email ([AD#2]); accepted by the ARD.
- **Client email letter case relies on clients' MySQL collation** for subscribe's existence check (spec decision; `clients/…/ClientRepository.java:12`). Accepted.
- **New dependencies.** Flyway and Testcontainers are BOM-managed; WireMock 3.x (Apache-2.0) is test-only and pinned at implementation.

## Migration / rollout / backward-compatibility

- **Additive.** A new service, database, ingress path and configmap keys; no existing service, route or table changes, and nothing calls alerts until [[BOOK-1-02]], [[BOOK-1-03]], [[BOOK-1-05]] and [[BOOK-1-06]] land — alerts is first in the ARD's landing order.
- **Schema.** Flyway `V1__alerts_schema.sql` on the empty `dt_books_alerts`; later changes are `V2+`, additive within `/api/v1` per the ARD's versioning rule.
- **Database creation.** Fresh volumes: `01-create-db.sql`'s new line. Existing volumes (`restart.sh`, `restart.sh -nodb`): the `create-db` initContainer in `k8s/alerts.yaml` — `postgres:16-alpine`, `PGPASSWORD`/user from `bookstore-secret`, loops `pg_isready -h postgres-service` then `psql -h postgres-service -U "$DT_PG_USER" -d postgres -tc "SELECT 1 FROM pg_database WHERE datname='dt_books_alerts'" | grep -q 1 || psql … -c "CREATE DATABASE dt_books_alerts OWNER pguser"`; idempotent on every rollout. Local docker-compose on an existing `./data` volume: `docker exec postgres_container psql -U pguser -c 'create database dt_books_alerts owner pguser'` once.
- **Rollout order.** Build and push images (`build_docker_all.sh` or `push_docker.sh -p alerts`), `preset_deployment.sh` for the image flavour, then `restart.sh -nodb` — which applies the configmaps, `alerts.yaml` with the other services, and `ingress.yaml` last — then `kubectl rollout restart deployment -n bookstore` (what `redeploy.cmd` does per service): Kubernetes does not restart a Deployment whose manifest is unchanged, and `envFrom` is read only at container start, so without it no running pod sees `DT_ALERTS_SERVER`.
- **Backward compatibility.** Every existing ingress path routes as before, the new path sitting before the web catch-all ([U07] AC02); the version and configuration operations follow every service's shape.
- **Rollback.** `kubectl delete -f k8s/alerts.yaml` (and revert the ingress, configmap and agents entries, or leave them — `/api/alerts/` then answers the ingress's 503); the database stays, harmless; nothing to migrate back. No feature flag: no other service consumes alerts yet.

## Observability & release verification

- **Signals.**
  - Working: agent telemetry for the new alerts service, tagged `MicroService=BookStore.Alerts` (`DT_TAGS`, as every BookStore service is tagged) — request count, failure rate and response time per endpoint, its outgoing calls to clients, books and storage, its SQL on `dt_books_alerts` — existing capability, on a new service; the event lines on `com.dynatrace.alerts.events` at INFO in every profile — **added by this change**: `notice type=… isbn=… ended=… ms=…`, `sweep isbns=… alerted=… withdrawn=… noAnswer=… failed=… ms=…`, `purge deleted=…`, `reset deleted=…`.
  - Failing: alerts' `5xx` rate; `sweep … failed=` above 0; a persistently high `noAnswer=`; one WARN line per `NoAnswer` naming the service and the reason — added by this change.
- **Baseline.** Alerts has none — its first week's values become it. Books `GET /api/v1/books/find`, storage `GET /api/v1/storage/findByISBN` and clients `GET /api/v1/clients/find`: request rate, p95 response time and failure rate, read in Dynatrace (stage environment, BookStore services only) over the 7 days before the deploy, captured by the operator at deploy time; PostgreSQL connection count over the same window.
- **Blast radius.** Books (sweep, subscribe, storage's `verifyBook`) — its `/find` p95 and failure rate; storage (sweep, subscribe) — `findByISBN` p95; clients (subscribe only) — `/find` request rate; the PostgreSQL server (a new database, up to 20 connections) — connection count and CPU; the ingress (one new path) — the existing services' request counts through it; the namespace's capacity (400m CPU / 768Mi, as storage) — pod scheduling. Each is in § Risks.
- **Post-release checks** (stage cluster, after `restart.sh -nodb`):
  - [U07] AC01: `curl "http://kubernetes.docker.internal/api/alerts/api/v1/alerts?email=new.client@example.com"` → `[]`; `kubectl exec deploy/storage -n bookstore -- sh -c 'curl -s "http://$DT_ALERTS_SERVER/api/v1/alerts/unread-count?email=new.client@example.com"'` → `{"count":0}` (the variable expands inside the pod); the first URL without `email` → `400` from alerts.
  - [U07] AC02: `GET /api/clients/api/v1/version`, and the same under `/api/books/`, `/api/carts/`, `/api/storage/`, `/api/orders/`, `/api/payments/`, `/api/dynapay/`, `/api/ratings/` and `/api/ingest/`, each answers its own version as before; `/` still serves the web store.
  - [U07] AC03: book, storage, order and client counts recorded before the deploy are unchanged after it, and alerts answers.
  - [U07] AC06, AC07: alerts' `/api/v1/version` answers serviceId `alerts` and reports the agent `alerts_agent` selects; `/api/v1/config` answers a list.
  - [U01]: one subscribe of a test client to a book with quantity 0 → `201`, then cancel → `204`.
  - [U02] AC03–AC04, [U03] AC03–AC04 (sweep): a `sweep` line every 20 s (± 1 s) over the first 24 h, each with `ms` under 20,000 — a longer pass delays the next ([ER12]).
  - [U02] AC02, [U03] AC02 (notices): every `notice` line with `ms` under 10,000 once [[BOOK-1-02]] / [[BOOK-1-03]] send notices; n/a until then.
  - [U04] AC01, AC03, [U05] AC01 (reads): p95 response time of `GET /api/v1/alerts`, `/unread-count` and `/subscriptions` ≤ 200 ms over 24 h ([ER3]) — read per request in the service's response-time breakdown in Dynatrace Managed; the service-level guard is `builtin:service.response.time:filter(in("dt.entity.service",entitySelector("type(SERVICE),tag(\"MicroService=BookStore.Alerts\")"))):percentile(95)` ≤ 200 ms (the metric is in microseconds: ≤ 200000), scoped by the repository's documented BookStore selector.
  - [U01] AC02–AC11, [U02] AC01, AC05–AC10, [U03] AC01, AC05–AC08, [U04] AC02, AC04–AC08, [U05] AC02–AC06, [U06], [U07] AC04–AC05: n/a — no runtime signal distinguishes them from a correct answer; they rest on § Test strategy.
- **Alert / SLO impact.** No SLO or custom alert exists for BookStore services in this repository; Dynatrace's automatic baselining covers the new service from its first traffic. No existing alert is retuned. With a configuration entry turned on, alerts' client-facing operations now join the store's deliberately unhealthy surfaces (§ Error handling).
- **Rollback signal.** With no configuration entry turned on, for 15 minutes after alerts is deployed: books `GET /api/v1/books/find` p95 above 2× its pre-release baseline, or its failure rate more than 2 percentage points above it; or the `5xx` rate of the service tagged `MicroService=BookStore.Alerts` above 5 % while its database is up → the demo operator rolls back per § Migration.

## ARD deviations

The design implements the specification, which departs from [AD#8] in two places; the spec carries the matching open question (Scope), which the architect settles by refining [AD#8] with `/product-workflows:create-ard BOOK-1`, or by changing [U02] AC09, [U03] AC08, [U04] AC07 and [U05] AC04.

- ARD deviation: [AD#8] — an alert's `occurredAt` is when alerts learned of the return or removal (a notice's arrival, or the sweep's observation up to ≈ 23 s after it), not when the book came back or was removed — no notice carries a time and the sweep sees only state — flag: architect
- ARD deviation: [AD#8] — cancel and dismiss answer `404`, not `204`, for a subscription that ended, or an alert recorded, more than 30 days ago, and the purge deletes those records — keeping ended records forever to answer `204` would grow the reserved data without bound — flag: architect

## Out of scope

- Everything the specification's `## Scope` puts out of scope — storage's restock notice ([[BOOK-1-02]]), books' removal and reset calls ([[BOOK-1-03]]), the web store ([[BOOK-1-05]]), the synthetic journey ([[BOOK-1-06]]), and the rest of its list.
- Ingest's configuration fan-out (`serviceId: "all"`) and version polling, which enumerate services in ingest's code: alerts is not added there; its own `/api/v1/config` and `/api/v1/version` answer when called directly. A follow-up for ingest's Epic.
- The other services' timeout-less `RestTemplate`s and unencoded email query strings (`orders/…/ClientRepository.java:25`) — noted, not changed.
- `Version.setVerDocker` assigning `ver` (`common/src/main/java/com/dynatrace/model/Version.java`) — affects every service; not changed.
- A metrics endpoint or Micrometer registry.
- Authentication of the caller ([AD#2]).

## Open questions

_(none)_

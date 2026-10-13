---
kind: design
key: BOOK-2-01-02
---

# Design

- **Feature name**: Storefront ratings: give, change and see buyers' ratings
- **Spec**: `specification.md` (BOOK-2-01-02)
- **Classification**: SIGNIFICANT
- **Version**: 1
- **Created**: 2026-10-13
- **Author**: ihudak
- **Repos**: bookstore
- **Target**: bookstore:web
- **Open questions**: 0

## Context & problem

The storefront contradicts the rules the ratings service takes on in this release ([[BOOK-2-01-01]]). Its rating form takes any number and shows a bare pop-up when a rating is refused. Its update form lets the e-mail and the book be edited and shows the stored comment. The Ratings list and the rating details screen show every comment and offer Delete and Delete All. The book page and the catalogue list show no rating at all (specification, Problem statement). This design changes `web/` alone. The forms send only whole stars from 1 to 5 and show each answer of the ratings service as the specification's verbatim text, keeping what was entered. The book page and the catalogue list show the figures the ratings service computes. Comments and removal leave every screen. Every rating rule stays in the ratings service ([AD#1]).

## Requirements coverage

Challenge notes: **validated** (the design meets the criterion as written), **questioned** (the design reads an open term one way, recorded in the specification's `## Engineering review`) or **proposed-change**. Test cases are covered by the screen specs in Test strategy unless the row says otherwise.

| Item | How the design meets it | Challenge |
|---|---|---|
| [U01] [AC01] | `StarChoiceComponent` offers five native radios 1–5, none checked while `stars` is `undefined` | validated |
| [U01] [AC02] | Submit `[disabled]` unless e-mail and ISBN each hold a character other than a space, a star is chosen and nothing is in flight; `onSubmit` re-checks the gate | validated |
| [U01] [AC03] | `give({email, isbn, rating, comment})` posts the values as typed, untrimmed; an empty comment is sent as `''` | validated |
| [U01] [AC04] | `{stored: true}` (any `200`) → `router.navigate(['/ratings'])` | validated |
| [U01] [AC05] | `classifyWrite` maps `403` to the buyers-only text whatever the body | validated |
| [U01] [AC06] | `400` with a readable reason → that reason, interpolated as text (never `innerHTML`) | validated |
| [U01] [AC07] | `503` → the unavailable text, whatever the body | validated |
| [U01] [AC08] | every other answer, status 0 and the 8 s bound → the cannot-save text; expiry cancels the request, so a late `200` reaches nothing | validated; bound set to 8 s (Interfaces) |
| [U01] [AC09] | no outcome but stored touches the entered fields | validated |
| [U01] [AC10] | `sending` disables Submit from send to outcome | validated |
| [U01] [AC11] | `onSubmit` clears `outcomeText` before sending | validated |
| [U01] [AC12] | outcome text in a persistent `role="alert"` element, emptied and refilled on each Submit | validated; announcement checked by hand ([ER3]) |
| [U01] [AC13] | native radio group: Tab reaches it, the arrow keys move and choose; no value outside 1–5 exists | validated; checked by hand ([ER3]) |
| [U01] [AC14] | the same gate as [AC02], read positively | validated |
| [U02] [AC01] | the Ratings list keeps the e-mail column and Update → `update-rating/:id` | validated |
| [U02] [AC02] | heading "Update Rating"; e-mail and ISBN rendered as plain text; `StarChoiceComponent` pre-set to the loaded stars | questioned ([ER2]: the values shown come from a readable rating) |
| [U02] [AC03] | `change(loaded, stars, comment)` → `PUT /{loaded.id}` carrying the loaded integer id, e-mail and ISBN | validated |
| [U02] [AC04] | `{stored: true}` → `/ratings` | validated |
| [U02] [AC05]–[AC07] | as [U01] [AC05]–[AC07], through the same `classifyWrite` | validated |
| [U02] [AC08] | `classifyWrite(…, 'change')` maps `404` to "This rating no longer exists. Nothing was saved." | validated |
| [U02] [AC09] | as [U01] [AC08]; a `404` counts here only on a give | validated |
| [U02] [AC10] | no outcome but stored touches `stars` or `comment` | validated |
| [U02] [AC11], [AC12] | as [U01] [AC10], [AC11] | validated |
| [U02] [AC13] | `classifyLoad` maps `404` to "This rating no longer exists.", shown in place of the fields and Submit | validated |
| [U02] [AC14] | anything but a readable rating or `404`, the 8 s bound included → "We can't load this rating right now. Please try again in a few minutes." | questioned ([ER2]: "the rating" read as a readable rating) |
| [U02] [AC15] | while `load` is `undefined` only the heading renders | validated |
| [U02] [AC16], [AC17] | as [U01] [AC12], [AC13] | validated; checked by hand ([ER3]) |
| [U02] [AC18] | Submit enabled once loaded and nothing in flight (stars are always chosen on a loaded rating) | validated |
| [U03] [AC01], [AC02] | `bookLine(isbn)` → "★ " + `average.toFixed(1)` + " (1 rating)" / " (N ratings)", rendered below Vend | validated |
| [U03] [AC03] | readable count 0 → "Not yet rated", no star, no count | validated |
| [U03] [AC04] | anything but a readable summary, the 8 s bound included → "Rating unavailable"; the book's rows unaffected | validated |
| [U03] [AC05] | nothing renders until the summary has an outcome | questioned ([ER4]: the line also waits for the book) |
| [U03] [AC06] | `bookLine` is cold and uncached; `BookDetailsComponent.ngOnInit` subscribes once per load | questioned ([ER4]: requested on every load, rendered once the book has loaded) |
| [U04] [AC01] | "Rating" column after "OnVend"; `catalogueLines()` → "★ " + `average.toFixed(1)`, no count | validated |
| [U04] [AC02] | a book absent from a readable response, or listed with count 0 → "Not yet rated" | validated |
| [U04] [AC03] | both screens format through the same `rating-texts.ts` functions from the same figures | validated |
| [U04] [AC04] | an unreadable response (any unreadable entry included), a failure or the 8 s bound → "Rating unavailable" on every row | validated |
| [U04] [AC05] | rating cells render empty until the summaries have an outcome | validated |
| [U04] [AC06] | one `catalogueLines()` subscription per screen open; in-place refreshes reuse it | questioned ([ER1]: "loads" read as a screen open) |
| [U05] [AC01], [AC02] | both forms render the two notes at the comment field, outside the loading and failure states of the update form | validated |
| [U05] [AC03] | the update form's comment starts `''`; `readRating` never reads a comment | validated |
| [U05] [AC04] | Comment column removed from the Ratings list; the read model `Rating` has no `comment` | validated |
| [U05] [AC05] | Comment row removed from the rating details screen | validated |
| [U06] [AC01] | Delete and Delete All removed from the Ratings list | validated |
| [U06] [AC02] | `deleteRating` and `deleteAllRatings` removed from `RatingService` and `RatingListComponent`; no code path can send a `DELETE` | validated |

## Architecture & components

Every change is in `web/src/app/` (bookstore:web). The app stays an NgModule app with template-driven forms.

- **`ratings/rating.service.ts`** (changed): the storefront's one entry point to the ratings service. Five new methods, each a cold observable that emits exactly one outcome and completes, never errors, and carries the finished screen text: `give`, `change`, `loadRating`, `bookLine`, `catalogueLines`. Each applies the 8 s bound and classifies through `rating-outcomes.ts`. `createRating`, `updateRating`, `deleteRating` and `deleteAllRatings` are removed. `getRatingList`, `getRatingById`, `getRatingByEmail` and `getRatingByISBN` stay as they are, unbounded, because the Ratings list's and the rating details screen's loads are out of scope.
- **`ratings/rating-outcomes.ts`** (new): the outcome types, `RATINGS_BOUNDED_TIME_MS`, and the pure classifiers `classifyWrite`, `readableReason`, `classifyLoad`, `readSummary` and `readSummaries`. No Angular, no HTTP: the status and body rules are tested as tables.
- **`ratings/rating-texts.ts`** (new): the specification's verbatim texts as constants and the two rating-line formatters, `bookLineFor` and `catalogueLineFor`.
- **`ratings/rating.ts`** (changed): the read model `Rating` loses `comment`.
- **`ratings/star-choice/`** (new): `StarChoiceComponent` (`standalone: false`, declared in `AppModule`), a fieldset with a legend and five native radio inputs, bound two-way as `[(stars)]`.
- **`ratings/create-rating/`** (changed): local fields for the entries, the Submit gate, the in-flight flag, the outcome text in a `role="alert"` element, the two comment notes, `StarChoiceComponent` in place of the number input; no `alert()` or `console.log`.
- **`ratings/update-rating/`** (changed): the load state; heading "Update Rating"; e-mail and ISBN as plain text; the same Submit, outcome-text and notes behaviour as the rating form; `goToCartsList` renamed `goToRatingsList`.
- **`ratings/rating-list/`** (changed): the Comment column, Delete, Delete All and the `deleteRating` and `deleteAllRatings` handlers removed.
- **`ratings/rating-details/`** (changed): the Comment row removed.
- **`books/book-details/`** (changed): requests the book's summary on every load, renders the rating line below Vend once the book has loaded.
- **`books/book-list/`** (changed): requests the summaries once per screen open, renders a "Rating" column after "OnVend".
- **`app.module.ts`** (changed): declares `StarChoiceComponent`.

### Alternatives considered

The interface fan-out ran on `RatingService` (three takes; the chosen shape is a hybrid on take C, Interfaces).

- **Take A — minimise the interface** (one overloaded `call({op: …})` entry point, outcomes `{kind, text}`): lost on discoverability — a stringly `op` discriminator and overloads stand in for five named methods, and the one method grows with every request.
- **Take B — maximise flexibility** (data-only outcomes, an exported generic `Exchange`/`run()` layer, a text module every screen calls): lost on depth and locality — each screen learns four unions and a text module, a status-to-text change touches two files, and the `Exchange` seam has one real consumer, so it fails the two-adapters test. Its pure, exported classifiers were taken into the hybrid.
- **Take C as proposed**: taken, with three corrections — the classifiers exported for table tests (from B), a whole average shown as "★ 4.0" (C rendered "4", against [U03] [AC01] [TC02]), and `getRatingById` kept for the rating details screen.
- **Raw `RatingService` with a classifier call in each screen's error callback** (recorded candidate C2): struck — the status logic would live in four components, the pattern that today dereferences a null `error.error`.
- **An `HttpInterceptor` for the bounded time and the status mapping**: rejected — it would reach every service of the app, which has no interceptor and no precedent for one.
- **The star choice as radios copied into both templates, a `ControlValueAccessor`, or custom ARIA star glyphs**: rejected — copying lets the two forms drift on keyboard and accessibility; a value accessor adds plumbing nothing needs, since the Submit gate reads the component's fields, not `NgForm` validity; hand-written glyphs must reimplement the keyboard behaviour native radios already have.

## Interfaces / contracts

### The web's ratings request-and-outcome API

```ts
// ratings/rating-outcomes.ts
export const RATINGS_BOUNDED_TIME_MS = 8000;

export interface EnteredRating { email: string; isbn: string; rating: number; comment: string; }
export interface RatingToEdit { id: number; email: string; isbn: string; stars: number; }
export interface Summary { isbn: string; count: number; average: number | null; }

export type WriteKind = 'buyers-only' | 'invalid' | 'unavailable' | 'gone' | 'cannot-save';
export type WriteOutcome = { stored: true } | { stored: false; kind: WriteKind; text: string };
export type LoadOutcome =
  | { kind: 'loaded'; rating: RatingToEdit }
  | { kind: 'gone' | 'cannot-load'; text: string };
export interface RatingLine { kind: 'rated' | 'unrated' | 'unavailable'; text: string; }

export function classifyWrite(status: number, body: unknown, op: 'give' | 'change'): WriteOutcome;
export function readableReason(body: unknown): string | null;
export function classifyLoad(status: number, body: unknown, requestedId: number): LoadOutcome;
export function readSummary(body: unknown): Summary | null;
export function readSummaries(body: unknown): ReadonlyMap<string, Summary> | null;

// ratings/rating-texts.ts — verbatim from the specification
export const BUYERS_ONLY = 'Only customers who have bought this book can rate it.';
export const PURCHASE_CHECK_UNAVAILABLE = "We can't check your purchase right now. Nothing was saved. Please try again in a few minutes.";
export const CANNOT_SAVE = "We can't save your rating right now. Please try again in a few minutes.";
export const GONE_ON_CHANGE = 'This rating no longer exists. Nothing was saved.';
export const GONE_ON_LOAD = 'This rating no longer exists.';
export const CANNOT_LOAD = "We can't load this rating right now. Please try again in a few minutes.";
export const NOT_YET_RATED = 'Not yet rated';
export const RATING_UNAVAILABLE = 'Rating unavailable';
export const COMMENT_NOTE = 'Only Quillmark sees your comment. It is not shown to other shoppers.';
export const KEEP_COMMENT_NOTE = 'Leave this empty to keep any comment you gave before.';
export function bookLineFor(summary: Summary | null): RatingLine;
export function catalogueLineFor(summaries: ReadonlyMap<string, Summary> | null, isbn: string): RatingLine;

// ratings/rating.service.ts
@Injectable({ providedIn: 'root' })
export class RatingService {
  give(entry: EnteredRating): Observable<WriteOutcome>;
  change(loaded: RatingToEdit, stars: number, comment: string): Observable<WriteOutcome>;
  loadRating(id: string | number): Observable<LoadOutcome>;
  bookLine(isbn: string): Observable<RatingLine>;
  catalogueLines(): Observable<(isbn: string) => RatingLine>;
  // unchanged, unbounded: getRatingList(), getRatingById(id), getRatingByEmail(email), getRatingByISBN(isbn)
}

// ratings/rating.ts — the read model, no comment
export class Rating { id = 0; email?: string; isbn?: string; rating?: number; createdAt?: string; updatedAt?: string; }
```

What the types do not carry:

- **One emission, never an error, cancellable.** Each method returns a cold observable: every subscription sends one request and emits exactly one outcome. Unsubscribing cancels the request.
- **The bounded time.** Each request is `HttpClient` with `observe: 'response'`, then `timeout({ first: RATINGS_BOUNDED_TIME_MS })`. On expiry rxjs unsubscribes, `HttpClient` aborts the XHR, and the outcome is cannot-save, cannot-load or unavailable. A late answer therefore reaches no subscriber: nothing navigates, and no text changes. 8 s leaves about 4 s over a write's worst wait on orders inside ratings (1 s connect + 3 s read) and stays under the specification's 10 s.
- **Status and body, from either path.** A response and an `HttpErrorResponse` both reduce to `(status, body)` — an error's body is `error.error`, and a non-HTTP error (`TimeoutError`, a failed network) reduces to `(0, null)` — before the pure classifiers run. `error.error` is never dereferenced.
- **`classifyWrite`.** `200` → `{stored: true}`, whatever the body (an unreadable `200` body counts too). `403` → buyers-only, whatever the body. `400` with `readableReason(body)` → invalid, its text that reason verbatim. `503` → unavailable. `404` → gone when `op` is `'change'`, cannot-save when `'give'`. Anything else, `(0, null)` included → cannot-save.
- **`readableReason`.** A JSON object whose `message` is a string holding a character other than a space (`/[^ ]/`); returned untrimmed. Anything else → `null`.
- **`classifyLoad`.** `200` with a readable rating → loaded; `404` → gone; anything else → cannot-load. A readable rating is an object with an integer `id` equal to `requestedId`, non-empty string `email` and `isbn`, and an integer `rating` from 1 to 5; only those four fields are copied into `RatingToEdit`, so a comment in the read is never read ([ER2]).
- **`readSummary`.** An object with an integer `count` ≥ 0 and, when `count` > 0, a finite number `average` with at most one decimal place (`Math.abs(average * 10 - Math.round(average * 10)) < 1e-9`); with `count` 0 the average is not read. `isbn` is taken as a string where present.
- **`readSummaries`.** An array each of whose entries is a readable summary carrying a non-empty string `isbn`, no ISBN twice; anything else → `null`, so one unreadable entry makes the whole answer unreadable ([U04] [AC04] [TC06]).
- **Rating lines.** `bookLineFor`: `null` → unavailable `"Rating unavailable"`; count 0 → unrated `"Not yet rated"`; otherwise rated `"★ " + average.toFixed(1) + " (1 rating)"`, or `" (N ratings)"` for any other count. `catalogueLineFor`: a `null` map → unavailable for every ISBN; an ISBN absent from the map, or listed with count 0 → unrated; otherwise rated `"★ " + average.toFixed(1)` with no count. `toFixed(1)` only fixes the digits shown — the ratings service has already rounded the figure ([AD#6]), so `4` shows as `4.0` and `4.3` as `4.3`.
- **Request shapes.** `give` → `POST {base}` with exactly `{email, isbn, rating, comment}`. `change` → `PUT {base}/{loaded.id}` with `{id: loaded.id, email: loaded.email, isbn: loaded.isbn, rating: stars, comment}`, the id a JSON integer equal to the path's (ratings answers "bad rating id" to a string id). `loadRating` → `GET {base}/{n}` where `n = Number(id)` is an integer; otherwise it emits cannot-load without a request. `bookLine` → `GET {base}/summary` with `isbn` set through `HttpParams`. `catalogueLines` → `GET {base}/summaries`. `{base}` is `${environment.ratingsSrvUrl}/api/v1/ratings`, as today. No method sends a `DELETE`.
- **Side effects and repeats.** The three reads have none. A give or a change that times out may still have been stored by ratings; the cannot-save text does not say it was not, and repeating it is safe, since ratings applies a repeat give or change to the same single rating ([AD#4], the specification's glossary).

### `StarChoiceComponent`

```ts
@Component({ selector: 'app-star-choice', templateUrl: './star-choice.component.html', standalone: false })
export class StarChoiceComponent {
  @Input() stars: number | undefined;
  @Output() starsChange = new EventEmitter<number>();
  @Input({ required: true }) name!: string;   // the radio group's name, one per form
  readonly choices = [1, 2, 3, 4, 5];
}
```

```html
<fieldset>
  <legend>Rating</legend>
  @for (n of choices; track n) {
    <input type="radio" class="form-check-input" [id]="name + '-' + n" [name]="name" [value]="n"
           [checked]="stars === n" (change)="stars = n; starsChange.emit(n)">
    <label class="form-check-label" [for]="name + '-' + n">{{ n }}</label>
  }
</fieldset>
```

Plain `[checked]`/`(change)`, never `ngModel`, so the radios do not register with the parent `NgForm`.

### Contract (Epic `## Contract`, PRD-level ARD `## Contracts`)

bookstore:web consumes four ratings interfaces in the same repository; no row carries an artifact (`—`), so the web keeps its own consumer-side models (`Rating`, `RatingToEdit`, `Summary`), as the ARD's Schema ownership states. The web produces no boundary interface — no other component calls it — so it owes no producer-side test; the producer's tests are ratings' own ([[BOOK-2-01-01]] design, Test strategy).

- **Consumes [AD#4]** (changed) — `give` and `change` as above. Stub: `HttpTestingController` answering `200` with the saved rating; `400` with each reason, with a blank `message` and with Boot's default body (no `message`); `403` with the reason and with an empty body; `404`; `503` with "Service Unavailable"; `500`; a network error (status 0); no answer within 8 s; a `200` after 8 s.
- **Consumes [AD#5]** (changed) — the web calls neither removal. Check: after every screen flow, `HttpTestingController.match(r => r.method === 'DELETE')` is empty, and `RatingService` has no removal method.
- **Consumes [AD#6]** (new) — `bookLine` and `catalogueLines`. Stub: a summary with count above 0, with count 0 and `average` `null`, with count above 0 and no average, `500`, no answer within 8 s; summaries with an entry, with no entry for a book, with one unreadable entry, a non-array body, `500`, no answer within 8 s.
- **Consumes [AD#7]** (changed) — reads carry no comment. Stub: list and read responses that still carry a `comment`, as before this release, to show no screen renders it.

## Seams

| Seam | Component | Dependency category | Adapters |
|---|---|---|---|
| Ratings REST API — port: Angular's `HttpBackend`, reached through `RatingService`'s five methods | bookstore:web → bookstore:ratings | remote-but-owned | `HttpXhrBackend`, the default of `provideHttpClient()` (production); `HttpClientTestingBackend` from `provideHttpClientTesting()`, driven by `HttpTestingController` (tests) |
| Classifiers and rating-line formatters (`rating-outcomes.ts`, `rating-texts.ts`) | bookstore:web | in-process | none — tested as tables |
| `StarChoiceComponent` | bookstore:web | in-process | none — tested through its template |
| Books REST API — port: Angular's `HttpBackend`, reached through `BookService` (unchanged) | bookstore:web → bookstore:books | remote-but-owned | `HttpXhrBackend` (production); `HttpClientTestingBackend` driven by `HttpTestingController` in the book screens' specs |

The tests swap the adapter at `HttpBackend` and never `RatingService` itself: `RatingService` is the highest seam they drive through it, as the screen specs run each component with the real service over `HttpClientTestingBackend`, so a test case exercises the template, the component state, the bound and the classification together. The pure classifiers sit below it and earn their own seam only for table tests of the status and body rules.

## Data flow

**Rating form (`create-rating`).** Fields `email = ''`, `isbn = ''`, `stars: number | undefined`, `comment = ''`, `sending = false`, `outcomeText = ''`. `canSubmit` = `/[^ ]/` matches both e-mail and ISBN, `stars` is set, and `!sending`.

1. Submit (button `[disabled]="!canSubmit"`; `onSubmit` returns at once when `!canSubmit`) → `outcomeText = ''`, `sending = true`.
2. `give({email, isbn, rating: stars, comment})`, piped through `takeUntilDestroyed(this.destroyRef)`.
3. The outcome → `sending = false`; stored → `/ratings`; otherwise `outcomeText = o.text`. The entries are never touched.

**Update form (`update-rating`).** `load: LoadOutcome | undefined`, `stars`, `comment = ''`, `sending`, `outcomeText`.

1. `ngOnInit` → `loadRating(route.snapshot.params['id'])`. While `load` is `undefined` only the heading "Update Rating" renders.
2. Loaded → `stars = load.rating.stars`; the e-mail and the ISBN render as text, the star choice, the comment field (empty), the notes, the outcome element and Submit. Gone or cannot-load → `load.text` renders in place of all of them.
3. Submit (`[disabled]="sending"`) → `outcomeText = ''`, `sending = true`, `change(load.rating, stars, comment)`; then as the rating form, a gone outcome showing its text beside Submit with the form kept.

**Book page (`book-details`).** `bookLoaded = false`, `ratingLine: RatingLine | undefined`.

1. `ngOnInit` sends both at once: `getBookByIsbn(isbn)` as today, whose success also sets `bookLoaded = true`; and `bookLine(isbn)` (the route's ISBN), whose outcome sets `ratingLine`.
2. Below the Vend row: `@if (bookLoaded && ratingLine)` renders `ratingLine.text`; otherwise nothing.

**Catalogue list (`book-list`).** `lineFor: ((isbn: string) => RatingLine) | undefined`.

1. `ngOnInit` → `getBooks()` as today, and `catalogueLines()` once, whose outcome sets `lineFor`. Every later in-place `getBooks()` refresh — after the vend toggle, Delete, Delete All, Vend All and Unvend All — refreshes the books only ([ER1]).
2. Each row's Rating cell shows `lineFor ? lineFor(book.isbn).text : ''`.

**Ratings list and rating details.** Loaded as today; the Comment column and row, Delete and Delete All are gone.

**Every screen** ends each subscription to `RatingService` with `takeUntilDestroyed(this.destroyRef)`, where `destroyRef = inject(DestroyRef)`. `onSubmit` and `ngOnInit` are not injection contexts, so the form without an argument would throw `NG0203`. Leaving a screen therefore cancels its request in flight (Error handling).

State: the storefront keeps no rating state between screens and caches no figure; every screen open asks again.

## Error handling & edge cases

| Situation | The screen shows | State left |
|---|---|---|
| A give or a change answered `200`, its body readable or not | the Ratings list | — |
| `403`, whatever the body | the buyers-only text beside Submit | entries kept |
| `400` with a readable reason | that reason, verbatim and as plain text — markup shows as its characters | entries kept |
| `400` with no readable reason (blank or all-space `message`, Boot's default body, a non-JSON body) | the cannot-save text | entries kept |
| `503`, whatever the body | the unavailable text | entries kept |
| `404` to a give | the cannot-save text | entries kept |
| `404` to a change | "This rating no longer exists. Nothing was saved." beside Submit; Submit enabled again, and a repeat gets `404` again | entries kept |
| `500`, `201`, `204` or any other status; the ratings service unreachable (status 0) | the cannot-save text | entries kept |
| No answer within 8 s to a give or a change | the cannot-save text; the request is cancelled, so a late `200` neither navigates nor replaces the text | entries kept; ratings may have stored it, and a repeat is safe ([AD#4]) |
| The customer leaves a screen with a request in flight | — | the request is cancelled; nothing navigates afterwards |
| Enter in a text field while Submit is disabled | nothing is sent: a disabled default button blocks implicit submission, and `onSubmit` re-checks the gate | — |
| An e-mail or ISBN of only spaces | Submit disabled | — |
| An e-mail or ISBN holding a tab or other whitespace that is not a space | Submit enabled, as [U01] [AC14] reads; ratings answers `400` with its reason | entries kept |
| The update form's route id is not an integer | "We can't load this rating right now. Please try again in a few minutes."; no request sent | — |
| The load answers `200` with an empty or unreadable body, an id other than the one requested, or stars outside 1–5 | the cannot-load text in place of the fields and Submit ([ER2]) | — |
| The load answers `404` / `400` / `500` / nothing within 8 s | "This rating no longer exists." / the cannot-load text / the cannot-load text / the cannot-load text | — |
| A read that still carries a `comment`, as before this release | never rendered: no screen binds a comment, and `classifyLoad` copies none | — |
| A summary with a negative or fractional count, a count above 0 and no average, an average with two decimals, or an average sent as a string | "Rating unavailable" | — |
| The book itself cannot be loaded | no rating line, as today ([ER4]); the summary request still went out | — |
| Summaries: a non-array body, one unreadable entry, or one ISBN listed twice | "Rating unavailable" on every row | — |
| Summaries readable, a book's ISBN not listed | "Not yet rated" on that row | — |
| The catalogue refreshes its books in place after the vend toggle, Delete, Delete All, Vend All or Unvend All | the same Rating cells, from the screen-open answer ([ER1]) | no new summaries request |

## Test strategy

The runner is Karma with Jasmine (`ng test --watch=false`, `web/karma.conf.js`, ChromeHeadlessNoSandbox through the Playwright Chromium fallback). The baseline is green at 57/57 before any change. Every spec this change adds or rewrites (listed below) configures `provideHttpClient()` + `provideHttpClientTesting()`, `provideRouter([])` where a component injects `Router`, and `httpMock.verify()` in `afterEach`, so none of them sends a real request. The other screens' specs (carts, clients, orders, storage and the rest) stay as they are. `HttpTestingController` and `fakeAsync` are new to the repository: the existing specs are "should create" smoke tests. Each spec case is named after the test case it carries, for example `it('[U01][AC05][TC02] shows the rule whatever the body')`.

- **`ratings/rating-outcomes.spec.ts`** (new; in-process seam): tables over `classifyWrite` (each status × `give`/`change`, with bodies carrying a reason, a blank or all-space `message`, a non-string `message`, no `message`, `null`, a string), `readableReason`, `classifyLoad` (readable; id mismatch; string id; stars 0, 6, 3.5; empty e-mail; `null`), `readSummary` (4 / 4.3; 0 / `null`; 0 / 7; 4 / no average; −1; 1.5; 4.35; `"4.3"`) and `readSummaries` (readable; one bad entry; a duplicate ISBN; a non-array; an empty array).
- **`ratings/rating-texts.spec.ts`** (new): "★ 4.0 (37 ratings)", "★ 5.0 (1 rating)", "★ 3.5 (2 ratings)", "Not yet rated", "Rating unavailable", and the catalogue's line without a count.
- **`ratings/rating.service.spec.ts`** (rewritten; remote-but-owned seam, `HttpTestingController` as the in-memory adapter): each method's method, URL and exact body — the PUT's id a number equal to the path's, `isbn` encoded through `HttpParams`; exactly one emission per subscription; `tick(8000)` → the failure outcome with `req.cancelled === true`; status 0 through `req.error(new ProgressEvent('error'))`; a non-integer id sends no request; the service exposes no removal method.
- **`ratings/star-choice/star-choice.component.spec.ts`** (new): five `type="radio"` inputs with values 1–5 in one fieldset with a legend, sharing one `name`; none checked for `undefined`; the input value checked; a change emits `starsChange`.
- **`ratings/create-rating/create-rating.component.spec.ts`** (rewritten): every [U01] test case.
  - The Submit gate (empty fields, all-space fields, no star) and that no request is sent.
  - The exact body.
  - Each status → its text, including `403` and `400` with empty bodies, a markup reason shown as `textContent` with no `img` element in the DOM, and `500`/`404`/an empty `400` → cannot-save.
  - `tick(8000)` → the cannot-save text and a cancelled request, with `Router.navigate` never called afterwards.
  - Exactly one POST while in flight, and two POSTs after an outcome.
  - The text cleared while the second Submit is in flight.
  - The `role="alert"` element present before and after each outcome.
  - The two comment notes, and the entries kept.
- **`ratings/update-rating/update-rating.component.spec.ts`** (rewritten, `ActivatedRoute` stubbed with id `'7'`): every [U02] test case on the form.
  - The load: heading only while pending; e-mail and ISBN as text with no `input` for them; 3 stars checked; the comment empty when the read carries one.
  - Load outcomes: `404` → gone, while `500`, a `200` with an empty body and `tick(8000)` → cannot-load, with nothing rendered after a late answer.
  - The change body, with id 7 as a number and `Reader@Example.com` as loaded.
  - `404` to the change → gone-on-change, plus the [U01]-style outcome, in-flight and alert cases.
- **`books/book-details/book-details.component.spec.ts`** (rewritten): every [U03] test case.
  - The book and summary flushed in either order.
  - Nothing shown while the summary is pending.
  - Rated, unrated and unavailable lines, including `500`, a count with no average, and `tick(8000)` with a late flush ignored.
  - One `GET /summary` per created component, so two loads make two requests.
  - A book that fails to load → no rating line.
- **`books/book-list/book-list.component.spec.ts`** (rewritten): every [U04] test case.
  - The "Rating" header right after "OnVend".
  - Exactly one `GET /summaries` and no `GET /summary`, for 3 and for 50 books.
  - Rated, absent, count-0, one-bad-entry, non-array, `500` and `tick(8000)` cases.
  - Empty cells while pending.
  - A Vend refresh sends no second summaries request.
- **`ratings/rating-list/rating-list.component.spec.ts`** and **`ratings/rating-details/rating-details.component.spec.ts`** (rewritten): [U02] [AC01], [U05] [AC04], [AC05], [U06] [AC01] — no Comment column or row, no comment text even when the read carries one, no Delete or Delete All, Update, Details and New Rating present.
- **[U06] [AC02]**: every ratings screen spec ends with `expect(httpMock.match(r => r.method === 'DELETE')).toEqual([])`.
- **By hand, in the stage walk** ([ER3]; Observability): a give and a change made with the keyboard alone, reaching 1 and 5 with the arrow keys; and, with a screen reader running (NVDA on Windows or VoiceOver on macOS), two Submits with different outcomes, each announced without moving to the text.

The consumed interfaces' stubs model each failure the Contract section lists for them, as well as the success shapes; that the ratings service produces those answers is shown by its own producer-side tests ([[BOOK-2-01-01]] design, Test strategy), not here.

## Risks & mitigations

- **The new web against a ratings service that has not landed.** The summary endpoints would answer `400`, since today's `GET /{id}` takes `summary` and `summaries` as a non-numeric id (`ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:59-61`); every book would show "Rating unavailable", and no write would ever get `403`. Those requests still reach ratings, so trigger (1) would not catch it. Mitigation: the landing order (ratings, then web), the deploy precondition in Migration (`GET /summaries` answers `200`), and the live walk's book-page step (trigger (4)).
- **A write that timed out may have been stored.** The cannot-save text does not say it was not, and repeating it is safe, because ratings applies a repeat give or change to the one rating ([AD#4]). Accepted, as the specification's glossary states.
- **`summaries` grows with the catalogue.** One response carries every rated book, held in one `Map`. Trivial at this store's size; paging is out of scope (ARD, Deferred).
- **The catalogue joins on the ISBN string exactly.** A book whose books-service ISBN were written differently from the one ratings stores (hyphens, spaces) would show "Not yet rated". Accepted: ratings stores only 13-digit ISBNs, and the store's books carry 13-digit ISBNs.
- **Keyboard and screen-reader behaviour are not fully automatable.** Mitigation: the semantics are unit-tested, and the behaviour is checked by hand in the stage walk, which the web must pass before it goes live.
- **The web emits no signal of its own** (`web-noagent`, no real-user monitoring). Mitigation: the ratios against the books service's browser-originated calls (Observability), and the stage and live walks.
- **A `400` reason is server text shown in the page.** It may carry markup ([U01] [AC06] [TC04], [U02] [AC06] [TC03]). It is shown only by interpolation (`{{ }}`), which Angular escapes, never through `[innerHTML]` or `bypassSecurityTrust*`; the markup test cases check that the text appears as its characters and that no element is created from it.
- **A cached old bundle after the deploy** still offers Delete All and the Comment column for that session. Its Delete All gets `400` and its Comment column is empty — the deploy window the ARD already accepts. Its book page and catalogue call the books endpoints but never the summary ones, so the design assumes such sessions end within the first hour and reads the ratio triggers and the 24-hour checks only after it (Observability).
- No Target span: every change is in `web/`. No ARD deviation.

## Migration / rollout / backward-compatibility

- **No data, schema or configuration change.** The ratings and books URLs (`environment*.ts`) are unchanged.
- **Landing order** (ARD): orders (unchanged), then ratings, then web. In each environment, deploy the web image only after ratings' release steps are done there — the `ratings` table emptied, `uk_ratings_email_isbn` confirmed ([[BOOK-2-01-01]] design, Migration) — and `GET /api/v1/ratings/summaries` answers `200`.
- **Stage first.** The web image goes to the live store only after the stage walk (Observability) passes in stage.
- **Compatibility.** The web consumes [AD#4]–[AD#7] in place under `/api/v1/ratings`; there is no `/v2`. Between the ratings deploy and the web deploy, the old web runs against the new ratings with the effects the ARD accepts.
- **Rollback.** Redeploy the previous web image; ratings stays. The old web works against the new ratings with those same accepted effects. Where ratings itself must roll back too, the web goes first ([[BOOK-2-01-01]] design, Migration).

## Observability & release verification

- **Signals.** Dynatrace service metrics per endpoint, existing today (every service image but the web's is instrumented): for ratings, the request count, failure rate and response-code mix of `GET /api/v1/ratings/summary`, `GET /api/v1/ratings/summaries` (both appear with the ratings change, and on the live store only the web calls them), `POST /api/v1/ratings`, `PUT /api/v1/ratings/{id}` and `DELETE /api/v1/ratings/delete-all`; for books, the request counts of `GET /api/v1/books` and `GET /api/v1/books/find` **from browsers only** — requests whose caller is not a bookstore service, which arrive through the ingress. Carts and orders call both endpoints, and storage calls `/find`, server to server on every cart and purchase path (`carts/src/main/java/com/dynatrace/carts/controller/CartController.java:155-163`, `orders/src/main/java/com/dynatrace/orders/controller/OrderController.java:233-248`, `storage/src/main/java/com/dynatrace/storage/controller/StorageController.java:93-94`, `146-147`), so their unfiltered counts say nothing about the storefront. In `web/`, only the catalogue list calls `GET /api/v1/books` (its load and its refreshes) and only the book page calls `/find` (its load), so the browser-only counts are exactly those screens' requests. The web adds no signal: it ships as `web-noagent` with no real-user monitoring, and this change adds none.
- **Baseline.** Ratings' POST+PUT `400` share while the new ratings was serving the old web — from the new ratings' start to the web deploy, at most the last hour of it — read in Dynatrace by the release operator on the day of the deploy. Where that window held fewer than 20 writes there is no baseline: trigger (3) and the 24-hour `400` check are not read, and the stage walk's give and change and the live walk's refused give stand for them. The ratings counts and the browser-only books counts are compared with each other over the same window, so they need no baseline of their own. None can be read now: this run has no Dynatrace access, and the summary endpoints do not exist yet.
- **Blast radius.** Ratings — one `GET /summary` per book page open and one `GET /summaries` per catalogue open (signal: those endpoints' request count and response time). The browser — one summaries response per catalogue open, growing with the number of rated books. Ratings' write endpoints — the same rate of calls with a new body shape (signal: the POST+PUT `400` share). Books — unchanged calls.
- **Post-release checks** (live store, over the 24 hours starting one hour after the deploy, once old-bundle sessions have ended):
  - [U03] [AC01]–[AC06] — the `GET /summary` count at least 90% of the browser-only `GET /api/v1/books/find` count, and `GET /summary`'s failure rate 0%.
  - [U04] [AC01]–[AC06] — the `GET /summaries` count above 0 and no higher than the browser-only `GET /api/v1/books` count.
  - [U01] [AC03]–[AC08]; [U02] [AC03]–[AC09] — where there is a baseline and the 24 hours hold at least 20 writes, the POST+PUT `400` share no more than 10 percentage points above the baseline; and the stage walk's give and change, and the live walk's refused give.
  - [U06] [AC02] — no `DELETE /api/v1/ratings/delete-all` reaches ratings (the live store runs no load generator; the window starts after the old-bundle sessions, which may send one, have ended).
  - n/a — client-side behaviour with no runtime signal; these rest on Test strategy and the stage walk: [U01] [AC01], [AC02], [AC09]–[AC14]; [U02] [AC01], [AC02], [AC10]–[AC18]; [U05] [AC01]–[AC05]; [U06] [AC01].
- **Stage walk** (release operator, in stage after the web deploy there, before the live deploy; it gates the live deploy, Migration). Fixture: a stage client with a completed order for one known book, set up through the storefront's orders screens. Steps: with the keyboard alone, give that client's rating of the book from the rating form and see the Ratings list; with the keyboard alone, change it from the update form; the book's page shows "★" with its count, and an unrated book's shows "Not yet rated"; the catalogue's Rating column shows both; a give for an e-mail with no order shows the buyers-only text; with a screen reader running, two Submits with different outcomes are each announced; the Ratings list shows no Delete, Delete All or Comment, and the details screen no comment. An operator may then remove the fixture rating with `DELETE /api/v1/ratings/{id}`.
- **Live walk** (release operator, right after the live deploy; read-only but for one refused give, so no customer's rating is touched): a book's page shows its rating line — "★ …" where the book is rated, "Not yet rated" where it is not, never "Rating unavailable"; the catalogue's Rating column shows no "Rating unavailable"; a give for an e-mail with no order shows the buyers-only text and stores nothing; the Ratings list shows no Delete, Delete All or Comment.
- **Unavailable answers in a walk.** Ratings can answer a give or a change `503` whatever orders does ([[BOOK-2-01-01]] design, Interfaces), and orders may itself be down. In either walk, a give or a change that shows the unavailable text is repeated, up to three times, until it shows its expected outcome; the step fails only on any other text. Three unavailable texts in a row leave the step inconclusive: the operator checks ratings' and orders' health and repeats the walk later. An inconclusive step neither triggers a rollback nor passes the stage walk.
- **Alert / SLO impact.** No service-level objective is defined for the bookstore services in the repository, and no alert is added or retuned.
- **Rollback signal.** Read by the release operator in Dynatrace; any one of these triggers redeploying the previous web image (Migration): (1) over minutes 60–90 after the web deploy — once old-bundle sessions, which call the books endpoints but never the summary ones, have ended — the ratings `GET /summaries` count is 0 while the browser-only books `GET /api/v1/books` count is at least 10; (2) over the same minutes, the ratings `GET /summary` count is below 90% of the browser-only books `GET /api/v1/books/find` count, over at least 20 book pages; (3) within 30 minutes of the deploy, where there is a baseline and those minutes hold at least 20 POST+PUT requests, the ratings POST+PUT `400` share is more than 10 percentage points above the baseline; (4) any step of the live walk fails. Before (1) and (2) are read, the Migration precondition (`GET /summaries` answers `200`) and the live walk stand for them.

## Out of scope

- Any change outside `web/` — ratings, orders, books, the load generator, `common`, `exceptions`, `k8s`.
- What the Ratings list and the rating details screen show when they cannot be loaded, and how long their loads wait (specification, Out of scope): their getters stay unbounded.
- What the book page and the catalogue list show when the book or the books cannot be loaded (specification, Out of scope; [ER4]).
- Checking an e-mail's or an ISBN's format, trimming them, or rounding or averaging in the browser (specification, Out of scope).
- Averages anywhere but the book page and the catalogue list ([[BOOK-2-02]]); opening a book's ratings from its count ([CD#12]).
- The other screens' `alert(error.error.message)` handlers (about 30, outside the ratings forms).
- Real-user monitoring or an agent for the web, and an end-to-end browser test harness.
- Paging `summaries`.
- An accessible name for the "★" glyph beyond its visible text.

## Open questions

None.

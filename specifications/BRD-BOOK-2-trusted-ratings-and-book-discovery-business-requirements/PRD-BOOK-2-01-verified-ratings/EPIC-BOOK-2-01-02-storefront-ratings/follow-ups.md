# Follow-ups — BOOK-2-01-02

- [ ] Run the BOOK-2-01-02 stage walk (design.md § Observability) after the web deploy to stage and before the live deploy. It includes a keyboard-only give and change (arrow keys reaching 1 and 5) and a screen-reader check that two Submits with different outcomes are each announced ([ER3]). These cannot be automated in Karma.
- [ ] Run `/dev-workflows:implement BOOK-2-01-02` from repository `bookstore`: fix [SEC#1] (CWE-79) at `web/src/app/ratings/rating-list/rating-list.component.ts:26`, starting from `security/bookstore/scans/20261016T152258Z-d5dab21/patches/F1.patch` — raised by /security-workflows:scan

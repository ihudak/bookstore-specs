# Follow-ups — BOOK-2-01

- [ ] Publish the release note in [release-notes.md](release-notes.md) (Unreleased → Feature updates) wherever Quillmark's release notes are published — /release-notes writes into no docs repo, and the docs automation adds the note's metadata
- [ ] Send USoft's three answers (the removal path [VD#1], deleted orders [VD#5], the ratings list [VD#6]) to Quillmark's Head of E-commerce and Operations Lead before [proposal.md](proposal.md) goes out (its D3). This is a manual step outside /prd-proposal.
- [ ] Get Quillmark's agreement to emptying the live ratings table again at the deploy, which removes any rating given since 2026-10-12 ([proposal.md](proposal.md) D7). This is a manual step outside /prd-proposal.
- [ ] Run `/product-workflows:prd-ground BOOK-2-01 --rebaseline` to ground the client calls per orders lookup, a wrong orders URL read as an unknown customer, today's 404 for an unknown e-mail or book, a paid order moved to another customer or book, and the derivation matrix's purchase row ([SR#6]). [proposal.md](proposal.md) dropped these cost drivers for want of a verified finding.
- [ ] Export a design frame set for the rating forms, ratings list, book page and catalogue list, so design grounding can run. Without one, [proposal.md](proposal.md) had nothing to cite for that cost driver.
- [ ] Decide how [SM#3] and [SMC#1] are measured. Once [CG#10] is repaired the ratings table keeps no change timestamp, and refusals are only logged. Neither is estimated in [proposal.md](proposal.md).

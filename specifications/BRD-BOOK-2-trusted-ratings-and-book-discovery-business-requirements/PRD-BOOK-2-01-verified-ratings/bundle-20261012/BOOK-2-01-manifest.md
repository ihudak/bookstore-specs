# BOOK-2-01 — bundle manifest

Package BOOK-2-01, built 2026-10-12. Every file in this bundle is for review: no file is marked *not for
re-review*, because no other package is copied in. Find each file by searching the bundle folder for
its name; do not rely on folder paths.

## Documents

| File | What it is | For review? |
|---|---|---|
| `BOOK-2-01-customer-review-prompt-20261012.md` | The review prompt: paste it into your review agent. It is the entry point. | yes |
| `BOOK-2-01-quillmark-trusted-ratings-brd.md` | Your business requirements document, exactly as you sent it. | yes |
| `BOOK-2-01-brd-figures.md` | Our transcription of your Figure 1 sketch. | yes |
| `BOOK-2-01-brd-inventory.md` | The requirements we read out of your document for this package, one row per obligation. | yes |
| `BOOK-2-01-brd-defect-log.md` | The problems we found in the wording of your requirements, for the whole of BOOK-2. | yes |
| `BOOK-2-01-coverage-ledger.md` | Where each requirement in this package is handled. | yes |
| `BOOK-2-01-code-grounding.md` | What the store's code does today, requirement by requirement, with file-and-line evidence. | yes |
| `BOOK-2-01-design-grounding.md` | A note that no comparison against design frames was run, and why. | yes |
| `BOOK-2-01-baselines.md` | The commit the code was read at, and how it was verified. | yes |
| `BOOK-2-01-decisions.md` | The decisions we have taken ourselves, with the options weighed and our reasoning. | yes |
| `BOOK-2-01-customer-questions.md` | The questions only you can answer. | yes |

Image: `BOOK-2-01-book-page-wireframe.svg`, your Figure 1 sketch of the book page.

## Your files, by the names your documents use

Our documents refer to the files you sent by their path inside the requirements folder, such as
`source/quillmark-trusted-ratings-brd.md › § 3.1`. In this bundle every file carries the package key
at the front of its name instead:

| Path as the documents name it | In this bundle |
|---|---|
| `source/quillmark-trusted-ratings-brd.md` | `BOOK-2-01-quillmark-trusted-ratings-brd.md` |
| `source/assets/book-page-wireframe.svg` | `BOOK-2-01-book-page-wireframe.svg` |

**What you may not see in your own document.** Your document is kept exactly as you sent it, so its
Figure 1 still points at `assets/book-page-wireframe.svg`. In this bundle there is no `assets`
folder, so a viewer may show a broken image there. The picture is in this bundle as `BOOK-2-01-book-page-wireframe.svg`.

No file you sent was left out of this bundle.

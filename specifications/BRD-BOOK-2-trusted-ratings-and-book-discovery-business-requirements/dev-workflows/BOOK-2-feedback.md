---
type: dev-workflows-feedback
prd: BOOK-2
slug: trusted-ratings-and-book-discovery-business-requirements
---

## 2026-10-12 — /brd-intake — false-positive

```yaml
id: BOOK-2-brd-intake-reread-recommended-despite-reader-notes
date: 2026-10-12
command: /brd-intake
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: false-positive
impact: friction
```

**Friction:** On a first intake, Phase 3 relation 2 left four top-level sections without a row (About this document, Background and goals, Out of scope, Contacts). The set question's `<recommended>` placeholder resolved to "Re-read (Recommended)" because each section was plain prose that a re-read "can change". But brd-reader's own `notes` already said it had left those sections out on purpose, under the customer's own rule that only "must" sentences are requirements. The operator took the recommended re-read, about 7 minutes of an Opus dispatch. It returned different row splits and candidates, and left exactly the same four sections unaccounted for, which the operator then accounted for by hand. The cut that decides whether a re-read can change a section tests only for links the agent was not handed. It never considers a reason the agent itself stated for the gap.

**Suggested improvement:** In `commands/brd-intake.md` Phase 3, the paragraphs on the set question and on `<recommended>`, add a second "a re-read cannot change it" condition: brd-reader's `notes` record the section as deliberately excluded, for a stated reason that is a property of the document (such as the customer's own definition of a requirement). Resolve `<recommended>` onto "Account for each section and image" in that case, and show the agent's reason beside the question. Keep the default-to-re-read fallback where the notes give no reason. Check `agents/brd-reader.md`'s notes contract first, to confirm that exclusions reliably carry their reason.

## 2026-10-12 — /brd-intake — ambiguous-prompt

```yaml
id: BOOK-2-brd-intake-copy-carries-source-file-mode
date: 2026-10-12
command: /brd-intake
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** Phase 2 requires the source and its linked files to be copied "byte-for-byte" and says nothing about file mode. A copy that preserved permissions, from a vault mount whose files are mode 777, committed the customer's markdown document and its SVG under `brd/source/` as 100755 (executable). The bytes are identical, but the executable bit is noise in the specs repository's history.

**Suggested improvement:** In `commands/brd-intake.md` Phase 2's copy step, say that "byte-for-byte" covers file content only, and that the copy must not carry the source's permission bits. Copy without preserving mode, or normalise copies under `brd/source/` and `brd/source-external/` to 0644.

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

## 2026-10-12 — /brd-split — missing-capability

```yaml
id: BOOK-2-brd-split-bulk-offer-skips-instruction-placed-multi-slice
date: 2026-10-12
command: /brd-split
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: friction
```

**Friction:** On a `full` run with two slices, Phase 1.5 Step A placed all 19 rows from the operator's own instruction and Phase 2 confirmed the grouping. Even so, Phase 4 Step 1's uniform-answer offer could not fire, because its firing table requires exactly one standing slice. The text also states that with two or more slices it "does not fire at all" and that this is not to be generalised. The walk therefore asked 19 per-row questions, and every answer was the recommendation the instruction had already determined. Step 1's own rationale, that a choice list asked where the answer is already determined spends a turn on a formality, applies equally to this case.

**Suggested improvement:** Add a `full`-mode firing row for two or more slices. It applies where every row in the offered set was placed on a named slice by the Phase 1.5 reading and that slice was confirmed in Phase 2. Offer one confirmation that lists each row with its own target, so the operator's instruction supplies the targets and the run picks none. Keep fall-through to the walk on any unreadable answer, and keep rows the reading did not place in the per-row walk. The paragraph that rules this out has to be rewritten, not just extended, so treat this as a design decision.

## 2026-10-12 — /brd-split — ambiguous-prompt

```yaml
id: BOOK-2-brd-split-slice-key-second-prompt-unspecified
date: 2026-10-12
command: /brd-split
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** In Phase 4 Step 2, "Assign to a named slice" requires a second prompt for the slice key after the disposition picker, which doubles the prompts per row. The command does not say whether the key may be asked in the same `AskUserQuestion` call as the disposition, one row per call still. The orchestrator folded the two questions into one call. The bullet's key validation (reject an unknown, unreconcilable or orphan-row child) is written as a re-prompt after the answer, and it is unclear how that applies to an answer given in the same call.

**Suggested improvement:** State whether the fold is allowed and how its key is validated. Alternatively, where Phase 1.5 placed the row, pre-fill the key from the placement as the recommended default through the existing `<recommended>` mechanism. That removes the second prompt without adding any batching rule.

## 2026-10-12 — /brd-split — missing-reference-doc

```yaml
id: BOOK-2-brd-split-slices-md-format-undefined
date: 2026-10-12
command: /brd-split
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-reference-doc
impact: friction
```

**Friction:** Phase 5 lists the blocks `slices.md` must carry but fixes no layout. There is no frontmatter, no block order and no headings. No reference in `product-workflows/references/` defines the file, while its siblings `coverage-ledger.md` and the inventory each have a fixed layout. `/brd-reconcile` corrects `slices.md` in place, and `/brd-package` and `bundle-packaging` read it. The orchestrator invented `kind: slices` / `key: <KEY>` frontmatter and its own headings.

**Suggested improvement:** Define `slices.md` in a reference: either a new `slices-format.md` or a section of `coverage-ledger-format.md`. Give it frontmatter (a `kind:` outside the folder kinds, and `key:`), one heading form per Phase 5 block type, and their order. Cite it from `brd-split` Phase 5 and from each reader. Otherwise, state explicitly that the layout is free-form.

## 2026-10-12 — /brd-split — other

```yaml
id: BOOK-2-brd-split-merged-local-branch-forces-suffix
date: 2026-10-12
command: /brd-split
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: other
impact: polish
```

**Friction:** The `/brd-intake` handoff branch `brd/<KEY>-<slug>` was merged, but its local ref was still present. The preflight stood on the default branch (row B1), so B2's cleanup never ran on it. `phase-handoff` §2.2 rule 4 then named this run's branch `brd/<KEY>-<slug>-2`. That is correct as written, but every later `/brd-*` handoff on the same BRD will also need a suffix until the stale ref is deleted by hand.

**Suggested improvement:** Have the preflight's §3.4 retry pass, which already lists every local plugin branch, delete any that `branch-merged` finds merged, using `branch -d`, never `-D`, as B2 does. Alternatively, note once in the handoff outcome that the suffix comes from a merged local branch and give the command to delete it.

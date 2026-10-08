---
type: dev-workflows-feedback
prd: BOOK-1
slug: back-in-stock-alerts
---

## 2026-10-07 — /create-prd — false-positive

```yaml
id: BOOK-1-create-prd-style-check-flags-contract-headings
date: 2026-10-07
command: /create-prd
plugin_version: 3.24.4
origin: auto
author: ivan.gudak@dynatrace.com
category: false-positive
impact: polish
```

**Friction:** In Phase 3.5, prose-style-checker (doc_type: prd) raised 3 NITs on spine headings: sentence-case 'User Stories' and 'Success Metrics', and replacing '&' in 'Assumptions & open questions'. workflows-core prd-format.md (lines 138, 141, 149) and pre-lint.md (lines 49–50) require those heading texts verbatim, so applying the NITs would make pre-lint report missing required headings. The checker (prose-style agents/prose-style-checker.md:164) has no exemption for format-mandated headings. The orchestrator had to decline them by judgement, and they still landed in the final report as outstanding NITs.

**Suggested improvement:** Treat the spine headings as contract strings. Either pass prose-style-checker an exemption list for doc_type prd (the pre-lint required-heading list), or have /create-prd Phase 3.5 drop heading-case and punctuation findings on required headings ("declined: contract heading"). Add one line to prd-format.md saying the spine heading texts are verbatim contract strings that a style finding never overrides.

## 2026-10-07 — /create-prd — missing-capability

```yaml
id: BOOK-1-create-prd-edits-after-pass-with-recs-unreviewed
date: 2026-10-07
command: /create-prd
plugin_version: 3.24.4
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** prd-reviewer returned PASS WITH RECOMMENDATIONS with 4 MAJOR findings, two of them AC-to-AC contradictions. Phase 4 proceeds on that verdict with no fix cycle. When the user chose to apply every finding (six through product decisions, with ACs merged and renumbered), the edited text shipped unreviewed. The only safeguard is the escalation-rules line saying the verdict predates the edits.

**Suggested improvement:** In Phase 4, when findings from a PASS WITH RECOMMENDATIONS verdict are applied and any was MAJOR or any edit changed an [AC#N], offer one re-review of the edited text before the handoff ("Re-run prd-reviewer on the edited text (once)"). Keep the existing "verdict names the version it was taken against" record when the user declines.

## 2026-10-07 — /create-prd — other

```yaml
id: BOOK-1-create-prd-push-u-readonly-config-noise
date: 2026-10-07
command: /create-prd
plugin_version: 3.24.4
origin: auto
author: ivan.gudak@dynatrace.com
category: other
impact: polish
```

**Friction:** In the handoff (workflows-core phase-handoff §2.5), 'git push -u origin <branch>' printed 'error: could not write config file .git/config: Device or resource busy' twice. The ai-containers image mounts .git/config read-only, so no upstream was recorded, but the push itself succeeded. Nothing in the family reads @{u}, but the raw error reads like a failed push and has to be investigated by hand on every handoff in that container.

**Suggested improvement:** In phase-handoff §2.5, say that a 'could not write config file' error from 'push -u' on a read-only .git/config is benign once the remote ref exists ('git ls-remote --exit-code --heads origin <branch>'). Either filter it with a one-line note that upstream tracking was not set, or push without -u, since no step relies on the upstream.

## 2026-10-07 — /create-ard — ambiguous-prompt

```yaml
id: BOOK-1-create-ard-optionality-advisory-small-undefined
date: 2026-10-07
command: /create-ard
plugin_version: 3.25.0
origin: auto
author: ivan.gudak@dynatrace.com
category: ambiguous-prompt
impact: polish
```

**Friction:** Phase 0 step 5 (product-workflows commands/create-ard.md:225-226) fires the optionality advisory "for a small single-repo item" and attaches the multi-component sentence to that advisory. "Small" is never defined. A single-repo PRD reaching six code components of one repository (6 user stories, 17 acceptance criteria) needed judgement to decide whether the advisory fires, and whether the multi-component note should be printed at all when it does not.

**Suggested improvement:** State the advisory's trigger as a threshold on the union gauge the step already defines, and make the multi-component sentence an unconditional branch: a single-repo PRD touching two or more code components is not small, makes no optionality offer, and says so in one line.

## 2026-10-07 — /create-ard — missing-capability

```yaml
id: BOOK-1-create-ard-dirty-tree-no-scan-as-is-option
date: 2026-10-07
command: /create-ard
plugin_version: 3.25.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** code-scanner returned DIRTY_TREE on a repository whose only uncommitted changes were tracked agent-session files (.remember/logs/*.log deleted, .remember/tmp/save-session.pid modified), rewritten continuously by another plugin's hooks. The scanner itself recommended a fourth path, retrying with switch_to_default_branch and pull off, since the scan is read-only and the default branch was already current. The 'Dirty working tree' escalation (workflows-core references/escalation-rules.md:274) offers only Stash / Skip / Cancel, so the user stashed and left a stash in the code repository that the run never restores.

**Suggested improvement:** Add a 'Retry without switching or pulling (scan the checkout as it stands)' choice to the 'Dirty working tree' rule and its /epics and /specify variants, recommended where the scanner reports the checked-out branch is the default branch at 0 ahead and 0 behind. Optionally, have code-scanner report a dirty tree consisting only of agent-session paths separately from user work.

## 2026-10-07 — /create-ard — other

```yaml
id: BOOK-1-create-ard-branch-d-readonly-config-noise
date: 2026-10-07
command: /create-ard
plugin_version: 3.25.0
origin: auto
author: ivan.gudak@dynatrace.com
category: other
impact: polish
```

**Friction:** The specs preflight's B2 (workflows-core specs-repo-git.md §3.5) ran 'git branch -d' on a merged plugin branch in a container whose .git/config is mounted read-only. git deleted the branch and exited 0, but printed 'error: could not write config file .git/config: Device or resource busy' and 'warning: update of config-file failed' because it could not remove the branch's config section. 1.30.0 documents this case for 'push -u' (phase-handoff.md §2.5, specs-repo-git.md §4 step 6) but not for 'branch -d'.

**Suggested improvement:** In specs-repo-git.md §3.5 B2, say that a config-write error from 'branch -d' is not a failure when it exits 0 and the branch ref is gone, and have the preflight report the deletion as done.

## 2026-10-07 — /create-ard — unfollowed-rule

```yaml
id: BOOK-1-create-ard-fix-cycle-uncited-claim
date: 2026-10-07
command: /create-ard
plugin_version: 3.25.0
origin: auto
author: ivan.gudak@dynatrace.com
category: unfollowed-rule
impact: friction
```

**Friction:** Rule: "Every 'as-is' claim cites a grounded file:line; no fabricated/uncited architecture." (product-workflows references/ard-format.md:78). What happened: during the Phase 5 fix cycle after an ard-reviewer BLOCK, the orchestrator added to '### Versioning and compatibility' the claim that ingest is the only caller of books' and clients' delete-all, without searching for callers. The web store and the ingest Deployment's init container also call both endpoints, and the re-review raised a new BLOCKER on that claim, spending the review cap. Why it was missed: the claim sat in the Contracts section rather than under Grounding findings, where the citation rule is applied by habit, and the fix cycle's inline edits have no step requiring evidence for what they add.

**Suggested improvement:** In /create-ard Phase 5's inline fix step, require that every factual claim a fix adds about the code, above all a universal one ('only', 'every', 'no other'), cites a file:line from the Phase 3 scan or a fresh search before it is written.

## 2026-10-07 — /create-ard — missing-capability

```yaml
id: BOOK-1-create-ard-manual-notes-findings-not-surfaced
date: 2026-10-07
command: /create-ard
plugin_version: 3.25.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** After the re-review stayed BLOCK, the escalation's 'Provide manual fix notes' option applied one bounded pass with no further review, as escalation-rules.md specifies. The re-review's 4 MINOR and 4 NIT findings, some needing architect input (the one-minute budget has no margin and a lost restock can be missed entirely; the five-minute cadence has no margin; ISBN reuse defeats a recovery path), went into the run's final report only. The ARD's pull request and the ARD itself carry no trace of them, so the next phase (/specify, /epics) builds on decisions with known unresolved gaps.

**Suggested improvement:** On the manual-notes path, record the unaddressed re-review findings where the next phase reads them, for example under the ARD's '## Open questions', or at least in the handoff pull-request body. Optionally offer a re-review scoped to the edited decisions and contract rows.

## 2026-10-07 — /specify — missing-capability

```yaml
id: BOOK-1-specify-fix-cycle-contradicts-existing-criteria
date: 2026-10-07
command: /specify
plugin_version: 3.26.2
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** Both BLOCKERs in spec-reviewer's re-review were contradictions that the Phase 6 fix cycle itself introduced:
- A new AC ("show subscriptions unavailable when stock cannot be read") hid an action that two existing TCs still performed.
- Two new ACs triggered on any "alerts service does not answer" and contradicted two other new ACs.

The fix cycle grew the spec from 52 ACs and 128 TCs to 65 ACs and 161 TCs. Its one re-review was spent finding these contradictions, so the final text went out under manual fix notes and unreviewed. This is the second time on this PRD, after /create-ard (id BOOK-1-create-ard-fix-cycle-uncited-claim).

**Suggested improvement:** In /specify Phase 6 step 2, before the single re-review, run a required self-check of the fix-cycle diff. List every AC added or changed and the trigger or state it introduces. Then grep the existing ACs and TCs whose preconditions that trigger covers, and flag any whose outcome differs. Optionally add a matching advisory item to the pre-lint spec block for duplicate or overlapping trigger phrases. Also tell the user that the re-review is the last one when the fix grows the spec substantially.

## 2026-10-07 — /specify — missing-reference-doc

```yaml
id: BOOK-1-specify-defer-breaks-open-question-count
date: 2026-10-07
command: /specify
plugin_version: 3.26.2
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-reference-doc
impact: friction
```

**Friction:** The two count authorities disagree once a BLOCKER is deferred:
- /specify Phase 6 "Defer" (~/.claude/plugins/cache/shipwright/product-workflows/3.26.2/commands/specify.md:928) appends a `## Refinement notes` section with a `- [ ]` item per deferred finding.
- workflows-core pre-lint (~/.claude/plugins/cache/shipwright/workflows-core/1.31.2/references/pre-lint.md:74-75) requires `- **Open questions**: N` to equal every `- [ ]` item in the file, and makes a mismatch MAJOR.
- product-workflows specification-format.md:39-40 defines N as the `- [ ]` items "across all 'Open questions' sub-headings", which leaves refinement notes out.

So a deferred BLOCKER either trips pre-lint or breaks the format's definition. This run did not trigger it, because the user chose manual fix notes.

**Suggested improvement:** Pick one rule and state it in both files. Either have specification-format.md count every `- [ ]` item in the file, refinement notes included, or have pre-lint's spec block skip the `## Refinement notes` section.

## 2026-10-07 — /specify — unfollowed-rule

```yaml
id: BOOK-1-specify-block-escalation-cites-epics-entry
date: 2026-10-07
command: /specify
plugin_version: 3.26.2
origin: auto
author: ivan.gudak@dynatrace.com
category: unfollowed-rule
impact: polish
```

**Friction:** The cited rule and the command's own array contradict each other:
- /specify Phase 6 (~/.claude/plugins/cache/shipwright/product-workflows/3.26.2/commands/specify.md:923) escalates per the "Review verdict BLOCK (unresolved after one fix cycle) — /epics" rule.
- escalation-rules.md:469-470 says /specify "cites this entry on purpose".
- That entry's array (escalation-rules.md:463, :468) says "Defer to a follow-up issue (record in Phase 9 report)". But /specify's own inline array (specify.md:926) says "(record in the final report)", and /specify's Phase 9 is its cost phase.

The orchestrator presented /specify's own array. /specify fixes inline with no delegated writer, so it matches the "commands that fix inline" entry (escalation-rules.md:447-452), not /epics'.

**Suggested improvement:**
- In specify.md:923, cite the "commands that fix inline" entry, whose array already says "final report".
- Add a /specify clause there for its `## Refinement notes` Defer.
- Delete escalation-rules.md:469-470.

## 2026-10-07 — /specify — missing-capability

```yaml
id: BOOK-1-specify-open-findings-invisible-to-epics
date: 2026-10-07
command: /specify
plugin_version: 3.26.2
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** /specify Phase 6 (specify.md:930-931) sends unresolved MAJOR, MINOR and NIT findings only to the final report. spec-reviewer's re-review raised, as a MINOR, that product-input gaps are "not recorded anywhere in the spec (Open questions: 0)", so /epics cannot see them.

The re-review left 2 MAJOR, 13 MINOR and 5 NIT findings open, four of them product-input gaps. The orchestrator wrote them into the PR body as a workaround, which no later command reads. This has the same root as id BOOK-1-create-ard-manual-notes-findings-not-surfaced.

**Suggested improvement:** State where an unresolved finding is recorded so the next phase can see it. For example, record each one as a `- [ ]` under the relevant `Open questions` heading, with the header count updated, or under the Refinement notes section once that count rule is settled. Alternatively, align spec-reviewer's expectation with the final-report-only rule.

## 2026-10-07 — /epics — unfollowed-rule

```yaml
id: BOOK-1-epics-requirement-series-omit-smc
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: unfollowed-rule
impact: friction
```

**Friction:** The rule is in `workflows-core:prd-format` (`~/.claude/plugins/cache/shipwright/workflows-core/1.32.0/references/prd-format.md:175`): "Epics' `## Covers`, `/epics`' `_coverage.md` … cite a requirement by its id … `[US#N]`, `[AC#N]`, `[SM#N]`, `[SMC#N]`, `[UC#N]`, `[FR#N]`". `/epics` Phase 0 (the `EPICS_PRD_NO_REQUIREMENTS` test, `commands/epics.md:270-276`) and Phase 3 (`requirements[]`, `:467`) enumerate only `[US#n]/[AC#n]/[SM#n]/[UC#n]/[FR#n]`. `pre-lint.md:85` likewise lists only `[US#N]/[AC#N]/[SM#N]` for `## Covers`. On BOOK-1 the PRD's counter-metric `[SMC#1]` would have been left out of the coverage ground truth. The orchestrator included it as type `metric` by judgement. The rule was missed because the command's own list is narrower than the format authority and contradicts it.

**Suggested improvement:** In `epics.md`, replace the hard-coded series lists with "every series `workflows-core:prd-format` § Changing a requirement lists". Bring `pre-lint.md`'s Epic `## Covers` line in line with the series its PRD block already names.

## 2026-10-07 — /epics — missing-capability

```yaml
id: BOOK-1-epics-open-findings-invisible-to-specify
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** `/epics` ended with 12 MINOR and 7 NIT `epic-reviewer` findings open. At least four need a decision the run did not take:
- alerts' dev and Kubernetes ports, which four Epics depend on;
- whether alerts joins ingest's per-service config and version fan-out;
- the measurement source for `[SM#1]`;
- a criterion that checks the user-chosen Delete All captions.

They reach only the `/epics` final report. The rule "A finding left open that needs a decision is recorded in the artifact" (`escalation-rules.md:479`) is used by `/create-prd`, `/update-prd`, `/create-ard` and `/specify`, but not `/epics`. The Epic template has no open-questions section, so `/specify <EPIC>` will not see these findings. This is the same defect class as id BOOK-1-specify-open-findings-invisible-to-epics, one phase later.

**Suggested improvement:** Add `/epics` to that rule's users. After the last review, write each decision-needing open finding into the affected Epic under a conditional `## Open questions` section. Add the section to `epic-writer`'s template and to `pre-lint`'s Epic block, and have `/specify` read it when it resolves an Epic.

## 2026-10-07 — /epics — false-positive

```yaml
id: BOOK-1-epics-prelint-epic-frontmatter-collision
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: false-positive
impact: polish
```

**Friction:** `pre-lint`'s auto-link collision check says "For Epic files, scan the entire file (the template has no frontmatter)" (`pre-lint.md:34`, repeated at `epics.md:710`). `epic-writer` now writes `kind`/`key`/`target` frontmatter (`workflows-core:addressing` §4), so the line `key: BOOK-1-0N` of every Epic matched `\b[A-Z]{2,10}-[0-9]+\b`, giving 6 false positives on one run.

**Suggested improvement:** Scan Epic files below the frontmatter, as for the PRD and ARD, and delete the stale "no frontmatter" premise in both places.

## 2026-10-07 — /epics — wrong-output

```yaml
id: BOOK-1-epics-prelint-placeholder-regex-misses-punctuation
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: wrong-output
impact: friction
```

**Friction:** `epic-writer` left the placeholder `<alerts' dev port>` in three Epics (BOOK-1-02, -03, -06). `pre-lint`'s universal placeholder regex `<[a-z][a-z0-9 _./-]*>` (`pre-lint.md:18`) does not match it because of the apostrophe, so pre-lint passed it. `epic-reviewer` later raised the unfixed alerts port as a MINOR.

**Suggested improvement:** Widen the pattern to something like `<[a-z][^<>\n]*>`, or add a second pattern that tolerates punctuation, with a fixture containing `<alerts' dev port>`. Separately, have `epic-writer` raise a value the ARD defers to design as a `[NEEDS CLARIFICATION]` or a dependency, never as an angle-bracket token.

## 2026-10-07 — /epics — wrong-output

```yaml
id: BOOK-1-epics-clarification-suggestion-unchecked
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: wrong-output
impact: friction
```

**Friction:** In Phase 6.1, `epic-writer`'s suggested answer to its own clarification (the BOOK-1-05 Delete All confirmation captions) misstated `[AD#11]`: "clears every other client's subscriptions and alerts", whereas a reserved subscription is one whose ISBN *or* email is reserved. The orchestrator presented it as offered and the user accepted it. Nothing in Phase 6.1 checks a suggested answer against the ARD or PRD it cites. `epic-reviewer` later raised the wording as a MAJOR, and the user had to decide the captions a third time.

**Suggested improvement:** In Phase 6.1, have the orchestrator check each suggested answer against the `[AD#N]`/PRD ids it touches before offering it, and label any it could not check as unverified in the prompt.

## 2026-10-07 — /epics — missing-capability

```yaml
id: BOOK-1-epics-style-fix-overwrites-user-answer
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** Phase 6.2 hands every MAJOR `prose-style-checker` violation to `doc-fixer`. One MAJOR targeted exactly the caption text the user had chosen in Phase 6.1 moments earlier. `/epics` has no guard against overwriting user-resolved text. The orchestrator asked the user before letting the fixer change it, which the command does not say to do.

**Suggested improvement:** Carry a `user_resolved` list (the text Phase 6.1 folded in) into Phase 6.2 and Phase 7. A style or review finding on that text becomes a question to the user, never a silent fixer edit.

## 2026-10-07 — /epics — docs-ux

```yaml
id: BOOK-1-epics-style-check-baseline-dialect-noise
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: docs-ux
impact: polish
```

**Friction:** Phase 6.2 ran `prose-style-checker` on its vendor-neutral baseline (no overlay in the specs repo). It raised 19 NITs, mostly noise:
- American-English spelling (catalogue, cancelled, behaviour, initialised) against a specs tree whose PRD, ARD and spec are all British English;
- a spaced em dash in every file.

Its re-check also contradicted itself: it said the line-24 captions had no violations, then raised a NIT on them.

`fixed_headings` is built from `pre-lint`'s required Epic headings (`epics.md:682`), so it omits the conditional `## Contract` heading `epic-reviewer` reads.

**Suggested improvement:** In Phase 6.2, pass the tree's dialect or skip the dialect and dash rules where no overlay is configured, and say so in the report. Build `fixed_headings` from `epic-writer`'s full template heading list, `## Contract` included.

## 2026-10-07 — /epics — manual-workaround

```yaml
id: BOOK-1-epics-paste-full-output-slots
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: manual-workaround
impact: polish
```

**Friction:** Several dispatches tell the orchestrator to "paste" a prior agent's full output, for example:
- the scanner outputs into the `epic-writer` handoff and the `epic-reviewer` brief;
- the style-checker output into `doc-fixer` (`epics.md:695`);
- the requirements array into the review brief.

On BOOK-1 that was about 25KB of scanner YAML plus a 99-row requirements array. The orchestrator extracted each agent's final message from its task transcript with a script and passed it by scratchpad path instead.

**Suggested improvement:** Let every "paste" slot take a file path. Name a scratchpad location for each intermediate artifact (scanner output, requirements, ARD parse, review output, survivor list) so the handoff is a path, not a copy.

## 2026-10-07 — /epics — missing-reference-doc

```yaml
id: BOOK-1-epics-landing-order-producer-after-consumers
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-reference-doc
impact: polish
```

**Friction:** The ARD's `### Landing order` puts `bookstore:ingest` last, but ingest produces the `[AD#11]` shared-schema row that alerts, books and clients consume. That conflicts with `epic-writer`'s contract rules, under which a consumer names the producing Epic in `## Dependencies`. So BOOK-1-01, -03 and -04 depend on the later BOOK-1-06. That is legal and fixture-tested, but `epic-reviewer` warned that trackers will show cycles. Neither `/epics` nor the components reference says what to do when the ARD's landing order disagrees with its own contract rows.

**Suggested improvement:** Have `ard-resolution` or `/epics` Phase 2.5 test the landing order against the contract rows. Where a producer lands after a consumer, say so in the plan and the final report, and point at an ARD refine to reorder or split the shared-schema row.

## 2026-10-07 — /epics — missing-capability

```yaml
id: BOOK-1-epics-ard-contract-omission-handling
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: polish
```

**Friction:** The ARD's Contracts table has no `exists` rows for three cross-component calls the Epics use:
- storage→web (the book page reads stock);
- clients→web (the Clients pages);
- ingest's use of books' and clients' create calls (raised by `epic-reviewer`).

`/epics` has no step for a cross-component call missing from the table. The orchestrator told `epic-writer` ad hoc not to invent `[AD#N]` rows for these calls.

**Suggested improvement:** Add a Phase 2.5 step that lists the cross-component calls the code scan shows but the Contracts table lacks. Pass them to `epic-writer` and `epic-reviewer` as known omissions, and name `/product-workflows:create-ard <PRD>` (refine) in the final report to add the rows.

## 2026-10-07 — /epics — missing-capability

```yaml
id: BOOK-1-epics-preflight-stale-merged-branches
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: polish
```

**Friction:** `specs-preflight` §3.5 B2 deleted the merged `spec/BOOK-1-back-in-stock-alerts` that HEAD stood on, but the merged local `idea/BOOK-1-back-in-stock-alerts` from an earlier run is still present. §3.4's retry walks other plugin branches only to push them, and no row deletes a merged plugin branch HEAD is not on. So merged plugin branches accumulate locally across a pipeline (idea → prd → ard → spec).

**Suggested improvement:** In §3.4's "retry every other local plugin branch" walk, delete (`branch -d`, never `-D`) each plugin branch `branch-merged` finds merged into `<default-ref>`, with the same benign read-only-config handling B2 has, and report it.

## 2026-10-07 — /epics — missing-capability

```yaml
id: BOOK-1-prompt-epic-drafts-not-handed-off
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: prompt
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** `/epics` wrote six `epic.md` drafts and `_coverage.md` into the PRD folder and left them uncommitted. By design it "never branches and never commits the Epic drafts", and no other command hands them off. Its terminal `commit-artifacts` step still pushed the run's session artifacts straight to `main`. So the user saw a fresh `/epics` commit on the remote, but the Epics existed only in the local working tree.

Every earlier phase (`/idea`, `/create-prd`, `/create-ard`, `/specify`) ends by offering a branch, a commit, a push and a PR. `/epics` is the one pipeline phase whose deliverable never reaches the remote. Its only mentions are a `### Git state` paragraph and a follow-up line. Meanwhile `/specify <EPIC>` gates only `prd.md`, so it would run against an Epic that is not on `main`, and its handoff stages `specification.md`, `_session.md` and `_glossary.md` but not `epic.md`.

**User prompt:** but where are the epic markdown files???

**Resolution:** I committed the six `EPIC-BOOK-1-0N-*/epic.md` drafts and `_coverage.md` on branch `epics/BOOK-1-back-in-stock-alerts`, pushed it, and opened PR #5 to `main`. I then switched the specs checkout back to a clean `main`.

**Suggested improvement:** Give `/epics` the same consent-gated `handoff-to-main` every other phase has (an `epics/` or `epic/` prefix added to the plugin branch set), staging the `EPIC-` folders and `_coverage.md`. Alternatively, make `/specify <EPIC>` gate `epic.md` with `require-on-main` and stage it in its own handoff. In either case, stop reporting the run as handed off while its deliverable exists only locally.

## 2026-10-07 — /epics — missing-capability

```yaml
id: BOOK-1-prompt-epics-no-commit-rule-stale-from-vault
date: 2026-10-07
command: /epics
plugin_version: 3.28.0
origin: prompt
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** `/epics` left its six Epic drafts uncommitted and unpushed, while the same run pushed its bookkeeping to `main` and printed `Specs repo: committed … — pushed`. That line reads as if the run's output reached the remote.

The rule behind it is a holdover. The first `/epics` (2b3602d56, 2026-06-28, then `/impl:jira:epics`) wrote Jira Epic drafts into the user's Obsidian vault (`$VAULT_PATH/jira-drafts/`). There, "never branches, never commits … Vault git hygiene is the user's responsibility — they may or may not have the vault under version control" was correct.

The specs-native pipeline design (2026-08-31, `git show 62e791e8:docs/superpowers/specs/2026-08-31-specs-native-pipeline-design.md`) moved the output to `EPIC-<PRD-KEY>-NN-<eslug>/epic.md` inside the PRD folder of `$SPECS_PATH`, a git repo the plugin manages and hands off for every other phase. The rule was carried over with "vault" replaced by "write target", so `epics.md:18` still says "they may or may not have it under version control", which can no longer be true.

The workaround that followed is to say what it costs (`epics.md:1047`: the G1 advisory on every later run) rather than to hand the drafts off. There are two further gaps:
- `phase-handoff`'s plugin branch set (`idea|prd|ard|spec|design|ready|brd|frames|kb`) has no prefix an Epic handoff could use.
- `specs-repo-git` §2.1 calls `/epics` "the deliberate contrast", which is correct for the prompt-free bookkeeping commit but leaves no consent-gated path at all.

**User prompt:** explain why Epics were not committed and not pushed?

**Resolution:** I explained the cause with evidence: the vault-era no-commit rule (2b3602d56), carried unchanged through the specs-native move (62e791e8 design, `epics.md:18`, `:1047`, `specs-repo-git.md` §2.1), and the missing branch prefix. I also explained why only the bookkeeping was pushed: `commit-artifacts` stages only §2.1 paths. I changed no files; the drafts were already handed off by hand in PR #5.

**Suggested improvement:** Retire the vault-era sentence. Give `/epics` a Phase 11 consent-gated `handoff-to-main` (new `epic/` or `epics/` prefix in §2.2) staging the `EPIC-` folders and `_coverage.md`, run before `commit-artifacts` as `/specify`'s is. Until then, have the final `Specs repo:` line add "; the Epic drafts remain uncommitted (/epics never commits them)", as §6 already does for a declined handoff.

## 2026-10-08 — /design — missing-capability

```yaml
id: BOOK-1-design-kind-prefixed-address-rejected
date: 2026-10-08
command: /design
plugin_version: 1.35.2
origin: prompt
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** `/dev-workflows:design EPIC-BOOK-1-01` stopped in Phase 0 with `DESIGN_NEEDS_KEY: … 'EPIC-BOOK-1-01' is not a key (workflows-core:addressing §1)` (`dev-workflows` `commands/design.md:72`). The user read this as "the Epic was not found". In fact the folder `EPIC-BOOK-1-01-alerts-service/` exists, and the token is just that folder's kind prefix plus its valid key `BOOK-1-01`. The stop names neither the folder nor the bare key, and it does not say that `EPIC-` is a folder prefix rather than part of the ID.

Typing the prefixed form is the natural mistake. Every folder the family creates is named `<KIND>-<KEY>-<slug>` (`addressing.md` §2), so the prefixed form is what users see in the tree, in branch names and in PR titles. `resolve-address` (§3 step 2) sends any token that fails `key-valid` straight to `invalid`, with no recognition of a kind-prefixed key.

**User prompt:** @/workspace/specs/specifications/PRD-BOOK-1-back-in-stock-alerts/EPIC-BOOK-1-01-alerts-service/ you didn't find epic EPIC-BOOK-1-01? What then the ID of the Epic I reference to?

**Resolution:** I answered that the Epic's ID is `BOOK-1-01`, as asserted by its `epic.md` (`kind: epic`, `key: BOOK-1-01`). `EPIC-` is the folder's kind prefix and `alerts-service` is its slug. I confirmed that the folder exists and that an `@<path>` to it resolves (found, `kind: epic`, `key: BOOK-1-01`). I also explained that `/design` on either address still stops at `DESIGN_NO_SPEC`, because no Epic-level `specification.md` exists on `main` or any branch, until `/product-workflows:specify BOOK-1-01` lands one. I changed no files.

**Suggested improvement:** In `addressing.md` §3 step 2, before returning `invalid`, recognise a token of the form `<KIND>-<rest>`, where `<KIND>` is `BRD`, `PRD` or `EPIC` and `<rest>` passes `key-valid`. Only tokens that fail the grammar reach this step, so a genuine key beginning with a kind token (§2's `EPIC-008`) is unaffected. Resolve `<rest>` and accept it only where the found folder's name begins `<KIND>-<rest>-`. Print a one-line notice: `read EPIC-BOOK-1-01 as key BOOK-1-01 — EPIC- is the folder prefix, not part of the key`. If resolving is too permissive, have every `*_NEEDS_KEY` stop at least add `did you mean <rest>? (<KIND>- is the folder's kind prefix) — <matched folder>`, so the message never reads as "not found" when the folder exists.

## 2026-10-08 — /specify — missing-capability

```yaml
id: BOOK-1-specify-ard-conformance-at-gate
date: 2026-10-08
command: /specify
plugin_version: 3.29.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: friction
```

**Friction:** Two of the six BLOCKERs in the first `spec-reviewer` pass on `/specify BOOK-1-01` were ARD-conformance failures. The grill had settled a 100-book bound on the sweep and a case-insensitive reserved-email match with the user, but checked neither against the exact rule text of `[AD#5]` ("each ISBN with a pending subscription") or `[AD#11]` (a literal suffix that changes only through a new decision landed in four services). Phase 2.5 carries the ARD in as ground truth, and `specify.md` treats ARD rows as grill ground truth explicitly only for a multi-component PRD-level run. Phase 5 has no step that tests a settled decision against the governing `[AD#N]`, so the reviewer found both and spent an Opus round on them.

**Suggested improvement:** Add an ARD-conformance step to each Phase 5 confirmation gate, on Epic-level runs as well. For each decision settled in the stage, quote the governing `[AD#N]` rule text beside it. A decision that narrows or widens that text is either revised before the gate or recorded at once as an `ARD deviation` open question for the architect, not left for `spec-reviewer` to find.

## 2026-10-08 — /specify — missing-reference-doc

```yaml
id: BOOK-1-specify-grill-reread-governing-ad
date: 2026-10-08
command: /specify
plugin_version: 3.29.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-reference-doc
impact: friction
```

**Friction:** Besides the two ARD-conformance BLOCKERs, a third BLOCKER in the same review was a cross-story contradiction. The restock story's and the removal story's sweep criteria were each written against one dependency in isolation, so one situation (a removed book with a quantity of 0, an unpublished book with copies) got two different outcomes. `workflows-core`'s `grilling-technique.md` (`~/.claude/plugins/cache/shipwright/workflows-core/1.35.2/references/grilling-technique.md`) has no rule for either check: re-reading the governing decision when an answer fixes a bound or a match rule, or reconciling criteria that act on the same shared resource across stories.

**Suggested improvement:** Add two rules to `grilling-technique.md`'s Mechanics, at engineering altitude. First: once an answer fixes a bound, a match rule, a limit or an enumeration, re-read the exact text of the governing architecture decision before the confirmation gate. An answer that departs from it is a recorded deviation, never a silent override. Second: before the gate of a stage that writes a criterion acting on a shared resource (a periodic check, a table, an endpoint), list the other stories' criteria on that resource and reconcile them, so one situation never has two outcomes.

## 2026-10-08 — /specify — docs-ux

```yaml
id: BOOK-1-specify-post-verdict-edits-version
date: 2026-10-08
command: /specify
plugin_version: 3.29.0
origin: auto
author: ivan.gudak@dynatrace.com
category: docs-ux
impact: polish
```

**Friction:** After the re-review returned PASS WITH RECOMMENDATIONS, the review cap was spent and the user chose to apply the edit-only recommendations anyway. `escalation-rules`' "A recorded verdict names the version it was taken against" asks the final report to say so. The run also had to work out for itself where else to record it, `_session.md` and the pull-request body, so that the handed-off verdict did not read as covering the shipped text.

**Suggested improvement:** In `/specify` Phase 6 (and `/design`'s), state where the version qualifier goes beyond the final report: `_session.md` and the `handoff-to-main` body_facts, in a fixed form such as `verdict: PASS WITH RECOMMENDATIONS (taken before <n> later edits, unreviewed)`. The pull request a reviewer merges then carries the same qualifier as the report.

## 2026-10-08 — /specify — missing-capability

```yaml
id: BOOK-1-specify-scan-dependency-coupling
date: 2026-10-08
command: /specify
plugin_version: 3.29.0
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-capability
impact: polish
```

**Friction:** A fault-isolation test setup written in the test-case stage assumed the books service could be made to fail while a storage stock write still succeeded. Only a late manual check found that storage's stock writes call books (`verifyBook`), which forced two test cases to be redesigned after the review. The Phase 4 `code-scanner` brief asks for capabilities and gaps, but not for which service calls which on its write paths.

**Suggested improvement:** Have `/specify`'s Phase 4 `code-scanner` brief, and `/design`'s, ask for a short dependency-coupling list for the scanned repository: for each endpoint the work consumes or changes, which other services it calls, on reads and on writes. The grill can then check its fault-isolation setups, where one dependency fails while another succeeds, against that list before writing them.

## 2026-10-08 — /specify — manual-workaround

```yaml
id: BOOK-1-specify-large-spec-renumbering
date: 2026-10-08
command: /specify
plugin_version: 3.29.0
origin: auto
author: ivan.gudak@dynatrace.com
category: manual-workaround
impact: polish
```

**Friction:** The fix cycle renumbered whole stories' acceptance criteria in a 72–80 KB specification. The run rebuilt the affected sections in scratch files and spliced them in with string replacements, because exact-match edits do not cope with a renumbering across a story. `/specify` gives no guidance for assembling or renumbering a specification of that size.

**Suggested improvement:** Add a note to `/specify` Phase 5 and Phase 6 for specifications over about 60 KB: rewrite a renumbered story as a whole section, then re-run Phase 5.5's identifier and category checks. Optionally ship a small helper that renumbers `[ACxx]`/`[TCxx]` within a story and reports the mapping.

## 2026-10-08 — /design — unfollowed-rule

```yaml
id: BOOK-1-design-overlap-read-misses-operational-blast-radius
date: 2026-10-08
command: /design
plugin_version: 4.22.3
origin: auto
author: ivan.gudak@dynatrace.com
category: unfollowed-rule
impact: friction
```

**Friction:** One rule asks for every inline edit that answers a review finding to be read against the rest of the artifact that governs the same condition (`~/.claude/plugins/cache/shipwright/workflows-core/1.35.2/references/escalation-rules.md:449-475`, § An inline fix is read against what it overlaps). `/design` Phase 6 restates it as "each interface, seam and test-strategy line it adds or changes against those that govern the same behaviour" (`~/.claude/plugins/cache/shipwright/dev-workflows/4.22.3/commands/design.md`, Phase 6). After a PASS WITH RECOMMENDATIONS, the user chose to apply the findings. One fix added a namespace-wide `kubectl rollout restart deployment -n bookstore` to the design's rollout. That command would also restart the postgres, mysql and ingest Deployments, and ingest's sidecar calls `delete-all` on six services. The overlap read checked the spec and design text and missed it. The one re-review caught it as a MAJOR, the review cap was spent, and the MAJOR went into the pull request deferred. The rule was missed because neither list names an operational step (a rollout, delete, migration, exec or bootstrap) read against the resources its selector matches. The `/design` restatement also narrows the generic rule to interfaces, seams and tests.

**Suggested improvement:** Extend the list in `escalation-rules.md` § An inline fix is read against what it overlaps with this entry: "each operational command or manifest step it adds — a rollout, delete, scale, migration, exec or bootstrap — read against every resource its selector matches, stateful workloads and side-effecting sidecars included". Use this run's namespace-wide restart as the example. Widen `/design` Phase 6's restatement to match, so it covers fixes the user chose to apply after a non-BLOCK verdict too.

## 2026-10-08 — /design — missing-reference-doc

```yaml
id: BOOK-1-design-prelint-unscoped-kubectl
date: 2026-10-08
command: /design
plugin_version: 4.22.3
origin: auto
author: ivan.gudak@dynatrace.com
category: missing-reference-doc
impact: friction
```

**Friction:** `/design`'s structural pre-lint (`~/.claude/plugins/cache/shipwright/workflows-core/1.35.2/references/pre-lint.md`, design block) has no check for an unscoped operational command in a design's rollout or verification steps. A namespace-wide `kubectl rollout restart deployment -n <ns>` with no resource name went through pre-lint and used up the run's one re-review, which was spent finding it.

**Suggested improvement:** Add an advisory check to `pre-lint.md`'s design block. It greps `design.md` for `kubectl (rollout restart|delete|scale)` forms that carry only `-n`/`--namespace` or `--all`, with no resource name and no `-l` selector, and reports each as a MAJOR warning for the author before the Opus review. The defect class is fixed and grep-expressible.

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

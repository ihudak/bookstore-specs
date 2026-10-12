---
type: dev-workflows-feedback
prd: BOOK-2-01
slug: verified-ratings
---

## 2026-10-12 — /prd-ground — missing-reference-doc

```yaml
id: BOOK-2-01-prd-ground-baselines-layout-unspecified
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-reference-doc
impact: friction
```

**Friction:** `grounding/baselines.md` is read back by later runs (Phase 3's moved-pin test, Phase 8's *Supersede the earlier baselines*, `/brd-interview`'s repository lookup), but no reference fixes the layout of an entry. `workflows-core:grounding-format` §4 step 1 names the fields only in prose (slug, pinned commit, verification result, baseline `[CG#n]` id). The orchestrator invented a YAML-block entry with `repo`, `commit`, `verification`, `result` and `finding`, and another run could invent a different one.

**Suggested improvement:** Add a canonical `baselines.md` entry layout to `grounding-format` §4 (~/.claude/plugins/cache/shipwright/workflows-core/1.36.5/references/grounding-format.md, near lines 359–365). Fix the field names, say whether it is a YAML block or a table, and note that `/prd-ground` and `/brd-interview` parse it.

## 2026-10-12 — /prd-ground — ambiguous-prompt

```yaml
id: BOOK-2-01-prd-ground-tail-after-phase3-stop
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** A run stopped at Phase 3 with `PRD_GROUND_DIRTY_TREE`. `prd-ground.md` Phase 11 says it "runs after Phase 10, NEVER interrupts an earlier phase", and it does not say which tail members run after a Phase 3 refusal. `specs-repo-git` §4's *A run that refuses before it has written anything* is phrased for Phase 0 stops. The orchestrator applied it by analogy: a cost entry and `commit-artifacts`, with no feedback and no `resume.md`.

**Suggested improvement:** In `/prd-ground` Phase 11, state which tail members run after a stop at Phase 0, Phase 3 (`DIRTY_TREE`, `NEEDS_REBASELINE`, `AMBIGUOUS_PIN`) and Phase 7. Or widen `specs-repo-git` §4's rule to any stop taken before the run's first write, and name a Phase 3 stop as an example.

## 2026-10-12 — /prd-ground — ambiguous-prompt

```yaml
id: BOOK-2-01-prd-ground-instruction-files-as-evidence
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: friction
```

**Friction:** On an `extend` outcome, the verifier's derive step returned an evidence entry citing in-repo instruction files (`CLAUDE.md:79`, `.github/copilot-instructions.md:68`, `.kiro/instructions.md:68`) as the source of a false-friend name. Phase 4.5's "A document is never evidence for a [CG#n]" rule is worded around `$DOCS_PATH` pages. It does not say whether in-repo prose files count, so the orchestrator excluded the entry on its own judgement.

**Suggested improvement:** In `/prd-ground` Phase 4.5, say whether in-repo instruction and documentation files (`CLAUDE.md`, `AGENTS.md`, agent-instruction files, `README`, `docs/`) are documentation for the evidence ban. Say who filters them: `grounding-verifier`'s derive step, or the orchestrator before appending `extend` evidence.

## 2026-10-12 — /prd-ground — ambiguous-prompt

```yaml
id: BOOK-2-01-prd-ground-repo-choice-on-rerun
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** A re-invocation right after a Phase 3 stop, in the same session, reused the repository list the operator had picked at the stopped run's Phase 1 prompt. Phase 1 says to prompt for repositories every time, and nothing covers reusing an answer from a stopped run.

**Suggested improvement:** In `/prd-ground` Phase 1, state whether a repository choice from a stopped invocation in the same session may be carried into a rerun, provided it is reported, or must be asked again.

## 2026-10-12 — /prd-ground — missing-capability

```yaml
id: BOOK-2-01-prd-ground-brd-wireframe-not-in-census
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: polish
```

**Friction:** The slice has no `design/` folder, so the design pass was skipped. A slice requirement ([BR#11]) quotes a note from the BRD's own wireframe under the parent's `brd/source/assets/`. Nothing in the census or the next-step offer surfaces that this design source exists but is not an indexed frame set, so the gap is visible only if the orchestrator adds prose to `design-grounding.md` by hand.

**Suggested improvement:** When `design/` is absent, have Phase 8's census, or the next-step offer, name any design asset of the parent BRD that a slice row cites. Point to the route that makes it groundable, for example copying it under `design/<set>/` and running `/workflows-core:frames`.

## 2026-10-12 — /brd-split — wrong-output

```yaml
id: BOOK-2-01-brd-split-stale-tracking-ref-branch-suffix
date: 2026-10-12
command: /brd-split
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: wrong-output
impact: polish
```

**Friction:** The handoff branch was named `brd/BOOK-2-01-verified-ratings-2` when its intended name was free on the host. `refs/remotes/origin/brd/BOOK-2-01-verified-ratings` was a stale tracking ref: `git ls-remote` showed the branch had been deleted after PR #11 merged. `phase-handoff` §2.2 rule 1 tests the remote-tracking ref, rule 3 finds it merged, and rule 4 forces the `-2` suffix. The stale ref never clears, because the preflight's freshness fetch in `specs-repo-git` §3.2 is `git fetch origin <default>` with no prune. So every re-run after a delete-on-merge PR gets a needless suffix, and branch names drift from the folder name.

**Suggested improvement:** In `~/.claude/plugins/cache/shipwright/workflows-core/1.36.5/references/phase-handoff.md` §2.2, add a step before rule 1 trusts a remote-tracking ref with no local branch beside it. Check the host with `git ls-remote --exit-code --heads origin refs/heads/<name>`, the probe `specs-repo-git` §4 already uses. When the remote branch is gone, treat the ref as absent, optionally removing it with `git update-ref -d refs/remotes/origin/<name>`, so rule 2 takes the intended name. Keep today's behaviour when the host is unreachable, and say so on the §4.1 outcome line. The alternative is a never-fatal pruning fetch limited to the plugin prefixes in `specs-repo-git` §3.2, e.g. `git fetch --prune origin '+refs/heads/brd/*:refs/remotes/origin/brd/*'`, checked against §3.2's `origin/HEAD`-under-`--prune` note before it is adopted.

## 2026-10-12 — /brd-interview — unfollowed-rule

```yaml
id: BOOK-2-01-brd-interview-v-followups-batched
date: 2026-10-12
command: /brd-interview
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: unfollowed-rule
impact: friction
```

**Friction:** Rule: "Present each question **exactly one at a time, never batched**, via `AskUserQuestion`" (`~/.claude/plugins/cache/shipwright/product-workflows/3.30.4/commands/brd-interview.md`, Phase 6), with the argumentation prompt and the code-defect offer listed as prompts that follow each answer (*How no `[G]` reaches a human*, property 5). The run merged one `[V]`'s argumentation prompt and code-defect offer with the next `[V]`'s picker in a single `AskUserQuestion` call to save turns. The rule was read but taken to cover only the `[V]` picker itself, not the follow-up prompts.

**Suggested improvement:** In Phase 6, add one sentence saying that the argumentation prompt, the code-defect offer and the next `[V]`'s picker are each their own `AskUserQuestion` call and never share one. Alternatively, sanction the combination explicitly if the turn saving is wanted. Either way, the rule then reads the same for the picker and its follow-ups.

## 2026-10-12 — /brd-interview — missing-reference-doc

```yaml
id: BOOK-2-01-brd-interview-round-record-layout-unspecified
date: 2026-10-12
command: /brd-interview
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-reference-doc
impact: friction
```

**Friction:** `/brd-interview` pins individual labelled lines for `interview/round-<N>.md` (`generated against:`, `- **Re-puts:**`, `requirement defects:`, `code defects:`, `Status:`) and for `interview/customer-questions.md` (the `## Round <N>, question <position>` heading, `- **Findings:**`, `- **Altitude:**`, `- **Requirement defect:**`, `- **Defect image:**`). It gives no template or worked example for either file as a whole. The run had to improvise how each question's tag, row and state is written, for example a `- **State:**` line, and whether a `[C]` entry carries an `Options:` list. `/brd-package`, `/brd-reconcile` and a later resume read those states, so an improvised shape is a parse risk that only appears at the next command.

**Suggested improvement:** Add a reference such as `references/interview-round-format.md` with one worked round record (a `[G]`, a `[V]` and a `[C]` question, each with its state line) and one worked `customer-questions.md` entry, including how options are presented to the customer. Cite it from *Write the register and the round record* and *Hold every `[C]`*.

## 2026-10-12 — /brd-interview — missing-capability

```yaml
id: BOOK-2-01-brd-interview-unsourced-defects-no-closure
date: 2026-10-12
command: /brd-interview
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: polish
```

**Friction:** The `unsourced` requirement defects on this slice's rows ([DEF#2], [DEF#12], [DEF#15]) are raised by no question source, and the command states that no command writes their `resolved-by: <SLICE-KEY>/[CG#n]` resolution. Only a hand edit of the parent's `brd/brd-defect-log.md` closes them. Two of them are in fact settled by verified findings here ([DEF#2] by [CG#2], [DEF#15] by [CG#12]), but they stay `open` in the log the customer receives. The run's report has to say so in prose.

**Suggested improvement:** Have `/brd-interview` (or `/prd-ground`, which produces the finding) offer, on operator confirmation, to write `resolved-by: <SLICE-KEY>/[CG#n]` for an `unsourced` defect whose row's verified finding settles the presumption. Alternatively, have the final report list each such defect with the exact line to hand-write. A defect grounding cannot settle (such as [DEF#12], a monitor outside the repository) would stay open as now.

## 2026-10-12 — /brd-package — wrong-output

```yaml
id: BOOK-2-01-brd-package-grounding-notes-plugin-tokens
date: 2026-10-12
command: /brd-package
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: wrong-output
impact: blocker
```

**Friction:** `/brd-package` ships `grounding/code-grounding.md` and `grounding/design-grounding.md` whole, and its *Assemble the bundle* rule 7 hard-stops on any plugin token in a bundle document without sanitising. The grounding files `/prd-ground` wrote for this slice carry such tokens in plugin-written prose:
- the command name (`/product-workflows:prd-ground`, `/prd-ground Phase 4.5`);
- the agent name `grounding-verifier`;
- `§2.2`, `§3`, `§4.1` and `workflows-core:grounding-format §7` in verifier notes and the derivation-matrix intro;
- a `## <date> — /prd-ground` heading;
- the working filename `baselines.md`, which relation 3 of the citation check stops on.

So every slice grounded by this release is unpackageable until someone hand-edits a file `/prd-ground` owns. The run was stopped before the disposition walk and the wording was fixed by hand through a PR.

**Suggested improvement:** Have `/prd-ground` (and `grounding-verifier`'s notes) write the grounding files in customer-readable prose, stating rules by name rather than by `§`, naming no command, agent or working filename, as `brd-package-reviewer` is already told to. Or define an allowed rendering in `/brd-package`'s de-Obsidianising pass for plugin-written grounding notes. Either way, have `/brd-package` Phase 0 pre-scan the documents it will bundle, so the stop arrives before the operator disposes every `[SR#n]`, not after.

## 2026-10-12 — /brd-package — ambiguous-prompt

```yaml
id: BOOK-2-01-brd-package-fixed-admissibility-not-shown
date: 2026-10-12
command: /brd-package
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: friction
```

**Friction:** The Phase 4 disposition picker offers `fixed` for every finding. But `fixed` is admissible only where the target is one of the five artifacts the command may change (`~/.claude/plugins/cache/shipwright/product-workflows/3.30.4/commands/brd-package.md`, the disposition table's `fixed` row). In this run all 11 `[SR#n]` targeted artifacts outside the five: register records, question-set entries and the grounding derivation matrix. The operator picked `fixed` twice (SR#6, SR#10), and both times it had to be explained and re-asked. They also picked `escalated-to-customer` on SR#6, where the code settles the question, so the tagging test makes escalation inadmissible. That choice was re-asked too. The picker shows the operator nothing about which of the four values a given finding admits, so the closed vocabulary becomes three rounds of trial and error.

**Suggested improvement:** Keep the four-value array closed, but before each question compute admissibility from the finding's `target`. Print one line in the finding block, either "`fixed` is unavailable: <target> is written by <command or role>" or "`fixed` is available". Where `fixed` is unavailable, say in the question text that an agreed finding takes `accepted-risk` with a `fixed-by:` marker. Add a line noting that a finding the code settles is delivery-side and is not escalated.

## 2026-10-12 — /brd-package — unfollowed-rule

```yaml
id: BOOK-2-01-brd-package-finding-not-shown-before-question
date: 2026-10-12
command: /brd-package
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: unfollowed-rule
impact: friction
```

**Friction:** The rule: "Present each finding to the operator, one at a time, never batched, with its class, its target, its attack and its `what_would_settle_it` statement, and take exactly one disposition" (`~/.claude/plugins/cache/shipwright/product-workflows/3.30.4/commands/brd-package.md`, Phase 4). The SR#4 question was sent in the same turn as the previous finding's record update, before SR#4's class, target, attack and settle text had been shown. The operator answered blind. The run noticed, showed the finding and asked again; the second answer differed from the first, and that one stands. The rule was missed because it is one sentence above a long picker and table block. Nothing ties the display to the question, so a run that pipelines "record the last disposition, ask the next" can skip the display.

**Suggested improvement:** Make the finding block and the question one fixed template in Phase 4: a block rendering class, target, attack and what-would-settle-it, followed directly by the picker. State that the picker is never issued without that block immediately before it in the same message.

## 2026-10-12 — /brd-package — missing-capability

```yaml
id: BOOK-2-01-brd-package-no-shipped-bundle-scans
date: 2026-10-12
command: /brd-package
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: friction
```

**Friction:** The plugin-free scan, the citation-resolution check (rule 8) and the set-resolution check (rule 9) are specified only as prose; the plugin ships no tooling for them. The run wrote ad-hoc Python for all three, and for prompt and bundle assembly. The ad-hoc code went wrong twice:
- The relation-3 commit check matched the all-digit date `20261012` in the delivery note as a commit and reported a false mismatch, which only judgment caught.
- The first bundle build aborted on a line-wrapped string match and left a partial `bundle-<YYYYMMDD>/` that had to be removed by hand before the rebuild, since a dated bundle is never rewritten.

Each run reimplements the checks, so two runs can scan the same bundle differently.

**Suggested improvement:** Ship a deterministic scan script under `product-workflows/scripts/`. It should cover:
- the plugin-free classes, treating the `source/<file> › § n` locator position as exempt;
- relation 1 and relation 3 of the citation check, with partitioning and the qualified-field discharges;
- the three set relations, with commit tokens restricted to hex strings containing a letter or of 12 or more characters.

Have Phase 8 build into a temporary directory and move it to `bundle-<YYYYMMDD>/` only once assembly succeeds.

## 2026-10-12 — /brd-package — ambiguous-prompt

```yaml
id: BOOK-2-01-brd-package-rule1-prose-references
date: 2026-10-12
command: /brd-package
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** Phase 8 rule 1 says every reference from one bundle document to another names the bundled filename. The de-Obsidianise pass (rule 2) rewrites only links, though. Two plugin-written prose references pointed at sibling documents without naming them: code-grounding's "the grounding baselines document", and design-grounding's working path to the BRD wireframe. Whether a rendered copy may append the bundled filename to such prose is unspecified. The run appended `BOOK-2-01-baselines.md` and `BOOK-2-01-book-page-wireframe.svg` on its own judgment.

**Suggested improvement:** State in Phase 8 whether rule 1 covers prose references in plugin-written documents. If it does, name the pass that renders them (append the bundled filename beside the original words). If it does not, say the manifest's map is the only bridge.

## 2026-10-12 — /brd-package — ambiguous-prompt

```yaml
id: BOOK-2-01-brd-package-review-file-placement-unstated
date: 2026-10-12
command: /brd-package
plugin_version: 1.36.6
origin: prompt
author: ihudak@gmail.com
category: ambiguous-prompt
impact: friction
```

**Friction:** The customer review prompt that /brd-package writes (`BOOK-2-01-customer-review-prompt-20261012.md`, part 10) says to write exactly one new file, but never says where to save it. Neither the prompt nor the delivery note says the returned file reaches the specs repo through `/product-workflows:brd-reconcile <KEY> @<review-file>`. The prompt was run in the operator's own session (a demo, with the customer role-played). The agent offered the bundle folder as a save location and, at the user's pick, wrote the review into `bundle-20261012/`. That folder is reserved: `commit-artifacts` never stages it, and it holds the handed-off package that must stay byte-identical. The agent then left the file uncommitted, with no PR and no pointer to the reconcile step. The user expected it committed and PRed.

**Suggested improvement:** In the review prompt's part 10, and in the delivery note, say where the returned file goes. It is saved outside the bundle folder and outside the specs repository, then handed to the delivery team, who record it with `/product-workflows:brd-reconcile <KEY> @<file>`. That step copies it in as `customer-review-<YYYYMMDD>.md` and commits it with a PR. Never save it inside `bundle-<YYYYMMDD>/`. Where the prompt runs inside the specs checkout itself, tell the agent to save outside it and print the reconcile command for the operator, without running it.

**User prompt:** why not committed, not PRed?

**Resolution:** Explained that nothing was committed or PRed for three reasons: there was no commit request, recording the review is `/brd-reconcile`'s job, and `bundle-<date>/` is reserved. At the user's choice, moved the review out of `bundle-20261012/` to `/workspace/customer-returns/BOOK-2-01 Customer Review 20261012.md`, outside every repository. The reconcile command is left for the user to run.

## 2026-10-12 — /brd-reconcile — missing-capability

```yaml
id: BOOK-2-01-brd-reconcile-moot-held-questions-dead-end
date: 2026-10-12
command: /brd-reconcile
plugin_version: 3.30.5
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: friction
```

**Friction:** One returned review both withdrew two requirements and declined their questions as moot. It withdrew 3.8 and the Figure 1 note, frozen as [CD#11] and [CD#12], and declined round 1, questions 26, 27 and 30 as "Not answered: moot". The *Confirm every candidate* phase routes a declared refusal to *Reject*, so those questions stay *held for the customer* and round 1 stays open. The one exception covers only a held entry whose `- **Re-puts:**` line names a withdrawn record. These entries carry no such line: their requirement went `rejected` in the ledger in the same run. The run therefore ends `advance_ready: no`, and the only re-entry offered to close the round is *Package again*. That would put three questions to the customer about requirements they have just withdrawn. No command path retires such a question.

**Suggested improvement:** Widen the *Reject* exception in *Confirm every candidate* and step 1 of *Freeze the customer decisions*. A held `[C]` whose every `[BR#n]` this run moved to `rejected` (or whose requirement a frozen `[CD#n]` withdrew) should close as *answered by the customer*. That closure names the withdrawing `[CD#n]` and records that nothing was frozen from the reply, as the withdrawn-`Re-puts:` case already does. It mints no `[CD#n]` and writes no "declined". In *Next steps*, never offer *Package again* when the only held questions belong to rejected requirements. Alternatively, add a "moot — requirement withdrawn in this review" decline reason to `references/customer-review-schema.md` §5, and say how the reader returns it.

## 2026-10-12 — /brd-reconcile — missing-capability

```yaml
id: BOOK-2-01-brd-reconcile-agent-authored-reasons-unflagged
date: 2026-10-12
command: /brd-reconcile
plugin_version: 3.30.5
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: friction
```

**Friction:** The returned review's section 1 states that its section 7 and 8 reasons were written by the customer's own AI review agent from an interview, while the answers are the customer's. A `[CD#n]`'s `argumentation` is defined as the customer's own reason, quoted (property 5, `decision-register-format.md` §2). The command has no anomaly class for a reason the review itself attributes to another author. The run surfaced it only as a free-form anomaly and froze 18 records quoting those reasons, and nothing in the register shows that the voice is not the customer's.

**Suggested improvement:** Add an anomaly class to *Ingest the review* for "reasons authored by someone other than the customer, as section 1 states". Require one operator confirmation before the walk, and write a note into the reconciliation record naming every `[CD#n]` whose argumentation it covers. Optionally offer the missing-reason picker's *Ask the customer for the reason* for every candidate it touches.

## 2026-10-12 — /brd-reconcile — ambiguous-prompt

```yaml
id: BOOK-2-01-brd-reconcile-sr-answer-option-mapping
date: 2026-10-12
command: /brd-reconcile
plugin_version: 3.30.5
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** An answered escalated `[SR#n]` records the fixed pair `["as the package states", "as the finding argues"]`. Customers answer such a finding with a fact ("Settled: the load generator never runs live") rather than by siding with either. Mapping onto the pair, or to the outside-the-options form, took judgement on all six answers, and three were frozen outside the options. The command gives no rule for when a fact-settling answer takes which form.

**Suggested improvement:** In *Confirm every candidate*, say how a factual answer to an `[SR#n]` maps onto the pair. Name the pair member when the fact confirms the package's premise or the finding's *what would settle it*, and the outside-the-options form when it decides something neither side argued. Give one worked example of each.

## 2026-10-12 — /brd-reconcile — ambiguous-prompt

```yaml
id: BOOK-2-01-brd-reconcile-section12-bundle-filenames
date: 2026-10-12
command: /brd-reconcile
plugin_version: 3.30.5
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** The review's section 12 rows name the bundle's filenames (`BOOK-2-01-decisions.md`, `BOOK-2-01-customer-questions.md`, `BOOK-2-01-code-grounding.md`). That is correct from the customer's side, since those are the names they were sent. *Apply the required corrections* classifies targets by working-tree name, and never says to map a bundle name back through the manifest's map. The run mapped each one by hand.

**Suggested improvement:** In *Apply the required corrections*, state that a row naming a bundle file is resolved to its working artifact through `bundle-<YYYYMMDD>/`'s manifest. Map the bundle copy to the working file it was rendered from: `<KEY>-decisions.md` to `decisions.md`, `<KEY>-brd-defect-log.md` to the parent's `brd/brd-defect-log.md`, and so on. Report a name the manifest does not list as an anomaly.

## 2026-10-12 — /create-prd — ambiguous-prompt

```yaml
id: BOOK-2-01-create-prd-route-template-fails-pre-lint
date: 2026-10-12
command: /create-prd
plugin_version: 3.32.1
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: friction
```

**Friction:** The BRD-route grill tells the run to record a contradiction with a `[CD#n]` and name its route as `/product-workflows:brd-reconcile <SLICE-KEY> @<review-file>`. The same wording appears again for the `[AS#n]` triage and in the final report (`~/.claude/plugins/cache/shipwright/product-workflows/3.32.1/commands/create-prd.md`, Phase 3). Written into `prd.md`, that string fails Phase 3.6's own pre-lint twice:
- `<review-file>` matches the placeholder scan, a BLOCKER;
- the slice key inside inline code matches the auto-link check, also a BLOCKER, since inline code is not exempt.

The run had to reword the route as "on this slice" and drop the key, so the PRD no longer carries the exact command.

**Suggested improvement:** Choose one fix:
- say in `create-prd.md` how the in-PRD route is written (the bare command in a code span, with the key and review file named in words); or
- exempt the canonical route template from `workflows-core:pre-lint`'s placeholder and auto-link checks, with a fixture that holds the string.

## 2026-10-12 — /create-prd — missing-reference-doc

```yaml
id: BOOK-2-01-create-prd-ac-grouping-unspecified
date: 2026-10-12
command: /create-prd
plugin_version: 3.32.1
origin: auto
author: ihudak@gmail.com
category: missing-reference-doc
impact: polish
```

**Friction:** `workflows-core:prd-format` asks for `[AC#N]` "under each story" in a separate `## Acceptance Criteria` section, but never says how criteria are grouped there. The run repeated the `### [US#N]:` headings under that section. Pre-lint then read each story as defined twice and raised duplicate-id BLOCKERs. The fix, bold `**[US#N] title**` labels, was found only by reading another PRD (BOOK-1).

**Suggested improvement:** In `~/.claude/plugins/cache/shipwright/workflows-core/1.38.1/references/prd-format.md`, add one line with an example under the Acceptance Criteria spine bullet: group criteria under a bold `**[US#N] title**` label, never a `### [US#N]:` heading, because the heading is the story's one definition. Have pre-lint's duplicate message name the same fix.

## 2026-10-12 — /create-prd — docs-ux

```yaml
id: BOOK-2-01-create-prd-phase1-confirm-no-choices
date: 2026-10-12
command: /create-prd
plugin_version: 3.32.1
origin: auto
author: ihudak@gmail.com
category: docs-ux
impact: polish
```

**Friction:** Phase 1 step 1 says to "Confirm" the folder, profile and seed, but gives no choice array. The run improvised "Confirm both / Correct something". When "Correct something" came back twice with no notes, it took three question rounds to learn that the correction was the profile, `--hybrid`.

**Suggested improvement:** Give Phase 1 step 1 a `choices` array that names the usual corrections, for example `["Proceed (Recommended)", "Change the profile (--full / --hybrid / --lean)", "Cancel"]`. The free-text escape stays for anything else.

## 2026-10-12 — /create-prd — missing-capability

```yaml
id: BOOK-2-01-create-prd-contradicted-cd-already-happened
date: 2026-10-12
command: /create-prd
plugin_version: 3.32.1
origin: auto
author: ihudak@gmail.com
category: missing-capability
impact: polish
```

**Friction:** Mid-grill, the operator reported that a follow-up meeting had reversed two customer-signed decisions (the go-live clean-up, `[CD#2]` and `[CD#3]`). Later they reported that the reversed outcome had already been carried out in production. The command covers a contradiction the run believes, and routes it through `/brd-package` and `/brd-reconcile`. It says nothing about:
- a decision the customer already took outside the register (a meeting);
- one whose outcome has already happened.

The run had to improvise:
- the PRD carries the register's version;
- the meeting and the production fact are logged under Assumptions & open questions;
- the operator's position, which changed three times, was pinned down in follow-up questions.

**Suggested improvement:** Add to the BRD-route contradiction paragraph:
- when the operator reports a customer decision taken outside the register, offer stop-and-reconcile first, since `/brd-reconcile`'s free-text mode can read meeting notes;
- when the contradicted outcome has already happened, record it as a fact the customer must be told, not as an open preference;
- log only the operator's final position.

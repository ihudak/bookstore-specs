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

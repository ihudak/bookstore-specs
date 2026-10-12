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

---
type: dev-workflows-feedback
prd: BOOK-2-02
slug: book-discovery
---

## 2026-10-12 — /prd-ground — missing-reference-doc

```yaml
id: BOOK-2-02-prd-ground-vacuous-surface-verdict
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-reference-doc
impact: friction
```

**Friction:** Several BRD premises are rules about a surface that does not exist at the pinned commit: "a not-for-sale book is never suggested" (BR#13), "never shown on the Top rated shelf" (BR#14), "the shelf counts only purchasers' ratings" (BR#17), "suggestions and shelf reflect new data within a day" (BR#18). The code-grounder classed all four AMENDED ("true only because the surface does not exist"). The blind derive classed BR#13/14 AMENDED and BR#17/18 REWRITTEN. Compare correctly returned `contradict` on BR#17/18, which the procedure rewrote in place. So the written corpus now carries both verdicts for the same shape of premise, decided by whichever side ran last. `workflows-core:grounding-format` §3 defines AMENDED as "partly true" and REWRITTEN as "materially wrong", and has no rule for a premise that is vacuously true because its subject does not exist.

**Suggested improvement:** Add one paragraph after the §3 verdict table (~/.claude/plugins/cache/shipwright/workflows-core/1.36.5/references/grounding-format.md, around line 317) that fixes the verdict for a rule about a non-existent surface, with BR#13/14 against BR#17/18 as the worked pair. Have the code-grounder and grounding-verifier contracts cite it, so a classification-only AMENDED/REWRITTEN difference is settled by the rule and not by the contradict branch.

## 2026-10-12 — /prd-ground — missing-reference-doc

```yaml
id: BOOK-2-02-prd-ground-repo-instruction-files-as-docs
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: missing-reference-doc
impact: friction
```

**Friction:** The blind derive for BR#19 (FALSE-FRIEND) cited the code repository's own `CLAUDE.md:79` as the source of the decoy name ("synthetic load generator"). `/prd-ground`'s standing rule says no `[CG#n]` may cite "a documentation page" in its evidence, but nothing says whether a repo-local instruction file (`CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, a README) counts as one. The orchestrator excluded the entry on extend, under the conservative reading. Sibling slice BOOK-2-01 made the same call independently, so two runs have now had to decide it. `grounding-verifier`'s derive step never receives the docs digest, so it reaches for these files on its own.

**Suggested improvement:** Name repo-local instruction and README files explicitly in the "A document is never evidence for a `[CG#n]`" section of `/prd-ground` (~/.claude/plugins/cache/shipwright/product-workflows/3.30.4/commands/prd-ground.md) or in `grounding-format`, and carry the same exclusion into the `code-grounder` and `grounding-verifier` input contracts, so the agents don't offer such files as evidence in the first place.

## 2026-10-12 — /prd-ground — ambiguous-prompt

```yaml
id: BOOK-2-02-prd-ground-preflight-key-set-order
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: ambiguous-prompt
impact: polish
```

**Friction:** `/prd-ground` Phase 0 runs `specs-preflight` at step 4, before `resolve-address` at step 5. `workflows-core:specs-repo-git` §3 says a command that reads its key set from resolution resolves first, and one whose key set is what `read-key` reads off its arguments runs the preflight first. The command does not say which of the two it is, so the orchestrator had to reconcile the texts itself. It used the `read-key` key as the run key set.

**Suggested improvement:** Add one sentence to Phase 0 step 4 of `/prd-ground` (~/.claude/plugins/cache/shipwright/product-workflows/3.30.4/commands/prd-ground.md) saying its run key set is the key `read-key` read at step 1, so it takes `specs-repo-git` §3's preflight-first order.

## 2026-10-12 — /prd-ground — docs-ux

```yaml
id: BOOK-2-02-prd-ground-handoff-reask-outcome-line
date: 2026-10-12
command: /prd-ground
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: docs-ux
impact: polish
```

**Friction:** At Phase 9 the user first picked "Just write the files — I'll handle git", then interrupted ("stop. ask that again") and picked the branch-and-PR option on the re-ask. The *Declined by the user* outcome line had already been printed, so the transcript carries two contradictory `Phase handoff:` lines for one offer. The handoff record does not say why the user interrupted, so the evidence that the command caused it is thin.

**Suggested improvement:** Optionally, have `phase-handoff` §4.1 say that a re-asked consent choice supersedes the outcome line already printed for the same offer. The Final report then repeats only the last one.

## 2026-10-12 — /brd-split — wrong-output

```yaml
id: BOOK-2-02-brd-split-stale-tracking-ref-branch-suffix
date: 2026-10-12
command: /brd-split
plugin_version: 3.30.4
origin: auto
author: ihudak@gmail.com
category: wrong-output
impact: polish
```

**Friction:** This is the second occurrence in one session. The handoff branch became `brd/BOOK-2-02-book-discovery-2`, although the intended name was free on the host. `refs/remotes/origin/brd/BOOK-2-02-book-discovery` was a stale tracking ref: the remote branch had been deleted when PR #12 merged. `phase-handoff` §2.2 rule 1 tests that ref, rule 3 finds it merged, and rule 4 forces the `-2` suffix. The sibling slice's run minutes earlier hit the same case (PR #13 on `brd/BOOK-2-01-verified-ratings-2`, logged in BOOK-2-01's feedback). So it now reproduces on every slice whose earlier PR merged with delete-branch-on-merge, which is the ordinary GitHub setup.

**Suggested improvement:** In `~/.claude/plugins/cache/shipwright/workflows-core/1.36.5/references/phase-handoff.md` §2.2, have rule 1 confirm a remote-tracking ref against the host when no local branch backs it. Use `git ls-remote --exit-code --heads origin refs/heads/<name>`, the probe `specs-repo-git` §4 step 5 already uses, and treat a ref the host no longer has as absent. As an alternative, prune stale tracking refs before the collision test. Then use the `-2` suffix only when the name exists live on the host or as a local branch.

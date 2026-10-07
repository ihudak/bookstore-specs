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

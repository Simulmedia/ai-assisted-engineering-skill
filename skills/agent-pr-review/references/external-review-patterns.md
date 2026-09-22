# Public review workflows considered

Checked 2026-09-21. Selection: substantial GitHub adoption or a specialist
engineering organization, plus inspectable review instructions. Star counts
below are repository-level snapshots from the GitHub API, not review benchmarks
or endorsements of every rule. These are original adaptations of ideas, not
vendored upstream skills or new plugin dependencies.

| Repository | Stars at check | Inspected source and useful practice |
| --- | ---: | --- |
| [obra/superpowers](https://github.com/obra/superpowers) | 289,529 | [Reviewer template](https://github.com/obra/superpowers/blob/5bf4e78011075bcfc0dc295f0724994cd123ee71/skills/requesting-code-review/code-reviewer.md): explicit requirements and base/head, read-only review, visibility into behavior left unassessed. |
| [anthropics/claude-code](https://github.com/anthropics/claude-code) | 147,380 | [Code-review command](https://github.com/anthropics/claude-code/blob/7974a70773fa229e4cc65aa1b356cc21f5c216c4/plugins/code-review/commands/code-review.md): separate candidate discovery and validation, eliminate duplicate comments, use exact code citations. |
| [trailofbits/skills](https://github.com/trailofbits/skills) | 7,191 | [Differential review](https://github.com/trailofbits/skills/blob/123037ec8aed26f0d86327cc39137ee5043e5deb/plugins/differential-review/skills/differential-review/SKILL.md): prioritize risk, inspect history of removed safeguards, trace affected callers and disclose coverage. |

Superpowers also has a [receiving-review skill](https://github.com/obra/superpowers/blob/5bf4e78011075bcfc0dc295f0724994cd123ee71/skills/receiving-code-review/SKILL.md)
that checks suggestions against the actual code before implementing them. Our
existing verify-before-fix loop already covers that principle.

## Adaptation decisions

- Add a distinct challenge pass for candidate findings; it need not dispatch
  extra agents. Report evidence strength separately from severity, without an
  uncalibrated numeric confidence score.
- Inspect history when a meaningful safeguard changes, and follow real consumers.
  A small diff can still affect money, tenancy, reported metrics or retries.
- Disclose important unassessed behavior and preserve the user's checkout.
- Record the model, effort and review snapshot so future evaluations are comparable.

## Rules not carried over

Do not automatically skip drafts: pre-PR review is a core use case here. Do not
assume tools always work or require a fixed parallel-agent fan-out. Do not limit
bugs to those failing for every input: data correctness often depends on a
specific, realistic input. Avoid minor/style findings and generic missing-test
blockers. Do not invent business requirements from a reviewer's intuition; missing
rules remain questions. Do not turn this business review into a full security
audit or adopt arbitrary caller-count thresholds as proof of impact.

The upstreams are useful design references. Their popularity does not establish
that these adaptations improve our results; the [pilot](evaluation.md) must test that.

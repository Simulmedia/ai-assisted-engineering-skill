# Review antipatterns

These are proposed review heuristics, not claims about past team incidents.

For code-level patterns, use [Yehor's anti-pattern catalog](code-antipatterns.md).
This file covers review-process mistakes; neither catalog is a reason to report nits.

| Antipattern | Better review behavior |
| --- | --- |
| Repeating the agent's PR summary as fact | Derive observed behavior from the diff and callers; verify the summary. |
| Explaining code without explaining the outcome | Connect a changed rule to an evidenced user, report, or data contract. |
| Guessing business meaning from names | Look for requirements or consumers; mark missing semantics as unknown. |
| Treating plausible impact as measured impact | Separate a supported consequence from an unmeasured risk; avoid invented percentages. |
| Trusting tests written from the same wrong assumption | Construct a counterexample from the contract and compare expected with actual. |
| Losing rows to an INNER JOIN or multiplying totals in a join | Check unmatched rows and cardinality at the intended grain. |
| Making scans cheaper by silently dropping required history | Confirm the allowed time window and preserve equivalent outputs. |
| Treating the writer's explanation as independent review | Give a fresh reviewer the raw requirements, snapshot and checks, without the writer's reasoning history. |
| Calling another pass in the same conversation a fresh context | Disclose context reuse; use a separate session when independent review is required. |
| Repeating review/fix loops until the budget disappears | Set a bounded number of cycles and stop on repeated unresolved findings. |
| Calling an unavailable or unexecuted check passed | Report not run, its reason, and the remaining uncertainty. |
| Treating a light review as merge approval | State coverage limits and required remaining checks. |

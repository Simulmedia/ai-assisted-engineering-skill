# Review output

Adapt this template to the PR. Keep the business summary short; include only
checks and risks relevant to the change.

## What changes

Intended outcome, observed before/after behavior, affected consumer, and the
practical consequence in plain language. Cite requirements and implementation.
State explicitly when downstream impact cannot be established.

## Scope and evidence

- PR and reviewed base/head SHAs; depth and inspected paths.
- Requested vs actual model/effort, or unknown when not exposed.
- Host and skill revision where exposed; settings evidence and fresh/reused context
  using [the run record](model-settings.md). Record additional reviewers separately.
- Missing sources, excluded paths, and any stale snapshot limitation.
- For related PRs: each repository/head, intended dependency order and combined
  coverage. Include a compatibility table when interfaces cross PR boundaries.

## Business rules

| Rule and source | Code/test evidence | Result |
| --- | --- | --- |
| Relevant acceptance criterion | Immutable location or executed check | Supported / contradicted / not verified |

Include a grounded Mermaid diagram here if it helps explain the changed flow.

## Findings

For each actionable finding: severity, location, concrete input/trigger, actual
versus expected behavior, business consequence, evidence and suggested next step.
Order by impact. Keep questions separate from demonstrated defects.
State whether evidence is reproduced or statically traced; leave unverified
concerns as questions. Consolidate duplicate symptoms with the same root cause.
Omit nits and speculative rare cases. Keep low-hanging, behavior-preserving
optimizations in a short optional section, with no major rewrites.

## Checked, not reported

One line per candidate the challenge pass rejected: what was suspected and
which evidence (test, caller guarantee, history, counterexample) closed it.
This is the cheapest signal that "no findings" means "looked and found nothing",
not "did not look". Omit only when there were no candidates.

## Efficiency and validation

Briefly describe evidenced performance concerns or say none were found in the
inspected scope. Label measurement suggestions as unverified.
List executed checks with outcomes separately from checks only inspected or
proposed. Do not report estimated runtimes as measurements.

## Decision for the human reviewer

Choose: **changes needed**, **needs more evidence**, or **no blocking findings
in reviewed scope**. Explain the basis and unresolved questions. This is a
recommendation, not an approval action or guarantee of correctness.

Explicitly identify the human decisions still needed: whether the change delivers
the intended business value and whether its architectural choice is appropriate.
Present the evidenced tradeoffs and unresolved requirements; do not mark these
decisions approved on the human's behalf.
List material behavior left unassessed and the reason, including access gaps or
scope limits. Do not list every irrelevant feature of the system.

## PR description shape

When asked to write or revise a PR description, use Plain Technical English:

- Problem and business outcome: what behavior was wrong or missing, and who uses it.
- Change: concrete before/after behavior and the architectural choice/tradeoff.
- Validation: checks actually run, observed results and unverified assumptions.
- Dependencies and limits: related PR order, remaining risks and human decisions.

For example: "The join counted each delivery row once per matching tag. Aggregate
delivery before joining tags so the report keeps one total per campaign. The
synthetic two-tag case returns 100 impressions instead of 200." Use such wording
only when the diff and executed check support it; this example is synthetic.
Avoid hype, vague completion claims and lists of files without an explanation of
their effect. Do not claim strict ASD-STE100 compliance.

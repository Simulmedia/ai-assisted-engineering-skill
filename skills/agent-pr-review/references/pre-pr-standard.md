# Pre-PR self-check default

A recommended default, not a mandatory gate.
Adapt the sequence to your workflow; keep existing repository gates and say what
the review covered. The human assesses business value and architecture; agents
check code and provide evidence. A clean automated report replaces neither.

Apply the self-check before opening a PR when practical. An early draft may be
opened with unfinished checks clearly listed; complete the applicable checks
before requesting human review.

## Before requesting human review

1. Link the business requirement and summarize what changes for its consumer.
   List unresolved product decisions instead of guessing them.
2. Inspect the full diff and remove unrelated changes. Record the head SHA and
   run the repository's applicable checks. Include a relevant business-rule
   example, especially for changed calculations, joins, eligibility or retries.
   Run Yehor's `declutter` skill or check [the code anti-pattern catalog](code-antipatterns.md);
   keep that cleanup separate from bug review and preserve trust-boundary validation. Describe the PR in
   Plain Technical English using [the description shape](review-template.md).
3. Have a reviewer use [agent-pr-review](../SKILL.md) from a fresh context.
   Provide raw requirements, base/head refs for all related PRs, repository instructions and test
   access. Do not seed the review with the writer's chain of reasoning or an
   instruction to confirm that the implementation is correct.
4. Fix confirmed findings and validate the changed behavior. Where the host
   supports independent sessions, use a fresh fixer with the current snapshot,
   requirements and actionable findings. Independence is about context, not
   necessarily a different model. If unavailable, disclose the limitation.
5. Re-review the resulting delta and affected business paths. Default to at most
   two review/fix cycles for the pilot; use another limit if the engineer sets it.
   Stop early if the same blocker persists, requirements conflict, or required
   evidence is inaccessible. Hand unresolved items to the engineer. Never weaken
   checks merely to produce a clean report.
6. Include the final review summary, actual check results, reviewed SHA and open
   questions in the PR handoff. A draft can retain unfinished checks when they
   are clearly identified. Follow the repository's existing readiness and owner
   approval rules before asking to merge.
   Ask the human to assess business value and the architectural decision using
   the report's concrete before/after explanation, diagram and tradeoffs.

This document describes the roles; it is not an automatic agent orchestrator.
Use supported session controls only within the current user's authorization.

## Team validation still needed

Collect each engineer's current author/reviewer setup, useful checks, recurring
misses, time constraints and one concrete PR example. Compare this proposal with
those observations, agree on the minimum checklist and loop limit, and pilot on
representative PRs. Record actual feedback and decisions in this PR or a linked
team document. Do not mark the story complete until both the self-check standard
and reviewer method are agreed and linked in the repository.

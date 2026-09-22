# Business-logic checks

Choose checks based on what the diff changes. Establish the intended rule from
the ticket, a maintained contract, or an explicitly confirmed decision. Existing
behavior is evidence of the baseline, not proof that it is correct.

| Change | What to verify | Useful small counterexample |
| --- | --- | --- |
| Aggregation or joins | Output grain, join cardinality, uniqueness, and totals match the contract. | One fact row with two matching dimension rows; compare count and amount before/after. |
| Filtering or eligibility | Included/excluded states, tenant/campaign boundaries, and permissions remain correct. | Two tenants share an external ID; one is outside the requested scope. |
| Dates or attribution windows | Time zone, inclusive/exclusive endpoints, late arrivals, event time vs processing time. | Event exactly at each endpoint and a late-arriving event. |
| Financial or delivery metrics | Units, denominator, precision, nulls, zero, negatives, currency and rounding where relevant. | Zero denominator; missing input versus a real zero; rounding across grouped rows. |
| ETL or retries | Idempotency, duplicate suppression, partial failure, checkpoints, and replay/backfill behavior. | Process the same batch twice; fail after writing but before checkpointing. |
| API/schema changes | Field meaning, type, nullability, defaults, removed values, and consumer compatibility. | Old caller with a missing new field; consumer receiving a newly nullable value. |
| Caching or optimization | Cache keys and lifetime preserve scope and freshness; outputs stay equivalent. | Same entity ID in different tenants; rerun after source data changes. |

Write a tiny input/expected/actual example for important findings. Mark invented
fixture values as synthetic. Do not imply they are real campaign data.

Check whether tests verify the business rule or merely mirror the new formula.
A green suite does not settle a rule that the suite never asserts. If code and
requirements disagree, cite both. If sources conflict, ask for resolution instead
of selecting the convenient interpretation.

## Multi-step pipelines and related PRs

Trace the actual flow from source through intermediate storage to the consumer.
For every changed handoff, verify meaning, grain, keys, units, types, nullability,
time semantics and freshness. Check that each step reads what the prior step
writes, including names/locations, partition conventions and schema versions.
Then check retries, checkpoint order and reuse of stale intermediate outputs.

Inspect intermediate writes and reads for unnecessary full rewrites, duplicated
materialization, repeated scans or missing applicable partition filters. Relate
the concern to actual consumers and data lifetime; do not delete an intermediate
artifact or change retention without knowing its purpose. Include end-to-end
latency when applicable, not just the runtime of one isolated step.

For multiple PRs, build a small compatibility table:

| Producer PR/head | Consumer PR/head | Shared contract | Compatibility / missing evidence |
| --- | --- | --- | --- |
| Inspected snapshot | Inspected snapshot | Meaning, schema and handoff | Evidence-backed assessment |

Check the intended combined state and supported intermediate deployment states.
Call out required merge/deploy order, rolling-version incompatibility, and flags
or migrations only when supported by the code or plan. Passing tests in each PR
does not establish interface consistency between them. Missing PR access means
the combined behavior is not verified.

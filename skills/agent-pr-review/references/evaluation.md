# Manual evaluation and model comparison

This is a pilot protocol, not an executed benchmark or an automatic model
switcher. Compare quality before claiming that a cheaper or stronger model is
better. Use exact model IDs and effort values supported by the running host.

## Runs

Use the same immutable PR snapshot, requirements, tools and test access for each
comparison. Start a fresh context for each run. Separate review depth from model
effort: first compare models at the same depth; then compare light/standard/hard
coverage. Keep unsupported configurations as not run, rather than substituting.
The default skill invocation is one review, not a request to run every model.

For each run, record:

| Field | Value to record |
| --- | --- |
| Input | PR/base/head or fixture version; requirement sources |
| Configuration | Requested and actual model, effort and depth |
| Evidence | Report artifact, executed checks, unavailable context/tools |
| Quality | Confirmed defects found/missed, false positives, unsupported claims |
| Usefulness | Whether a human can explain the business effect and next action |
| Cost | Observed duration and usage/cost only if exposed; otherwise unknown |

Score against observable behavior, not exact wording. A plausible diagnosis
without evidence is not a pass. Keep ground truth away from the reviewer during
the run; a human compares the report afterward. Do not reuse the author's
conversation as the independent evaluation context.

## Synthetic smoke cases

These cases are intentionally small and are not production evidence. Supply only
the requirement and code excerpt to the reviewer, then assess against the
expectation. They can be run manually without warehouse or network access.

### Duplicate join

Requirement: report one row per campaign with the sum of delivered impressions.
Before: `SELECT campaign_id, SUM(impressions) FROM delivery GROUP BY campaign_id`.
After: join `delivery d JOIN tags t ON d.campaign_id = t.campaign_id` before the
same aggregation. Fixture: one delivery row `(campaign_id=7, impressions=100)`
and two tag rows `(7, 'sports')`, `(7, 'news')`.

Evaluator expectation: identify 200 instead of 100; explain overstated delivery
in the report with the join as evidence. Do not invent affected client counts or
revenue loss. A possible fix must retain the actual purpose of the tags filter.

### Missing business rule

Requirement: ticket is unavailable. Diff changes `age_days <= 7` to
`age_days < 7`; no consumers or tests supplied.

Evaluator expectation: describe the exclusion at exactly day 7; ask whether the
window is inclusive. Do not declare an attribution policy violation, invent a
consumer, or recommend merge as though the intended rule were known.

### Safe optimization

Requirement: return active IDs, preserving source order.
Before: `result = [r['id'] for r in rows if r['active']]`.
After: `result = []; result.extend(r['id'] for r in rows if r['active'])`.
Input is a local list of dictionaries with boolean `active` and integer `id`.

Evaluator expectation: both produce the same ordered IDs for these inputs;
report no demonstrated business regression. Do not claim measured speed or
memory savings. Include empty/all-inactive inputs when depth warrants checks.

### Retry duplication

Requirement: each source event contributes to delivery once, including after retries.
Before: upsert by event ID. After: unconditional append followed by checkpoint.
Fixture: event `e1` with 100 impressions is appended, process fails before the
checkpoint, and the same event is replayed.

Evaluator expectation: demonstrate two rows/200 impressions instead of one/100;
connect the checkpoint failure to duplicate reporting. Ask about downstream
deduplication if not supplied rather than pretending to have inspected it.

## Pilot acceptance

In addition to the four smoke cases, use these cases to check the team requirements:

### Related PR contract mismatch

Requirement: a two-step pipeline writes and reads delivered impressions as whole
counts. Producer PR A changes the intermediate field from `impressions` to
`impressions_thousands` and divides the value by 1000. Consumer PR B reads the new
field name but still labels and aggregates the value as whole impressions.
Fixture: producer input 120000; stored value 120; consumer output 120 impressions.

Evaluator expectation: identify the units mismatch across PRs and the 1000-fold
understatement, even though names align. Record both snapshots and ask for the
supported deployment order. Do not approve business value or architecture for
the human. If PR B cannot be read, mark combined behavior unverified instead.

### Intermediate scan with no database access

Requirement: a downstream stage needs only the current daily partition. The
changed stage writes partitioned output, but the reader selects all partitions
and filters the date in application code. No plans, volumes or database access
are available.

Evaluator expectation: explain the evidenced avoidable read and suggest moving
the equivalent filter to the query, subject to the actual date semantics. Mention
latency as unmeasured and identify an appropriate plan check; do not fabricate an
EXPLAIN result, claim quantified savings or attempt to gain new access.

### Cosmetic declutter and real boundary validation

Requirement: validate external API input before processing it. The diff adds
`# initialize results` before `results = []` and a null check on an untrusted
request body. No functional defect is supplied.

Evaluator expectation: do not report the comment as a serious issue; preserve
trust-boundary validation even if the function has type annotations. No edits
are authorized by this review. Do not infer a bug from a catalog match.

Require the material join/retry defects to be recognized, the ambiguous boundary
to remain a question, and the equivalent-code case to avoid false blockers.
All reports must separate executed tests from proposed checks and avoid invented
business facts. Test on representative real PRs, including a held-out PR, before
adopting a model/depth default. Team acceptance remains a human decision.
Also require cross-PR unit consistency, honest handling of missing plan access,
and suppression of cosmetic findings. The business/architecture decision must
remain with a human. These cases are a protocol until actual runs are recorded.

# Model, reasoning effort and run record

Review depth is how much code and behavior is inspected. Reasoning effort is a
host/model setting. Output length is a third, separate choice. A long answer
does not prove deeper reasoning, and a model name does not prove review quality.

## Selection

Keep the host's current model and effort unless the user chooses otherwise.
`model` and `effort` in a request are asks to the host, not settings this
Markdown can apply. Set them through the host's own controls before invoking
the skill, then verify what the host reports:

| Host | Model | Reasoning effort |
| --- | --- | --- |
| Claude Code | `/model` (or `--model` at launch) | the session effort setting (`/effort` where available, or the host's effort/thinking control) |
| Codex CLI | `model` in config | `model_reasoning_effort` in config |
| Other | that host's documented controls | that host's documented controls |

Record an exact model ID rather than "fast" or "strong". If the host exposes only
an alias, record the alias and say the resolved version is unknown. Never infer
the model from its writing style. Do not translate one host's effort enum into
another's; `hard` depth is not a synonym for any vendor's effort value.

## Pilot configurations

Optional starting points for the team pilot, not benchmark winners or defaults.
Use whatever the host actually offers; the labels are ours.

| Pilot | Depth | Model | Effort |
| --- | --- | --- | --- |
| Light coverage | light | host default | low |
| Regular coverage | standard | host default | medium/default |
| Deep coverage | hard | host default | high |
| Model comparison at fixed depth | standard | one alternative model | same as regular |

The first three vary coverage and effort together, so they cannot show which
one caused a quality difference. To compare models, hold depth, effort, input
snapshot and tools constant. To compare effort, hold model and depth constant.
Use [evaluation.md](evaluation.md) before adopting a preferred configuration.

Example request:

```text
Use agent-pr-review on <PR URL>, depth=standard, explanation=plain.
Report the actual model and effort before presenting results.
```

Do not write global host configuration as part of a review. If a requested
setting is unsupported, mark that run *not run* and continue only within the
user's constraints; never substitute silently when the setting matters for a
comparison.

## Report record

Include a compact record for each reviewer/validator run:

```yaml
review_depth: standard
host: <product and version if exposed>
requested_model: <ID or "host default">
actual_model: <host-reported ID or unknown>
requested_effort: <value or "host default">
actual_effort: <host-reported value or unknown>
settings_evidence: <host metadata/config source or unavailable>
reviewed_heads: <repository and head SHA for each PR>
skill_revision: <commit containing this skill, if known>
context: <fresh or reused; single reviewer or authorized independent pass>
checks: <executed checks and coverage gaps>
```

A requested setting is never copied into an `actual_*` field without
verification. Record elapsed time and usage only when the host exposes them.

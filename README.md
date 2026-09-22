# AI-Assisted Engineering Skill

A skill for taking engineering work from discovery through implementation, testing, and release using a structured AI-assisted workflow.

**Supports:** Cursor, Claude, Codex, and other agents that follow the [Agent Skills](https://agentskills.io) standard.

## Suggested Use Cases

- Implementing a non-trivial feature from a ticket or PRD
- Working across multiple repos, services, or data pipelines
- Starting a task with incomplete context that needs structured discovery
- Planning and executing work that requires iterative validation and cleanup

## What's Included

```text
.
├── SKILL.md
├── README.md
├── references/
│   └── carmack-smp-plan-1998.md
└── skills/
    └── agent-pr-review/
        ├── SKILL.md
        └── references/
```

## Structure

- `SKILL.md` - the main workflow definition and operating guidance
- `references/carmack-smp-plan-1998.md` - reference material for the technical narrative style used by the skill

## Agent-written PR review (draft)

Proposal for [sc-84584](https://app.shortcut.com/vamos-tribe/story/84584):

- [Reviewer skill and algorithm](skills/agent-pr-review/SKILL.md): business impact,
  business logic, basic efficiency, plain-language summaries and grounded diagrams.
- [Pre-PR self-check standard](skills/agent-pr-review/references/pre-pr-standard.md):
  author, fresh-context reviewer and fixer handoff with a bounded review loop.
- [Review-process antipatterns](skills/agent-pr-review/references/review-antipatterns.md)
  and [Yehor's code anti-pattern catalog](skills/agent-pr-review/references/code-antipatterns.md) (vendored verbatim).
- [Manual evaluation and model comparison](skills/agent-pr-review/references/evaluation.md):
  light/standard/hard depth, explicit model/effort requests and synthetic pilot cases.
- [Model and reasoning-effort settings](skills/agent-pr-review/references/model-settings.md)
  and [public workflow comparison](skills/agent-pr-review/references/external-review-patterns.md).

The review skill is a separate bundle at `skills/agent-pr-review/`; copy or
symlink that whole directory (with `references/`) into your skills folder. Its
`allowed-tools` frontmatter keeps a review read-only by default. The root
engineering skill remains the implementation workflow. Example request:
`Use agent-pr-review on <PR URL>, depth=standard, explanation=plain.`

Status: draft for sc-84584. Team interviews, agreement and a real-PR pilot are
tracked in [sc-86709](https://app.shortcut.com/vamos-tribe/story/86709).

### Origin

The proposal comes from the Data team's
[August 18-19, 2026 thread](https://simulmedia.slack.com/archives/C022S15NCJ0/p1787057765549379):
two outputs (pre-PR self-check standard, reviewer algorithm as a skill), humans on
business value and architecture with agents on code, Yehor's declutter catalog as
review input, a common default rather than a mandatory rule, Plain Technical English
instead of ASD-STE100 for PR descriptions, and no Poderv'yansky-style retellings.
Nazarii's shared review prompt (August 19, verbatim) is what the skill's efficiency
step and findings policy implement:

> Review this from consistency, business logic and code correctness and performance (incl latency where applicable) perspectives. Make sure to bring up only serious issues and bugs, no nits or super rare edge cases.
>
> For applied performance analysis you can use the correct tools, like EXPLAINs for Redshift, Postgres RDS and Databricks, if you have access via AWS SSM or via Databricks CLI accordingly.
>
> For any analysis of multi-step pipelines you should make sure they are semantically correct and are storing and querying any intermediate data efficiently. If I am sending several PRs, check them for their logical and interface consistency.
>
> If you see any low-hanging fruit opportunities for optimization without major rewrites, mention them in your review as well

The proposal is a recommended default. Humans assess business value and architecture;
agents check code, business-rule implementation, consistency and performance.
Reviews cover related PR interfaces and intermediate pipeline data, report serious
issues plus small useful optimizations, and use Plain Technical English without nits
or stylized retellings. Query-plan analysis is conditional on appropriate access.

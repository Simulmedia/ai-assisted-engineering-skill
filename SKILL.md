---
name: ai-assisted-engineering
description: >
  AI-assisted engineering workflow for taking tickets/PRDs from discovery through
  implementation, testing, and release. Covers context collection, scope negotiation,
  planning, iterative development, testing, and cleanup. Use this skill when an engineer
  asks you to implement a feature, work on tickets, build something from a PRD, or
  when starting any non-trivial coding task that involves multiple repos, data pipelines,
  or unclear requirements. Also trigger when the engineer says "work on this ticket,"
  "implement this feature," "here's what we need to build," provides Shortcut links,
  or drops a bunch of repo links and says "figure it out." If the task is bigger than a
  single-file fix, this skill applies.
author: Nazarii Melnychuk
---

# AI-Assisted Engineering Workflow

A structured but flexible process for taking engineering work from ticket to shipped
feature using AI coding agents. Not a rigid pipeline — a set of checkpoints that
prevent the usual failure modes: building the wrong thing, losing context mid-session,
repeating mistakes, shipping without tests.

## Phase 0: Environment Check

Before starting any work, verify available integrations. The agent should check for
and note the status of:

- **AWS Redshift MCP** — querying production data, validating outputs,
  understanding existing schemas
- **AWS Docs MCP** — referencing AWS service documentation
- **Neon MCP** — read-only database access for development/staging data
- **Databricks agent skills** — if the task involves Databricks, check for
  [databricks-agent-skills](https://github.com/databricks/databricks-agent-skills).
  Cursor will prompt installation if you name-drop Databricks; other harnesses
  may need manual setup
- **AWS CLI** — verify it's authenticated via SSO with credentials exported
- **Databricks CLI** — verify it's configured and authenticated
- **SM API skill** — if interacting with Simulmedia's internal APIs
- **`gh` CLI** — GitHub CLI for creating PRs, reviewing, linking issues
- **`s5cmd`** — fast S3 operations, especially useful for long listings and
  bulk copies which are common in data pipeline work

If any of these are missing and the task requires them, tell the engineer what's
unavailable and what it limits. Don't silently work around missing access — surface
it early.

Also check for:
- `agents.md` or `AGENTS.md` — check repo roots, but also subdirectories and
  library folders within repos. These are repo- or module-specific agent
  instructions and can live anywhere in the tree
- `.cursor/rules`, `.claude/settings`, or similar agent config files
- Available MCP servers beyond the ones listed above
- Any simplify/refactor/best-practices skills enabled in the current agent
  harness — note them for Phase 6

## Phase 1: Context Collection

The quality of the output is directly proportional to the quality of the input context.
This is the most important phase.

### If the engineer provides rich context

When the engineer drops repo links, ticket links, docs, and architectural notes —
read everything before writing a single line of code. Organize what you've received:

- **Tickets/requirements** — what are we building and why
- **Repos and branches** — where does the code live, any feature branches with
  prior work or POCs
- **Docs and specs** — PM specs, architectural docs, Notion pages
- **Constraints** — what NOT to touch, what's experimental, what's production-critical

### If the engineer provides sparse context

Do not start work. Ask for more. Specifically:

1. "What are we building and why?" — plain language, not ticket-speak
2. "Which repos are involved?" — links, not names
3. "Is there any prior work or POC?" — branches, spikes, PM prototypes
4. "What are the hard constraints?" — don't touch X, don't deploy to Y, must be
   compatible with Z
5. "Who owns the product decisions if I hit ambiguity?" — so you know who to
   direct questions to

Push the engineer to get this from their PM, tech lead, or whoever has context.
A well-scoped 15-minute conversation saves hours of rework.

### Context organization

Once you have context, organize it into a single working document (not scattered
across 10 markdown files). Structure:

```
## What We're Building
Plain language summary. One paragraph.

## Moving Parts
Which repos, services, tables, APIs are involved and how they connect.

## Decisions Made
Scope choices, architecture decisions, constraints. With rationale.

## Open Questions
Things that need answers before or during implementation.

## References
Links to repos, branches, tickets, docs. Annotated with relevance:
- [HIGH] primary implementation repo — where we're writing code
- [SKIM] upstream data pipeline repo — for understanding data lineage if needed
- [REF] product spec / PM doc — source of truth for requirements
```

## Phase 2: Scope and Attention Routing

Before writing code, get explicit alignment on scope.

### Ask the engineer

- "Full implementation or MVP first?" — this changes everything about how you
  approach it
- "Which parts are critical path vs nice-to-have?" — so you don't gold-plate
  the wrong thing
- "Any parts I should skim vs study deeply?" — manages your context window
  like a budget

This is partly a product question. If the engineer doesn't know, guide them to
ask their PM specific questions rather than guessing. Frame it as: "I want to
build the right thing the first time — can you confirm X with your PM?"

### Attention budget

Not everything deserves equal attention. Explicitly mark:

- **Deep read** — repos where you're writing code, schemas you're extending
- **Skim** — related services for understanding interfaces
- **Reference only** — check if you hit a specific question, otherwise ignore

## Phase 3: Planning

Create a structured plan before writing code. This plan must survive context
compactions — it's your memory across long sessions.

**Important:** Do NOT use your harness's built-in "Plan mode" (Cursor Plan mode,
Claude Code plan mode, Codex plan mode etc.) for this. Those modes are typically sandboxed to
local file operations, can't make MCP calls, can't query databases, and can't do
the exploratory analysis that real planning requires. Do your planning in normal
mode where you have full tool access.

### Scratch directory

Create a `.scratch/` directory (add to `.gitignore`) as a working space for:
- Downloaded reference files, sample data, exploratory outputs
- Intermediate analysis artifacts
- Anything the agent needs during work but shouldn't commit

This keeps the repo clean while giving the agent a place to think.

### Plan document structure

Single document, not a sprawl of files:

```
# Implementation Plan: [Feature Name]

## Index
1. Summary
2. Architecture
3. Action Items
4. Assumptions
5. Open Questions
6. Notes & Decisions

## Summary
What we're building, where, why. 2-3 sentences max.

## Architecture
Data flow, component interactions, new tables/APIs.
Use Mermaid UML diagrams with proper types for architecture visualization.
Show how components connect, what data flows where, what's new vs existing.

## Action Items
Ordered, specific, with clear done-criteria:
1. [repo] Add new attribution model class — done when: model passes
   unit tests with sample data
2. [repo] Extend API endpoint — done when: returns correct response
   for test campaign
...

## Assumptions
Things you're assuming to be true. Flag for engineer to confirm.

## Open Questions
Blocking or non-blocking, with suggested resolution path.

## Notes & Decisions
Running log of choices made during implementation. Date-stamped.
"Chose X over Y because Z. If this turns out wrong, reverting to Y
requires changing [specific files]."
```

### Work log

Maintain a separate lightweight log:

```
# Work Log

## [timestamp] Started — read all context, created plan
## [timestamp] Action item 1 — implemented, tests pass
## [timestamp] Hit issue — [description]. Tried A, didn't work because B.
   Tried C, worked. Root cause was D.
## [timestamp] Action item 2 — implemented, but found edge case in [X],
   added handling
```

This is not a diary. It's a compaction-surviving breadcrumb trail so the agent
(or a fresh agent session) doesn't repeat failed approaches.

## Phase 4: Implementation

### Code quality calibration

This is roughly 60% of what determines whether the output is usable or throwaway.

Your code should be **elegant, readable, correct, very well inline-documented,
using what we have efficiently, not duplicating stuff.**

- **Elegant** — clean structure, clear intent, nothing extraneous
- **Readable** — the next person reading this might not be an AI. If a clever
  one-liner takes 30 seconds to parse, write it as three obvious lines
- **Correct** — obviously correct is better than cleverly correct
- **Well inline-documented** — explain WHY, not WHAT. Document non-obvious
  decisions, edge cases, and "this looks weird but it's because X"
- **Reuse what exists** — check the codebase for existing utilities, patterns,
  abstractions before creating new ones. "Most was done before us already, we
  just don't know it because we haven't searched"
- **No duplication** — if you're copying a pattern, extract it
- **Idempotent where possible** — especially for data pipelines and ETL
- **Performant by default** — think about data volumes, query plans, memory.
  Be specific to the tech stack you're actually working with. Examples:
  Spark/Databricks (partition strategies, broadcast joins, predicate pushdown),
  Redshift (sort/dist keys, query plan analysis, WLM queue behavior),
  Postgres (index usage, EXPLAIN ANALYZE, connection pooling, vacuum),
  Pandas (vectorized ops over iterrows, chunked reading for large CSVs,
  categorical dtypes for memory), Lambda (cold starts, memory allocation,
  connection reuse), Ruby (N+1 queries, eager loading, object allocation),
  TypeScript/Node (event loop blocking, stream processing for large payloads,
  connection pooling). The point is: generic "make it fast" is useless.
  Know the specific levers for the specific tech

### Research before implementing

For statistical, mathematical, or domain-specific logic — research the methodology
first. Don't invent an approach when established ones exist. If you're implementing
an attribution model, understand attribution modeling. If you're doing time-series
aggregation, know the standard approaches.

Check your work against the methodology. If something feels off about the numbers
or the approach, pause and verify rather than shipping something subtly wrong.

If the methodology feels genuinely complex — not just unfamiliar but actually
requiring domain expertise (statistical modeling, ML pipeline design, custom
attribution logic) — flag it to the engineer: "This might benefit from Data
Science team input before we commit to an approach." Sometimes the right move
is a 20-minute conversation with someone who's solved this class of problem
before, not three hours of the agent researching from scratch.

### Guardrails

Hard rules that override everything:

- **Do NOT deploy to production** unless explicitly told to
- **Do NOT post/update tickets** in Shortcut or any project management tool
  unless explicitly asked
- **Do NOT modify schemas in production databases**
- **Do NOT delete data**
- **Do NOT commit directly to main/master** — use feature branches
- **Use git worktrees** for parallel work across branches when needed, to avoid
  stashing/switching overhead and keep work isolated
- If unsure whether an action is safe — ask first

### Commit and PR discipline

Don't let work sit uncommitted and unpushed. Engineers often assume code is
already pushed — especially when juggling multiple branches — and base their
next steps on that assumption. Early pushes also enable early reviews, which
catch issues before they compound.

For larger changes, prefer **PR chains** over monolithic 30-file PRs:

- PR1 → merges into main, contains foundational changes
- PR2 → branches from PR1, builds on top
- PR3 → branches from PR2, etc.

Each PR description should explain what THIS PR does on its own — not just
"part 2 of the chain." Include links to the other PRs in the chain so
reviewers can see the full picture. Use `gh` CLI if available to create
and link PRs efficiently.

## Phase 5: Testing

### Unit tests

Write them. For every non-trivial function. Not as a checkbox exercise — as
actual verification that the thing works.

### Invariant tests

For data-heavy work, identify and test invariants:

- Row counts before and after transformations
- Sum/aggregate consistency
- Null handling
- Edge cases in date ranges, empty datasets, single-record scenarios
- Idempotency — running the same operation twice produces the same result

### Validation against real data

If you have database access (Redshift MCP, Neon MCP, Databricks), validate against
actual data. This is not optional polish — it's where you catch the real bugs.

- Pick a recent entity with known characteristics and verify your output matches
  expectations
- Run ad-hoc piecewise validation queries during development — don't wait until
  the end. Query the database to check "would this actually work?" before writing
  the full implementation. A quick `SELECT` that confirms your assumptions about
  the data shape, value distributions, or join cardinality saves hours of debugging
  code that was built on wrong assumptions
- Don't just trust that tests pass — look at the actual numbers

## Phase 6: Cleanup and Handoff

After the core work is done and tests pass:

### Code review (self)

Before proposing anything to the engineer, review your own work:

1. **Self-consistency** — do all the pieces agree with each other? Are naming
   conventions consistent across files? Do comments match what the code does?
2. **Logical correctness** — trace the data flow end to end. Does every branch
   get handled? Are edge cases covered or explicitly documented as out of scope?
3. **Data grain correctness** — for data pipeline work, verify you're operating
   at the correct grain. Aggregating at the wrong level is a silent killer.
4. **Data contracts** — if work spans multiple repos or services, verify that
   data contracts hold across boundaries. Field names, types, nullability,
   enum values, timestamp formats — all aligned. This is critical with many
   moving pieces and easy to miss.
5. **Low-hanging performance** — things that are obvious in hindsight but
   invisible when you're deep in the task: selecting only the fields you need
   instead of `SELECT *`, adding timestamp filters to narrow scan ranges,
   using the correct join type semantically (LEFT vs INNER when it matters),
   making use of existing sort/dist/partition keys, avoiding unnecessary
   materializations. These are often the difference between "runs in 2 minutes"
   and "runs in 45 minutes" and they cost almost nothing to fix.

### Simplification

Can anything be made more readable without changing behavior? Tests are the
safety net here — behavior and results must hold exactly, which is why tests
come before this step. Propose specific changes: "I can simplify the
aggregation logic in X and add missing docstrings in Y — want me to?"
Don't refactor as a self-goal. Don't refactor without asking.

If a simplify/refactor/best-practices skill is available in the current agent
harness (noted in Phase 0), propose enabling it: "There's a [skill name] that
could help clean this up — want me to run it?"

### Documentation — commit to the repo

Commit documentation to `docs/` under a clear name tied to the feature and
ticket(s). This is gold for the next person (human or AI) who touches this code.

**Working artifacts** (from earlier phases, updated with final state):

1. **Implementation plan** — the plan doc from Phase 3, updated with final
   decisions and outcomes
2. **Work log** — what was tried, what worked, what didn't
3. **Validation instructions** — this is critical. Not generic "run the tests"
   instructions. The specific commands, parameter values, queries, and flow
   that YOU used to validate the work. This encodes assumptions clearly, so
   the next person can go "they tested with campaign X using parameters Y —
   let me try with a different dimension and see if it holds." A good validation
   doc is a ready-made baseline for the next round of work.

**Synthesized docs** (written after work is done):

4. **Elevator pitch** — a short, specific summary for meetings and standups
   where you need to give context without getting into details. Not hype.
   "We built [feature] that does [this]. Our assumptions were [that].
   It supports [these methods/params/inputs]. We tested by [methodology]."
   Write it the way you'd explain it to a competent colleague over coffee.
   If you wouldn't say it out loud without feeling embarrassed, rewrite it.

5. **Technical narrative** — a longer-form write-up that tells the story of
   what was built, why certain approaches were chosen or abandoned, what
   surprised you, and what the key technical decisions were. This is not a
   formal report. It's an informal but precise engineering diary entry —
   personal in voice, rigorous in content. It blends the implementation plan
   (what we set out to do), the work log (what actually happened), and a
   summary of outcomes into one readable document focused on the things that
   mattered.

   The tone: honest, specific, unhurried. State what you tried, what the
   numbers were, why something didn't work, what you learned from it. If you
   abandoned an approach, explain why with enough detail that someone else
   won't re-attempt it without new information. If something worked
   unexpectedly well, say that too.

   Not everything deserves a technical narrative — routine features don't
   need one. But anything that involved non-obvious decisions, failed
   approaches, performance investigation, or architectural choices that
   future engineers will wonder about — write it up. See
   `references/carmack-smp-plan-1998.md` for the gold standard of this
   format.

### Inline documentation

Update or create relevant inline docs. Code comments where the logic is
non-obvious. README updates if the setup process changed.

## Iterative Research-Build Loops

Not every task benefits from this section. Most straightforward feature work doesn't need
a research loop — you know what to build, you build it. But for tasks where the
cycle is research → design → implement → test → measure → adjust — where the
"right answer" isn't clear upfront — consider using (and suggesting to the engineer) structured research harnesses:

- [autoresearch](https://github.com/uditgoenka/autoresearch) — Claude-based
  research loop automation
- [pi-autoresearch](https://github.com/davebcn87/pi-autoresearch) — Pi harness,
  works with GPT-5.4 business tier subscription and other models

Good candidates for research loops: novel algorithm implementation, performance
optimization where the bottleneck is unclear, latency improvement, data modeling where the shape of
the problem is ambiguous, methodology-heavy work (statistics, ML, attribution).

Bad candidates: CRUD features, API extensions, schema migrations, anything where
the spec is clear and the implementation is mechanical. Use judgment.

## Anti-Patterns

Things this skill explicitly prevents:

- **Starting to code without understanding what you're building** — Phase 1 exists
  for a reason
- **Losing context after compaction** — the plan doc, work log, and scratch dir
  survive you
- **Repeating failed approaches** — the work log tracks what didn't work and why
- **Building the wrong scope** — Phase 2 forces explicit alignment
- **Shipping without tests** — Phase 5 is not optional
- **Silent refactoring** — always propose, never unilaterally restructure
- **Guessing at methodology** — research first, implement second
- **Working around missing access silently** — surface limitations in Phase 0
- **Using Plan mode for real planning** — use normal mode with full tool access
- **Junking the repo** — scratch dir exists, use it for temporary artifacts
- **Monolithic PRs** — split into chains, push early, describe each PR properly

## Standards and Inspiration

The writing and communication standard for all artifacts this skill produces —
plan docs, work logs, technical narratives, elevator pitches, PR descriptions,
code comments — is clarity and specificity over style. No filler, no hype,
no padding. No slop.

Good technical communication reads like someone explaining their work to a
smart colleague who doesn't have context yet. It's specific about what was
done and why, honest about what's uncertain, and short enough that people
actually read it.

Reference points for the overall engineering philosophy:

- **John Carmack** — shipped under hard deadlines at id Software while
  maintaining clean, structured, performant code. The orientation is: time
  pressure is not an excuse for sloppy work, it's a reason to be disciplined
  about what matters and ruthless about what doesn't. His `.plan` files
  (1996–2010) are the original engineering diaries — informal, precise,
  honest about failure, and read by thousands. The technical narrative
  format in Phase 6 is directly inspired by these. See
  `references/carmack-smp-plan-1998.md` for the tone.
- **Linus Torvalds** — whatever you think of his interpersonal style, his
  technical communication in kernel mailing lists is precise, specific, and
  wastes zero words explaining exactly why something is wrong and what the
  fix should be. That specificity is the standard.
- **The tradition of good engineering notebooks** — document what you did,
  what you observed, what you concluded, and why. Not for posterity. For the
  next person who needs to understand this at 2 AM during an incident.

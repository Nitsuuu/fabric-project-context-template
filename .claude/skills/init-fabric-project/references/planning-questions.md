# Part 1 Planning Conversation — Question Set

Run after the identity questions (Step 2). Ask one at a time. Push back on vague answers. The output of this conversation is captured material for the new project's `project-overview.md` and `architecture.md`.

## The conversation in brief

> "I'm going to ask seven questions, one at a time, to capture the design of {{PROJECT_NAME}} before any code gets scaffolded. Be specific. If something is uncertain, say so — vague answers produce a vague project."

## Q1 — What does this project actually do?

> "Describe what this project does in 2-3 sentences. What goes in (sources), what comes out (consumers), and the business outcome it enables. Avoid vague language. Bad: 'improves data quality.' Good: 'reduces month-end close from 5 days to 2 days by automating GL extraction from {{source}} into Power BI semantic models.'"

Capture: a paragraph for `project-overview.md > What this is`.

## Q2 — Who uses it and what do they need?

> "Who consumes the gold layer? Power BI semantic models? Operational dashboards? Downstream apps? An analytics team running ad-hoc queries? Be specific about the consumer and what they need from this project."

Capture: feeds into `project-overview.md > Core user flow` (the consumer end) and helps shape success criteria.

## Q3 — What are the core user flows from start to finish?

> "Walk me through the data flow from source to consumer. Step by step, no gaps. When do pipelines run? What happens at each layer? Where are the human checkpoints?"

Capture: numbered steps for `project-overview.md > Core user flow`.

## Q4 — What are the most technically complex parts?

> "What's the hardest part of this project, technically? Examples: a source system with quirky pagination or rate limits, a data quality issue that's hard to detect, a join that produces unexpected duplicates, a SCD Type 2 with edge cases, an unusual file format. Naming the hard parts up front lets us plan around them."

Capture: feeds into `architecture.md > Invariants` (project-specific) and surfaces topics for the first REQs.

## Q5 — What could go wrong?

> "What's the most likely failure mode? Source goes down? Schema changes? Duplicate keys? Late-arriving data? Stale credentials? Naming the failure modes lets us decide which need automation, which need monitoring, and which the team will catch by hand."

Capture: feeds into `architecture.md > Invariants` and `project-overview.md > Out of scope`.

## Q6 — What are the technology decisions and why?

> "We've already locked Microsoft Fabric as the platform. What other technology decisions matter? Which sources are batch via API vs. file drop? Which lakehouse design (single vs multi-domain)? Are you using Variable Library for env config? Key Vault for secrets? Any non-default choices?"

Capture: feeds into `architecture.md > Stack` and `architecture.md > Storage model`.

## Q7 — What is explicitly out of scope?

> "List what this project will NOT do. Be specific. 'No real-time' and 'no ML' are common; what else? Power BI report design? Source-system master data corrections? Historical backfill? Listing what's out of scope is more important than listing what's in — it stops scope creep from day one."

Capture: bullet list for `project-overview.md > Out of scope`.

## After the seven questions

Summarize all captured material back to the user as structured bullets per question. Confirm before proceeding to Step 4 (substitution). If the user wants to adjust any answer, do so before moving on.

## Tone

- One question at a time.
- Push back on vague answers. Examples: "What does '{{source}}' mean specifically?", "When you say 'fast', what's the actual latency target?", "By 'all sources', do you mean the three you listed or are there more?"
- Do not write code suggestions during this step. The conversation is design, not implementation.
- Do not skip a question because it seems redundant. Skipping leaves a gap in the project-overview.

## When the user wants to skip

If the user genuinely wants to skip the planning conversation (e.g., they already have a planning doc), accept it but record this in `progress-tracker.md` Session Notes: "Planning conversation skipped at user request — refer to [link] for original planning doc." This makes the skip visible later when someone wonders why `project-overview.md` is sparse.

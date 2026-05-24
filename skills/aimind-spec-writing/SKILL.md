---
name: aimind-spec-writing
description: Structured spec/PRD/brainstorm authoring for Dave's freelance and product work. Takes a vague idea, feature name, or problem statement and produces a scannable PRD with problem, goals, non-goals, user stories, P0/P1/P2 requirements, success metrics, open questions, and timeline considerations. Also supports brainstorm-partner mode for fuzzy early-stage thinking.
tier: installed
catalog_entry: aimind-spec-writing
user-invocable: true
enabled: true
priority: 4

allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - mcp__aimind-memory__search_memory
  - mcp__aimind-memory__store_memory
  - mcp__google__docs_documents_create
  - mcp__google__docs_documents_batchUpdate

personality-mode: warm

metadata:
  category: product
  channel: telegram
  author: nova-catalogue
  source_attribution: anthropic/knowledge-work-plugins product-management (write-spec, product-brainstorming)
  triggers:
    - write a spec
    - draft a PRD
    - draft a spec
    - product brief
    - feature spec
    - brainstorm
    - competitive analysis
    - product brainstorming
    - acceptance criteria
---

# AIMind Spec Writing

Dave does a lot of spec work — for Nike, for Wolters-Kluwer, for Velph, for his own product ideas. This skill turns vague problem statements into structured PRDs, and is also his brainstorm partner for the fuzzy front-end.

Adapted from Anthropic's `product-management/write-spec` and `product-brainstorming` skills. The PRD structure and brainstorm patterns are the value; the connector list maps to Nova's tools.

## Two modes

### Mode A: Brainstorm partner (fuzzy → structured)

When Dave says "let's brainstorm X" or "I've been thinking about Y":

1. **Listen first.** Reflect back what you heard in one paragraph. Verify the framing before going deeper.
2. **Ask one question at a time.** Cover (in order):
   - What problem does this solve? Who feels it?
   - What's the current workaround? Why isn't it good enough?
   - Who's the user? Be specific.
   - What does success look like in 6 months?
   - What's the laziest version that would still be valuable?
3. **Push on assumptions.** Where appropriate, challenge:
   - "Why do you think users want X and not Y?"
   - "What's the evidence?"
   - "What would make this fail?"
4. **Diverge then converge.** Throw out 3-5 angles. Then narrow.
5. **End with a choice.** "Want me to draft a spec on the angle we landed on, or sit with it for a day?"

Brainstorm mode does **not** produce a PRD. It produces a 1-page notes file in `~/obsidian/nova/Projects/<category>/<project>/brainstorm-<date>.md` and offers to graduate it to a spec when Dave's ready.

### Mode B: Spec writing (structured PRD)

When Dave says "write a spec for X" or graduates a brainstorm:

#### 1. Gather context

Ask conversationally — don't dump all questions at once. Cover:
- **User problem**: what + who experiences it
- **Target users**: which segments
- **Success metrics**: how we know it worked
- **Constraints**: tech, timeline, regulatory, dependencies
- **Prior art**: previous attempts, existing solutions

If this is for a known project, pull context first:
- `mcp__aimind-memory__search_memory("<project name>")`
- `Grep` for the project name in `~/obsidian/nova/Projects/`
- Don't ask Dave for context Nova already has.

#### 2. Generate the PRD

Output a markdown PRD with these sections. Keep it scannable — bold the things a busy stakeholder needs to skim.

```markdown
# PRD — <Feature Name>

**Author:** Dave (via Nova)
**Date:** <YYYY-MM-DD>
**Status:** Draft

## Problem Statement
<2-3 sentences: what problem, who experiences it, cost of not solving it. Ground in evidence: user research, support data, metrics, customer feedback.>

## Goals
1. <Specific measurable outcome — answer "how will we know this succeeded?">
2. <...>
3-5 goals total. Outcomes, not outputs. Distinguish user goals from business goals.

## Non-Goals
1. <Adjacent capability explicitly out of scope> — <brief rationale>
2. <...>
3-5 non-goals. Prevent scope creep, set stakeholder expectations.

## User Stories
Grouped by persona. Format: "As a <user type>, I want <capability> so that <benefit>."

### <Persona 1>
- As a <specific user type>, I want <capability> so that <benefit>
- (include edge cases: error states, empty states, boundary conditions)

### <Persona 2>
- ...

## Requirements

### Must-Have (P0)
The feature cannot ship without these. Minimum viable version.
- [ ] <Requirement> — <acceptance criteria>
- [ ] ...

### Nice-to-Have (P1)
Significantly improves the experience but core works without them. Often fast follow-ups.
- [ ] ...

### Future Considerations (P2)
Explicitly out of scope for v1 but design should support them later. Architectural insurance.
- [ ] ...

## Success Metrics

### Leading Indicators (days to weeks)
- <metric>: <target> within <window>, measured via <method>
- ...

### Lagging Indicators (weeks to months)
- <metric>: <target> within <window>, measured via <method>
- ...

## Open Questions
Tagged with owner and blocking/non-blocking status.
- [Eng, blocking] <question>
- [Design, non-blocking] <question>
- [Stakeholder, blocking] <question>

## Timeline Considerations
- Hard deadlines: <if any>
- Dependencies: <other teams, releases, vendor work>
- Suggested phasing: <if feature is too large for one release>

## Acceptance Criteria
Given/When/Then for each P0 requirement.

- **Given** <precondition>
- **When** <user action>
- **Then** <expected outcome>
```

#### 3. Iterate

After producing the draft:
- "Want me to expand any section?"
- "Should I draft the engineering ticket breakdown next?"
- "Want a competitive-analysis matrix to go with this?"

#### 4. Persist

Save to `~/obsidian/nova/Projects/<category>/<project>/spec-<feature-name>-<date>.md`. If Dave wants to share externally, offer to also create a Google Doc via `mcp__google__docs_documents_create` — **external sharing only**, not for internal tracking.

## Quality bar — what makes a good spec

Pull from Anthropic's source skill:
- **Be ruthless about P0s.** If everything's P0, nothing is. Challenge every must-have.
- **Non-goals are as important as goals.** They prevent scope creep during implementation.
- **Goals are outcomes, not outputs.** "Reduce time to first value by 50%" not "build onboarding wizard."
- **Acceptance criteria cover happy path, error cases, AND negative cases** (what should NOT happen).
- **Avoid weasel words** in acceptance criteria: "fast", "user-friendly", "intuitive" — define them concretely.
- **Open questions should be genuinely open.** Don't include questions you can answer from context.

### Good user stories (INVEST)
- **Independent** — can be developed standalone
- **Negotiable** — details are discussable, story isn't a contract
- **Valuable** — delivers value to a USER, not just the team
- **Estimable** — team can roughly estimate effort
- **Small** — fits in one sprint
- **Testable** — clear way to verify

### Common user-story mistakes
- Too vague: "I want the product to be faster" — what specifically?
- Solution-prescriptive: "I want a dropdown menu" — describe the need, not the UI widget
- No benefit: "I want to click a button" — why?
- Too large: "I want to manage my team" — break into capabilities
- Internal focus: "We want to refactor the database" — that's a task, not a user story

## Output location

| Type | Path |
|------|------|
| Brainstorm notes | `~/obsidian/nova/Projects/<category>/<project>/brainstorm-<YYYY-MM-DD>.md` |
| PRD draft | `~/obsidian/nova/Projects/<category>/<project>/spec-<feature-name>-<YYYY-MM-DD>.md` |
| External share | Google Doc via `mcp__google__docs_documents_create` (only when explicitly requested) |

## Migration notes

- Sources merged:
  - `anthropic/knowledge-work-plugins/product-management/skills/write-spec/SKILL.md` (Mode B)
  - `anthropic/knowledge-work-plugins/product-management/skills/product-brainstorming/SKILL.md` (Mode A)
- Combined into one skill because Dave's flow is fluid — brainstorms graduate to specs in the same session.
- Skipped Anthropic's `competitive-brief`, `metrics-review`, `roadmap-update`, `sprint-planning`, `stakeholder-update`, `synthesize-research` — defer to follow-up skills if Dave wants them.
- Output is Obsidian-first; Google Doc is opt-in for external sharing only (per Nova's storage rules: Obsidian = internal, Drive = external audience).

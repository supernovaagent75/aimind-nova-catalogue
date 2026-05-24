---
name: velph-sales
description: Pre-sales workflow for Velph client work — account research, call prep, outreach drafting, and competitive intelligence. Adapted from Anthropic's sales plugin for Dave's freelance/Velph context (no CRM, no enrichment SaaS — uses web search, Gmail, calendar, Obsidian as the data layer). Complements the existing aimind-offer skill, which handles the proposal stage.
tier: installed
catalog_entry: velph-sales
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
  - mcp__google__gmail_users_messages_list
  - mcp__google__gmail_users_messages_get
  - mcp__google__gmail_users_drafts_create
  - mcp__google__calendar_events_list
  - mcp__google__calendar_events_get
  - mcp__aimind-memory__search_memory
  - mcp__aimind-memory__store_memory
  - mcp__aimind-memory__search_entities

personality-mode: warm

metadata:
  category: sales
  channel: telegram
  author: nova-catalogue
  source_attribution: anthropic/knowledge-work-plugins sales (account-research, call-prep, draft-outreach, competitive-intelligence)
  client: velph
  triggers:
    - velph
    - client research
    - research client
    - prep for client call
    - call prep
    - draft outreach
    - competitive analysis
    - intel on
    - look up the person
---

# Velph Sales — Pre-Sales Workflow

Dave's actively doing Velph client work (BodemBrief, Vera persona, Hanne/Dennis as the client team). This skill is the pre-sales companion to the existing `aimind-offer` skill: it covers account research, call prep, outreach drafting, and competitive intel — the work that happens **before** a proposal lands.

Adapted from Anthropic's `sales` plugin (account-research, call-prep, draft-outreach, competitive-intelligence). The CRM and enrichment connectors are swapped for Nova's actual data layer: web search + Gmail + calendar + Obsidian + memory.

## Sub-workflows

This skill bundles four related workflows. Pick the right one based on Dave's ask.

---

### 1. Account Research

**Trigger phrases**: "research <company>", "look up <person>", "intel on <prospect>", "who is <name> at <company>", "tell me about <company>".

#### Step 1: Parse
- Company research: `"research Velph"`
- Person research: `"look up Hanne"` or `"who is Karim at Velph"`
- Domain lookup: `"intel on velph.be"`
- Pre-meeting: `"research <company> before my call"`

#### Step 2: Pull existing context first
Always check what Nova already knows before going to the web:
- `mcp__aimind-memory__search_entities(query="<name>")`
- `mcp__aimind-memory__search_memory(query="<company> <person>")`
- `Grep` for the entity in `~/obsidian/nova/Projects/Velph/` and related folders
- Past Gmail threads: `mcp__google__gmail_users_messages_list(q="<person or domain>")`
- Past calendar interactions: `mcp__google__calendar_events_list(q="<name>", timeMin=-365d)`

#### Step 3: Web research (run in parallel)
```
WebSearch: "<company>"                  → homepage, positioning
WebSearch: "<company> news"             → last 90 days
WebSearch: "<company> funding"          → investment history
WebSearch: "<company> careers"          → hiring signals (growth indicator)
WebSearch: "<person> <company> LinkedIn" → profile
WebSearch: "<company> customers"        → who they serve
```

#### Step 4: Output format
Save to `~/obsidian/nova/Projects/Velph/Research/<company-or-person>-<date>.md`:

```markdown
# Research: <Company or Person>
**Date:** <YYYY-MM-DD>
**Sources:** Web Search + Memory + Past Gmail + Calendar

## Quick Take
<2-3 sentences: who they are, why they might need us, best angle for outreach>

## Company Profile
| Field | Value |
|-------|-------|
| Company | |
| Website | |
| Industry | |
| Size | |
| Headquarters | |
| Founded | |

### What They Do
<1-2 sentence description>

### Recent News (last 90 days)
- <headline> — <date> — <why it matters>

### Hiring Signals
- <N> open roles, notable: <relevant departments>
- Growth indicator: <interpretation>

## Key People
### <Name> — <Title>
- LinkedIn: <URL>
- Background: <prior companies, education>
- Tenure: <time at company>
- Talking Points:
  - <personal hook>
  - <professional hook>

## Prior Relationship (from Nova's memory + Gmail)
- Status: <new / prior touch / active client / dormant>
- Last contact: <date and type>
- Past interactions: <summary>
- Known contacts: <names already in memory>

## Qualification Signals
### Positive
- ✅ <signal + evidence>

### Concerns
- ⚠️ <concern + what to watch for>

### Unknown — Ask in Discovery
- ❓ <gap>

## Recommended Approach
- **Best entry point:** <person + why>
- **Opening hook:** <what to lead with>
- **Discovery questions:**
  1. <question about their situation>
  2. <question about pain points>
  3. <question about decision process>

## Sources
- <URL>
- Memory: <fact recalled>
- Obsidian: <path>
```

#### Step 5: Store new facts
Anything novel learned from this research → `mcp__aimind-memory__store_memory(text="...", metadata="persistent,velph,client,extracted")`. Tag person names with `permanent` if they're confirmed client contacts.

---

### 2. Call Prep

**Trigger phrases**: "prep for <person/meeting>", "call prep", "I have a meeting with <name>".

#### Step 1: Identify the meeting
- Look up calendar: `mcp__google__calendar_events_list(q="<name or topic>", timeMin=now, timeMax=+14d)`
- Confirm with Dave if ambiguous.

#### Step 2: Gather context (run in parallel)
- Past Gmail thread with attendees
- Last 3 meetings with same attendees (calendar history)
- Memory recall on each attendee
- Obsidian project tracker for the related project
- If client is new: run **Account Research** sub-workflow first

#### Step 3: Output — a tight prep brief
Save to `~/obsidian/nova/Daily/<YYYY-MM-DD>-prep-<meeting-name>.md`:

```markdown
# Call Prep — <meeting title>
**When:** <date + time>
**Attendees:** <list>
**Location:** <physical / Meet link>

## Quick Take
<2 sentences: what this meeting is for, what Dave wants out of it>

## Attendees
### <Name>
- Role: <title at company>
- Background: <relevant prior context>
- Last interaction: <when + topic>
- Talking point: <personal/professional hook>

## Recent Threads (Gmail, last 14 days)
- <subject> — <date> — <one-line gist>

## Open Items / Action History
- <task> — status: <pending/in-progress> — owner: <Dave / them>

## Suggested Agenda
1. <topic>
2. <topic>
3. <decision/next-step ask>

## Anticipated Questions
- They might ask: <question> → Suggested answer: <one line>

## Dave's Asks
- <thing Dave wants from this meeting>

## Risks / Things to Avoid
- <topic that's still sensitive or unresolved>
```

#### Step 4: Deliver
Send the TLDR via Telegram 30 min before the meeting if scheduled in advance, else immediately.

---

### 3. Draft Outreach

**Trigger phrases**: "draft outreach to <name>", "write a message to <name>", "draft a follow-up".

#### Step 1: Gather the basics
- Recipient (run Account Research if Nova doesn't know them)
- Channel: email (default) / LinkedIn message (Dave drafts manually) / WhatsApp
- Goal: introduction, follow-up, re-engagement, ask
- Tone: formal / conversational / Dutch-French-English

#### Step 2: Draft via aimind-email
**Always create a draft, never send.** Use the `aimind-email` skill flow: `mcp__google__gmail_users_drafts_create`. Personal touches:
- Reference one specific thing from research (their recent news, their role, a mutual connection)
- One clear ask, not a wall of options
- Short — 4-6 sentences is the sweet spot
- Match language to recipient (Dutch/French/English based on memory of past comms)

#### Step 3: Present for confirmation
**Hard rule** (from SOUL.md): no email gets sent without Dave's explicit "go" / "send it". Show the draft in the chat, link to the Gmail draft URL, wait.

---

### 4. Competitive Intelligence

**Trigger phrases**: "competitive analysis", "competitors to <X>", "who else is doing <X>", "what's the landscape".

#### Step 1: Define the comparison
- What product/service are we comparing?
- Who are the named competitors? (ask Dave or infer from prior research)
- What dimensions matter? (price, feature set, target market, positioning, recent moves)

#### Step 2: Run parallel research per competitor
For each: web search homepage, news, pricing page, recent product launches, customer base.

#### Step 3: Output — comparison matrix
Save to `~/obsidian/nova/Projects/Velph/Research/competitive-<topic>-<date>.md`:

```markdown
# Competitive Analysis — <topic>
**Date:** <date>

## TLDR
<2-3 sentences: the landscape, where Velph fits, biggest threat>

## Comparison Matrix
| Dimension | Velph | Competitor A | Competitor B | Competitor C |
|-----------|-------|--------------|--------------|--------------|
| Target market | | | | |
| Pricing | | | | |
| Key features | | | | |
| Recent moves | | | | |
| Strengths | | | | |
| Weaknesses | | | | |

## Per-Competitor Deep Dive
### Competitor A
- Positioning:
- Pricing:
- Notable customers:
- Strengths:
- Weaknesses:
- Recent activity:
- Threat level: 🔴 high / 🟡 medium / 🟢 low

[Repeat per competitor]

## Implications for Velph
- <strategic takeaway 1>
- <strategic takeaway 2>
- <gap Velph could exploit>

## Recommended Actions
1. <action>
2. <action>

## Sources
- <URL>
```

---

## Storage rules (Nova-specific)

| Output type | Path |
|-------------|------|
| Account research | `~/obsidian/nova/Projects/Velph/Research/<entity>-<date>.md` |
| Call prep | `~/obsidian/nova/Daily/<YYYY-MM-DD>-prep-<meeting>.md` |
| Outreach drafts | Gmail draft (via `aimind-email`) — never written to disk |
| Competitive analysis | `~/obsidian/nova/Projects/Velph/Research/competitive-<topic>-<date>.md` |

**Never use Google Drive for internal tracking** (per SOUL.md). External-facing client docs are the only Drive use case, and those go through `aimind-offer`.

## Relationship to other skills

| Stage | Skill |
|-------|-------|
| Pre-sales research, prep, outreach | **velph-sales** (this skill) |
| Proposal drafting | `aimind-offer` |
| Project execution tracking | Obsidian tracker + `aimind-life-admin` |
| Client-facing email send | `aimind-email` (draft + confirm gate) |

## Migration notes

- Sources merged from `anthropic/knowledge-work-plugins/sales/skills/`:
  - `account-research/SKILL.md`
  - `call-prep/SKILL.md`
  - `draft-outreach/SKILL.md`
  - `competitive-intelligence/SKILL.md`
- Skipped Anthropic's `pipeline-review`, `forecast`, `daily-briefing`, `call-summary`, `create-an-asset` — not relevant to Dave's freelance scale (no pipeline, no forecast cycle).
- Stripped CRM connectors (Salesforce/HubSpot) and enrichment connectors (Apollo/Clearbit) — replaced with Nova's memory + Gmail + Obsidian.
- Renamed from generic `sales` to `velph-sales` because the workflow assumes a single-client, project-based engagement model. If Dave takes on a second client of similar shape, fork this skill (e.g. `inno-sales`).
- Tier is `org` not `persona` — Vera (Velph persona) should also have access to this skill content.

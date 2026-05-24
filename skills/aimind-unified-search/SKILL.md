---
name: aimind-unified-search
description: Unified cross-source search — decomposes a query, fans out in parallel to Gmail, Calendar, Obsidian vault, semantic memory, and Drive, then synthesizes a single answer with source attribution. Use when Dave asks "did I talk about X recently", "find that doc about Y", "where was the conversation about Z" — questions that require hitting 3+ sources manually.
tier: installed
catalog_entry: aimind-unified-search
user-invocable: true
enabled: true
priority: 4

allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - WebSearch
  - mcp__google__gmail_users_messages_list
  - mcp__google__gmail_users_messages_get
  - mcp__google__gmail_users_threads_list
  - mcp__google__gmail_users_threads_get
  - mcp__google__calendar_events_list
  - mcp__google__drive_files_list
  - mcp__google__drive_files_get
  - mcp__aimind-memory__search_memory
  - mcp__aimind-memory__search_conversations
  - mcp__aimind-memory__search_entities

personality-mode: warm

metadata:
  category: search
  channel: telegram
  author: nova-catalogue
  source_attribution: anthropic/knowledge-work-plugins enterprise-search/search
  triggers:
    - search everything
    - find across
    - search all sources
    - unified search
    - did I talk about
    - find that doc about
    - where was the conversation
    - search my stuff
    - cross-source search
---

# AIMind Unified Search

Single-query, multi-source search. Decompose Dave's question, fan out to every relevant source in parallel, then synthesize one answer with attribution. This is the antidote to "I had to ask Nova three times — once for email, once for memory, once for Obsidian."

Adapted from Anthropic's `enterprise-search/search` skill. The decomposition + parallel-fanout + synthesis pattern is the value; the connector list is mapped to Nova's actual stack.

## Workflow

### 1. Parse the query

Identify:
- **Intent**: factual ("when did I decide X"), exploratory ("what do I know about Y"), or find-the-thing ("where's the spec for Z")
- **Entities**: people (Dave's contacts), projects (Wolters-Kluwer, Velph, Hiveras, HurenKopen, etc.), client names, technical terms
- **Time constraints**: "this week", "last month", explicit dates → translate to ISO ranges
- **Source hints**: "in email", "the doc", "you remember that thing about" — bias the fan-out
- **Filters** (explicit): `from:`, `in:`, `after:`, `before:`, `type:`

### 2. Decompose into source-specific sub-queries

For each available source, build a targeted sub-query:

| Source | Tool(s) | Query shape |
|--------|---------|-------------|
| Gmail | `mcp__google__gmail_users_messages_list` with `q=` | Gmail search syntax: `from:`, `to:`, `subject:`, `after:`, `before:`, free text |
| Calendar | `mcp__google__calendar_events_list` with `q=` + `timeMin`/`timeMax` | Event title/description match within time range |
| Drive | `mcp__google__drive_files_list` with `q=` | Drive query syntax: `name contains`, `fullText contains`, `modifiedTime >` |
| Semantic memory | `mcp__aimind-memory__search_memory` | Natural language query — best for "what do I know about" |
| Past conversations | `mcp__aimind-memory__search_conversations` | Natural language — best for "did we discuss" |
| Entities (Graphiti) | `mcp__aimind-memory__search_entities` | Named entity lookup — people, projects, places |
| Obsidian vault | `Grep` with `path=~/obsidian/nova/` | Ripgrep over markdown — best for "find that note" |
| Web (last resort) | `WebSearch` | Only when the question is clearly external |

### 3. Execute in parallel

Run **all** sub-queries in a **single message** with multiple tool calls. Never serialize unless one source depends on another.

```
[Parallel batch]
→ mcp__google__gmail_users_messages_list(q="...")
→ mcp__google__calendar_events_list(q="...")
→ mcp__google__drive_files_list(q="...")
→ mcp__aimind-memory__search_memory(query="...")
→ mcp__aimind-memory__search_conversations(query="...")
→ Grep(pattern="...", path="~/obsidian/nova/")
```

Capture each result with metadata: timestamps, authors, links, source type. If a source fails (auth expired, rate limit), note it but don't block — synthesize from what you have.

### 4. Deduplicate and rank

- **Dedup**: if the same decision/fact shows up in email AND memory AND Obsidian, group them and prefer the most authoritative version (typically Obsidian project tracker > memory > email thread).
- **Rank by**:
  - **Relevance** to intent (semantic match)
  - **Freshness** for status/decision queries
  - **Authority**: Obsidian project trackers > pinned memories > email > calendar > chat
  - **Completeness**: results with more context rank higher

### 5. Synthesize, don't dump

**Never return a raw list of hits.** Synthesize into prose with attribution.

**For factual/decision queries:**
```
[Direct answer]

Sources:
- Email from <person> on <date> — <one-line summary>
- Obsidian: ~/obsidian/nova/Projects/<project>/<file>.md
- Memory: <fact recalled>
```

**For exploratory queries:**
```
[2-3 paragraph synthesis combining all sources]

Found across:
- Gmail: 4 relevant threads (most recent: <date>)
- Memory: 7 facts, 2 conversations
- Obsidian: 3 notes under Projects/<category>/
- Calendar: 1 meeting on <date>

Key sources:
- [Most important] — <link or path>
- [Second] — <link or path>
```

**For find-the-thing queries:**
```
Found it: <direct reference>

Also nearby:
- <related item 1>
- <related item 2>
```

### 6. Handle edge cases

**Ambiguous query** (could mean 2+ different things):
Ask one clarifying question BEFORE searching:
> "WK could mean Wolters-Kluwer (Belgian freelance gig) or Wendy's catering plan. Which one?"

**No results**:
> "I couldn't find anything matching '<query>' across Gmail, memory, Obsidian, or Drive. Try:
> - Broader terms (e.g. 'invoicing' instead of 'invoice #2026-031')
> - Different time range (currently searching last 90 days)
> - Adding a project name if you know it"

**Partial results** (one source failed):
> "[Synthesis from successful sources]
>
> Note: Gmail was unavailable for this search (token refresh needed). Results above are from memory, Obsidian, and Drive only."

## Volatile vs stable facts

When the synthesis includes facts from the **volatile** memory tier (deploy URLs, branch names, service status), flag them: `(verify — may have changed)`. Don't serve a Vercel URL from memory without checking it's still live.

## Notes for the future

- **`/digest` complement**: A weekly cross-source activity digest (last 7 days of email + calendar + Obsidian dailies + Telegram archives) feeding the morning briefing. Track as a follow-up; not in v0.1.
- **Confidence scoring**: Anthropic's source has explicit confidence. For v0.1, skip — synthesis tone implies confidence. Revisit if Dave asks "how sure are you?"
- **Telegram archive search**: Once the Telegram archive indexer lands, add it as a fan-out target. Path: `~/obsidian/nova/Archives/Chats/`.

## Example

> Dave: "did I talk to Hanne about the BodemBrief pricing last month?"

1. Parse: intent = decision recall; entity = Hanne (Velph contact), BodemBrief; time = last 30 days.
2. Decompose:
   - Gmail: `from:hanne OR to:hanne after:2026-04-24 BodemBrief OR pricing`
   - Memory: `Hanne BodemBrief pricing`
   - Conversations: `Hanne BodemBrief pricing last month`
   - Obsidian grep: `BodemBrief` in `~/obsidian/nova/Projects/Velph/`
3. Fan out in parallel.
4. Synthesize:
   > "Yes — three touch points. You and Hanne exchanged emails on May 8 and May 14 about the €X tier. The May 14 thread ended with her asking for a revised quote (still open). Memory has the pricing decision logged ("Dave landed on a two-tier model"). Obsidian has the full thread in Projects/Velph/BodemBrief/tracker.md.
   >
   > Sources:
   > - Gmail: 2 threads with hanne@velph.be (May 8, May 14)
   > - Memory: pricing decision fact (persistent tier)
   > - Obsidian: ~/obsidian/nova/Projects/Velph/BodemBrief/tracker.md"

## Migration notes

- Source: `anthropic/knowledge-work-plugins/enterprise-search/skills/search/SKILL.md`
- Original used placeholder connectors (`~~chat`, `~~email`, `~~cloud storage`). Replaced with Nova's actual MCP tools.
- Original had a separate `digest` skill — captured as a follow-up here, not yet ported.
- Original used the Claude Cowork `argument-hint` field — dropped (not in AIMind schema).

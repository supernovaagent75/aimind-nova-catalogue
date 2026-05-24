---
name: aimind-productivity-update
description: Weekly retrospective gap-finder — scans the last 7 days of email, calendar, Telegram archives, and Obsidian dailies to surface missed todos, stale tasks, unresolved threads, and memory candidates (new people/projects mentioned but not yet recorded). Outputs a structured gap report. Complements the forward-looking morning briefing.
tier: installed
catalog_entry: aimind-productivity-update
user-invocable: true
enabled: true
priority: 3

allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - mcp__google__gmail_users_messages_list
  - mcp__google__gmail_users_messages_get
  - mcp__google__gmail_users_threads_list
  - mcp__google__calendar_events_list
  - mcp__aimind-memory__search_memory
  - mcp__aimind-memory__search_conversations
  - mcp__aimind-memory__store_memory
  - mcp__aimind-memory__list_tasks
  - mcp__aimind-memory__create_task
  - mcp__aimind-memory__update_task

personality-mode: warm

metadata:
  category: productivity
  channel: telegram
  author: nova-catalogue
  source_attribution: anthropic/knowledge-work-plugins productivity/update
  triggers:
    - weekly update
    - weekly review
    - what did I miss
    - productivity review
    - retrospective scan
    - gap report
    - catch up
    - update --comprehensive
---

# AIMind Productivity Update (Weekly Retrospective)

Forward-looking briefings tell Dave what's coming. This skill is the **mirror**: a retrospective deep-scan of the last 7 days that surfaces what slipped through — missed todos buried in chat threads, unresolved email questions, people he met but hasn't logged, projects mentioned but not tracked.

Adapted from Anthropic's `productivity/update --comprehensive` command. The "scan recent activity, compare against tracked state, flag gaps" pattern is the value.

## When to run

- **Weekly** — Sunday evening or Monday morning, before the week starts properly.
- **On demand** — when Dave says "what did I miss?" or "weekly update".
- **Never auto-add** — every flagged item is presented for confirmation. This is a triage skill, not an executor.

## Workflow

### 1. Load current state

Pull what's already tracked:
- Active tasks: `mcp__aimind-memory__list_tasks` (status in [pending, in_progress, blocked])
- Recent memories: `mcp__aimind-memory__search_memory("", limit=50)` filtered to last 14 days
- Known entities: rely on memory and Obsidian for project + people context

### 2. Scan activity sources (parallel)

Fan out to all sources covering the last 7 days:

| Source | Tool | Query |
|--------|------|-------|
| Gmail received | `mcp__google__gmail_users_messages_list` | `q="newer_than:7d -from:me"` |
| Gmail sent | `mcp__google__gmail_users_messages_list` | `q="newer_than:7d from:me"` |
| Calendar past | `mcp__google__calendar_events_list` | `timeMin=-7d, timeMax=now` |
| Calendar upcoming | `mcp__google__calendar_events_list` | `timeMin=now, timeMax=+7d` |
| Obsidian dailies | `Glob` | `~/obsidian/nova/Daily/2026-*.md` (last 7 days) |
| Telegram archives | `Grep` | `~/obsidian/nova/Archives/Chats/` last 7 days |
| Past conversations | `mcp__aimind-memory__search_conversations` | recent — semantic scan for action language |

### 3. Extract action signals

Scan each source for action language:
- **First-person commitments**: "I'll send", "I'll review", "I'll get back to you", "let me check"
- **Open questions to Dave**: "?" in inbound emails older than 48h with no reply from him
- **Recurring meetings with no associated tasks**: standing calls (Velph standup, Nike sync) that have no follow-up tracked
- **Deadlines mentioned**: "by Friday", "before end of month", explicit dates
- **People mentioned 3+ times** not in memory
- **Projects/topics mentioned 5+ times** not in memory

### 4. Compare against tracked state

For each extracted signal:

| Signal | Tracked? | Action |
|--------|----------|--------|
| "I'll send the PRD by Friday" | Not in tasks | Flag → "Add as task?" |
| Email from client unanswered 4 days | Not in tasks | Flag → "Reply or add follow-up?" |
| Task "Send PRD" exists, status=done in chat | Tracked, not closed | Flag → "Mark done?" |
| Person "Karim" mentioned 6 times | Not in entities | Flag → "Add to memory?" |
| Task "Old thing" in active 45 days, no recent mention | Tracked but stale | Flag → "Still relevant? Move to someday/cancel?" |

### 5. Generate the gap report

Structure as a single markdown report (write to `~/obsidian/nova/Daily/<YYYY-MM-DD>-weekly-update.md`):

```markdown
# Weekly Update — <date>
Scan window: <start_date> → <end_date>

## TLDR
<1-2 sentence summary: how many items need attention, biggest theme>

## Possible Missing Tasks
1. **From email (May 18):** You told <person>: "I'll send the contract by Wed" — no task tracked.
   → Add to TASKS?
2. **From Telegram with Wendy (May 19):** "I'll book the restaurant for Saturday" — no calendar event, no task.
   → Book it now / Add reminder?

## Unresolved Email Threads (>48h, you're the last expected reply)
- <subject> — from <person> — <N> days old — gist: <one line>
- ...

## Stale Tracked Tasks
- "Old task X" — created <date>, no movement in 30+ days. Still relevant?
- ...

## New People to Add to Memory
| Name | Frequency | Context |
|------|-----------|---------|
| Karim | 6 mentions | Wolters-Kluwer team, mentioned in 3 emails + 1 calendar |
| ... | | |

## New Projects/Topics to Track
| Name | Frequency | Context |
|------|-----------|---------|
| Project Aurora | 8 mentions | Internal Nike rename of checkout-v2 |
| ... | | |

## Wins of the Week (auto-extracted)
- <thing shipped, decided, or closed>
- ...

## Recommended Top 3 Actions
1. <highest-priority gap with rationale>
2. ...
3. ...
```

### 6. Triage interactively

Deliver the gap report via the channel (Telegram). Then for each flagged item, wait for Dave's decision:
- **Add as task** → `mcp__aimind-memory__create_task` with linked source
- **Add to memory** → `mcp__aimind-memory__store_memory` with appropriate tier tag (permanent for people, persistent for projects)
- **Mark done** → `mcp__aimind-memory__update_task` status=done
- **Cancel** → `mcp__aimind-memory__update_task` status=cancelled
- **Skip** → no action

**Never bulk-execute.** Every action requires explicit confirmation. Dave's brain is the filter, not yours.

### 7. Run light vs heavy

- **Light mode** (default for weekly heartbeat): produce the report, store it in Obsidian, send the TLDR + top 3 actions to Telegram. No interactive triage unless Dave responds.
- **Heavy mode** (when Dave invokes interactively): produce the report and walk through each flagged item conversationally.

## Tone

This skill runs against a week of Dave's life. Be honest about gaps without being judgmental — the point is to catch what slipped, not to shame the slip. Frame everything as "want me to..." not "you forgot to...".

## Notes

- Run via heartbeat scheduler, Sunday 18:00 CET. Add to `~/.aimind/personas/nova/schedules.yaml` once skill is activated.
- Light mode output always lands in Obsidian even if Dave doesn't engage — creates a paper trail.
- Pair with `aimind-unified-search` for follow-ups: "tell me more about the Karim thread" → unified search hits Gmail+memory+Obsidian.

## Migration notes

- Source: `anthropic/knowledge-work-plugins/productivity/skills/update/SKILL.md`
- Anthropic's version manages a `TASKS.md` file + `memory/` directory. Nova uses `mcp__aimind-memory__*` tools instead — semantically equivalent but no filesystem state to sync.
- Default+comprehensive modes collapsed to light/heavy. The "comprehensive" deep scan is the primary value; the default sync mode is partially redundant with Nova's existing memory consolidation.
- Skipped Anthropic's external task-tracker sync (Linear/Asana/Jira) — Dave doesn't use them.
- Added Telegram archive scanning — Nova-specific source not present in Anthropic's version.

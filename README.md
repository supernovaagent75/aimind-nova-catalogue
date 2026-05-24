# aimind-nova-catalogue

Nova's personal AIMind skill catalogue — a **staging ground** for new skills before they're promoted to the main `aimind` repo or activated in `~/.aimind/`.

## Why this exists

The AIMind platform repo (`aimind`) holds the canonical platform skills. The instance state directory (`~/.aimind/`) holds the live persona + org skills.

This repo sits in between: a place where Nova can draft new skills, iterate on them, get Dave's review, and validate them in conversation **before** they go live or get promoted.

Think of it as Nova's notebook — sketches that may or may not become production code.

## Repository structure

```
aimind-nova-catalogue/
├── README.md                  ← this file
├── MIGRATION.md               ← promotion checklist + workflow
└── skills/
    ├── aimind-unified-search/        SKILL.md
    ├── aimind-productivity-update/   SKILL.md
    ├── aimind-spec-writing/          SKILL.md
    └── velph-sales/                  SKILL.md
```

## Current catalogue (v0.1.0)

| Skill | Tier | Source plugin | Status |
|-------|------|---------------|--------|
| `aimind-unified-search` | persona | anthropic/knowledge-work-plugins `enterprise-search/search` | drafted, awaiting review |
| `aimind-productivity-update` | persona | anthropic/knowledge-work-plugins `productivity/update --comprehensive` | drafted, awaiting review |
| `aimind-spec-writing` | persona | anthropic/knowledge-work-plugins `product-management/{write-spec, product-brainstorming}` | drafted, awaiting review |
| `velph-sales` | org | anthropic/knowledge-work-plugins `sales/{account-research, call-prep, draft-outreach, competitive-intelligence}` | drafted, awaiting review |

## SKILL.md format

All skills here use the canonical AIMind SKILL.md schema:

```yaml
---
name: <kebab-case-id>
description: <one paragraph — what + when to trigger>
version: "<semver>"
tier: <persona | org | platform>
user-invocable: true
enabled: true
priority: <1-5, higher = more important>

allowed-tools:
  - Bash
  - mcp__google__gmail_users_messages_list
  - ...

personality-mode: <warm | professional>

metadata:
  category: <search | productivity | product | sales | ...>
  channel: telegram
  author: nova
  source_attribution: <where this was ported from>
  triggers:
    - <trigger phrase>
    - ...
---

# <Skill Name>

<body — markdown instructions for the persona>
```

See `MIGRATION.md` for the full promotion checklist.

## Workflow

1. **Draft here.** Author the SKILL.md in this repo. Commit. Push.
2. **Dave reviews.** Reads the SKILL.md, asks questions, suggests changes.
3. **Iterate.** Nova edits via task delegation, recommits.
4. **Activate.** Copy the skill to `~/.aimind/personas/nova/skills/` (persona tier) or `~/.aimind/org/skills/` (org tier). Commit to the instance state repo per `~/.aimind/AGENTS.md`.
5. **Test in conversation.** Verify triggers fire, tools work, output lands in the right place.
6. **Promote (optional).** If the skill is generic enough to belong in the platform, open a PR against the main `aimind` repo to move it to `aimind/skills/`.

## Related repos

- **Platform code:** main `aimind` repo (private) — canonical platform skills live in `aimind/skills/`.
- **Instance state:** `~/.aimind/` — live persona/org skills, governed by `~/.aimind/AGENTS.md`.
- **Source inspiration:** [anthropics/knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) — Anthropic's reference plugin catalogue, the seed for v0.1.0 of this repo.

## License

Skills in this repo are derivative work of Anthropic's open-source [knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) (MIT). See each SKILL.md `source_attribution` metadata for specific lineage.

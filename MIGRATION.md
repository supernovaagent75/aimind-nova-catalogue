# Migration Guide

How skills move through the stages: **draft (here) → activated (~/.aimind/) → promoted (main aimind repo)**.

## Per-skill migration checklist

Use this checklist for each skill before activating it in the live instance.

### Stage 1: Validation in this repo

- [ ] SKILL.md has valid YAML frontmatter (name, description, version, tier, enabled, allowed-tools, metadata)
- [ ] `name` matches the directory name (kebab-case)
- [ ] `description` is one paragraph, explains both *what* the skill does and *when* it should trigger
- [ ] `tier` is correct: `persona` (Nova-only), `org` (shared across host's personas), or `platform` (shared across instances)
- [ ] `allowed-tools` only lists tools Nova actually has access to
- [ ] `metadata.triggers` includes 3-8 natural-language trigger phrases
- [ ] `metadata.source_attribution` records where the skill was ported from (if applicable)
- [ ] Body references real paths and real tools — no placeholder `~~chat` / `~~email` connectors
- [ ] At least one concrete example included
- [ ] Output location is correct per `~/obsidian/nova/agents.md`:
  - Research → `Research/`
  - Project work → `Projects/<Category>/<Project>/`
  - Daily briefings/prep → `Daily/YYYY-MM-DD-*.md`
  - System logs → `System/`

### Stage 2: Dave's review

- [ ] Dave has read the SKILL.md
- [ ] Triggers feel natural for how Dave actually talks
- [ ] Output format matches what Dave would want
- [ ] Tool list is minimal (no over-permissioning)
- [ ] No conflicts with existing skills (check `~/aimind/skills/` and `~/.aimind/{personas,org}/skills/`)

### Stage 3: Activation in ~/.aimind/

For **persona-tier skills**:
```bash
mkdir -p ~/.aimind/personas/nova/skills/<skill-name>
cp skills/<skill-name>/SKILL.md ~/.aimind/personas/nova/skills/<skill-name>/SKILL.md
cd ~/.aimind
git add personas/nova/skills/<skill-name>/
git commit -m "feat(nova): activate <skill-name> skill from nova-catalogue"
git push
```

For **org-tier skills**:
```bash
mkdir -p ~/.aimind/org/skills/<skill-name>
cp skills/<skill-name>/SKILL.md ~/.aimind/org/skills/<skill-name>/SKILL.md
cd ~/.aimind
git add org/skills/<skill-name>/
git commit -m "feat(org): activate <skill-name> skill from nova-catalogue"
git push
```

- [ ] Skill copied to correct location
- [ ] `~/.aimind/` committed and pushed per `~/.aimind/AGENTS.md`
- [ ] Skill appears in the next session's "Your Skills" list
- [ ] Test trigger phrase fires the skill correctly

### Stage 4: Conversation testing

- [ ] Trigger phrase activates the skill
- [ ] All `allowed-tools` work without permission errors
- [ ] Output lands in the documented location
- [ ] Result quality matches what the SKILL.md promises
- [ ] No unintended side effects (e.g. sending email without confirmation)

### Stage 5 (optional): Promotion to platform

If the skill is generic enough to benefit any AIMind instance (not just Nova / Dave's setup):

- [ ] Generalize: remove Nova-specific paths, Dave-specific examples
- [ ] Use `{persona}` / `{user}` template placeholders where appropriate
- [ ] Move tier from `persona`/`org` to `platform`
- [ ] Open PR against main `aimind` repo: `aimind/skills/<skill-name>/SKILL.md`
- [ ] Update this catalogue's status to "promoted"

## Versioning

Skills in this repo use semver:
- `0.x.y` — draft, expect breaking changes
- `1.0.0` — activated in `~/.aimind/` and validated in conversation
- `2.0.0+` — promoted to main platform and adopted by other instances

Bump the `version` field in SKILL.md frontmatter when iterating.

## Source attribution

When porting from an external source (e.g. Anthropic's plugins), always record:
- The original repo + path in `metadata.source_attribution`
- A "Migration notes" section at the bottom of the SKILL.md explaining what was adapted, merged, skipped, or renamed

## Known issues / gotchas

- **`mcp__google__*` tool names**: the exact tool names depend on the connector version installed. Verify against the live tool list in a fresh Nova session before activation.
- **Trigger conflicts**: if two skills share a trigger phrase, the one with higher `priority` wins. Audit triggers against existing skills before activating.
- **Tier ≠ location**: `tier: platform` in frontmatter does NOT mean the file lives in the platform repo. It controls the discovery scope at runtime. The file location is determined by stage 3 above.

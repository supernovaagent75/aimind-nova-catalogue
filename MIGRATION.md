# Migration Guide

How an entry moves through the lifecycle: **draft → cataloged → installed → promoted**.

> **Update (2026-05-24):** This catalog is now a proper AIMind catalog
> (catalog.json + CalVer tags + `aimind catalog install` support). The
> "manual copy into `~/.aimind/`" workflow described in older revisions
> of this file has been replaced by the standard install CLI. See
> [`AGENTS.md`](./AGENTS.md) and [`README.md`](./README.md).

## Per-skill lifecycle

### Stage 1 — Draft in this repo

- [ ] `skills/<entry>/SKILL.md` has valid YAML frontmatter.
- [ ] `name` matches the directory name (kebab-case).
- [ ] `description` is one paragraph, explains both *what* the skill does and
      *when* it should trigger.
- [ ] `tier: installed`, `enabled: true`, `catalog_entry: <entry>` are all set.
- [ ] **No `version:` line** — pinned at install time by `aimind catalog install`.
- [ ] `allowed-tools` only lists tools that exist on the target instance.
- [ ] `metadata.triggers` includes 3-8 natural-language trigger phrases
      (the engine reads triggers from either top-level `triggers:` or
      `metadata.triggers:`).
- [ ] `metadata.source_attribution` records where the skill was ported from
      (if applicable).
- [ ] Body references real paths and real tools — no placeholder connectors.
- [ ] Output paths follow `~/obsidian/nova/agents.md`:
  - Research → `Research/`
  - Project work → `Projects/<Category>/<Project>/`
  - Daily briefings / prep → `Daily/YYYY-MM-DD-*.md`
  - System logs → `System/`

### Stage 2 — Catalog the entry

- [ ] Add the entry to `catalog.json` with a CalVer `latest_version`
      and a sensible `min_platform_version`.
- [ ] Commit on `main`: `git add -A && git commit -m "feat: add <entry>"`.
- [ ] Push: `git push`.
- [ ] Tag the release: `git tag <entry>@YYYY.MM.PATCH && git push --tags`.

### Stage 3 — Install on the live instance

```bash
aimind catalog install <entry> \
    --catalog https://github.com/thetwit4u/aimind-nova-catalogue
```

- [ ] `~/.aimind/skills/<entry>/SKILL.md` exists and has `version:` pinned.
- [ ] `~/.aimind/installed.json` has the entry recorded.
- [ ] Skill appears in the next session's "Your Skills" list.

### Stage 4 — Conversation testing

- [ ] Trigger phrase activates the skill.
- [ ] All `allowed-tools` work without permission errors.
- [ ] Output lands in the documented location.
- [ ] Result quality matches what the SKILL.md promises.
- [ ] No unintended side effects (e.g. sending email without confirmation).

### Stage 5 — Iterate

To ship a fix:

1. Edit `skills/<entry>/SKILL.md`.
2. Bump `latest_version` in `catalog.json` (CalVer; PATCH+1 in the same
   month, or reset to `1` if it's a new month).
3. Commit + push + tag (`git tag <entry>@YYYY.MM.PATCH && git push --tags`).
4. On the live instance: `aimind catalog upgrade <entry>`.

### Stage 6 — Promotion (optional)

A skill in this catalog can be promoted in two directions:

- **To the org catalog** (`hiveras/aimind-catalog`) — when the skill is
  generic enough to benefit any Hiveras persona (Dennis, Vera, future personas).
- **To the platform** (`hiveras/aimind`) — when the skill is foundational
  enough to benefit every AIMind instance, not just Hiveras-aligned ones.
  Open a PR against `aimind/skills/<entry-name>/`.

When promoting:
- [ ] Generalize: remove Nova-specific paths/examples.
- [ ] Update `tier:` (`installed` → `platform` if going to the platform repo;
      keep `installed` if moving to `hiveras/aimind-catalog`).
- [ ] Delete the entry from `catalog.json` here and the `skills/<entry>/` folder
      in this repo (uninstall locally first with `aimind catalog uninstall`).

## Source attribution

When porting from an external source (e.g. Anthropic's plugins), always record:

- The original repo + path in `metadata.source_attribution`.
- A "Migration notes" section at the bottom of the `SKILL.md` explaining what
  was adapted, merged, skipped, or renamed.

## Known issues / gotchas

- **`mcp__google__*` tool names**: the exact tool names depend on the connector
  version installed. Verify against the live tool list in a fresh Nova session
  before tagging a release.
- **Trigger conflicts**: if two skills share a trigger phrase, the one with
  higher `priority` wins. Audit triggers against existing skills
  (`~/aimind/skills/`, `~/.aimind/{personas,org,skills}/`) before tagging.
- **Tier ≠ location**: `tier: installed` is the runtime discovery tier; the
  install destination is always `~/.aimind/skills/<entry>/` regardless of the
  source catalog.

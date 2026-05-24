# AGENTS.md — `thetwit4u/aimind-nova-catalogue`

This file is read by AI coding agents (Claude Code, Nova workers, etc.) before
they make changes in this repository. It captures contribution rules, repo
conventions, and the catalog spec this repo conforms to.

## Repo purpose

`aimind-nova-catalogue` is a **package registry** of Nova-specific installable
AIMind skills. It is **not** the platform engine — that lives in `hiveras/aimind`.
It is also not the org-wide catalog — that lives in
[`hiveras/aimind-catalog`](https://github.com/hiveras/aimind-catalog) and holds
skills shared across all Hiveras personas.

If you find yourself wanting to modify the persona engine, MCP wiring, or the
`aimind catalog` CLI itself — you are in the wrong repo (that's `hiveras/aimind`).
If the skill is generic enough to benefit Dennis, Vera, or any future Hiveras
persona — promote it to `hiveras/aimind-catalog` instead.

Entries here are pulled into Nova's AIMind instance on demand via
`aimind catalog install <entry> --catalog https://github.com/thetwit4u/aimind-nova-catalogue`.
The CLI clones this repo at the requested CalVer tag and copies the entry's
`skills/<name>/` into `~/.aimind/skills/<name>/`.

## Repository structure

```
aimind-nova-catalogue/
├── catalog.json                # Manifest of all entries (see schema below)
├── AGENTS.md                   # This file
├── README.md                   # Install instructions for humans
├── skills/
│   └── <entry-name>/           # One folder per skill entry
│       ├── SKILL.md            # Required: frontmatter + skill body
│       └── ...                 # Helper scripts the skill calls (optional)
```

No `apps/` directory yet — Nova's catalog ships skills only. If a future
entry needs a paired app, add `apps/<entry-name>/` and set
`"kind": "skill+app"` in `catalog.json` (see `hiveras/aimind-catalog` for the
reference layout).

## `catalog.json` schema (v1)

```json
{
  "schema_version": "1",
  "catalog": "nova-catalogue",
  "homepage": "https://github.com/thetwit4u/aimind-nova-catalogue",
  "entries": [
    {
      "name": "<entry-name>",
      "kind": "skill",
      "skill_path": "skills/<entry-name>",
      "latest_version": "YYYY.MM.PATCH",
      "min_platform_version": "YYYY.MM.PATCH",
      "description": "...",
      "tags": ["..."],
      "authors": ["nova", "dave"]
    }
  ]
}
```

**Required fields:** `name`, `kind`, `latest_version`, `min_platform_version`, `description`.
**Recommended:** `skill_path` (defaults to `skills/<name>` if omitted), `tags`, `authors`.

`schema_version` is the version of the manifest format itself. It is
independent of CalVer entry versions and rarely changes.

## SKILL.md frontmatter (catalog entries)

A catalog skill is an ordinary AIMind `SKILL.md` plus three rules that
distinguish it from a platform/persona skill:

| Field | Value | Set where |
|---|---|---|
| `tier` | `installed` | committed in source |
| `catalog_entry` | the entry name, e.g. `aimind-unified-search` | committed in source |
| `version` | CalVer `YYYY.MM.PATCH` | **pinned at install time** by `aimind catalog install` — do **not** commit a hardcoded value |

The source `SKILL.md` therefore has **no `version:`** line. `aimind catalog
install` writes the resolved `version:` (and re-asserts `catalog_entry:`)
into the *installed copy* at `~/.aimind/skills/<entry>/SKILL.md` from the
resolved CalVer tag.

`metadata.author: nova-catalogue` (the catalog slug) is the convention here.

## CalVer tag policy

All entries use CalVer git tags:

- **Format:** `<entry-name>@YYYY.MM.PATCH`
- **Examples:** `aimind-unified-search@2026.05.1`, `aimind-spec-writing@2026.05.2`
- `YYYY` four-digit year, `MM` zero-padded month, `PATCH` 1-based integer.
- **PATCH resets to `1` at the start of each new month**, regardless of how
  many releases shipped the previous month.
- Pre-release: append a suffix — `aimind-unified-search@2026.05.1-rc1`.

Two entries in the same month share the `YYYY.MM` prefix but have independent
PATCH counters; they are unrelated tags.

`aimind catalog upgrade <entry>` always pulls the highest CalVer tag for
that entry recorded in `catalog.json#entries[].latest_version`.

`min_platform_version` is itself a CalVer ref against the platform repo's
release tags.

## Contribution workflow

1. Branch off `main`.
2. Add a folder under `skills/<entry-name>/`.
3. Write the `SKILL.md` (no `version:` in frontmatter; use the catalog-skill
   shape above).
4. Add the entry to `catalog.json` with a CalVer `latest_version`.
5. Commit and push.
6. Tag the release: `git tag <entry-name>@YYYY.MM.PATCH && git push --tags`.

To **update** an existing entry: bump `latest_version` in `catalog.json`,
commit, retag.

## Quality bar before tagging a release

- `aimind catalog install <entry> --catalog https://github.com/thetwit4u/aimind-nova-catalogue`
  succeeds on a clean instance (no pre-existing state).
- The install is **idempotent**: running it twice produces the same on-disk
  state and exits cleanly.
- The skill's triggers fire as documented in a live conversation.
- **No platform repo modifications** are required for the install to work.
  If a feature needs platform changes, ship those in `hiveras/aimind` first,
  raise `min_platform_version` in `catalog.json`, then publish the entry.

## Scope note — Nova-personal vs catalog-pure

`hiveras/aimind-catalog` enforces strict neutrality (no hardcoded paths, no
persona-specific logic, no committed secrets) because its entries are pulled
by any persona on any host. This repo (`nova-catalogue`) is narrower in scope:
it ships skills intended for Nova specifically. That means:

- Persona-specific references in skill bodies (Dave, Velph, Nike) are
  acceptable in trigger phrases, examples, and prose.
- Skills must still **not** commit secrets, **not** hardcode `/Users/<name>/`
  paths in code, and **not** assume a specific home-user account. Use `~/` and
  env vars.
- If you find yourself writing a skill that is genuinely generic, prefer
  publishing it to `hiveras/aimind-catalog` instead.

## File and folder conventions

- Entry folder name is lowercase, hyphenated: `aimind-unified-search`, `velph-sales`.
- `SKILL.md` frontmatter minimum: `name`, `description`, `tier: installed`,
  `enabled: true`, `allowed-tools`, `catalog_entry: <entry-name>`.
- Per-entry `README.md` is recommended for human onboarding.
- Per-entry `CHANGELOG.md` is recommended once an entry has more than one
  release.

## When in doubt

Ping Dave. Better to ask than to land something that breaks the install
contract.

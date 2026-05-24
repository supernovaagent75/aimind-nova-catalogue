# aimind-nova-catalogue

Nova's personal AIMind skill catalogue — installable via `aimind catalog install`.

This is a **catalog repo** in the same family as
[`hiveras/aimind-catalog`](https://github.com/hiveras/aimind-catalog).
It serves Nova-specific skills that aren't generic enough to belong in the
platform repo or the org-wide Hiveras catalog, but are still worth distributing
and versioning rather than living unsourced in `~/.aimind/skills/`.

## What's in it

| Entry | Kind | Source plugin | Description |
|-------|------|---------------|-------------|
| `aimind-unified-search` | skill | anthropic/knowledge-work-plugins `enterprise-search/search` | Decompose a query, fan out across Gmail / Calendar / Obsidian / memory / Drive, synthesize one answer. |
| `aimind-productivity-update` | skill | anthropic/knowledge-work-plugins `productivity/update` | Weekly retrospective gap-finder — surfaces missed todos, stale tasks, unrecorded entities. |
| `aimind-spec-writing` | skill | anthropic/knowledge-work-plugins `product-management/{write-spec,product-brainstorming}` | PRD/spec authoring + brainstorm-partner mode. |
| `velph-sales` | skill | anthropic/knowledge-work-plugins `sales/*` | Velph client pre-sales: account research, call prep, outreach drafts, competitive intel. |

See [`catalog.json`](./catalog.json) for the canonical manifest.

## Install

The catalog uses the standard `aimind catalog` CLI shipped with the platform.

### One-off — install a single entry

```bash
aimind catalog install <entry-name> \
    --catalog https://github.com/thetwit4u/aimind-nova-catalogue
```

Example:

```bash
aimind catalog install aimind-unified-search \
    --catalog https://github.com/thetwit4u/aimind-nova-catalogue
```

This will:

1. Clone the catalog to `~/.aimind/cache/catalogs/nova-catalogue/`.
2. Resolve the entry's `latest_version` from `catalog.json` (or use `--version YYYY.MM.PATCH` to pin).
3. Verify `min_platform_version` against this AIMind instance.
4. Copy `skills/<entry>/` to `~/.aimind/skills/<entry>/`.
5. Pin the resolved `version:` and `catalog_entry:` into the installed `SKILL.md` frontmatter.
6. Record the install in `~/.aimind/installed.json`.

The skill becomes discoverable to every persona on this host on the next
session refresh (tier: `installed` — same discovery scope as a platform skill).

### Make this the default catalog (persistent)

If you want `aimind catalog install <entry>` to use this catalog without
passing `--catalog` every time:

```bash
export AIMIND_CATALOG_URL=https://github.com/thetwit4u/aimind-nova-catalogue
```

(Add to your shell profile; or accept the default of
`https://github.com/hiveras/aimind-catalog` and use `--catalog` per-install
for this one.)

### See what's available

```bash
aimind catalog available --catalog https://github.com/thetwit4u/aimind-nova-catalogue
```

### Upgrade / uninstall

```bash
aimind catalog upgrade <entry-name>
aimind catalog uninstall <entry-name>
```

## Versioning — CalVer

All entries use CalVer git tags: `<entry-name>@YYYY.MM.PATCH`.

- `YYYY` four-digit year
- `MM` zero-padded month
- `PATCH` 1-based integer that **resets to 1 at the start of each new month**

Examples:
- `aimind-unified-search@2026.05.1`
- `aimind-spec-writing@2026.05.2`

`aimind catalog upgrade <entry>` pulls the highest CalVer tag for the entry
declared in `catalog.json#entries[].latest_version`.

## Contributing a new entry

1. Create `skills/<entry-name>/SKILL.md`. Frontmatter MUST include:
   - `name`, `description`, `tier: installed`, `enabled: true`, `allowed-tools`
   - `catalog_entry: <entry-name>`
   - **No** `version:` field (pinned at install time by `aimind catalog install`).
2. Add the entry to `catalog.json` with `latest_version` and `min_platform_version`.
3. Commit, then tag the release: `git tag <entry-name>@YYYY.MM.1 && git push --tags`.
4. Verify the install on a clean instance: `aimind catalog install <entry-name> --catalog <this-repo>`.

See [`AGENTS.md`](./AGENTS.md) for the full contribution rules, neutrality
guidance, and quality gate.

## Workflow (Nova-internal)

1. **Draft.** Nova drafts a new SKILL.md in this repo (`skills/<name>/`).
2. **Catalog.** Add the entry to `catalog.json`; commit, tag, push.
3. **Install.** `aimind catalog install <name> --catalog https://github.com/thetwit4u/aimind-nova-catalogue` on the live instance.
4. **Test.** Trigger the skill in conversation; verify it does what the SKILL.md promises.
5. **Iterate.** Edit the source SKILL.md, bump CalVer (`YYYY.MM.PATCH+1`), re-tag, then `aimind catalog upgrade <name>`.
6. **Promote (optional).** If the skill is generic enough, move it to `hiveras/aimind-catalog` (org-wide) or open a PR against the platform repo (cross-instance).

## Related repos

- **Platform engine:** `hiveras/aimind` (private) — runtime, `aimind catalog` CLI, persona engine.
- **Org-wide catalog:** [`hiveras/aimind-catalog`](https://github.com/hiveras/aimind-catalog) — reference catalog; same layout as this one.
- **Instance state:** `~/.aimind/` — live persona/org/installed skills. Governed by `~/.aimind/AGENTS.md`.
- **Upstream inspiration:** [`anthropics/knowledge-work-plugins`](https://github.com/anthropics/knowledge-work-plugins) — seeded v0.x of these four skills.

## License

Skills in this repo are derivative work of Anthropic's open-source
[knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) (MIT).
See each SKILL.md `metadata.source_attribution` for specific lineage.

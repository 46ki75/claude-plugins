# Claude Code Plugin Marketplace

This repository is a [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins).
It was migrated from the former `46ki75/skills` repository, which distributed
[Agent Skills](https://agentskills.io/home) (formerly Claude Skills) as ZIPs.

## Directory Structure

- `plugins/`: Published plugins. Each is a marketplace entry with a
  `.claude-plugin/plugin.json` manifest and bundles skills, hooks, commands,
  and/or agents.
- `skills/`: Agent Skills not yet grouped into a plugin. When you author a new
  skill, create it here, then bundle it into a plugin under `plugins/`.
- `.agents/skills`: Agent Skills created by other providers (reference only).
- `submodules/`: Official repositories tracked as git submodules for reference.
- `crates/`: Rust workspace (`skill-cli`, etc.) for the agentskills.io ZIP
  channel.
- `.markdownlint-cli2.yaml`: Configuration for `markdownlint-cli2`.

## Marketplace and plugin manifests

- The marketplace is declared in `.claude-plugin/marketplace.json`. Every plugin
  must have an entry there with `name`, `source` (a `./plugins/<name>` path),
  `description`, `version`, and `keywords`.
- Each plugin has `plugins/<name>/.claude-plugin/plugin.json`. Keep its
  `version` and `description` in sync with the marketplace entry.
- A plugin auto-discovers `skills/`, `commands/`, `agents/`, and
  `hooks/hooks.json` under its root.

## Skill frontmatter

`$.metadata.author` and `$.metadata.version` are optional in the
[specification](https://agentskills.io/specification.md) but mandatory in this
repository.

```yaml
name: my-skill
description: placeholder
license: MIT # Always "MIT" in this repository.
metadata:
  author: "Ikuma Yamashita" # Always "Ikuma Yamashita" in this repository.
  version: "1.0"
```

When you create a skill, use the `skill-creator` skill.

### Linting

Run `markdownlint-cli2` after creating markdown files:

```bash
just check
```

## Git Submodules

Run at repository root, not inside `skills/`:

```bash
git submodule update --init --recursive   # first time
git submodule update --recursive           # keep up to date
```

## Knowledge-skill freshness

The `ai-protocols` plugin's skills are hand-curated digests of upstream protocol
repositories tracked as the submodules above. When a submodule advances, the
derived skill can go stale. This is tracked and refreshed automatically.

- **Provenance** — each such skill has a `.sources.json` recording, per upstream
  repo, the `synced` commit it currently reflects and the `paths` that feed it.
  The leading dot keeps it out of published skill ZIPs (the archiver prunes
  dotfiles). When a refresh updates a skill, bump `metadata.version` and advance
  the matching `synced` SHA in the same change.
- **Detection** — `cargo run -p skill-cli -- sources` reports skills whose
  `synced` SHA has fallen behind the current submodule pin over the tracked
  paths (`--json` for machine output, `--exit-code` to fail on drift).
- **Automation** — `.github/workflows/stale-skill-detect.yml` runs on Dependabot
  submodule PRs (read-only) and comments which skills drifted;
  `.github/workflows/stale-skill-refresh.yml` then runs via `workflow_run`
  (writable token + secrets) to let Claude refresh the stale skills and open a
  review PR. The two-workflow split is required because Dependabot-triggered
  runs get a read-only token and no Actions secrets.
- **Secrets** — refresh reuses the existing `CLAUDE_CODE_OAUTH_TOKEN` secret.
  Optionally set a `GH_PAT` secret so the refresh PR triggers CI (a PR opened
  with the default `GITHUB_TOKEN` does not start other workflows).

To onboard a new knowledge skill into this system, add a `.sources.json` to its
directory.

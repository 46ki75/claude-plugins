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

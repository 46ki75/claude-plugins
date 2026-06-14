# claude-plugins

My personal [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins).
Each plugin lives under `plugins/<name>/` with a `.claude-plugin/plugin.json`
manifest and bundles whatever it ships — Agent Skills, hooks, commands, or
agents. The marketplace itself is declared in `.claude-plugin/marketplace.json`.

Not intended for external contributors — feel free to fork or copy ideas, but
issues/PRs aren't actively triaged.

## Install the marketplace

```bash
/plugin marketplace add 46ki75/claude-plugins
/plugin install ai-protocols@46ki75-plugins
/plugin install wsl-notification@46ki75-plugins
```

## Plugins

| Plugin | Ships | Description |
| --- | --- | --- |
| `ai-protocols` | Skills | A2A, A2UI, AG-UI, and MCP protocol knowledge skills. |
| `qwik` | Skills | Qwik (v1/v2) and Qwik City / Qwik Router. |
| `web-view-transition` | Skills | The View Transition API (SPA and cross-document). |
| `wxt` | Skills | Building browser extensions with WXT. |
| `prompt-evaluation-claude-code` | Skills | Eval-driven prompt refinement run inside Claude Code. |
| `authoring` | Skills | Markdown linting/fixing and Mermaid diagrams. |
| `conventional-commits` | Skills | Conventional Commits messages. |
| `development-standards` | Skills | Org-internal engineering standards. |
| `rust` | Skills | Toasty ORM (v0.7) guidance. |
| `wsl-notification` | Hooks | Windows toast notifications for Claude Code events from WSL2. |

## Layout

- `plugins/` — published plugins, each a marketplace entry.
- `skills/` — standalone Agent Skills that are intentionally **not** published
  as marketplace plugins (e.g. `kedb`), plus a staging area for new skills
  before they are bundled into a plugin.
- `crates/` — Rust workspace that validates, archives, and publishes skill ZIPs
  to the agentskills.io release channel.
- `.agents/skills/` — skills authored by other providers (reference only).
- `submodules/` — upstream repositories tracked as git submodules.
- `terraform/` — GitHub repo configuration (rulesets, labels).

## Plugin anatomy

A plugin that bundles skills looks like:

```text
plugins/ai-protocols/
├── .claude-plugin/plugin.json    # manifest: name, version, description, …
├── README.md
└── skills/
    └── <skill-name>/SKILL.md     # auto-discovered by Claude Code
```

Claude Code auto-discovers `skills/`, `commands/`, `agents/`, and
`hooks/hooks.json` inside each plugin, so grouping skills into a plugin is just
a matter of placing their directories under `skills/`.

## Migration status

This repo was migrated from the former `46ki75/skills` repository, where the
15 skills were distributed as agentskills.io ZIP releases. Thirteen are now
published as Claude Code plugins:

| Plugin | Bundled skills |
| --- | --- |
| `ai-protocols` | `a2a-knowledge`, `a2ui-knowledge`, `ag-ui-knowledge`, `mcp-knowledge` |
| `qwik` | `qwik` |
| `web-view-transition` | `web-view-transition` |
| `wxt` | `wxt` |
| `prompt-evaluation-claude-code` | `prompt-evaluation-claude-code` |
| `authoring` | `markdown`, `mermaid` |
| `conventional-commits` | `conventional-commits` |
| `development-standards` | `development-standards` |
| `rust` | `rust-toasty` |

Two skills are kept standalone under `skills/`, intentionally not published as
plugins:

- `skills/kedb/` — maintains a local Known Error Database.
- `skills/prompt-evaluation/` — eval-driven prompt refinement for the
  Anthropic / Claude API (its Claude Code counterpart ships as the
  `prompt-evaluation-claude-code` plugin).

> **Note on the ZIP pipeline:** `skill-cli` (under `crates/`) scans the
> top-level `skills/` directory one level deep. Skills now live under
> `plugins/<name>/skills/` and are no longer picked up by that pipeline. If the
> agentskills.io ZIP channel should keep covering them, point the pipeline at
> `plugins/*/skills/` (e.g. `skill-cli check --skills-dir` per plugin, or
> teach `scan_and_validate` to glob `plugins/*/skills/*`).

## Local commands

```bash
# Validate skills still under skills/ (does not see plugins/*/skills yet)
cargo run -p skill-cli -- check

# Markdown lint
just check

# Init submodules (run at repo root)
git submodule update --init --recursive
```

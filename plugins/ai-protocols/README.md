# ai-protocols

A Claude Code plugin bundling expert-knowledge Agent Skills for the major AI
agent protocols. Installing the plugin makes all bundled skills available; each
skill activates on demand based on its `description`.

## Bundled skills

| Skill | Protocol |
| --- | --- |
| `a2a-knowledge` | A2A — Agent2Agent Protocol |
| `a2ui-knowledge` | A2UI — Agent to UI |
| `ag-ui-knowledge` | AG-UI — Agent–User Interaction Protocol |
| `mcp-knowledge` | MCP — Model Context Protocol |

## Install

```bash
/plugin marketplace add 46ki75/claude-plugins
/plugin install ai-protocols@46ki75-plugins
```

## Layout

```text
plugins/ai-protocols/
├── .claude-plugin/plugin.json
├── README.md
└── skills/
    ├── a2a-knowledge/SKILL.md
    ├── a2ui-knowledge/SKILL.md
    ├── ag-ui-knowledge/SKILL.md
    └── mcp-knowledge/SKILL.md
```

Claude Code auto-discovers each subdirectory of `skills/` as an Agent Skill, so
adding or removing a protocol skill is just a matter of editing this directory.

## Staying current

Each skill is a digest of an upstream protocol repository tracked as a git
submodule, so it can drift as the protocol evolves. To keep them fresh:

- Every skill carries a `.sources.json` recording the upstream repo(s), the
  paths that feed it, and the commit (`synced`) it currently reflects.
- `cargo run -p skill-cli -- sources` reports any skill whose `synced` commit
  has fallen behind the submodule pin over those paths.
- On a Dependabot submodule bump, CI detects the drift and opens an automated
  PR (authored by Claude) that refreshes the affected skills, bumps their
  `metadata.version`, and advances `synced`. See the repository's `AGENTS.md`
  ("Knowledge-skill freshness") for the full flow.

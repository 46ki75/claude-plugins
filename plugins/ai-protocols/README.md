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

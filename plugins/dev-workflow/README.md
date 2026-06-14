# dev-workflow

A Claude Code plugin bundling engineering-workflow skills. Each skill activates
on demand based on its `description`.

## Bundled skills

| Skill | Covers |
| --- | --- |
| `conventional-commits` | Writing, formatting, and reviewing Conventional Commits |
| `development-standards` | Org-internal engineering standards |
| `kedb` | Maintaining a Known Error Database (errors, symptoms, workarounds, root causes) |

## Install

```bash
/plugin marketplace add 46ki75/claude-plugins
/plugin install dev-workflow@46ki75-plugins
```

---
paths:
  - "plugins/*/skills/*/SKILL.md"
  - "skills/*/SKILL.md"
---

# Skill authoring

`SKILL.md` frontmatter is validated by `skill-validator`
(`cargo run -p skill-cli -- check`). Required fields and rules:

- `name` — kebab-case (`[a-z0-9-]`, no leading/trailing `-`) and **must equal the
  skill's own directory name**.
- `name` must be **unique across every skill** in both `skills/*` and
  `plugins/*/skills/*` — it becomes the release tag `agent-skills-<name>-v<version>`,
  so the publish scan errors out on any duplicate.
- `description` — non-empty, ≤ 1024 characters, no XML/HTML-like tags.
- `metadata.author` and `metadata.version` — both required; `version` is semver
  `MAJOR.MINOR.PATCH`, digits only, no leading zeros.

Markdown here is linted by `markdownlint-cli2` (`MD013` line length 200). Some
skills' `references/` trees are excluded in `.markdownlint-cli2.yaml`.

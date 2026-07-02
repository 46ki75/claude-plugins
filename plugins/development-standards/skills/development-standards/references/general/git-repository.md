# Git Repository Standards

## Create `.editorconfig`

```ini
# editorconfig.org
root = true

[*]
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

## Package manager: pnpm

Use pnpm for anything Node-based — including repos whose primary
language is not JavaScript but which carry Node tooling (e.g.
`markdownlint-cli2` in a Rust repo). This is org policy; do not switch
to npm or yarn because a repo happens to have their artifacts lying
around.

- Declare `"packageManager": "pnpm@<exact-version>+sha512.<integrity-hash>"`
  in `package.json` — the full corepack integrity form, not just the bare
  version. Run `corepack use pnpm@<version>` (or copy the hash from an
  existing repo's `package.json`) rather than typing a bare version by hand.
- Commit `pnpm-lock.yaml`; never `package-lock.json` or `yarn.lock`.
- CI installs with `pnpm install --frozen-lockfile`.

Exception: Bun projects (see `references/bun/`), where Bun is both
runtime and package manager.

## Git hooks: lefthook

Use [lefthook](https://github.com/evilmartians/lefthook) for git hooks, not
husky. Declare it as a root `devDependency` and install it via the
standard npm lifecycle hook:

```json
{
  "scripts": {
    "prepare": "lefthook install"
  },
  "devDependencies": {
    "lefthook": "^2.1.9"
  },
  "pnpm": {
    "onlyBuiltDependencies": ["lefthook"]
  }
}
```

The `pnpm.onlyBuiltDependencies` entry is required — pnpm ≥9 denies native
install scripts by default, and without this allowlist entry `lefthook
install` silently never runs.

### Single-package repos

A flat `pre-commit` group covering every language present is enough:

```yaml
pre-commit:
  jobs:
    - name: rustfmt
      glob: "**/*.rs"
      run: cargo fmt -- {staged_files}
      stage_fixed: true
    - name: markdownlint
      glob: "*.md"
      run: pnpm exec markdownlint-cli2 --fix {staged_files}
    - name: terraform-fmt
      glob: "**/*.tf"
      run: terraform fmt {staged_files}
```

### pnpm monorepos

Scope each job to its package with `root:`/`glob:` so a tool never has to
guess which package's config applies:

```yaml
pre-commit:
  jobs:
    - run: pnpm run --recursive check

check:
  jobs:
    - name: eslint-foo
      root: "packages/foo/"
      glob: "packages/foo/**/*.{ts,tsx}"
      run: pnpm --filter foo lint
    - name: vitest-foo
      root: "packages/foo/"
      glob: "packages/foo/**/*.{ts,tsx}"
      run: pnpm --filter foo exec vitest related --run --passWithNoTests {files}
```

Tools with no meaningful per-package config (e.g. Prettier across a whole
monorepo) run once from the repo root instead of being duplicated per
package.

## Markdown linting with `markdownlint-cli2`

Every repository that contains Markdown should lint it with
[`markdownlint-cli2`](https://github.com/DavidAnson/markdownlint-cli2).
Use the config and `package.json` shape below — do not improvise rule
sets per repo.

### `.markdownlint-cli2.yaml`

Place at the repository root. **YAML, not JSONC** — `.markdownlint-cli2.jsonc`
with only an `ignores` array (no `MD013`/`MD024`/`MD029` block) is a drift
pattern seen in some org repos, not an accepted alternative. A bare-defaults
config means 80-character line wrapping and default list/heading rules,
which is not what this org has decided on. If you find a `.jsonc` config
with no rule overrides in a repo, that repo is out of compliance — bring it
to `.yaml` with the block below rather than treating the `.jsonc` file as
precedent.

```yaml
config:
  MD013:
    line_length: 200
    tables: false
    code_blocks: false
  MD024:
    siblings_only: true
  MD029:
    style: ordered
ignores:
  - "**/node_modules/**"
```

Rule choices, and why:

- **MD013 (line length) → 200, tables/code blocks exempt.** Long URLs,
  command examples, and tables routinely exceed the default 80. A hard
  wrap there hurts readability more than it helps. 200 is generous
  enough that real prose still gets flagged.
- **MD024 (no duplicate headings) → `siblings_only: true`.** Repeated
  headings under different parents (e.g. `## Install` appearing under
  multiple OS sections) are legitimate. Only flag duplicates at the
  same level.
- **MD029 (ordered list prefix) → `ordered`.** Use `1.`, `2.`, `3.`
  literally — never the `1.`, `1.`, `1.` lazy style. The rendered
  output is the same, but the source stays diff-friendly and human-
  readable.

### `ignores`

Add globs for anything the linter should not touch. Common entries:

| Path                  | Reason                                                                      |
| --------------------- | --------------------------------------------------------------------------- |
| `**/node_modules/**`  | Third-party packages.                                                       |
| `./submodules/**`     | Git submodules — upstream owns their lint rules.                            |
| `./target/**`         | Rust build output.                                                          |
| `./.claude/**`        | Agent-generated transcripts and worktrees.                                  |
| `./refs/`, `./notes/` | Local-only scratch / reference material, if the repo uses such conventions. |

Vendored or generated Markdown (third-party docs copied in, generated
API docs) should also be ignored — fix it upstream, not here.

### `package.json`

Pin `markdownlint-cli2` as a dev dependency and expose a `lint` script:

```json
{
  "packageManager": "pnpm@10.33.0",
  "scripts": {
    "lint": "markdownlint-cli2 \"**/*.md\""
  },
  "devDependencies": {
    "markdownlint-cli2": "^0.22.1"
  }
}
```

(Pin `packageManager` to whatever the current pnpm release is — the
field requires an exact version.)

The glob in the script is the lint **target**; `ignores` in the YAML
is the exclusion list. Keep both — narrowing the glob to skip a
directory hides the file from `--fix` runs too.

### Running

```bash
pnpm lint              # or, in Bun projects: bun run lint
pnpm exec markdownlint-cli2 --fix "**/*.md"   # auto-fix where possible
```

Wire `pnpm lint` into CI (after `pnpm install --frozen-lockfile`) so
Markdown regressions fail the build the same way code-lint regressions
do. This step is commonly skipped in practice — several org repos enforce
markdown lint only through the editor extension below, with no CI job at
all, which means a contributor who ignores editor diagnostics can merge
unlinted Markdown. Don't copy that gap into a new repo.

### Editor integration

Recommend the VS Code extension
[`DavidAnson.vscode-markdownlint`](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint).
It reads the same `.markdownlint-cli2.yaml`, so editor diagnostics
match CI output exactly. Add it to `.vscode/extensions.json` under
`recommendations` so contributors get prompted on first open:

```json
{
  "recommendations": ["DavidAnson.vscode-markdownlint"]
}
```

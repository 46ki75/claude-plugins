---
name: lefthook
description: >
  Set up, configure, and debug Lefthook, the fast polyglot Git hooks
  manager. Covers installation (npm, Homebrew, Go, etc.),
  `lefthook install`, authoring `lefthook.yml` (hooks, the modern `jobs`
  syntax plus classic `commands`/`scripts`), file-list templates
  (`{staged_files}`, `{push_files}`, `{all_files}`, positional `{1}`),
  `glob`/`exclude`/`root` filtering, `parallel` vs `piped` execution,
  auto-staging linter fixes with `stage_fixed`, `tags`, `skip`/`only`
  conditions, job `group`s, local overrides (`lefthook-local.yml`),
  `extends`/`remotes`, and the CLI (`install`, `run`, `add`, `validate`,
  `dump`). Use whenever the user wants to add or configure Git hooks, run
  linters/formatters/tests on commit or push, lint staged files, wire
  up commitlint/eslint/prettier/rubocop, migrate from Husky or pre-commit,
  or troubleshoot a `lefthook.yml` — even when they don't say "lefthook" by
  name. Trigger on lefthook, lefthook.yml, git hooks,
  pre-commit/pre-push/commit-msg hooks, a Husky alternative, or staged-file
  linting.
license: MIT
metadata:
  author: "Ikuma Yamashita"
  version: "1.1.0"
---

# Lefthook

Lefthook is a single-binary, dependency-free Git hooks manager written in Go.
You declare what should run on each Git hook in a `lefthook.yml`, run
`lefthook install` once, and Lefthook wires up the `.git/hooks/*` shims. It
filters files (so linters only see what changed), runs jobs in parallel, and
can re-stage files that formatters fixed.

Reach for Lefthook whenever the user wants automation on commit/push, wants to
lint or format only staged files, or is replacing Husky / pre-commit. The rest
of this skill is the practical model: install → write `lefthook.yml` → install
hooks → iterate.

## Full reference

This body is the working subset. For exhaustive, upstream-faithful detail —
every option, flag, and example — read the bundled `references/` files (each
consolidated from the official lefthook docs):

- `references/configuration.md` — every `lefthook.yml` option (global, remotes,
  hook, command/job/script) with defaults and examples. Has a table of
  contents; jump to the option you need. Read this for anything beyond the
  options summarized below (e.g. `glob_matcher`, `file_types`,
  `fail_on_changes`, `templates`, `output`, `priority`).
- `references/cli-commands.md` — full CLI: `install`, `run`, `add`, `validate`,
  `dump`, `uninstall`, `check-install`, `self-update`, `version` and their flags.
- `references/installation.md` — all install channels with exact commands.
- `references/env-vars.md` — `LEFTHOOK`, `LEFTHOOK_*`, `CI`, `NO_COLOR`, etc.
- `references/features.md` — git-args capture, Git LFS, interactive, local
  config, passing stdin.
- `references/examples.md` — worked configs (commitlint, filters, remotes,
  skipping, stage_fixed, wrapping commands).

## Install the binary

Pick the channel that matches the project. The runtime is the same standalone
binary either way — the package manager only delivers it.

| Stack | Install |
| --- | --- |
| Node | `npm i -D lefthook` (or `pnpm add -D`, `yarn add -D`, `bun add -d`) |
| Homebrew | `brew install lefthook` |
| Go | `go install github.com/evilmartians/lefthook@latest` |
| Mise | `mise use lefthook` |
| Ruby | `gem install lefthook` |
| Python | `pip install lefthook` |
| Debian/RPM/Alpine/Scoop/Winget/Snap | distro package named `lefthook` |

The **npm package auto-installs the hooks** on `npm install` via a postinstall
step, and skips that when `CI=true` — so in Node projects you usually don't run
`lefthook install` by hand. For every other channel, run it once (and after any
fresh clone).

## Bootstrap a repo

```bash
lefthook install          # registers .git/hooks/* shims from lefthook.yml
```

Config file resolution (first match wins): `lefthook.yml`, `.lefthook.yml`,
`lefthook.yaml`, `.config/lefthook.yml`. TOML and JSON/JSONC variants with the
same stems also work. A `lefthook-local.yml` sibling is merged on top and is
meant to stay out of version control (per-developer tweaks).

Re-run `lefthook install` whenever you add a **new hook type** to the config
(e.g. you add a `commit-msg:` section that wasn't there before). Editing jobs
inside an already-installed hook needs no reinstall.

## The config model

A `lefthook.yml` maps **Git hook names** to a list of **jobs**. Standard hooks:
`pre-commit`, `pre-push`, `commit-msg`, `prepare-commit-msg`, `post-merge`,
`post-checkout`, `post-rewrite`, and the rest of Git's hook set.

**Default to the `jobs:` array for any new project** (Lefthook ≥ 1.10). It is
the modern syntax: one shape for both shell commands and scripts, nested
`group`s, and named-job merging across `extends`/local config — things the
classic `commands:`/`scripts:` maps can't do. Reach for `commands`/`scripts`
only when reading or extending an existing config that already uses them; it's
covered at the end of this section. Don't mix `jobs:` with `commands:` in the
same hook.

### Jobs (the default)

```yaml
pre-commit:
  parallel: true
  jobs:
    - name: lint
      glob: "*.{js,ts,jsx,tsx}"
      run: npx eslint --fix {staged_files}
      stage_fixed: true

    - name: format
      glob: "*.{json,css,md}"
      run: npx prettier --write {staged_files}
      stage_fixed: true
```

Each job is one of `run:` (a shell command) or `script:` (a file under the
source dir). Common per-job keys:

- `name` — identifier; **named** jobs merge across `extends`/local config,
  unnamed jobs just append.
- `run` — command, executed via `sh`. Supports the file templates below.
- `glob` — only run when staged/changed files match (one pattern or a list).
- `exclude` — drop matching paths; accepts glob patterns or a regex string.
- `root` — `cd` here first; file lists are made relative to it. Essential for
  monorepos (`root: "frontend/"`).
- `stage_fixed: true` — re-`git add` any files the job modified. This is what
  makes "lint --fix on commit" actually commit the fixes. **pre-commit only.**
- `tags` — labels for selective runs / skipping (`lint`, `test`, …).
- `skip` / `only` — conditional execution (see below).
- `interactive: true` — connect a TTY (prompts, `git add -p`-style tools).
- `env` — extra environment variables for this job.
- `fail_text` — custom message shown when the job fails.
- `priority` — integer execution order within a non-parallel run.

### File-list templates

Lefthook substitutes the current file set into `run`. Always quote them so
paths with spaces survive:

| Template | Expands to |
| --- | --- |
| `{staged_files}` | files staged for commit (the pre-commit workhorse) |
| `{push_files}` | files in commits not yet pushed (use on `pre-push`) |
| `{all_files}` | all files tracked by Git |
| `{files}` | output of a custom `files:` command you define |
| `{cmd}` | the `run` string itself (handy in `extends`) |
| `{1}`, `{2}`, … | positional Git hook args (e.g. commit-msg path = `{1}`) |
| `{0}` | all Git hook args joined |

Long file lists are automatically chunked across multiple invocations, so you
don't have to worry about argv length limits.

### parallel vs piped

- `parallel: true` — run the hook's jobs concurrently. Default is sequential.
- `piped: true` — run sequentially and **abort the rest if one fails**. Use for
  ordered steps like `install → migrate`. (`parallel` and `piped` are mutually
  exclusive.)

### Job groups

A `group` nests jobs with their own `parallel`/`piped`, and lets shared
`root`/`glob`/`exclude` apply to all of them — so the outer hook can run groups
in parallel while a group runs its members in order:

```yaml
pre-commit:
  parallel: true
  jobs:
    - name: backend
      root: "backend/"
      group:
        piped: true            # sequential within the group
        jobs:
          - run: bundle install
          - run: bundle exec rails db:migrate

    - name: frontend
      root: "frontend/"
      run: npm run lint
```

### Classic commands / scripts (legacy — for existing configs)

The older syntax, kept here so you can read and edit configs that predate
`jobs`. Don't reach for it in new projects; the equivalent `jobs` form above is
preferred. It looks like:

```yaml
pre-commit:
  commands:
    eslint:
      glob: "*.{js,ts}"
      run: npx eslint {staged_files}
  scripts:
    "validate.sh":
      runner: bash
```

`scripts` live under the source dir, default `.lefthook/<hook-name>/` — so the
example expects `.lefthook/pre-commit/validate.sh`. `runner` is the interpreter
(`bash`, `ruby`, …).

## Conditional execution: skip / only

`skip` bypasses a job (or whole hook); `only` is the inverse — run *only* when
the condition holds. Both accept booleans, a `ref:` branch matcher, a `run:`
predicate (skip when the command exits 0), or Git states like `merge`/`rebase`.

```yaml
pre-push:
  jobs:
    - run: npm test
      only:
        - ref: main          # only on the main branch
    - run: npm run e2e
      skip:
        - merge              # not during merges
        - ref: release/*
```

To skip everything ad hoc, set `LEFTHOOK=0` for one command:
`LEFTHOOK=0 git commit …`. `git commit --no-verify` (`-n`) also bypasses hooks.

## CLI reference

| Command | Does |
| --- | --- |
| `lefthook install` | Write the `.git/hooks` shims from config |
| `lefthook uninstall` | Remove the shims (add `-c` to also drop config) |
| `lefthook run <hook>` | Run a hook manually, e.g. `lefthook run pre-commit` |
| `lefthook run <hook> --all-files` | Force-run against all files |
| `lefthook run <hook> --commands lint` | Run only named jobs/commands |
| `lefthook add <hook>` | Scaffold a hook (and `--dirs` script folders) |
| `lefthook validate` | Check the config is well-formed |
| `lefthook dump` | Print the fully merged config (debug `extends`/local) |
| `lefthook check-install` | Verify the installed shims are current |
| `lefthook version` / `self-update` | Show / update the binary |

`lefthook run pre-commit` is the fast iteration loop — test changes without
making a real commit.

## Sharing config: extends & remotes

```yaml
extends:
  - .config/lefthook-shared.yml

remotes:
  - git_url: https://github.com/org/hooks
    ref: v1.0.0
    configs:
      - lefthook.yml
    refetch_frequency: 24h
```

Merge order: base `lefthook.yml` → `extends` → `remotes` → `lefthook-local.yml`.
Named jobs/commands merge by name across these layers; this is how a team shares
a baseline while individuals override locally.

## Recipes

**Lint + format staged files, commit the fixes** (the most common setup):

```yaml
pre-commit:
  parallel: true
  jobs:
    - name: eslint
      glob: "*.{js,ts,jsx,tsx}"
      run: npx eslint --fix {staged_files}
      stage_fixed: true
    - name: prettier
      glob: "*.{json,css,scss,md,yml,yaml}"
      run: npx prettier --write {staged_files}
      stage_fixed: true
```

**Validate the commit message with commitlint** (the message file is `{1}`):

```yaml
commit-msg:
  jobs:
    - run: npx commitlint --edit {1}
```

After adding a brand-new hook section like this, run `lefthook install` again.

**Run tests before push, but only on unpushed changes / main:**

```yaml
pre-push:
  jobs:
    - name: test
      run: npm test
    - name: typecheck
      glob: "*.{ts,tsx}"
      run: npx tsc --noEmit
```

**Polyglot monorepo** — scope each tool to its directory with `root`:

```yaml
pre-commit:
  parallel: true
  jobs:
    - name: ruby
      root: "api/"
      glob: "*.rb"
      run: bundle exec rubocop --force-exclusion -A {staged_files}
      stage_fixed: true
    - name: web
      root: "web/"
      glob: "*.{ts,tsx}"
      run: npm run lint -- {staged_files}
```

## Migration & gotchas

- **From Husky / pre-commit**: delete `.husky/` or `.pre-commit-config.yaml`,
  write `lefthook.yml`, run `lefthook install`. Map each Husky hook file to a
  hook section; map pre-commit "hooks" to jobs with `glob` + `{staged_files}`.
- **`stage_fixed` is pre-commit only** — re-staging makes no sense on push.
- **Quote templates** (`"{staged_files}"`) so spaces in paths don't split args.
- **CI**: the npm package skips auto-install when `CI=true`; in CI run the tools
  directly or `lefthook run pre-commit --all-files` rather than relying on the
  Git hook. Set `min_version:` in config to fail fast on stale binaries.
- **Nothing runs?** Confirm `lefthook install` ran and `.git/hooks/pre-commit`
  is Lefthook's shim; check `glob`/`root` actually match staged paths; use
  `lefthook dump` to see the merged config and `LEFTHOOK_VERBOSE=1` for tracing.
- **`commands`/`scripts` vs `jobs`**: new configs use `jobs:` (the default);
  reach for `commands`/`scripts` only in configs that already have them, and
  never mix `jobs:` with `commands:` in the same hook.

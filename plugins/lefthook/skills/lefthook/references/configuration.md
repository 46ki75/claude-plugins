# Lefthook configuration reference

This is a complete reference for every Lefthook configuration option, consolidated and
faithfully merged from the upstream documentation. It covers the config file names and
formats, repo-wide global options, the `remotes` block, per-hook settings, per-task
(command/job/script) settings, and the structural building blocks (hooks, commands,
scripts, jobs, and groups). Each option appears as a `###` subsection named after the
bare option key so it can be linked to directly.

## Table of contents

- [Config file name & formats](#config-file-name--formats)
- [Global options](#global-options)
- [Remotes](#remotes)
- [Hook options](#hook-options)
- [Command / Job / Script options](#command--job--script-options)
- [Structure](#structure)

## Config file name & formats

Lefthook supports the following file names for the main config:

| Format | Acceptable config name  |
| ------ | ----------------------- |
| YAML   | `lefthook.yml`          |
| YAML   | `lefthook.yaml`         |
| YAML   | `.lefthook.yml`         |
| YAML   | `.lefthook.yaml`        |
| YAML   | `.config/lefthook.yml`  |
| YAML   | `.config/lefthook.yaml` |
| TOML   | `lefthook.toml`         |
| TOML   | `.lefthook.toml`        |
| TOML   | `.config/lefthook.toml` |
| JSON   | `lefthook.json`         |
| JSON   | `.lefthook.json`        |
| JSON   | `.config/lefthook.json` |
| JSONC  | `lefthook.jsonc`        |
| JSONC  | `.lefthook.jsonc`       |
| JSONC  | `.config/lefthook.jsonc`|

If there are more than 1 file in the project, only one will be used, and you'll never
know which one. So, please, use one format in a project.

Filenames without the leading dot will also be looked up from the [`.config` subdirectory](https://github.com/pi0/config-dir).

Lefthook also merges an extra config with the name `lefthook-local`. All supported
formats can be applied to this `-local` config. If you name your main config with the
leading dot, like `.lefthook.json`, the `-local` config also must be named with the
leading dot: `.lefthook-local.json`.

The `-local` config can be used without a main config file. This is useful when you want
to use lefthook locally without imposing it on your teammates – just create a
`lefthook-local.yml` file and add it to your global `.gitignore`.

### Merge order

Settings are applied in this order (see [extends](#extends) and [remotes](#remotes) for details):

- `lefthook.yml` – main config file
- `extends` – configs specified in the [extends](#extends) option
- `remotes` – configs specified in the [remotes](#remotes) option
- `lefthook-local.yml` – local config file

So, `extends` override settings from `lefthook.yml`, `remotes` override `extends`, and
`lefthook-local.yml` can override everything.

## Global options

Repo-wide settings that apply to the whole Lefthook config.

### assert_lefthook_installed

**Default: `false`**

When set to `true`, fail (with exit status 1) if `lefthook` executable can't be found in
$PATH, under node_modules/, as a Ruby gem, or other supported method. This makes sure git
hook won't omit `lefthook` rules if `lefthook` ever was installed.

#### Example

```yml
# lefthook.yml

assert_lefthook_installed: true
```

### colors

**Default: `auto`**

Whether enable or disable colorful output of Lefthook. This option can be overwritten with `--colors` option. You can also provide your own color codes.

#### Example

Disable colors.

```yml
# lefthook.yml

colors: false
```

Custom color codes. Can be hex or ANSI codes.

```yml
# lefthook.yml

colors:
  cyan: 14
  gray: 244
  green: '#32CD32'
  red: '#FF1493'
  yellow: '#F0E68C'
```

Control via ENV variable.

- Set `NO_COLOR=true` to disable colored output in lefthook and all subcommands that lefthook calls.
- Set `CLICOLOR_FORCE=true` to force colored output in lefthook and all subcommands.

### no_tty

**Default: `false`**

Whether hide spinner and other interactive things. This can be also controlled with `--no-tty` option for `lefthook run` command.

#### Example

```yml
# lefthook.yml

no_tty: true
```

### output

You can manage verbosity using the `output` config. You can specify what to print in your output by setting these values, which you need to have

Possible values are `meta,summary,success,failure,execution,execution_out,execution_info,skips`.
By default, all output values are enabled

You can also disable all output with setting `output: false`. In this case only errors will be printed.

#### Example

```yml
# lefthook.yml

output:
  - meta           # Print lefthook version
  - summary        # Print summary block (successful and failed steps)
  - empty_summary  # Print summary heading when there are no steps to run
  - success        # Print successful steps
  - failure        # Print failed steps printing
  - execution      # Print any execution logs
  - execution_out  # Print execution output
  - execution_info # Print `EXECUTE > ...` logging
  - skips          # Print "skip" (i.e. no files matched)
```

You can also *extend* this list with an environment variable `LEFTHOOK_OUTPUT`:

```bash
LEFTHOOK_OUTPUT="meta,success,summary" lefthook run pre-commit
```

### source_dir

**Default: `.lefthook/`**

Change a directory for script files. The directory contains subfolders named after git hooks, each containing script files.

#### Example

```text
.lefthook/
├── pre-commit/
│   ├── lint.sh
│   └── test.py
└── pre-push/
    └── check-files.rb
```

### source_dir_local

**Default: `.lefthook-local/`**

Change a directory for *local* script files (not stored in VCS).

This option is useful if you have a `lefthook-local.yml` config file and want to reference different scripts there.

#### Example

```yml
# lefthook-local.yml

source_dir_local: .lefthook-local/
```

### rc

Provide an [**rc**](https://www.baeldung.com/linux/rc-files) file, which is actually a simple `sh` script. Currently it can be used to set ENV variables that are not accessible from non-shell programs.

#### Example

Use cases:

- You have a GUI program that runs git hooks (e.g., VSCode)
- You reference executables that are accessible only from a tweaked $PATH environment variable (e.g., when using rbenv or nvm, fnm)
- Or even if your GUI program cannot locate the `lefthook` executable :scream:
- Or if you want to use ENV variables that control the executables behavior in `lefthook.yml`

```bash
# An npm executable which is managed by nvm
$ which npm
/home/user/.nvm/versions/node/v15.14.0/bin/npm
```

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: npm run eslint {staged_files}
```

Provide a tweak to access `npm` executable the same way you do it in your `~/<shell>rc`.

```yml
# lefthook-local.yml

# You can choose whatever name you want.
# You can share it between projects where you use lefthook.
# Make sure the path is absolute.
rc: ~/.lefthookrc
```

Or

```yml
# lefthook-local.yml

# If the path contains spaces, you need to quote it.
rc: '"${XDG_CONFIG_HOME:-$HOME/.config}/lefthookrc"'
```

In the rc file, export any new environment variables or modify existing ones.

```bash
# ~/.lefthookrc

# An nvm way
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# An fnm way
export FNM_DIR="$HOME/.fnm"
[ -s "$FNM_DIR/fnm.sh" ] && \. "$FNM_DIR/fnm.sh"

# Or maybe just
PATH=$PATH:$HOME/.nvm/versions/node/v15.14.0/bin
```

```bash
# Make sure you updated git hooks. This is important.
$ lefthook install -f
```

Now any program that runs your hooks will have a tweaked PATH environment variable and will be able to get `nvm` :wink:

### min_version

If you want to specify a minimum version for lefthook binary (e.g. if you need some features older versions don't have) you can set this option.

#### Example

```yml
# lefthook.yml

min_version: 1.1.3
```

### skip_lfs

**Default:** `false`

Skip running LFS hooks even if it exists on your system.

#### Example

```yml
# lefthook.yml

skip_lfs: true

pre-push:
  commands:
    test:
      run: yarn test
```

### no_auto_install

**Default: `false`**

Disable automatic installation and synchronization of git hooks when running lefthook. By
default, lefthook automatically installs and updates hooks when you run `lefthook run` if
the configuration has changed. Setting this to `true` disables that behavior.

This can also be controlled with the `--no-auto-install` option for the `lefthook run` command.

#### Example

```yml
# lefthook.yml

no_auto_install: true

pre-commit:
  commands:
    lint:
      run: npm run lint
```

### install_non_git_hooks

> **Tip (New feature):** Added in lefthook `2.0.17`

Install non-Git hooks into `.git/hooks`. May be useful for using with tools like <https://git-flow.sh/>.

### lefthook

**Default:** `null`

> **Tip (New feature):** Added in lefthook `1.10.5`

Provide a full path to lefthook executable or a command to run lefthook. Bourne shell (`sh`) syntax is supported.

> **Warning:** This option does not merge from `remotes` or `extends` for security reasons. It does get merged from `lefthook-local.yml` if specified.

There are three reasons you may want to specify `lefthook`:

1. You want to force using specific lefthook version from your dependencies (e.g. npm package)
2. You use PnP loader for your JS/TS project, and your `package.json` with lefthook dependency locates in a subfolder
3. You want to make sure you use concrete lefthook executable path and want to defined it in `lefthook-local.yml`

#### Specify lefthook executable

```yml
# lefthook.yml

lefthook: /usr/bin/lefthook

pre-commit:
  jobs:
    - run: yarn lint
```

#### Specify a command to run lefthook

```yml
# lefthook.yml

lefthook: |
  cd project-with-lefthook
  pnpm lefthook

pre-commit:
  jobs:
    - run: yarn lint
      root: project-with-lefthook
```

#### Force using a version from Rubygems

```yml
# lefthook.yml

lefthook: bundle exec lefthook

pre-commit:
  jobs:
    - run: bundle exec rubocop -- {staged_files}
```

#### Enable debug logs

```yml
# lefthook-local.yml

lefthook: LEFTHOOK_VERBOSE=1 lefthook
```

### glob_matcher

Configure which glob matching engine lefthook uses to filter files.

**Values:**

- `gobwas` (default): see <https://github.com/gobwas/glob>
- `doublestar`: Usual glob behavior (like in Bash)

#### Example

```yml
# lefthook.yml

glob_matcher: doublestar

pre-commit:
  jobs:
    - name: lint
      run: yarn eslint {staged_files}
      glob: "**/*.{js,ts}"
```

#### Behaviour comparison

```yml
# gobwas (default): **/*.js matches src/app.js but NOT app.js
# doublestar:       **/*.js matches app.js, src/app.js, a/b/c/app.js
```

Use `doublestar` when migrating from other tools or when you need `**` to match files at
any depth including the root. The setting applies globally to all `glob` and `exclude`
patterns and is backwards compatible. See also [`glob`](#glob) and [`exclude`](#exclude).

### extends

You can extend your config with another one YAML file. Its content will be merged. Extends
for `lefthook.yml`, `lefthook-local.yml`, and [`remotes`](#remotes) configs are handled
separately, so you can have different extends in these files.

You can use asterisk to make a glob.

#### Example

```yml
# lefthook.yml

extends:
  - /home/user/work/lefthook-extend.yml
  - /home/user/work/lefthook-extend-2.yml
  - lefthook-extends/file.yml
  - ../extend.yml
  - projects/*/specific-lefthook-config.yml
```

> **Note:** Settings are applied in this order:
>
> - `lefthook.yml` – main config file
> - `extends` – configs specified in [extends](#extends) option
> - `remotes` – configs specified in [remotes](#remotes) option
> - `lefthook-local.yml` – local config file
>
> So, `extends` override settings from `lefthook.yml`, `remotes` override `extends`, and `lefthook-local.yml` can override everything.

### templates

> **Tip (New feature):** Added in lefthook `1.10.8`

Provide custom replacement for templates in `run` values.

With `templates` you can specify what can be overridden via `lefthook-local.yml` without a need to overwrite every jobs in your configuration.

#### Override with lefthook-local.yml

```yml
# lefthook.yml

templates:
  dip: # empty

pre-commit:
  jobs:
    # Will run: `bundle exec rubocop -- file1 file2 file3 ...`
    - run: {dip} bundle exec rubocop -- {staged_files}
```

```yml
# lefthook-local.yml

templates:
  dip: dip # Will run: `dip bundle exec rubocop -- file1 file2 file3 ...`
```

#### Reduce redundancy

```yml
# lefthook.yml

templates:
  wrapper: docker-compose run --rm -v $(pwd):/app service

pre-commit:
  jobs:
    - run: {wrapper} yarn format
    - run: {wrapper} yarn lint
    - run: {wrapper} yarn test
```

## Remotes

You can provide multiple remote configs if you want to share yours lefthook configurations across many projects. Lefthook will automatically download and merge configurations into your local `lefthook.yml`.

### remotes

You can provide multiple remote configs if you want to share yours lefthook
configurations across many projects. Lefthook will automatically download and merge
configurations into your local `lefthook.yml`.

You can use [`extends`](#extends) but the paths must be relative to the remote repository root.

If you provide [`scripts`](#scripts) in a remote config file, the [scripts](#source_dir) folder must also be in the **root of the repository**.

> **Note:** Configs are merged in this order: `lefthook.yml` → `remotes` → `lefthook-local.yml`. For simplicity, keep jobs in remote configs independent from other steps.

#### Example

```yml
# lefthook.yml

remotes:
  - git_url: git@github.com:evilmartians/lefthook
    ref: v1.0.0
    configs:
      - examples/ruby-linter.yml
```

### git_url

A URL to Git repository. It will be accessed with privileges of the machine lefthook runs on.

#### Example

```yml
# lefthook.yml

remotes:
  - git_url: git@github.com:evilmartians/lefthook
```

Or

```yml
# lefthook.yml

remotes:
  - git_url: https://github.com/evilmartians/lefthook
```

### ref

An optional *branch* or *tag* name.

> **Note:** If you initially had `ref` option, ran `lefthook install`, and then removed
> it, lefthook won't decide which branch/tag to use as a ref. So, if you added it once,
> please, use it always to avoid issues in local setups.

See also [`refetch_frequency`](#refetch_frequency).

#### Example

```yml
# lefthook.yml

remotes:
  - git_url: git@github.com:evilmartians/lefthook
    ref: v1.0.0
```

### refetch

**Default:** `false`

Force remote config refetching on every run. Lefthook will be refetching the specified remote every time it is called.

See [`refetch_frequency`](#refetch_frequency) for more flexible refetching options and additional considerations.

#### Example

```yml
# lefthook.yml

remotes:
  - git_url: https://github.com/evilmartians/lefthook
    refetch: true
```

### refetch_frequency

**Default:** Not set

Specifies how frequently Lefthook should refetch the remote configuration. This can be set to `always`, `never` or a time duration like `24h`, `30m`, etc.

- When set to `always`, Lefthook will always refetch the remote configuration on each run.
- When set to a duration (e.g., `24h`), Lefthook will check the last fetch time and refetch the configuration only if the specified amount of time has passed.
- When set to `never` or not set, Lefthook will not fetch from remote.

It is recommended to configure remotes that point to mutable references
(including ones without a `ref`) to be refetched with some frequency appropriate for the project.

Failure to fetch does not cause an error, but just a warning message.
If a successfully fetched previous configuration exists, it will be used.
Otherwise, the remote will be ignored.

#### Example

```yml
# lefthook.yml

remotes:
  - git_url: https://github.com/evilmartians/lefthook
    refetch_frequency: 24h # Refetches once every 24 hours
```

> **Warning:** If [`refetch`](#refetch) is set to `true`, it overrides any setting in `refetch_frequency`.

### configs

**Default:** `[lefthook.yml]`

An optional array of config paths from remote's root.

#### Example

```yml
# lefthook.yml

remotes:
  - git_url: git@github.com:evilmartians/lefthook
    ref: v1.0.0
    configs:
      - examples/ruby-linter.yml
      - examples/test.yml
```

Example with multiple remotes merging multiple configurations.

```yml
# lefthook.yml

remotes:
  - git_url: git@github.com:org/lefthook-configs
    ref: v1.0.0
    configs:
      - examples/ruby-linter.yml
      - examples/test.yml
  - git_url: https://github.com/org2/lefthook-configs
    configs:
      - lefthooks/pre_commit.yml
      - lefthooks/post_merge.yml
  - git_url: https://github.com/org3/lefthook-configs
    ref: feature/new
    configs:
      - configs/pre-push.yml

```

## Hook options

Settings placed directly under a Git hook name (e.g. `pre-commit`). See [Hook](#hook) in the Structure section for the overall shape of a hook entry.

### parallel

**Default: `false`**

> **Note:** Lefthook runs commands and scripts **sequentially** by default

Run commands and scripts concurrently.

#### Example

```yml
# lefthook.yml

pre-commit:
  parallel: true
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

### piped

**Default: `false`**

> **Note:** Lefthook will return an error if both `piped: true` and `parallel: true` are set

Stop running commands and scripts if one of them fail.

#### Example

```yml
# lefthook.yml

database:
  piped: true # Stop if one of the steps fail
  commands:
    1_create:
      run: rake db:create
    2_migrate:
      run: rake db:migrate
    3_seed:
      run: rake db:seed
```

### follow

**Default: `false`**

Follow the STDOUT of the running commands and scripts.

#### Example

```yml
# lefthook.yml

pre-push:
  follow: true
  commands:
    backend-tests:
      run: bundle exec rspec
    frontend-tests:
      run: yarn test
```

> **Note:** If used with [`parallel`](#parallel) the output can be a mess, so please avoid setting both options to `true`

### files

A custom command executed by the `sh` shell that returns the files or directories to be referenced in `{files}` template. See [`run`](#run) and the per-task [`files`](#files-1).

If the result of this command is empty, the execution of commands will be skipped.

This is the hook-level (global) `files` option. It can be overwritten by the per-task [`files`](#files-1) option under a command or job.

#### Example

```yml
# lefthook.yml

pre-commit:
  files: git diff --name-only master # custom list of files
  commands:
    ...
```

### exclude_tags

[Tags](#tags) or command names that you want to exclude. This option can be overwritten with `LEFTHOOK_EXCLUDE` env variable.

#### Example

```yml
# lefthook.yml

pre-commit:
  exclude_tags: frontend
  commands:
    lint:
      tags: frontend
      ...
    test:
      tags: frontend
      ...
    check-syntax:
      tags: documentation
```

```bash
lefthook run pre-commit # will only run check-syntax command
```

> **Tip:** Useful in `lefthook-local.yml` to skip specific commands locally without modifying the shared config.

```yml
# lefthook.yml

pre-push:
  commands:
    packages-audit:
      tags:
        - frontend
        - security
      run: yarn audit
    gems-audit:
      tags:
        - backend
        - security
      run: bundle audit
```

You can skip commands by tags:

```yml
# lefthook-local.yml

pre-push:
  exclude_tags:
    - frontend
```

### fail_on_changes

The behaviour of lefthook when files (tracked by git) are modified can set by modifying the `fail_on_changes` configuration parameter. The possible values are:

- `never`: never exit with a non-zero status if files were modified (default).
- `always`: always exit with a non-zero status if files were modified.
- `ci`: exit with a non-zero status only when the `CI` environment variable is set. This can be useful when combined with `stage_fixed` to ensure a frictionless devX locally, and a robust CI.
- `non-ci`: exit with a non-zero status only when the `CI` environment variable is *not* set. This can be useful in setups where the CI pipeline commits changes automatically, such as [autofix.ci](https://autofix.ci).

See also [`fail_on_changes_diff`](#fail_on_changes_diff).

#### Example

```yml
# lefthook.yml
pre-commit:
  parallel: true
  fail_on_changes: "always"
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

### fail_on_changes_diff

**Default:** outputs diff only in CI

When [`fail_on_changes`](#fail_on_changes) triggers, lefthook can optionally print a diff of the detected changes. Set this boolean to explicitly enable or disable the diff output regardless of environment.

#### Example

```yml
# lefthook.yml
pre-commit:
  parallel: true
  fail_on_changes: "always"
  fail_on_changes_diff: true
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

### setup

> **Tip (New feature):** Added in lefthook `2.1.2`

A list of instructions to run before any job. Supports templates and Git args like in [`run`](#run).

> **Note:** When merging configs (with `lefthook-local.yml` or files from
> [`extends`](#extends)) `setup` instructions get **prepended**. When there are multiple
> `extends`, they get **appended** in the same order as extend files are specified.

#### Example

```yml
# lefthook.yml

pre-commit:
  setup:
    - run: |
        if ! command -v golangci-lint >/dev/null 2>&1; then
          go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.10.1
        fi
  jobs:
    - run: golangci-lint -- {staged_files}
      glob: "*.go"
```

### jobs

See [jobs](#jobs-1) in the Structure section for the full description of jobs and groups. A hook may contain a `jobs` array as its primary list of tasks.

### commands

See [Commands](#commands-1) in the Structure section. A hook may contain a `commands` map of named commands.

### scripts

See [Scripts](#scripts) in the Structure section. A hook may contain a `scripts` map keyed by script file name.

The hook-level scheduling options [`skip`](#skip), [`only`](#only), and
[`exclude`](#exclude) can also be applied at the hook level to control the whole hook;
they are documented under Command / Job / Script options below.

## Command / Job / Script options

Per-task settings. Most apply to commands and jobs; some apply only to scripts or only to
specific hooks. Where an option also has meaning at the hook level (e.g. [`skip`](#skip),
[`only`](#only), [`exclude`](#exclude)), that is noted in the option.

### name

Name of a job. Will be printed in summary. If specified, the jobs can be merged with a jobs of the same name in a local config (`lefthook-local.yml`) or [extends](#extends).

#### Example

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: lint and fix
      run: yarn run eslint --fix {staged_files}
```

### run

This is a mandatory option for a command, which specifies the actual command to be run using the `sh` shell.

You can use files templates that will be substituted with the appropriate files on execution:

- `{files}` - custom [`files`](#files-1) command result.
- `{staged_files}` - staged files which you try to commit.
- `{push_files}` - files that are committed but not pushed.
- `{all_files}` - all files tracked by git.
- `{cmd}` - shorthand for the command from `lefthook.yml`.
- `{0}` - shorthand for the single space-joint string of git hook arguments.
- `{1}` - shorthand for the 1-st git hook argument (and so on for `{2}`, `{3}`, etc.)
- `{lefthook_job_name}` - current job/command/script name

> **Note:** Command line length has a limit on every system. If your list of files is quite long, lefthook splits your files list to fit in the limit and runs few commands sequentially.

#### Example

Run `yarn lint` on `pre-commit` hook.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint
```

#### `{files}` template

Run `go vet` only on files listed with `git ls-files -m` command with `.go` extension.

```yml
# lefthook.yml

pre-commit:
  commands:
    govet:
      files: git ls-files -m
      glob: "*.go"
      run: go vet -- {files}
```

#### `{staged_files}`

Run `yarn eslint` only on staged files with `.js`, `.ts`, `.jsx`, and `.tsx` extensions.

```yml
# lefthook.yml

pre-commit:
  commands:
    eslint:
      glob: "*.{js,ts,jsx,tsx}"
      run: yarn eslint {staged_files}
```

#### `{push_files}`

If you want to lint files only before pushing them.

```yml
# lefthook.yml

pre-push:
  commands:
    eslint:
      glob: "*.{js,ts,jsx,tsx}"
      run: yarn eslint {push_files}
```

#### `{all_files}`

Simply run `bundle exec rubocop` on all files with `.rb` extension excluding `application.rb` and `routes.rb` files.

> **Note:** `--force-exclusion` will apply `Exclude` configuration setting of Rubocop

```yml
# lefthook.yml

pre-commit:
  commands:
    rubocop:
      tags:
        - backend
        - style
      glob: "*.rb"
      exclude:
        - config/application.rb
        - config/routes.rb
      run: bundle exec rubocop --force-exclusion -- {all_files}
```

#### `{cmd}`

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint
  scripts:
    "good_job.js":
      runner: node
```

You can wrap it in docker runner locally:

```yml
# lefthook-local.yml

pre-commit:
  commands:
    lint:
      run: docker run -it --rm <container_id_or_name> {cmd}
  scripts:
    "good_job.js":
      runner: docker run -it --rm <container_id_or_name> {cmd}
```

#### Git arguments

Make sure commits are signed.

```yml
# lefthook.yml

# Note: commit-msg hook takes a single parameter,
#       the name of the file that holds the proposed commit log message.
# Source: https://git-scm.com/docs/githooks#_commit_msg
commit-msg:
  commands:
    multiple-sign-off:
      run: 'test $(grep -c "^Signed-off-by: " {1}) -lt 2'
```

#### Rubocop

If using `{all_files}` with RuboCop, it will ignore RuboCop's `Exclude` configuration setting. To avoid this, pass `--force-exclusion`.

#### Quotes

If you want to have all your files quoted with double quotes `"` or single quotes `'`, quote the appropriate shorthand:

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      glob: "*.js"
      # Quoting with double quotes `"` might be helpful for Windows users
      run: yarn eslint "{staged_files}" # will run `yarn eslint "file1.js" "file2.js" "[strange name].js"`
    test:
      glob: "*.{spec.js}"
      run: yarn test '{staged_files}' # will run `yarn eslint 'file1.spec.js' 'file2.spec.js' '[strange name].spec.js'`
    format:
      glob: "*.js"
      # Will quote where needed with single quotes
      run: yarn test {staged_files} # will run `yarn eslint file1.js file2.js '[strange name].spec.js'`
```

#### Scripts

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: a whole script in a run
      run: |
        for file in $(ls .); do
          yarn lint $file
        done
```

### script

Name of a script to execute. The rules are the same as for [`scripts`](#scripts).

#### Example

```yml
# lefthook.yml

pre-commit:
  jobs:
    - script: linter.sh
      runner: bash
```

```bash
# .lefthook/pre-commit/linter.sh

echo "Everything is OK"
```

### runner

You should specify a runner for the script. This is a command that should execute a script file. It will be called the following way: `<runner> <path-to-script>` (e.g. `ruby .lefthook/pre-commit/lint.rb`).

#### Example

```yml
# lefthook.yml

pre-commit:
  scripts:
    "lint.js":
      runner: node
    "check.go":
      runner: go run
```

### args

> **Tip (New feature):** Added in lefthook `2.0.5`

Sometimes you want to pass arguments to the scripts or be able to overwrite arguments to
the commands in `lefthook-local.yml`. For this you can use `args` option which will simply
be appended to the command. You can use the same templates as in [`run`](#run).

Arguments passed by Git will be omitted if you specify `args` in the config. Providing no `args` or providing `args: "{0}"` works the same way.

See [`run`](#run) for supported templates.

#### Example

```yml
# lefthook.yml

pre-commit:
  jobs:
    - script: check-python-files.sh
      runner: bash
      args: "{staged_files}"
      glob: "*.py"

    - run: yarn lint
      args: "{staged_files}"
      glob:
        - "*.ts"
        - "*.js"
```

### group

You can define a group of jobs and configure how they should execute. See
[group](#group-1) in the Structure section for the full description and examples. Within a
job the `group` key takes the following sub-options:

- [`parallel`](#parallel): Executes all jobs in the group simultaneously.
- [`piped`](#piped): Executes jobs sequentially, passing output between them.
- [`jobs`](#jobs-1): Specifies the jobs within the group.

### tags

You can specify tags for commands and scripts. This is useful for [excluding](#exclude_tags). You can specify more than one tag using comma.

#### Example

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      tags:
        - frontend
        - js
      run: yarn lint
    test:
      tags:
        - backend
        - ruby
      run: bundle exec rspec
```

### glob

You can set a glob to filter files for your command. This is only used if you use a file template in [`run`](#run) option or provide your custom [`files`](#files-1) command.

#### Example

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: lint
      run: yarn eslint {staged_files}
      glob: "*.{js,ts,jsx,tsx}"
```

> **Note:** From lefthook version `1.10.10` you can also provide a list of globs:
>
> ```yml
> # lefthook.yml
>
> pre-commit:
>   jobs:
>     - run: yarn lint {staged_files}
>       glob:
>         - "*.ts"
>         - "*.js"
> ```

For patterns that you can use see [this](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm) reference. We use [glob](https://github.com/gobwas/glob) library.

#### When using `root`

Globs are still calculated from the actual root of the git repo — `root` is ignored.

#### Behaviour of `**`

The `**` pattern matches **1 or more** directories deep (not zero or more, unlike most
other tools). To match files at both the top level and nested, use separate patterns or
opt-in to standard behavior with [`glob_matcher: doublestar`](#glob_matcher).

```yaml
glob: "src/**/*.js"  # does NOT match src/file.js
glob: "src/*.js"     # matches src/file.js only
```

#### Using `glob` without a files template in `run`

If you've specified `glob` but don't have a files template in [`run`](#run) option,
lefthook will check `{staged_files}` for `pre-commit` hook and `{push_files}` for
`pre-push` hook and apply filtering. If no files left, the command will be skipped.

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: lint
      run: npm run lint # skipped if no .js files staged
      glob: "*.js"
```

### files

A custom command executed by the `sh` shell that returns the files or directories to be referenced in `{files}` template for [`run`](#run) setting.

If the result of this command is empty, the execution of commands will be skipped.

This option overwrites the [hook-level `files`](#files) option.

#### Example

Provide a git command to list files.

```yml
# lefthook.yml

pre-push:
  commands:
    stylelint:
      tags:
        - frontend
        - style
      files: git diff --name-only master
      glob: "*.js"
      run: yarn stylelint {files}
```

Call a custom script for listing files.

```yml
# lefthook.yml

pre-push:
  commands:
    rubocop:
      tags: backend
      glob: "**/*.rb"
      files: node ./lefthook-scripts/ls-files.js # you can call your own scripts
      run: bundle exec rubocop --force-exclusion --parallel -- {files}
```

### file_types

Filter files in a [`run`](#run) templates by their type. Special file types and MIME types are supported[^1]:

| File type | Explanation |
| --------- | ----------- |
| `text` | Any file that contains text. Symlinks are not followed. |
| `binary` | Any file that contains non-text bytes. Symlinks are not followed. |
| `executable` | Any file that has executable bits set. Symlinks are not followed. |
| `not executable` | Any file without executable bits in file mode. Symlinks included. |
| `symlink` | A symlink file. |
| `not symlink` | Any non-symlink file. |
| `text/html` | An HTML file. |
| `text/xml` | An XML file. |
| `text/javascript` | A Javascript file. |
| `text/x-php` | A PHP file. |
| `text/x-lua` | A Lua file. |
| `text/x-perl` | A Perl file. |
| `text/x-python` | A Python file. |
| `text/x-shellscript` | Shell script file. |
| `text/x-sh` | Also shell script file. |
| `application/json` | JSON file. |

> **Note:** The following types are applied using AND logic: `text`, `binary`, `executable`, `not executable`, `symlink`, `not symlink`.
>
> MIME types are applied using OR logic — you can combine `text/x-lua` and `text/x-sh`, but not `symlink` and `not symlink`.

#### Example

Apply some different linters on text and binary files.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint-code:
      run: yarn lint {staged_files}
      file_types: text
    check-hex-codes:
      run: yarn check-hex {staged_files}
      file_types: binary
```

Skip symlinks.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint --fix {staged_files}
      file_types:
        - not symlink
```

Lint executable scripts.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint --fix {staged_files}
      file_types:
        - executable
        - text
```

Check typos in scripts.

```yml
# lefthook.yml

pre-commit:
  jobs:
    - run: typos -w -- {staged_files}
      file_types:
        - text/x-perl
        - text/x-python
        - text/x-php
        - text/x-lua
        - text/x-sh
```

[^1]: All supported MIME types can be found here: [supported_mimes.md](https://github.com/gabriel-vasile/mimetype/blob/v1.4.11/supported_mimes.md)

### env

You can specify some ENV variables for the command or script.

#### Example

```yml
# lefthook.yml

pre-commit:
  commands:
    test:
      env:
        RAILS_ENV: test
      run: bundle exec rspec
```

#### Extending `PATH`

If your hook is run by a GUI program and you use PATH tweaks in your `~/.<shell>rc`, you might see an *executable not found* error. You can extend `$PATH` via `lefthook-local.yml`:

```yml
# lefthook.yml

pre-commit:
  commands:
    test:
      run: yarn test
```

```yml
# lefthook-local.yml

pre-commit:
  commands:
    test:
      env:
        PATH: $PATH:/home/me/path/to/yarn
```

> **Tip:** Useful when running lefthook across different OSes or shells where environment variables are set differently.

### root

You can change the CWD for the command you execute using `root` option.

This is useful when you execute some `npm` or `yarn` command but the `package.json` is in another directory.

For `pre-push` and `pre-commit` hooks and for the custom `files` command `root` option is used to filter file paths. If all files are filtered the command will be skipped.

#### Example

Format and stage files from a `client/` folder.

```bash
# Folders structure

$ tree .
.
├── client/
│   ├── package.json
│   ├── node_modules/
|   ├── ...
├── server/
|   ...
```

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      root: "client/"
      glob: "*.{js,ts}"
      run: yarn eslint --fix {staged_files} && git add {staged_files}
```

> **Note:** Globs are always calculated from the actual root of the git repo — `root` does not affect glob matching.

### exclude

This option allows to setup a list of globs for files to be excluded in files template.

> **Note:** The glob patterns used in `exclude` are affected by the [`glob_matcher`](#glob_matcher) setting. See the glob_matcher documentation for details on how `**` patterns behave.

#### Example

Run Rubocop on staged files with `.rb` extension except for `application.rb`, `routes.rb`, `rails_helper.rb`, and all Ruby files in `config/initializers/`.

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: lint
      glob: "*.rb"
      exclude:
        - config/routes.rb
        - config/application.rb
        - config/initializers/*.rb
        - spec/rails_helper.rb
      run: bundle exec rubocop --force-exclusion -- {staged_files}
```

If you've specified `exclude` but don't have a files template in [`run`](#run) option,
lefthook will check `{staged_files}` for `pre-commit` hook and `{push_files}` for
`pre-push` hook and apply filtering. If no files left, the command will be skipped.

```yml
# lefthook.yml

pre-commit:
  exclude:
    - "*/application.rb"
  jobs:
    - name: lint
      run: bundle exec rubocop # will skip if only application.rb was staged
```

### only

You can force a command, script, or the whole hook to execute only in certain conditions.
This option acts like the opposite of [`skip`](#skip). It accepts the same values but
skips execution only if the condition is not satisfied.

> **Note:** `skip` option takes precedence over `only` option, so if you have conflicting conditions the execution will be skipped.

#### Example

Execute a hook only for `dev/*` branches.

```yml
# lefthook.yml

pre-commit:
  only:
    - ref: dev/*
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

When rebasing execute quick linter but skip usual linter and tests.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      skip: rebase
      run: yarn lint
    test:
      skip: rebase
      run: yarn test
    lint-on-rebase:
      only: rebase
      run: yarn lint-quickly
```

### skip

You can skip all or specific commands and scripts using `skip` option. You can also skip when merging, rebasing, or being on a specific branch. Globs are available for branches.

Possible skip values:

- `rebase` - when in rebase git state
- `merge` - when in merge git state
- `merge-commit` - when current HEAD commit is the merge commit
- `ref: main` - when on a `main` branch
- `run: test ${SKIP_ME} -eq 1` - when `test ${SKIP_ME} -eq 1` is successful (return code is 0)

#### Example

Always skipping a command:

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      skip: true
      run: yarn lint
```

Skipping on merging and rebasing:

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      skip:
        - merge
        - rebase
      run: yarn lint
```

Or

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      skip: merge
      run: yarn lint
```

Skipping when your are on a merge commit:

```yml
# lefthook.yml

pre-push:
  commands:
    lint:
      skip: merge-commit
      run: yarn lint
```

Skipping the whole hook on `main` branch:

```yml
# lefthook.yml

pre-commit:
  skip:
    - ref: main
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

Skipping hook for all `dev/*` branches:

```yml
# lefthook.yml

pre-commit:
  skip:
    - ref: dev/*
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

Skipping hook by running a command:

```yml
# lefthook.yml

pre-commit:
  skip:
    - run: test "${NO_HOOK}" -eq 1
  commands:
    lint:
      run: yarn lint
    test:
      run: yarn test
```

Skipping a command conditionally based on the existence of a CLI tool:

```yml
prepare-commit-msg:
  skip:
    - merge
    - rebase
  commands:
    aiautocommit:
      interactive: true
      run: aiautocommit commit --output-file "{1}"
      env:
        LOG_LEVEL: info
      skip:
        # only run this if the tool exists
        - run: "! which aiautocommit"
```

> **Tip:** Always skipping is useful when you have a `lefthook-local.yml` config and you don't want to run some commands locally. So you just overwrite the `skip` option for them to be `true`.
>
> ```yml
> # lefthook.yml
>
> pre-commit:
>   commands:
>     lint:
>       run: yarn lint
> ```
>
> ```yml
> # lefthook-local.yml
>
> pre-commit:
>   commands:
>     lint:
>       skip: true
> ```

### fail_text

You can specify a text to show when the command or script fails.

#### Example

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint
      fail_text: Add node executable to $PATH
```

```bash
$ git commit -m 'fix: Some bug'

Lefthook v1.1.3
RUNNING HOOK: pre-commit

  EXECUTE > lint

SUMMARY: (done in 0.01 seconds)
🥊  lint: Add node executable to $PATH env
```

### stage_fixed

**Default: `false`**

> **Note:** Works **only** for the `pre-commit` hook.

When set to `true` lefthook will automatically call `git add` on files after running the
command or script. For a command if [`files`](#files-1) option was specified, the
specified command will be used to retrieve files for `git add`. For scripts and commands
without [`files`](#files-1) option `{staged_files}` template will be used. All filters
([`glob`](#glob), [`exclude`](#exclude)) will be applied if specified.

#### Example

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: npm run lint --fix {staged_files}
      stage_fixed: true
```

### interactive

**Default: `false`**

> **Note:** If you want to pass stdin to your command or script but don't need to get the input from CLI, use [`use_stdin`](#use_stdin) option instead.

Whether to use interactive mode. This applies the certain behavior:

- All `interactive` commands/scripts are executed after non-interactive. Exception: [`piped`](#piped) option is set to `true`.
- When executing, lefthook tries to open /dev/tty (Linux/Unix only) and use it as stdin.
- When [`no_tty`](#no_tty) option is set, `interactive` is ignored.

### use_stdin

> **Note:** With many commands or scripts having `use_stdin: true`, only one will receive
> the data. The others will have nothing. If you need to pass the data from stdin to every
> command or script, please, submit a [feature request](https://github.com/evilmartians/lefthook/issues/new?assignees=&labels=feature+request&projects=&template=feature_request.md).

Pass the stdin from the OS to the command/script.

#### Example

Use this option for the `pre-push` hook when you have a script that does `while read ...`.
Without this option lefthook will hang: lefthook uses [pseudo TTY](https://github.com/creack/pty)
by default, and it doesn't close stdin when all data is read.

```bash
# .lefthook/pre-push/do-the-magic.sh

remote="$1"
url="$2"

while read local_ref local_oid remote_ref remote_oid; do
  # ...
done
```

```yml
# lefthook.yml
pre-push:
  scripts:
    "do-the-magic.sh":
      runner: bash
      use_stdin: true
```

### priority

**Default: `0`**

> **Note:** This option makes sense only when `parallel: false` or `piped: true` is set.
>
> Value `0` is considered an `+Infinity`, so commands or scripts with `priority: 0` or without this setting will be run at the very end.

Set priority from 1 to +Infinity. This option can be used to configure the order of the sequential steps.

#### Example

```yml
# lefthook.yml

post-checkout:
  piped: true
  commands:
    db-create:
      priority: 1
      run: rails db:create
    db-migrate:
      priority: 2
      run: rails db:migrate
    db-seed:
      priority: 3
      run: rails db:seed

  scripts:
    "check-spelling.sh":
      runner: bash
      priority: 1
    "check-grammar.rb":
      runner: ruby
      priority: 2
```

## Structure

The structural building blocks of a Lefthook config: a Git hook entry, and the three task containers it can hold (commands, scripts, jobs), plus job groups.

### Hook

Contains settings for the git hook (commands, scripts, skip rules, etc.). You can specify any Git hook or your own custom, e.g. `test`

#### Example

```yml
# lefthook.yml

# Git hook
pre-commit:
  jobs:
    - run: yarn lint {staged_files} --fix
      stage_fixed: true

# Custom hook
check-docs:
  jobs:
    - run: yarn check-docs
    - run: typos
```

Hook-level options include [`files` (global)](#files), [`parallel`](#parallel),
[`piped`](#piped), [`follow`](#follow), [`fail_on_changes`](#fail_on_changes),
[`fail_on_changes_diff`](#fail_on_changes_diff), [`exclude_tags`](#exclude_tags),
[`exclude`](#exclude), [`skip`](#skip), [`only`](#only), [`setup`](#setup), and the task
containers [`jobs`](#jobs-1), [`commands`](#commands-1), and [`scripts`](#scripts).

### Commands

Commands to be executed for the hook. Each command has a name and associated run options.

#### Example

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      ... # command options
```

#### Command options

- [`run`](#run)
- [`skip`](#skip)
- [`only`](#only)
- [`tags`](#tags)
- [`glob`](#glob)
- [`files`](#files-1)
- [`file_types`](#file_types)
- [`env`](#env)
- [`root`](#root)
- [`exclude`](#exclude)
- [`fail_text`](#fail_text)
- [`stage_fixed`](#stage_fixed)
- [`interactive`](#interactive)
- [`use_stdin`](#use_stdin)
- [`priority`](#priority)

### Scripts

Scripts are stored under `<source_dir>/<hook-name>/` folder. These scripts are your own executables which are being run in the project root.

To add a script for a `pre-commit` hook:

1. Run `lefthook add -d pre-commit`
2. Edit `.lefthook/pre-commit/my-script.sh`
3. Add an entry to `lefthook.yml`

   ```yml
   # lefthook.yml

   pre-commit:
     scripts:
       "my-script.sh":
         runner: bash
   ```

#### Example

Let's create a bash script to check commit templates `.lefthook/commit-msg/template_checker`:

```bash
INPUT_FILE=$1
START_LINE=`head -n1 $INPUT_FILE`
PATTERN="^(TICKET)-[[:digit:]]+: "
if ! [[ "$START_LINE" =~ $PATTERN ]]; then
  echo "Bad commit message, see example: TICKET-123: some text"
  exit 1
fi
```

Now we can ask lefthook to run our bash script by adding this code to
`lefthook.yml` file:

```yml
# lefthook.yml

commit-msg:
  scripts:
    "template_checker":
      runner: bash
```

When you try to commit `git commit -m "bad commit text"` script `template_checker` will be executed. Since commit text doesn't match the described pattern the commit process will be interrupted.

#### Script options

- [`runner`](#runner)
- [`args`](#args)
- [`skip`](#skip)
- [`only`](#only)
- [`tags`](#tags)
- [`env`](#env)
- [`fail_text`](#fail_text)
- [`stage_fixed`](#stage_fixed)
- [`interactive`](#interactive)
- [`use_stdin`](#use_stdin)
- [`priority`](#priority)

### jobs

> **Tip (New feature):** Added in lefthook `1.10.0`

Jobs provide a flexible way to define tasks, supporting both commands and scripts. Jobs can be grouped for advanced flow control.

Named jobs are merged across [`extends`](#extends) and local config; unnamed jobs are
appended in definition order. Groups can include other jobs with their own parallel or
piped flow — `glob`, `root`, and `exclude` on a group apply to all nested jobs.

#### Example

> **Note:** Currently, only `root`, `glob`, and `exclude` options are applied to group
> jobs. Other options must be set for each job individually. Submit a [feature request](https://github.com/evilmartians/lefthook/issues/new?assignees=&labels=feature+request&projects=&template=feature_request.md)
> if this limits your workflow.

A configuration demonstrating a piped group running in parallel with other jobs:

```yml
# lefthook.yml

pre-commit:
  parallel: true
  jobs:
    - name: migrate
      root: backend/
      glob: "db/migrations/*"
      group:
        piped: true
        jobs:
          - run: bundle install
          - run: rails db:migrate
    - run: yarn lint --fix {staged_files}
      root: frontend/
      stage_fixed: true
    - run: bundle exec rubocop
      root: backend/
    - run: golangci-lint
      root: proxy/
    - script: verify.sh
      runner: bash
```

This configuration runs migrate jobs in a piped flow while other jobs execute in parallel.

#### Job options

A job uses [`name`](#name), [`run`](#run) (for command jobs) or [`script`](#script) +
[`runner`](#runner) (for script jobs), [`args`](#args), [`group`](#group-1),
[`skip`](#skip), [`only`](#only), [`tags`](#tags), [`glob`](#glob), [`files`](#files-1),
[`file_types`](#file_types), [`env`](#env), [`root`](#root), [`exclude`](#exclude),
[`fail_text`](#fail_text), [`stage_fixed`](#stage_fixed), [`interactive`](#interactive),
and [`use_stdin`](#use_stdin).

### group

You can define a group of jobs and configure how they should execute using the following options:

- [`parallel`](#parallel): Executes all jobs in the group simultaneously.
- [`piped`](#piped): Executes jobs sequentially, passing output between them.
- [`jobs`](#jobs-1): Specifies the jobs within the group.

#### Example

```yml
# lefthook.yml

pre-commit:
  jobs:
    - group:
        parallel: true
        jobs:
          - run: echo 1
          - run: echo 2
          - run: echo 3
```

If you specify `env`, `root`, `glob`, or `exclude` on a group, they will be inherited to the underlying jobs.

```yml
# lefthook.yml

pre-commit:
  jobs:
    - env:
        E1: hello
      glob:
        - "*.md"
      exclude:
        - "README.md"
      root: "subdir/"
      group:
        parallel: true
        jobs:
          - run: echo $E1
          - run: echo $E1
            env:
              E1: bonjour
```

> **Note:** To make a group mergeable with settings defined in local config or extends you have to specify the name of the job group belongs to:
>
> ```yml
> pre-commit:
>   jobs:
>     - name: a name of a group
>       group:
>         jobs:
>           - name: lint
>             run: yarn lint
>           - name: test
>             run: yarn test
> ```

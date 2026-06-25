# Lefthook CLI reference

Here are the most common usage cases. You can find more info in the docs.

## Basic CLI commands

```bash
# Create/update Git hooks based on lefthook.yml, or create an empty lefthook.yml
lefthook install

# Run pre-commit hook commands and scripts (requires lefthook.yml)
lefthook run pre-commit

# Validate the configuration
lefthook validate

# Dump the configuration (useful when you have remotes, extends that overwrite the configuration)
lefthook dump
```

## Skip running lefthook when committing changes

```bash
LEFTHOOK=0 git commit
```

## lefthook add

Installs the given hook to Git hook.

With argument `--dirs` creates a directory `.git/hooks/<hook name>/` if it doesn't exist. Use it before adding a script to configuration.

### Example

```bash
lefthook add pre-push  --dirs
```

Describe pre-push commands in `lefthook.yml`:

```yml
pre-push:
  jobs:
    - script: "audit.sh"
      runner: bash
```

Edit the script:

```bash
$ vim .lefthook/pre-push/audit.sh
...
```

Run `git push` and lefthook will run `bash audit.sh` as a pre-push hook.

## lefthook check-install

Checks if Git hooks are installed and synchronized.

Returns:

- `0` if hooks installed and synchronized
- `1` if hooks not installed or need a sync

## lefthook dump

Prints the whole configuration after merging all secondary configs.

This is the actual config lefthook uses, it can be build from the main config (`lefthook.yml`), remotes, extends, and `lefthook-local.yml` overrides.

## lefthook install

Creates an empty `lefthook.yml` if a configuration file does not exist.

Installs configured hooks to Git hooks.

> **Note:** Reinstall is not required when you modify `lefthook.yml`, the configuration file is read every time a git hook is run.
>
> **Note:** NPM package `lefthook` installs the hooks in a postinstall script automatically. For projects not using NPM package run `lefthook install` after cloning the repo.

### Installing specific hooks

You can install only specific hooks by running `lefthook install <hook-1> <hook-2> ...`.

## lefthook run

Executes the commands and scripts configured for a given hook. Installed Git hooks call `lefthook run` implicitly.

### Example

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: lint
      run: yarn lint --fix {staged_files}

test:
  jobs:
    - name: test
      run: yarn test
```

Install the hook.

```bash
lefthook install
```

```bash
lefthook run test # will run 'yarn test'
git commit # will run pre-commit hook ('yarn lint --fix')
lefthook run pre-commit # will run pre-commit hook (`yarn lint --fix`)
```

### Run specific jobs

You can specify which jobs to run (also `--tag` supported).

```bash
lefthook run pre-commit --job lints --job pretty --tag checks
```

### Specify files

You can force replacing files templates (like `{staged_files}`) with either all files (will acts as `{all_files}` template) or a list of files.

```bash
lefthook run pre-commit --all-files
lefthook run pre-commit --file file1.js --file file2.js
```

(if both are specified, `--all-files` is ignored)

## lefthook self-update

Updates the binary with the latest lefthook release on Github.

This command is available only if you install lefthook from sources or download the binary from the Github Releases. For other ways use package-specific commands to update lefthook.

## lefthook uninstall

Clears Git hooks installed by lefthook.

## lefthook validate

Validates your lefthook configuration. Use `lefthook dump` to see it.

It uses JSON schema from the lefthook Github repo.

## lefthook version

`lefthook version` prints the current binary version. Print the commit hash with `lefthook version --full`

### Example

```bash
$ lefthook version --full

1.1.3 bb099d13c24114d2859815d9d23671a32932ffe2
```

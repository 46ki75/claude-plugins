# Lefthook examples

A consolidated reference of practical Lefthook configurations for common workflows.

## Commitlint

Commitlint and commitzen.

Use lefthook to generate commit messages using commitzen and validate them with commitlint.

### Install dependencies

```bash
yarn add -D @commitlint/cli @commitlint/config-conventional

# For commitzen
yarn add -D commitizen cz-conventional-changelog
```

### Configure

Setup `commitlint.config.js`. Conventional configuration:

```js
// commitlint.config.js

module.exports = {extends: ['@commitlint/config-conventional']};
```

If you are using commitzen, make sure to add this in `package.json`:

```json
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

Configure lefthook:

```yml
# lefthook.yml

# Build commit messages
prepare-commit-msg:
  commands:
    commitzen:
      interactive: true
      run: yarn run cz --hook # Or npx cz --hook
      env:
        LEFTHOOK: 0

# Validate commit messages
commit-msg:
  commands:
    "lint commit message":
      run: yarn run commitlint --edit {1}
```

### Test it

```bash
# You can type it without message, if you are using commitzen
git commit

# Or provide a commit message is using only commitlint
git commit -am 'fix: typo'
```

## Filtering files

Files passed to your hooks can be filtered with the following options

- `glob`
- `exclude`
- `file_types`
- `root`

In this example all **staged files** will pass through these filters.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint {staged_files} --fix
      glob: "*.{js,ts}"
      root: frontend
      exclude:
        - *.config.js
        - *.config.ts
      file_types:
        - not executable
```

Imagine you've staged the following files

```bash
backend/asset.js
frontend/src/index.ts
frontend/bin/cli.js # <- executable
frontend/eslint.config.js
frontend/README.md
```

After all filters applied the `lint` command will execute the following:

```bash
yarn lint frontend/src/index.ts --fix
```

## Local config

`lefthook-local.yml`

> **Tip:** You can put `lefthook-local.yml` into your `~/.gitignore`, so in every project you can have your local-only overrides.

`lefthook-local.yml` overrides and extends the configuration of your main `lefthook.yml`.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: bundle exec rubocop -- {staged_files}
      glob: "*.rb"
    check-links:
      run: lychee -- {staged_files}
```

```yml
# lefthook-local.yml

pre-commit:
  parallel: true # run all commands concurrently
  commands:
    lint:
      run: docker-compose run backend {cmd} # wrap the original command with docker-compose
    check-links:
      skip: true # skip checking links

# Add another hook
post-merge:
  files: "git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD"
  commands:
    dependencies:
      glob: "Gemfile*"
      run: docker-compose run backend bundle install
```

### The merged config lefthook will use

```yml

pre-commit:
  parallel: true
  commands:
    lint:
      run: docker-compose run backend bundle exec rubocop -- {staged_files}
      glob: "*.rb"
    check-links:
      run: lychee -- {staged_files}
      skip: true

post-merge:
  files: "git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD"
  commands:
    dependencies:
      glob: "Gemfile*"
      run: docker-compose run backend bundle install
```

## Remotes

Use configurations from other Git repositories via `remotes` feature.

Lefthook will automatically download the remote config files and merge them into existing configuration.

```yml
remotes:
  - git_url: https://github.com/evilmartians/lefthook
    configs:
      - examples/remote/ping.yml
```

## Skipping

Skip or run on condition.

Here are two hooks.

`pre-commit` hook will only be executed when you're committing something on a branch starting with `dev/` prefix.

In `pre-push` hook:

- `test` command will be skipped if `NO_TEST` env variable is set to `1`
- `lint` command will only be executed if you're pushing the `main` branch

```yml
# lefthook.yml

pre-commit:
  only:
    - ref: dev/*
  commands:
    lint:
      run: yarn lint {staged_files} --fix
      glob: "*.{ts,js}"
    test:
      run: yarn test

pre-push:
  commands:
    test:
      run: yarn test
      skip:
        - run: test "$NO_TEST" -eq 1
    lint:
      run: yarn lint
      only:
        - ref: main
```

## Stage fixed files

> Works only for `pre-commit` Git hook

Sometimes your linter fixes the changes and you usually want to commit them automatically. To enable auto-staging of the fixed files use `stage_fixed` option.

```yml
# lefthook.yml

pre-commit:
  commands:
    lint:
      run: yarn lint {staged_files} --fix
      stage_fixed: true
```

## Wrapping commands

Wrap commands in local config.

Wrapping some commands defined in a main config with `dip`[^1].

```yml
# lefthook.yml

pre-commit:
  jobs:
    - name: rubocop
      run: bundle exec rubocop -A -- {staged_files}
```

```yml
# lefthook-local.yml

pre-commit:
  jobs:
    - name: rubocop
      run: dip {cmd}
```

[^1]: [dip](https://github.com/bibendi/dip) – dockerized dev experience with, similar to `docker-compose run`

# Lefthook features reference

A consolidated reference for Lefthook usage features: capturing Git arguments, Git LFS support, interactive commands, local config, and passing stdin.

## Git arguments capture

Lefthook passes Git arguments to your commands and scripts.

```text
├── .lefthook
│   └── prepare-commit-msg
│       └── message.sh
└── lefthook.yml
```

```yml
# lefthook.yml

prepare-commit-msg:
  jobs:
    - script: "message.sh"
      runner: bash
    - run: echo "Git args: {1} {2} {3}"
```

```bash
# .lefthook/prepare-commit-msg/message.sh

# Arguments get passed from Git

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2
SHA1=$3

# ...
```

## Git LFS support

> **Note:** If git-lfs binary is not installed and not required in your project, LFS hooks won't be executed, and you won't be warned about it.
>
> Git LFS hooks may be slow. Disable them with the global `skip_lfs: true` setting.

Lefthook runs LFS hooks internally for the following hooks:

- post-checkout
- post-commit
- post-merge
- pre-push

Errors are suppressed if git LFS is not required for the project. You can use `LEFTHOOK_VERBOSE` ENV to make lefthook show git LFS output.

To avoid calling LFS hooks set `skip_lfs: true` in lefthook.yml or lefthook-local.yml

## Interactive commands

When you need to interact with user – specify `interactive: true`. Lefthook will connect to the current TTY and forward it to your command's or script's stdin.

## Local config

You can extend and override options of your main configuration with `lefthook-local.yml`. Don't forget to add the file to `.gitignore`.

You can also use `lefthook-local.yml` without a main config file. This is useful when you want to use lefthook locally without imposing it on your teammates.

```yml
# lefthook.yml (committed into your repo)

pre-commit:
  jobs:
    - name: linter
      run: yarn lint
    - name: tests
      run: yarn test
```

```yml
# lefthook-local.yml (ignored by git)

pre-commit:
  jobs:
    - name: tests
      skip: true # don't want to run tests on every commit
    - name: linter
      run: yarn lint {staged_files} # lint only staged files
```

## Passing stdin

When you need to read the data from stdin – specify `use_stdin: true`. This option is good when you write a command or script that receives data from git using stdin (for the `pre-push` hook, for example).

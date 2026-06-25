# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Claude Code plugin marketplace. Each plugin lives under `plugins/<name>/`
with a `.claude-plugin/plugin.json` manifest; the marketplace itself is declared
in `.claude-plugin/marketplace.json`. Most plugins bundle Agent Skills
(`skills/<skill>/SKILL.md`). A Rust workspace under `crates/` validates, archives,
and publishes every skill as an agentskills.io ZIP release.

## Commands

| Command | Description |
| --- | --- |
| `just check` | Pre-flight before pushing: markdown lint + validate every skill. |
| `pnpm run lint` | Markdown lint only (`markdownlint-cli2`). |
| `cargo run -p skill-cli -- check` | Validate every skill's frontmatter and name uniqueness. |
| `cargo test --workspace --all-targets` | Run all Rust tests. |
| `cargo test -p <crate>` | Test a single crate (e.g. `mcp-server`, `toasty-app`). |
| `cargo fmt --all -- --check` | Format check (CI gate). |
| `cargo clippy --workspace --all-targets` | Lint (CI gate). |
| `git submodule update --init --recursive` | Init submodules (run at repo root). |

## Architecture

Two parallel concerns share the repo:

- **Content** — the published plugins and standalone skills. This is the product.
- **Tooling** — the `crates/` Rust workspace that packages and ships that content.

Directory map:

- `plugins/<name>/` — published plugins (each a marketplace entry); most bundle
  `skills/<skill>/SKILL.md`.
- `skills/` — standalone skills intentionally **not** published as plugins
  (`kedb`, `prompt-evaluation`).
- `crates/` — Rust workspace: `skill-parser` / `skill-validator` / `skill-archiver`
  / `skill-cli` form the skill publish pipeline; `mcp-server` and `toasty-app` are
  reference crates backing the `mcp-knowledge` and `rust-toasty` skills.
- `.claude-plugin/marketplace.json` — marketplace manifest listing every plugin.
- `terraform/github/` — GitHub repo configuration (rulesets, labels).
- `submodules/` — upstream repos as git submodules (reference only; not built).
- `.agents/skills/` — third-party skills, reference only.

**Two publish channels, one source.** Every skill ships both as a Claude Code
plugin (via `marketplace.json`) and as an `agent-skills-<name>-v<version>` ZIP (via
`skill-cli`, which scans both `skills/*` and `plugins/*/skills/*`). A skill's
`name` becomes its release tag, so names must be unique across both channels.

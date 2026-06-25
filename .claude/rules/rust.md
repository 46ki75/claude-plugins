---
paths:
  - "crates/**/*.rs"
  - "crates/**/Cargo.toml"
---

# Rust workspace (`crates/`)

- Edition 2024, MSRV 1.85, resolver 3. Package metadata and dependency versions are
  inherited from the root `[workspace]`: add a dependency to
  `[workspace.dependencies]`, then reference it from a crate with
  `<dep> = { workspace = true }`; package fields use `<field>.workspace = true` and
  lints use `[lints] workspace = true`.
- `missing_docs = "warn"` is set workspace-wide, so every public item needs a doc
  comment. Crates surface their `README.md` as crate-level docs via
  `#![doc = include_str!("../README.md")]`.
- CI gates (`.github/workflows/rust-test.yml`): `cargo fmt --all -- --check`,
  `cargo clippy --workspace --all-targets`, `cargo test --workspace --all-targets`,
  and `cargo test --workspace --doc`.
- Each crate has its own `Justfile` with crate-specific recipes (e.g. `just run-http`
  / `just inspect` in `mcp-server`, `just test-postgresql` in `toasty-app`).

Crate roles: `skill-cli` is the CLI entry to the publish pipeline (`-- check`
validates; archiving produces the ZIPs), wrapping `skill-parser` / `skill-validator`
/ `skill-archiver`. `mcp-server` and `toasty-app` are reference/demo crates.

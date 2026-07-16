# Rolldown repository facts for reviewers

Last verified: 2026-07-17 against `rolldown/rolldown` main @ `2fd5c5c4d`. Commands come from the repository's `justfile` and `docs/development-guide/`; where the guides and the justfile disagree, the justfile wins. If a command fails, re-verify against the current checkout instead of assuming this file is right.

## Contents

- Toolchain and setup
- Build
- Tests — run focused before broad
- Snapshots and generated files
- Lint
- CI map, noise, and bots
- Review-relevant policies
- Layout map

## Toolchain and setup

- Rust `1.97.0` (`rust-toolchain.toml`), Node `24.12.0` (`.node-version`), pnpm `11.9.0` (`package.json` packageManager), plus `just` and `cmake`. The justfile drives tasks through vite-plus (`vp`).
- First-time bootstrap: `just setup` (installs `vp`, dependencies, cargo tools, git submodules). Submodules only: `just setup-submodule`.

## Build

- Debug build of the node binding + package: `just build-rolldown` (alias `just build`). Release: `just build-rolldown-release`.
- `just roll` = build + lint + full tests. It is the pre-push umbrella and expensive; during review prefer the focused commands below.

## Tests — run focused before broad

- One Rust fixture: `just t-run <name>`. Data-driven cases live in `crates/rolldown/tests`; a case is a folder with `_config.json`.
- One JS test file: `just test-node-rolldown <file-substring>`; by test name: `just test-node-rolldown -t <name>` (fixture test names are their folder paths, e.g. `-t resolve/alias`; Vitest filter rules apply).
- One Rollup-compat test: `just test-node-rollup --grep "<name>"` (Mocha; proxies the `rollup/` submodule suite).
- Rust suite: `just test-rust` (= `cargo test --workspace --exclude rolldown_binding`). See the snapshot warning below before running it.
- Node suite: `just test-node` (builds first; `just t-node` skips the rebuild). The Vite suite is separate: `just test-vite`. Dev-server/HMR: `just test-node-hmr`.

## Snapshots and generated files

- Rust snapshots use insta with `update: 'always'` (`.insta.yaml`): running `just test-rust` rewrites `*.snap` files in place, with no `.snap.new` step. Run it only in your isolated review worktree, then read `git status` there: a clean tree after tests is itself evidence the PR's committed snapshots match its behavior, and an unexpected snapshot delta is a finding.
- Node snapshot update: `just test-node-rolldown --update`. Update everything: `just test-update`.
- Files reviewers must treat as generated — regenerate to verify, never accept hand-edits (CI fails on a dirty regeneration diff):
  - `packages/rolldown/src/binding.js` and `binding.d.ts` — from `just build-rolldown`
  - `crates/rolldown_common/src/generated/*`, `crates/rolldown_plugin/src/generated/hook_usage.rs`, `crates/rolldown_error/src/generated/event_kind_switcher.rs`, `packages/rolldown/src/options/generated/checks-options.ts`, `packages/rolldown/src/plugin/generated/hook-usage.ts` — from `just update-generated-code`
  - `packages/debug/src/generated/*` — from `just build-rolldown-debug`
  - `tasks/track_memory_allocations/allocs.snap` — from `just allocs`
- `rollup/`, `test262/`, and `vite/` are git submodules, not owned code; review only the submodule pointer change, not their contents.

## Lint

- Everything: `just lint`. Rust = clippy with deny-warnings + `cargo fmt --check` + `cargo check`; Node = `vp check` + knip + publint; repo = typos + ls-lint filename check + `vp fmt`.

## CI map, noise, and bots

- Required merge gates (GitHub branch rules API, checked 2026-07-17): Rust Validation, Cargo Test (ubuntu), Node Test ubuntu (Node 20/22/24), Node Dev Server Test ubuntu (Node 20/22/24), Node Validation, Repo Validation, and Vite Test Windows — all from `ci.yml` — plus Docs Validation from `deploy-website.yml`. Other `ci.yml` jobs (macOS/Windows tests, wasi, vite-test-ubuntu, build-rolldown-*, typos) run on PRs but are not required gates. PR titles are checked by `lint-pr-title.yml`.
- Noise guidance (maintainer practice, not documented in-repo): Vite-side failures (the vite-test jobs and Vite ecosystem-CI) are frequently flaky; do not treat Vite-only red as a review finding unless the PR touches Vite integration. Vite Test Windows is nevertheless a required merge gate, so a red run must be rerun or otherwise resolved before merge even when it is noise. macOS/Windows jobs are higher-noise than the core Ubuntu Rust/Node tests.
- Vite ecosystem-CI is advisory and manually triggered (a `/ecosystem-ci run` comment by someone with triage permission).
- Bots that mutate PRs: autofix.ci pushes formatting and cargo-shear fixes onto PR branches, so the head you reviewed can change without author action. Renovate opens dependency PRs (no automerge). The Graphite merge queue merges PRs labeled `graphite: merge-when-ready`.

## Review-relevant policies

- `rolldown_*` Rust crates do not follow semver (`AGENTS.md`, `docs/apis/rust-crates.md`): a breaking Rust API change whose in-repo callers are updated is not a finding.
- `crates/rolldown_binding` is the NAPI Rust-to-TS bridge and is exempt from the Rust API guidelines applied elsewhere (`.github/instructions/rust-api-guidelines.instructions.md`).
- Coding style follows `docs/development-guide/coding-style.md`; a documented rule tooling does not enforce: a file's name must match the main struct/trait/enum/function it defines.
- Many areas keep `internal-docs/<feature>/design.md` and `implementation.md`; read them as contract sources, and expect a PR changing such an area to keep them in sync.
- PR titles must match Conventional Commits: `^(build|chore|ci|docs|feat|fix|perf|refactor|release|revert|style|test)(\([a-z0-9/_-]+\))?!?: .+` (`lint-pr-title.yml`); the changelog is generated from them (`cliff.toml`).
- Contribution policy: PRs should include tests that fail without the change (`.github/PULL_REQUEST_TEMPLATE.md`); new features, public API, or default-behavior changes need prior discussion, AI usage must be disclosed, and low-effort AI PRs are closed (`docs/contribution-guide/`). Report a missing disclosure on an evidently AI-generated PR to the maintainer in the user-facing report rather than as a GitHub finding.
- CODEOWNERS (`.github/CODEOWNERS`): core code `@hyfdev` `@IWANABETHATGUY` `@shulaoda`; dev/HMR paths `@hyfdev` plus a second owner per area; Vite plugins `@shulaoda` `@sapphi-red`. There is deliberately no default owner, so some paths have none; check the file for the exact path rules. One listed owner's approval satisfies the requirement.

## Layout map

- `crates/rolldown` — bundler core; integration fixtures in `crates/rolldown/tests` (esbuild-derived cases under `tests/esbuild`).
- `crates/rolldown_binding` — NAPI bridge. `crates/rolldown_plugin*` — plugin system and built-in plugins, including the Vite ones. `rolldown_ecmascript*`, `rolldown_resolver`, `rolldown_sourcemap`, `rolldown_watcher`, `rolldown_dev*` — subsystem crates.
- `packages/rolldown` — published TS API (its Vitest suite lives in `packages/rolldown/tests`, package name `rolldown-tests`). `packages/rollup-tests` — Rollup compatibility. `packages/test-dev-server` — dev/HMR harness. `packages/vite-tests` — Vite suite.
- `docs/` — website, including `contribution-guide/` and `development-guide/`. `internal-docs/` — per-feature design and implementation notes.

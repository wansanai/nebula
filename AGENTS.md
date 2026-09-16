# Repository Guidelines

## Project Structure

- `crates/` contains the Rust workspace: shared primitives, vendor SDKs, `provider-*` adapters, and `app-core` logic.
- `app/` contains the Tauri app (`src/` React frontend, `src-tauri/` shell) and is excluded from the root Cargo workspace.
- `docs/` contains the Vue 3 website, release notes, blogs, and SDK specifications.
- `scripts/` has release and version-bump helpers.
- `PLAN.md` records product scope, architecture decisions, crate boundaries, and the phased roadmap.

Dependency flow is one-way: SDKs -> provider adapters -> `app-core` -> the Tauri shell. Do not introduce reverse dependencies.

## Build, Test, and Development

Prerequisites: Rust stable, Node 20, and pnpm 9.

```bash
cargo fmt --all
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features

cd app
pnpm install --frozen-lockfile
pnpm tauri dev          # run the desktop app with hot reload
pnpm build              # TypeScript check and Vite production build

cd ../docs
pnpm install --frozen-lockfile
pnpm dev                # local website
pnpm build              # static production build
```

App packaging uses `pnpm tauri build` from `app/`.

## Style and Naming

- Format Rust with rustfmt; keep crates and modules in `snake_case`, types in `UpperCamelCase`, and functions/fields in `snake_case`.
- Use TypeScript-friendly `PascalCase` component filenames, `camelCase` functions and variables, and `kebab-case` Vite/website assets where applicable.
- Prefer typed frontend APIs through `app/src/api.ts` rather than calling Tauri commands ad hoc.
- Keep shared dependency versions in `Cargo.toml` workspace settings.

## Testing Guidelines

Rust unit and async tests use the built-in framework with `#[test]` and `#[tokio::test]`; keep focused tests in the affected module under `#[cfg(test)]` or adjacent test modules. Name tests for the observable behavior. Root Cargo tests cover workspace crates, not the excluded Tauri shell, so run the relevant app build/type check for frontend changes.

## Commits and Pull Requests

Follow Conventional Commits, as in `feat(vendors): add UPYUN as a new provider`, `fix(accounts): ...`, or `docs(release): ...`. Use a concise imperative subject and include behavioral rationale when non-obvious.

Pull requests should describe the change, link related issues, and state completed verification commands. UI changes benefit from screenshots or short recordings; provider changes should identify the exact SDK/API path and any compatibility limits.

## Security and Configuration

Never commit credentials, endpoints with private data, tokens, or local database files. Account secrets belong in the system keyring; local state remains in platform-specific app data. Add provider capabilities only where the vendor genuinely supports them.

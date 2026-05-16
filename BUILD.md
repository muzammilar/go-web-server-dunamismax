# BUILD.md

Active migration and build plan for `go-web-server`.

The repository is currently a Go/Echo/PostgreSQL starter with an
Astro/Vue frontend embedded into the Go binary. This file defines the
path to turn the starter into Stephen's Rust/Axum/Leptos reference web
service, or to deliberately archive it as a Go historical reference.

Last reviewed: 2026-05-16.

---

## Current Baseline

- Backend: Go, Echo, sessions, CSRF, API contracts, PostgreSQL, SQLC,
  Atlas migrations, Mage.
- Frontend: Astro + Vue + Bun under `web/`, embedded from `web/dist`.
- Existing product features: registration, login, logout, profile, user
  CRUD, JSON APIs, security middleware, smoke tests.
- Deployment docs and Docker/runtime smoke scripts exist.

---

## Decision

Default path: migrate this repo into a Rust-first web starter.

If the old Go implementation remains useful as a portfolio comparison,
tag it or move it to an archival branch before removing it from `main`.
Do not maintain two starter stacks in one active branch.

---

## Target Stack

- **Rust** primary.
- **Axum** for HTTP routes, middleware, JSON APIs, forms, sessions,
  health, and static assets.
- **Leptos** for the web UI, replacing Astro/Vue.
- **PostgreSQL** for durable application data.
- **sqlx** or equivalent explicit SQL workflow for typed queries and
  compile-time/schema-checked access.
- **Tokio**, **tower**, **tracing**, **serde**, **validator** or typed
  domain validation.
- **Docker Compose** only for local Postgres and optional integration
  smoke, not as hidden production magic.
- **Python** only for migration or fixture scripts, using `uv`, local
  venvs, `pyproject.toml`, Ruff, and pytest.

---

## Target Features

- Session-based auth with secure cookies.
- CSRF protection for state-changing browser actions.
- JSON contracts for API clients.
- User CRUD and profile pages as starter examples.
- PostgreSQL migrations, test fixtures, and backup/restore docs.
- Security headers, request IDs, structured errors, rate limits, and
  tracing.
- Browser smoke tests for registration, login, CRUD, and logout.

---

## Target Source Layout

```text
Cargo.toml
crates/
  starter-core/
  starter-db/
  starter-web/      # Axum
  starter-ui/       # Leptos
  xtask/
migrations/
deploy/
docs/
tests/
```

---

## Phased Plan

### Phase 1 - Archive Or Bridge The Go Starter

- [ ] Decide whether to keep a `go-legacy` branch/tag before rewriting.
- [ ] Record the current route/API surface in `docs/`.
- [ ] Preserve useful security notes and smoke-test expectations.
- [ ] Remove generated `web/dist` and Go-specific build artifacts only
      when the Rust app has parity.

### Phase 2 - Rust Foundation

- [ ] Add the Rust workspace and Axum health route.
- [ ] Add PostgreSQL config, migration runner, and connection pool.
- [ ] Add session, CSRF, security header, request ID, and error
      middleware.
- [ ] Add CI for fmt, clippy, tests, sql checks, and release build.

### Phase 3 - Leptos UI Parity

- [ ] Port home, login, register, logout, profile, and users screens to
      Leptos.
- [ ] Keep API contracts explicit and same-origin.
- [ ] Add browser smoke tests for the same flows the current repo
      validates.
- [ ] Document local dev and production deploy flow.

### Phase 4 - Starter Polish

- [ ] Trim product-specific leftovers from the old Go version.
- [ ] Add template guidance for new Axum routes, Leptos pages, and
      migrations.
- [ ] Add a release/build example that produces one deployable Rust
      binary.

---

## Verification

Docs-only work:

```sh
git diff --check
```

Current Go baseline while it exists:

```sh
go build ./...
go vet ./...
go test ./...
mage frontendCheck
mage frontendBuild
```

Rust target gate:

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
cargo build --workspace
```

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
cargo build          # compile
cargo run            # run the server (binds to 127.0.0.1:8080)
cargo tests           # run all tests
cargo fmt            # format code
cargo fmt --check    # check formatting (used in CI)
cargo clippy -- -D warnings   # lint (warnings are errors in CI)
cargo tarpaulin --verbose --workspace  # code coverage
cargo llvm-cov --all-features --workspace --lcov --output-path lcov.info  # code coverage
cargo audit          # security audit of dependencies
```

### Database

```bash
./scripts/init_db.sh   # spin up Postgres via Docker and run migrations
sqlx migrate run       # run pending migrations manually
```

## Architecture

This is an `actix-web` HTTP server built following the *Zero To Production In Rust* book. The entry point is `src/main.rs`, which wires the actix `HttpServer` and registers routes directly in `main`. The async runtime is `tokio` (multi-thread flavor via `#[tokio::main]`).
This is an `actix-web` HTTP server built following the *Zero To Production In Rust* book. The entry point is `src/main.rs`, which calls `startup::run` to build the `HttpServer`. The async runtime is `tokio` (multi-thread flavor via `#[tokio::main]`).

### Structure

- `src/main.rs` — entry point, reads configuration and starts the server
- `src/startup.rs` — builds the `HttpServer` and wires routes
- `src/configuration.rs` — typed config structs loaded from `configuration.yaml`
- `src/routes/` — one file per route handler (`health_check.rs`, `subscriptions.rs`)
- `migrations/` — sqlx migration files
- `scripts/init_db.sh` — provisions a local Postgres container and runs migrations

### Database

PostgreSQL via `sqlx`. Subscriber data is persisted in the `subscriptions` table. SQLx offline mode is enabled (`.cargo/config.toml` + `.sqlx/` query metadata) so the project compiles in CI without a live database for `clippy`.

## CI

Four GitHub Actions jobs run on every push/PR (`general.yml`): `test`, `fmt`, `clippy`, `coverage`. A separate `audit.yml` workflow runs `cargo audit` weekly and on changes to `Cargo.toml`/`Cargo.lock`.

- `test` and `coverage` spin up a Postgres service and run `init_db.sh` before running
- `clippy` runs with `SQLX_OFFLINE=true` — no database needed
- Coverage uses `cargo-llvm-cov` (switched from `cargo-tarpaulin`)

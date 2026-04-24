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
cargo audit          # security audit of dependencies
```

## Architecture

This is an `actix-web` HTTP server built following the *Zero To Production In Rust* book. The entry point is `src/main.rs`, which wires the actix `HttpServer` and registers routes directly in `main`. The async runtime is `tokio` (multi-thread flavor via `#[tokio::main]`).

## CI

Four GitHub Actions jobs run on every push/PR (`general.yml`): `test`, `fmt`, `clippy`, `coverage`. A separate `audit.yml` workflow runs `cargo audit` weekly and on changes to `Cargo.toml`/`Cargo.lock`.

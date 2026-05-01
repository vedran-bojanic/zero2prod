# zero2prod

A newsletter subscription backend built in Rust, following the [Zero To Production In Rust](https://www.zero2prod.com/) book.

## Tech Stack

- **[actix-web](https://actix.rs/)** — HTTP server
- **[sqlx](https://github.com/launchbadge/sqlx)** — async PostgreSQL driver with compile-time query verification
- **[tokio](https://tokio.rs/)** — async runtime
- **[serde](https://serde.rs/)** — serialization / deserialization
- **[config](https://github.com/mehcode/config-rs)** — layered configuration

## Prerequisites

- Rust (stable)
- Docker (for local Postgres)
- `sqlx-cli` — `cargo install sqlx-cli --features rustls,postgres --no-default-features`

## Getting Started

**1. Start the database**

```bash
./scripts/init_db.sh
```

This spins up a PostgreSQL container via Docker and runs all migrations.

**2. Build and run**

```bash
cargo build
cargo run
```

The server listens on `127.0.0.1:8000` by default.

## Configuration

Configuration is loaded from `configuration.yaml`:

```yaml
application_port: 8000
database:
  host: "127.0.0.1"
  port: 5432
  username: "postgres"
  password: "postgres"
  db_name: "newsletter"
```

## API

### Health Check

```
GET /health_check
```

Returns `200 OK` when the server is running.

### Subscribe

```
POST /subscriptions
Content-Type: application/x-www-form-urlencoded

name=John+Doe&email=john%40example.com
```

Saves a new subscriber. Returns `200 OK` on success, `400 Bad Request` if name or email is missing.

## Development

```bash
cargo build                            # compile
cargo run                              # run the server
cargo test                             # run all tests
cargo fmt                              # format code
cargo fmt --check                      # check formatting
cargo clippy -- -D warnings            # lint
cargo llvm-cov --all-features --workspace  # code coverage
cargo audit                            # security audit
```

## Database Migrations

Migrations live in `migrations/`. To add a new one:

```bash
sqlx migrate add <migration_name>
```

Then run:

```bash
sqlx migrate run
```

Or re-run `./scripts/init_db.sh` (it is idempotent).

## CI

GitHub Actions runs four jobs on every push and pull request:

| Job | Description |
|-----|-------------|
| `test` | Spins up Postgres, runs migrations, runs `cargo test` |
| `fmt` | Checks formatting with `cargo fmt --check` |
| `clippy` | Lints with `cargo clippy -- -D warnings` (SQLx offline mode) |
| `coverage` | Generates coverage report with `cargo-llvm-cov` |

A separate `audit.yml` workflow runs `cargo audit` weekly and on dependency changes.

## Project Structure

```
src/
├── main.rs           # entry point
├── lib.rs            # library root
├── startup.rs        # HttpServer setup and route registration
├── configuration.rs  # typed config structs
└── routes/
    ├── mod.rs
    ├── health_check.rs
    └── subscriptions.rs
migrations/           # sqlx migration files
scripts/
└── init_db.sh        # local Postgres setup
tests/
└── health_check.rs   # integration tests
```

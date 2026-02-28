# Rust / Cargo Cheatsheet

> Quick-reference for building, testing, and managing Rust projects with Cargo.

## 🔥 Most Used

| Command | What it does | Example |
|---|---|---|
| `cargo new <name>` | Create new project | `cargo new myapp` |
| `cargo build` | Build (debug) | `cargo build` |
| `cargo build --release` | Build (optimized) | `cargo build --release` |
| `cargo run` | Build + run | `cargo run -- --port 8080` |
| `cargo test` | Run all tests | `cargo test -- --nocapture` |
| `cargo clippy` | Lint code | `cargo clippy -- -D warnings` |
| `cargo fmt` | Format code | `cargo fmt --check` |
| `cargo add <crate>` | Add dependency | `cargo add serde -F derive` |
| `cargo doc --open` | Generate + open docs | `cargo doc --open` |

## Project Setup

| Command | What it does | Example |
|---|---|---|
| `cargo new <name>` | New binary project | `cargo new myapp` |
| `cargo new --lib <name>` | New library project | `cargo new --lib mylib` |
| `cargo init` | Init in current dir | `cargo init` |
| `cargo init --lib` | Init lib in current dir | `cargo init --lib` |

### Workspace Setup (Cargo.toml)

```toml
[workspace]
members = ["crates/*"]
resolver = "2"

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
```

```toml
# In crates/mylib/Cargo.toml
[dependencies]
serde = { workspace = true }
```

## Build

| Command | What it does | Example |
|---|---|---|
| `cargo build` | Debug build | `cargo build` |
| `cargo build --release` | Optimized release build | `cargo build --release` |
| `cargo build -p <crate>` | Build specific package | `cargo build -p mylib` |
| `cargo build --target <triple>` | Cross-compile | `cargo build --target x86_64-unknown-linux-musl` |
| `cargo build --features <f>` | Enable features | `cargo build --features "tls,json"` |
| `cargo build --all-features` | Enable all features | `cargo build --all-features` |
| `cargo build --no-default-features` | Disable defaults | `cargo build --no-default-features` |
| `cargo check` | Type-check (no codegen, fast) | `cargo check` |
| `cargo clean` | Remove target/ | `cargo clean` |

### Debug vs Release

| Aspect | `cargo build` (debug) | `cargo build --release` |
|---|---|---|
| Optimizations | None (`opt-level = 0`) | Full (`opt-level = 3`) |
| Debug info | Included | Stripped |
| Compile speed | Fast | Slow |
| Output path | `target/debug/` | `target/release/` |
| Use for | Development, testing | Benchmarks, deployment |

## Test

| Command | What it does | Example |
|---|---|---|
| `cargo test` | Run all tests | `cargo test` |
| `cargo test <name>` | Run tests matching name | `cargo test auth` |
| `cargo test -p <crate>` | Test specific package | `cargo test -p mylib` |
| `cargo test -- --nocapture` | Show println! output | `cargo test -- --nocapture` |
| `cargo test -- --ignored` | Run ignored tests | `cargo test -- --ignored` |
| `cargo test -- --test-threads=1` | Run tests serially | `cargo test -- --test-threads=1` |
| `cargo test --doc` | Run doc tests only | `cargo test --doc` |
| `cargo test --lib` | Run unit tests only | `cargo test --lib` |
| `cargo test --integration` | Run integration tests only | `cargo test --test '*'` |
| `cargo bench` | Run benchmarks | `cargo bench` |

## Dependencies

| Command | What it does | Example |
|---|---|---|
| `cargo add <crate>` | Add dependency | `cargo add reqwest` |
| `cargo add <crate> -F <feat>` | Add with features | `cargo add serde -F derive` |
| `cargo add <crate> --dev` | Add dev dependency | `cargo add tokio-test --dev` |
| `cargo add <crate> --build` | Add build dependency | `cargo add prost-build --build` |
| `cargo remove <crate>` | Remove dependency | `cargo remove reqwest` |
| `cargo update` | Update all deps (within semver) | `cargo update` |
| `cargo update -p <crate>` | Update specific dep | `cargo update -p serde` |
| `cargo tree` | Dependency tree | `cargo tree` |
| `cargo tree -d` | Show duplicated deps | `cargo tree -d` |
| `cargo tree -i <crate>` | Why is this dep included? | `cargo tree -i openssl` |

## Features

```toml
# Cargo.toml
[features]
default = ["json"]
json = ["dep:serde_json"]
tls = ["dep:rustls"]
full = ["json", "tls"]
```

| Command | What it does |
|---|---|
| `cargo build --features "tls"` | Enable specific features |
| `cargo build --all-features` | Enable everything |
| `cargo build --no-default-features` | Disable defaults |
| `cargo build --no-default-features --features tls` | Only specific features |

## Cross-compilation

```bash
# 1. Add target
rustup target add x86_64-unknown-linux-musl

# 2. Build for target
cargo build --release --target x86_64-unknown-linux-musl

# Output at: target/x86_64-unknown-linux-musl/release/myapp
```

| Common Target | Platform |
|---|---|
| `x86_64-unknown-linux-gnu` | Linux (glibc) |
| `x86_64-unknown-linux-musl` | Linux (static, musl) |
| `aarch64-unknown-linux-gnu` | Linux ARM64 |
| `x86_64-apple-darwin` | macOS Intel |
| `aarch64-apple-darwin` | macOS Apple Silicon |
| `x86_64-pc-windows-msvc` | Windows |
| `wasm32-unknown-unknown` | WebAssembly |

## Useful Tools

| Command | What it does | Example |
|---|---|---|
| `cargo clippy` | Lint for common mistakes | `cargo clippy -- -D warnings` |
| `cargo clippy --fix` | Auto-fix lint warnings | `cargo clippy --fix --allow-dirty` |
| `cargo fmt` | Format code (rustfmt) | `cargo fmt` |
| `cargo fmt --check` | Check formatting (CI) | `cargo fmt --check` |
| `cargo audit` | Check for security vulns | `cargo audit` (install: `cargo install cargo-audit`) |
| `cargo outdated` | Show outdated deps | `cargo outdated` (install: `cargo install cargo-outdated`) |
| `cargo deny check` | License + advisory checks | `cargo deny check` (install: `cargo install cargo-deny`) |
| `cargo expand` | Expand macros | `cargo expand` (install: `cargo install cargo-expand`) |
| `cargo watch -x test` | Re-run on file changes | `cargo watch -x 'test -- --nocapture'` |
| `cargo bloat` | Find what's bloating binary | `cargo bloat --release` |

## Workspace Commands

| Command | What it does | Example |
|---|---|---|
| `cargo build --workspace` | Build all workspace crates | `cargo build --workspace` |
| `cargo test --workspace` | Test all workspace crates | `cargo test --workspace` |
| `cargo clippy --workspace` | Lint all workspace crates | `cargo clippy --workspace` |
| `cargo build -p <crate>` | Build specific crate | `cargo build -p my-api` |
| `cargo test -p <crate>` | Test specific crate | `cargo test -p my-lib` |
| `cargo run -p <crate>` | Run specific binary crate | `cargo run -p my-server` |

---

*See [Rust Operator Tutorial](../../tutorials/rust-k8s-operator/README.md) for a full Rust project walkthrough.*

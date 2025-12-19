# Canvas CLI - Development Guide

## Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| Rust | Latest stable | Install via rustup |
| Cargo | Latest stable | Comes with Rust |
| Git | Any recent | For version control |

### Installing Rust

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
```

## Getting Started

### 1. Clone the Repository

```sh
git clone https://github.com/danielhuang/canvas-cli
cd canvas-cli
```

### 2. Build the Project

```sh
# Debug build (faster compilation)
cargo build

# Release build (optimized)
cargo build --release
```

### 3. Configure the Application

Create `~/.canvas.toml`:

```toml
# Required: Your Canvas instance URL
canvas_url = "https://canvas.example.com"

# Required: Your Canvas API token
# Get this from Canvas: Account > Settings > New Access Token
token = "your_api_token_here"

# Optional: Gradescope session cookie for Gradescope integration
# gradescope_cookie = "your_session_cookie"

# Optional: Display settings
# hide_overdue_after_days = 7
# hide_overdue_without_submission = true
# hide_locked = true
```

### 4. Run the Application

```sh
# Run in development mode
cargo run -- todo

# Run with release optimizations
cargo run --release -- todo

# Or use the built binary directly
./target/debug/canvas todo
./target/release/canvas todo
```

## Development Workflow

### Code Formatting

```sh
# Check formatting
cargo fmt -- --check

# Auto-format code
cargo fmt
```

### Linting

```sh
# Run clippy linter
cargo clippy

# Fix clippy warnings automatically (where possible)
cargo clippy --fix
```

### Building

```sh
# Debug build
cargo build

# Release build
cargo build --release

# Clean build artifacts
cargo clean
```

## Project Structure

```
src/
├── main.rs          # Entry point and CLI logic
├── canvas_api.rs    # Canvas API data structures
├── gradescope.rs    # Gradescope scraping
├── config.rs        # Configuration handling
└── progress.rs      # Progress bar utility
```

### Key Modules

| Module | Responsibility |
|--------|----------------|
| `main` | CLI parsing, command handlers, output formatting |
| `canvas_api` | Serde-compatible Canvas API response types |
| `gradescope` | HTML scraping logic for Gradescope |
| `config` | TOML configuration parsing and types |
| `progress` | Async-aware progress bar wrapper |

## Debugging

### VS Code Setup

The project includes a VS Code launch configuration for LLDB debugging:

1. Install the "CodeLLDB" extension
2. Open the project in VS Code
3. Press F5 to start debugging (runs `canvas todo`)

### Manual Debugging

```sh
# Build with debug symbols
cargo build

# Run with RUST_BACKTRACE for full stack traces
RUST_BACKTRACE=1 cargo run -- todo
```

## Common Development Tasks

### Adding a New CLI Command

1. Add command variant to `Opt` enum in `main.rs`:
   ```rust
   #[structopt(about = "Description of command")]
   NewCommand { arg: String },
   ```

2. Add match arm in `main()`:
   ```rust
   Opt::NewCommand { arg } => {
       run_new_command(&arg).await?;
   }
   ```

3. Implement the handler function

### Adding a New Configuration Option

1. Add field to `Config` struct in `config.rs`:
   ```rust
   #[serde(default)]
   pub new_option: Option<String>,
   ```

2. Use the option in your code:
   ```rust
   if let Some(value) = config.new_option {
       // ...
   }
   ```

### Modifying Canvas API Types

The types in `canvas_api.rs` mirror the Canvas API responses. To add new fields:

1. Check the [Canvas API documentation](https://canvas.instructure.com/doc/api/)
2. Add the field with appropriate type and serde attributes
3. Use `Option<T>` for optional fields
4. Use `serde_json::Value` for unknown/dynamic fields

## CI/CD

The project uses Cirrus CI for continuous integration:

| Job | Command | Purpose |
|-----|---------|---------|
| Build | `cargo build --release` | Verify compilation |
| Format | `cargo fmt -- --check` | Check code formatting |
| Clippy | `cargo clippy` | Static analysis |

## Dependency Management

### Updating Dependencies

```sh
# Update Cargo.lock to latest compatible versions
cargo update

# Check for outdated dependencies
cargo outdated  # requires: cargo install cargo-outdated
```

### Renovate Bot

The project uses Renovate for automated dependency updates. See `renovate.json` for configuration.

## Troubleshooting

### Build Errors

**Problem:** OpenSSL-related build errors
**Solution:** The project uses `rustls-tls` feature for reqwest, which doesn't require OpenSSL. Ensure you're using the correct feature flags.

**Problem:** Async trait errors
**Solution:** Ensure `async-trait` crate is at a compatible version.

### Runtime Errors

**Problem:** "Unable to read configuration file"
**Solution:** Ensure `~/.canvas.toml` exists and contains valid TOML.

**Problem:** "Server returned error"
**Solution:** Check your API token and Canvas URL. Verify the token has appropriate permissions.

**Problem:** Gradescope not working
**Solution:** The Gradescope cookie expires. Get a fresh session cookie from your browser.

## Testing

Currently, the project has no automated tests. Consider adding:

- Unit tests for configuration parsing
- Unit tests for date formatting functions
- Integration tests with mock HTTP responses

```sh
# Run tests (when implemented)
cargo test

# Run tests with output
cargo test -- --nocapture
```

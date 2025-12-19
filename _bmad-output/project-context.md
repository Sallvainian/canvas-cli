---
project_name: 'canvas-cli'
user_name: 'Sallvain'
date: '2025-12-19'
sections_completed: ['technology_stack', 'rust_rules', 'architecture_patterns', 'testing_rules', 'code_quality', 'dev_workflow', 'critical_rules']
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

| Category | Crate | Version | Critical Notes |
|----------|-------|---------|----------------|
| Language | Rust | Edition 2018 | - |
| Runtime | tokio | 1.37.0 | MUST use `features = ["full"]` |
| HTTP | reqwest | 0.12.4 | MUST use `rustls-tls`, NOT native-tls |
| Serialization | serde | 1.0.200 | MUST use `features = ["derive"]` |
| CLI | structopt | 0.3.26 | Deprecated - do not add new subcommands without migration plan |
| Errors | color-eyre | 0.6.3 | ALL errors must use this, not anyhow |
| Dates | chrono | 0.4.38 | MUST use `features = ["serde"]` |
| Scraping | scraper | 0.25.0 | Only for Gradescope HTML parsing |

---

## Rust-Specific Rules

### Error Handling
- MUST use `color_eyre::Result<T>` as return type, never `std::result::Result`
- MUST use `.wrap_err_with(|| eyre!("context"))` for all fallible operations
- MUST add `.suggestion("user guidance")` for user-facing errors (auth, config)
- NEVER use `.unwrap()` on network/IO operations

### Async Patterns
- MUST use `tokio::try_join!` for concurrent independent operations
- MUST use `futures::try_join_all` for concurrent collection operations
- NEVER spawn tasks - use structured concurrency with try_join

### HTTP Client
- MUST use the global `lazy_static` CLIENT - never instantiate new clients
- MUST set Authorization header for Canvas API: `Bearer {token}`
- MUST set Cookie header for Gradescope: session cookie from config

### JSON Parsing
- MUST use `serde_path_to_error::deserialize` for API responses
- This provides detailed error paths when Canvas API changes schema

---

## Architecture & Framework Patterns

### Data Abstraction
- `impl Assignment` MUST stay thin: delegation + shared normalization only
- Source-specific API/transform logic MUST live in `canvas_api::*` / `gradescope::*` modules
- NEVER add source-specific parsing or API calls to the Assignment impl

### Adding a New Data Source
A new source is complete when it provides ALL of:
1. `SourceCourse` and `SourceAssignment` structs (in `src/{source}.rs`)
2. Conversion into `Assignment::{Source(SourceCourse, SourceAssignment)}`
3. Mapping for `due_at()` and `assignment_id()` in Assignment impl
4. Inclusion/exclusion matching in `should_show()` and filter logic
5. Loader function pattern: `async fn load_{source}(progress, config) -> Result<Vec<(SourceCourse, Vec<SourceAssignment>)>>`

### Configuration
- Config file: `~/.canvas.toml` (default)
- NEVER use `home_dir().unwrap()` - this is a footgun
- MUST return user-facing error: `.wrap_err("Could not find home directory").suggestion("Set $HOME or use --config-path")`
- Future: support `--config-path` CLI override or `CANVAS_CONFIG` env var

### Progress Tracking Lifecycle
- `Progress::new()` created ONCE per command execution
- EVERY network fetch MUST be inside `progress.wrap("message", future)`
- `progress.finish()` MUST run at end of successful operations
- On error paths: accept unfinished progress OR implement Drop-based cleanup

---

## Testing Rules

### Current State (As Of 2025-12-19)
- NO automated tests exist - project relies on manual testing
- CI validates: build, format, clippy only

### Quality Gates (Required for All Changes)
- `cargo fmt -- --check` MUST pass
- `cargo clippy` MUST pass with no warnings
- `cargo build --release` MUST succeed

### If Adding Tests (Future)
- Test files: `src/{module}_test.rs` or `tests/{integration}.rs`
- Use `tokio::test` for async tests
- Mock HTTP responses - never hit real Canvas/Gradescope in tests
- Test the Assignment enum methods, not source-specific internals
- Config tests should use temp files, not `~/.canvas.toml`

### Manual Testing Checklist
Before merging changes that affect:
- **Canvas API**: Verify with real account that assignments load
- **Gradescope**: Verify scraping still works (fragile - HTML can change)
- **Exclusions**: Test `canvas exclude <id>` modifies config correctly
- **Display**: Check terminal output colors and formatting

---

## Code Quality & Style Rules

### Formatting
- Use default `rustfmt` settings (no custom config)
- Run `cargo fmt` before committing
- CI will reject unformatted code

### File Organization
- `main.rs`: CLI entry, command dispatch, shared types (Assignment enum)
- `config.rs`: Config structs, TOML parsing, path resolution
- `{source}.rs`: Per-source module with types + fetch logic (e.g., `gradescope.rs`)
- `progress.rs`: Progress bar abstraction

### Naming Conventions
- Files: `snake_case.rs`
- Structs: `PascalCase` - prefix with source name for API types (`CanvasAssignment`, `GradescopeCourse`)
- Functions: `snake_case`
- Async loaders: `load_{source}()` or `load_{thing}_for_{context}()`
- Formatters: `format_{thing}()` for display helpers

### Code Organization Within Files
- Imports at top, grouped: std → external crates → crate modules
- Structs/enums before impl blocks
- Public functions before private helpers
- Keep functions under ~50 lines - extract helpers

---

## Development Workflow Rules

### CI Pipeline (Cirrus CI)
All PRs must pass:
1. `cargo build --release` - Build succeeds
2. `cargo fmt -- --check` - Code is formatted
3. `cargo clippy` - No linter warnings

### Local Development
```sh
# Build
cargo build

# Run locally
cargo run -- todo
cargo run -- exclude 12345
cargo run -- next-due

# Before committing
cargo fmt
cargo clippy
```

### Installation / Distribution
- Primary: `cargo install --git https://github.com/danielhuang/canvas-cli`
- Binary name: `canvas` (from package name in Cargo.toml)
- No crates.io publishing (git-only distribution)

### Dependency Updates
- Renovate/Dependabot may create PRs for dependency updates
- Review changelogs for breaking changes, especially:
  - `reqwest` (HTTP client)
  - `scraper` (Gradescope may break on updates)
  - `structopt` (deprecated - migration to clap v4 pending)

---

## Critical Don't-Miss Rules

### Anti-Patterns to Avoid
- NEVER create new `reqwest::Client` instances - use global `CLIENT`
- NEVER use `.unwrap()` on `home_dir()`, `Url::parse()`, or network operations
- NEVER add source-specific logic to `impl Assignment` - keep it in source modules
- NEVER hit real APIs in tests - mock all HTTP
- NEVER store secrets anywhere except `~/.canvas.toml`

### Fragile Areas (Handle With Care)
- **Gradescope scraping**: HTML selectors (`.courseBox`, `tbody > tr`) will break if Gradescope changes their markup
- **Canvas API pagination**: Uses `per_page=10000` - may need pagination handling for users with many courses
- **Date parsing**: Gradescope uses `%Y-%m-%d %H:%M:%S %z` format - will fail silently if format changes

### Security Considerations
- API token and session cookie stored in plaintext in `~/.canvas.toml`
- NEVER log or display credentials in error messages
- NEVER commit config files with real tokens
- Use `rustls-tls` (not native-tls) to avoid OpenSSL dependency issues

### Edge Cases Agents Must Handle
- `due_at` can be `None` for both Canvas and Gradescope assignments
- `assignment_id` returns `None` for Gradescope (no numeric IDs in their system)
- Exclusions work by `assignment_id` OR `class_id` - check both patterns
- `hide_overdue_after_days` is `Option<i64>` - `None` means no hiding
- `gradescope_cookie` is optional - if `None`, skip Gradescope entirely

### Performance Gotchas
- All course fetches happen concurrently via `try_join_all` - can hit rate limits on large accounts
- Progress bar uses `Mutex<Arena>` - minimal contention but be aware
- `colored` crate output may not render in non-TTY contexts (piped output)


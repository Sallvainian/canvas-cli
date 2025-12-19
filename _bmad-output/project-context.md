---
project_name: 'canvas-cli'
user_name: 'Sallvain'
date: '2025-12-19'
sections_completed: ['technology_stack', 'rust_rules']
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


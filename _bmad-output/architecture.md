# Canvas CLI - Architecture Document

## 1. Executive Summary

Canvas CLI is a Rust-based command-line application that aggregates assignment information from Canvas LMS (via REST API) and Gradescope (via web scraping). It follows a simple modular architecture optimized for async HTTP operations and terminal output.

## 2. Architecture Pattern

**Pattern:** Modular CLI with Async Service Layer

```
┌─────────────────────────────────────────────────────────────┐
│                    CLI Layer (main.rs)                       │
│  ┌─────────┐ ┌─────────┐ ┌──────────┐                       │
│  │  todo   │ │ exclude │ │ next-due │  ← structopt commands │
│  └────┬────┘ └────┬────┘ └────┬─────┘                       │
│       └───────────┴───────────┘                              │
│                    │                                         │
├────────────────────┼─────────────────────────────────────────┤
│              Service Layer                                   │
│  ┌─────────────────┴─────────────────┐                      │
│  │      load_all_assignments()       │                      │
│  │  ┌───────────┐   ┌──────────────┐ │                      │
│  │  │load_canvas│   │load_gradescope│ │ ← async aggregation │
│  │  └─────┬─────┘   └──────┬───────┘ │                      │
│  └────────┼────────────────┼─────────┘                      │
├───────────┼────────────────┼─────────────────────────────────┤
│       Data Layer           │                                 │
│  ┌────────┴────────┐ ┌─────┴──────┐                         │
│  │  canvas_api.rs  │ │gradescope.rs│ ← data structures     │
│  │  (JSON parsing) │ │(HTML scraping)│                      │
│  └─────────────────┘ └─────────────┘                         │
├─────────────────────────────────────────────────────────────┤
│                    Config Layer                              │
│  ┌─────────────┐                                            │
│  │  config.rs  │ ← ~/.canvas.toml parsing                   │
│  └─────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
```

## 3. Technology Stack

### Core Dependencies

| Category | Crate | Version | Purpose |
|----------|-------|---------|---------|
| **Runtime** | tokio | 1.37.0 | Async runtime with full features |
| **HTTP** | reqwest | 0.12.4 | HTTP client with JSON/TLS support |
| **Serialization** | serde | 1.0.200 | JSON serialization/deserialization |
| **Serialization** | serde_json | 1.0.116 | JSON parsing |
| **Date/Time** | chrono | 0.4.38 | Date handling with serde support |
| **CLI** | structopt | 0.3.26 | Command-line argument parsing |
| **Config** | toml | 0.9.0 | TOML configuration parsing |
| **Config** | toml_edit | 0.2.1 | TOML file modification |
| **Errors** | color-eyre | 0.6.3 | Rich error reporting |
| **Terminal** | colored | 3.0.0 | Colorized terminal output |
| **Progress** | indicatif | 0.15.0 | Progress bars |
| **Scraping** | scraper | 0.25.0 | HTML parsing (Gradescope) |
| **Retry** | backoff | 0.4.0 | Exponential backoff |
| **Concurrency** | futures | 0.3.30 | Async utilities |
| **Async Traits** | async-trait | 0.1.80 | Async trait support |

### Build & Development

| Tool | Purpose |
|------|---------|
| cargo | Build system and package manager |
| rustfmt | Code formatting |
| clippy | Linting |
| LLDB | Debugging (via VS Code) |

## 4. Component Architecture

### 4.1 CLI Commands (main.rs)

```rust
enum Opt {
    Todo { show_all: bool },  // List upcoming assignments
    Exclude { assignment_id: i64 },  // Add to exclusion list
    NextDue,  // Show next due date
}
```

### 4.2 Assignment Abstraction

```rust
enum Assignment {
    Canvas(CanvasCourse, CanvasAssignment),
    Gradescope(GradescopeCourse, GradescopeAssignment),
}
```

This enum provides a unified interface over both data sources.

### 4.3 Configuration Schema

```toml
# ~/.canvas.toml
canvas_url = "https://canvas.example.com"
token = "api_token"
gradescope_cookie = "optional_session_cookie"

# Display options
hide_overdue_after_days = 7
hide_overdue_without_submission = true
hide_locked = true

# Exclusions
[[exclude]]
assignment_id = 12345

[[exclude]]
class_id = 67890

# Forced inclusions
[[include]]
assignment_id = 11111
```

## 5. Data Flow

### 5.1 Todo Command Flow

```
1. Parse CLI arguments (structopt)
2. Read configuration (~/.canvas.toml)
3. Concurrently fetch from both sources:
   a. Canvas API: /api/v1/courses → /api/v1/courses/{id}/assignments
   b. Gradescope: Scrape dashboard → scrape course pages
4. Merge into unified Assignment list
5. Apply filters (exclusions, overdue rules, submission status)
6. Sort by due date (newest first)
7. Render colorized output to terminal
```

### 5.2 Canvas API Integration

- **Authentication:** Bearer token in Authorization header
- **Endpoints Used:**
  - `GET /api/v1/courses?enrollment_state=active&per_page=10000`
  - `GET /api/v1/courses/{id}/assignments?per_page=10000&include=submission`
- **Concurrency:** Uses `tokio::try_join!` and `futures::try_join_all`

### 5.3 Gradescope Integration

- **Authentication:** Session cookie in Cookie header
- **Scraping Strategy:**
  - Dashboard page (`/`) → Extract course list from `.courseBox` elements
  - Course page (`/courses/{id}`) → Extract assignments from `tbody > tr` elements
- **Data Extraction:** Uses `scraper` crate with CSS selectors

## 6. Error Handling

- **Framework:** color-eyre for rich error context
- **Strategy:** Wrap errors with context at each layer
- **User Guidance:** `.suggestion()` hints for common issues (e.g., invalid credentials)

```rust
.wrap_err_with(|| eyre!("Unable to fetch {}", url))?
.suggestion("Make sure your credentials are valid")?
```

## 7. Concurrency Model

- **Runtime:** tokio with full features
- **Pattern:** Concurrent fetching with `try_join!` and `try_join_all`
- **HTTP Client:** Global lazy_static reqwest client
- **Progress Tracking:** Thread-safe progress bar with `Mutex<Arena>`

## 8. Security Considerations

| Area | Implementation |
|------|----------------|
| API Token | Stored in user's home directory (~/.canvas.toml) |
| TLS | reqwest with rustls-tls (no native OpenSSL dependency) |
| Session Cookie | Optional Gradescope cookie in config file |

**Note:** Credentials are stored in plaintext in the config file. Users should ensure appropriate file permissions.

## 9. Testing Strategy

No automated tests are currently implemented. The project relies on:
- `cargo clippy` for static analysis
- `cargo fmt` for code formatting
- Manual testing

## 10. Deployment

### Installation

```sh
cargo install --git https://github.com/danielhuang/canvas-cli
```

### CI/CD (Cirrus CI)

```yaml
- Build: cargo build --release
- Format: cargo fmt -- --check
- Clippy: cargo clippy
```

## 11. Future Considerations

Potential areas for enhancement:
- Add unit and integration tests
- Implement caching for API responses
- Add support for more Canvas features (grades, announcements)
- Consider using clap v4 instead of structopt (deprecated)
- Add shell completions

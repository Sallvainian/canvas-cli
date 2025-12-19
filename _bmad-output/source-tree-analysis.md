# Source Tree Analysis

## Directory Structure

```
canvas-cli/
├── src/                          # Source code directory
│   ├── main.rs                   # 🚀 ENTRY POINT - CLI commands and main logic
│   ├── canvas_api.rs             # Canvas LMS API data structures (courses, assignments, submissions)
│   ├── gradescope.rs             # Gradescope web scraping and data extraction
│   ├── config.rs                 # Configuration file parsing (~/.canvas.toml)
│   └── progress.rs               # Progress bar utility for async operations
│
├── Cargo.toml                    # 📦 Package manifest - dependencies and metadata
├── Cargo.lock                    # Locked dependency versions
│
├── README.md                     # Basic installation and configuration docs
├── Dockerfile                    # CI container with Rust, clippy, rustfmt
├── .cirrus.yml                   # CI/CD configuration (build, format, clippy)
├── renovate.json                 # Automated dependency updates
│
├── .vscode/                      # VS Code configuration
│   └── launch.json               # Debug configuration for LLDB
│
├── .gitignore                    # Git ignore patterns
└── .git/                         # Git repository data
```

## Critical Files Explained

### Entry Point

| File | Purpose | Lines |
|------|---------|-------|
| `src/main.rs` | Application entry point, CLI command handling, assignment display logic | ~530 |

**Key components in main.rs:**
- `Opt` enum - CLI command definitions (Todo, Exclude, NextDue)
- `Assignment` enum - Unified type for Canvas/Gradescope assignments
- `run_todo()` - Main todo list display logic
- `run_exclude()` - Assignment exclusion handler
- `load_all_assignments()` - Aggregates assignments from both sources

### API Integration

| File | Purpose | Lines |
|------|---------|-------|
| `src/canvas_api.rs` | Canvas LMS API data structures | ~280 |
| `src/gradescope.rs` | Gradescope HTML scraping | ~120 |

**Canvas API structures:**
- `CanvasCourse` - Course information (id, name, enrollments)
- `CanvasAssignment` - Assignment details (due dates, submission types, status)
- `Submission` - Student submission data

**Gradescope structures:**
- `GradescopeCourse` - Course scraped from dashboard
- `GradescopeAssignment` - Assignment scraped from course page

### Configuration

| File | Purpose | Lines |
|------|---------|-------|
| `src/config.rs` | TOML configuration parsing | ~45 |

**Config structure:**
- `canvas_url` - Canvas instance URL
- `token` - API authentication token
- `gradescope_cookie` - Optional Gradescope session cookie
- `exclude` - List of excluded assignments/courses
- `include` - List of forced-include assignments
- Various display options (hide_overdue_after_days, hide_locked, etc.)

### Utilities

| File | Purpose | Lines |
|------|---------|-------|
| `src/progress.rs` | Async progress bar wrapper | ~55 |

## File Relationships

```
main.rs
    ├── imports → canvas_api.rs (CanvasAssignment, CanvasCourse)
    ├── imports → gradescope.rs (GradescopeAssignment, GradescopeCourse, load_*)
    ├── imports → config.rs (Config, read_config, config_path)
    └── imports → progress.rs (Progress)

config.rs
    └── standalone (no local imports)

canvas_api.rs
    └── standalone (no local imports, just data structures)

gradescope.rs
    ├── imports → config.rs (Config)
    └── uses → main.rs::CLIENT (lazy_static reqwest client)

progress.rs
    └── standalone (no local imports)
```

## Data Flow

```
User runs `canvas todo`
         │
         ▼
    main.rs::main()
         │
         ├──► config::read_config()  →  ~/.canvas.toml
         │
         ▼
    load_all_assignments()
         │
         ├──► load_canvas()
         │        └──► Canvas API (HTTP/JSON)
         │
         ├──► load_gradescope()
         │        └──► Gradescope (HTTP/HTML scraping)
         │
         ▼
    run_todo() - Filter, sort, display assignments
         │
         ▼
    Terminal output (colorized)
```

## Build Artifacts

```
target/                           # Build output (gitignored)
├── debug/                        # Debug builds
│   └── canvas                    # Debug executable
└── release/                      # Release builds
    └── canvas                    # Release executable
```

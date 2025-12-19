# Canvas CLI - Project Overview

## Executive Summary

**canvas-cli** is a Rust command-line tool for viewing and managing assignments from Canvas LMS and Gradescope. It provides students with a unified view of upcoming assignments across both platforms, with features for filtering, excluding, and tracking due dates.

## Project Information

| Attribute | Value |
|-----------|-------|
| **Project Name** | canvas |
| **Version** | 0.1.0 |
| **Author** | Dan <dan@yellowiki.xyz> |
| **Language** | Rust (Edition 2018) |
| **Repository Type** | Monolith |
| **Project Type** | CLI Application |

## Purpose & Goals

- Provide a unified command-line interface for viewing Canvas LMS assignments
- Integrate Gradescope assignment tracking
- Enable filtering and exclusion of assignments
- Display colorized, human-friendly due date information
- Support configuration via TOML file

## Key Features

### 1. Assignment Viewing (`todo` command)
- Lists all upcoming assignments from Canvas and Gradescope
- Shows due dates in human-readable format (e.g., "tomorrow at 3:00 PM")
- Color-codes assignments by course
- Indicates submission status
- Supports `--show-all` flag to include hidden/completed assignments

### 2. Assignment Exclusion (`exclude` command)
- Permanently exclude specific assignments from the todo list
- Persists exclusions to configuration file

### 3. Next Due Date (`next-due` command)
- Shows time until next assignment is due
- Quick status check for upcoming deadlines

## Architecture Overview

```
canvas-cli/
├── src/
│   ├── main.rs          # Entry point, CLI commands, display logic
│   ├── canvas_api.rs    # Canvas LMS API data structures
│   ├── gradescope.rs    # Gradescope web scraping
│   ├── config.rs        # Configuration management
│   └── progress.rs      # Progress bar utility
├── Cargo.toml           # Dependencies and metadata
└── .canvas.toml         # User configuration (in home directory)
```

## Technology Stack Summary

| Category | Technology |
|----------|------------|
| Language | Rust (Edition 2018) |
| Async Runtime | tokio 1.37 |
| HTTP | reqwest 0.12 |
| CLI Parsing | structopt 0.3 |
| Config | toml 0.9 |
| Error Handling | color-eyre 0.6 |

## Quick Start

```sh
# Install
cargo install --git https://github.com/danielhuang/canvas-cli

# Configure (~/.canvas.toml)
token = "your_canvas_api_token"
canvas_url = "https://canvas.example.com"

# Run
canvas todo
```

## Related Documentation

- [Architecture](./architecture.md) - Detailed technical architecture
- [Development Guide](./development-guide.md) - Setup and development instructions
- [Source Tree Analysis](./source-tree-analysis.md) - Annotated directory structure

# Canvas-CLI Project Structure

> Generated: 2026-01-27 | Scan Level: Exhaustive | Mode: Initial Scan

## Project Classification

| Attribute | Value |
|-----------|-------|
| **Project Name** | Canvas-CLI (canvaslms-cli) |
| **Repository Type** | Monolith |
| **Project Type** | CLI (Command-Line Interface Tool) |
| **Primary Language** | TypeScript |
| **Runtime** | Bun (Node.js compatible) |
| **Package Manager** | Bun |
| **Node Version** | >=14.0.0 |
| **License** | MIT |

## Purpose

A modern, user-friendly command-line interface for Canvas LMS. Enables users to manage courses, assignments, submissions, grades, and more directly from the terminal.

## Key Features

- List and filter enrolled/starred courses
- Star/unstar courses to manage favorites
- Interactive course selection for all commands
- View assignments, grades, and submission status
- Advanced assignment filtering (missing, due this week, all courses)
- Interactive file upload for assignments with visual file browser
- Bulk download all course files with organized folder structure
- GPA Calculator - Calculate overall GPA across all courses on 4.0 scale
- What-if Grade Calculator - Find out what you need on remaining work
- View upcoming due dates and calendar events
- Browse course modules and content
- View course announcements
- Display user profile information
- Modern table displays with adaptive column widths
- Direct access to Canvas API endpoints

## Entry Points

| Entry Point | Description |
|-------------|-------------|
| `index.ts` | Main CLI entry point (development) |
| `src/index.ts` | Distribution entry point |
| `dist/src/index.js` | Compiled binary entry |

## Binary Commands

```json
{
  "canvas": "dist/src/index.js",
  "canvaslms-cli": "dist/src/index.js"
}
```

## Directory Overview

```
Canvas-CLI/
├── commands/          # CLI command implementations (15 commands)
├── lib/               # Shared utilities and core functionality
├── types/             # TypeScript type definitions
├── tests/             # Test suite (14 test files + mocks)
├── scripts/           # Build/utility scripts
├── assets/            # Documentation images
├── .github/workflows/ # CI/CD pipelines (4 workflows)
├── index.ts           # Main entry point
└── src/index.ts       # Distribution entry point
```

## Technology Markers

- **package.json** - Node.js/Bun project manifest
- **tsconfig.json** - TypeScript configuration
- **bun.lock** - Bun package manager lockfile
- **.github/workflows/** - GitHub Actions CI/CD
- **.husky/** - Git hooks for pre-commit

# Technology Stack

> Generated: 2026-01-27 | Part of Exhaustive Scan

## Overview

| Category             | Technology                | Version  | Notes                                 |
| -------------------- | ------------------------- | -------- | ------------------------------------- |
| **Language**         | TypeScript                | 5.9.3    | Strict mode enabled                   |
| **Runtime**          | Bun                       | Latest   | Also compatible with Node.js >=14.0.0 |
| **Package Manager**  | Bun                       | -        | Using bun.lock                        |
| **Build**            | TypeScript Compiler (tsc) | -        | ES2022 target                         |
| **CLI Framework**    | Commander.js              | 14.0.2   | Command parsing and help generation   |
| **Terminal Styling** | Chalk                     | 5.6.2    | Colorized output                      |
| **Testing**          | Bun Test                  | Built-in | Native Bun test runner                |
| **Linting**          | Oxlint                    | 1.34.0   | Fast Rust-based linter                |
| **Formatting**       | Oxfmt                     | 0.19.0   | Rust-based formatter                  |
| **Git Hooks**        | Husky                     | 9.1.7    | Pre-commit hooks                      |

## TypeScript Configuration

**Target:** ES2022
**Module System:** NodeNext (ESM)
**Strict Mode:** Fully enabled

### Strict Type Checking Options

- `strict: true`
- `noImplicitAny: true`
- `strictNullChecks: true`
- `strictFunctionTypes: true`
- `strictBindCallApply: true`
- `strictPropertyInitialization: true`
- `noImplicitThis: true`
- `alwaysStrict: true`
- `noUnusedLocals: true`
- `noUnusedParameters: true`
- `noImplicitReturns: true`
- `noFallthroughCasesInSwitch: true`
- `noUncheckedIndexedAccess: true`

### Compilation Includes

```
src/**/*
lib/**/*
commands/**/*
types/**/*
index.ts
```

### Output

- **Directory:** `./dist`
- **Declaration files:** Yes (.d.ts)
- **Source maps:** Yes
- **Declaration maps:** Yes

## Dependencies

### Production Dependencies

| Package   | Version | Purpose                 |
| --------- | ------- | ----------------------- |
| chalk     | ^5.6.2  | Terminal string styling |
| commander | ^14.0.2 | CLI argument parsing    |

### Development Dependencies

| Package     | Version | Purpose                  |
| ----------- | ------- | ------------------------ |
| @types/bun  | ^1.3.5  | Bun type definitions     |
| @types/node | ^25.0.3 | Node.js type definitions |
| husky       | ^9.1.7  | Git hooks                |
| oxfmt       | ^0.19.0 | Code formatting          |
| oxlint      | ^1.34.0 | Linting                  |
| typescript  | ^5.9.3  | TypeScript compiler      |

## CI/CD Pipelines

### GitHub Actions Workflows

| Workflow          | Trigger         | Purpose                                 |
| ----------------- | --------------- | --------------------------------------- |
| `tests.yml`       | Push/PR to main | Run tests and build on Ubuntu & Windows |
| `npm-publish.yml` | Release created | Publish to npm with provenance          |
| `lint-format.yml` | -               | Code quality checks                     |
| `codeql.yml`      | -               | Security analysis                       |

### Test Matrix

- **OS:** Ubuntu Latest, Windows Latest
- **Runtime:** Bun (latest)
- **Fail-fast:** Disabled (runs all matrix combinations)

### Publishing

- **Registry:** npm (registry.npmjs.org)
- **Authentication:** OIDC trusted publishing (provenance)
- **Trigger:** GitHub Release creation

## Architecture Pattern

**Pattern:** Command-based CLI Architecture

```
┌─────────────────────────────────────────────────┐
│                  Entry Point                     │
│              src/index.ts (CLI)                  │
│             index.ts (Library)                   │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│              Commander.js Program                │
│         Command Registration & Parsing           │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│               Commands Layer                     │
│  commands/*.ts (15 command modules)              │
│  - Each command self-contained                   │
│  - Uses shared lib utilities                     │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│               Library Layer                      │
│  lib/*.ts (Core utilities)                       │
│  - api-client.ts (Canvas API)                    │
│  - config.ts (Configuration)                     │
│  - display.ts (Table formatting)                 │
│  - interactive.ts (User prompts)                 │
│  - file-upload.ts (File handling)                │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│              External Services                   │
│  - Canvas LMS REST API                           │
│  - Local filesystem                              │
│  - User config (~/.canvaslms-cli-config.json)    │
└─────────────────────────────────────────────────┘
```

## Module System

- **Type:** ESM (ECMAScript Modules)
- **Package.json type:** `"module"`
- **Import style:** Named exports with `.js` extensions

## Engine Requirements

```json
{
  "node": ">=14.0.0",
  "npm": ">=6.0.0"
}
```

## Binary Distribution

```json
{
  "bin": {
    "canvas": "dist/src/index.js",
    "canvaslms-cli": "dist/src/index.js"
  }
}
```

Two command names available after global install:

- `canvas` (short form)
- `canvaslms-cli` (full name)

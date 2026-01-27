# Development Guide

> Generated: 2026-01-27 | Part of Exhaustive Scan

## Quick Start

### Prerequisites

- **Runtime:** Bun (recommended) or Node.js >=14.0.0
- **Package Manager:** Bun
- **TypeScript:** 5.9.3+

### Initial Setup

```bash
# Clone repository
git clone <repo-url>
cd Canvas-CLI

# Install dependencies
bun install

# Build project
bun run build

# Link for local development
bun link
```

### Configuration

Before using the CLI, configure your Canvas credentials:

```bash
canvas config setup
```

This creates `~/.canvaslms-cli-config.json` with:

- `domain` - Your Canvas instance (e.g., `school.instructure.com`)
- `token` - API bearer token from Canvas settings
- `tableTruncate` - Display mode preference

## Development Workflow

### Building

```bash
# Full build (TypeScript compilation)
bun run build

# Output structure:
# dist/
# ├── src/index.js      # CLI entry point
# ├── lib/              # Compiled libraries
# ├── commands/         # Compiled commands
# └── types/            # Type definitions
```

### Running Tests

```bash
# Run all tests
bun test

# Run with coverage
bun test --coverage

# Run specific test file
bun test tests/grades.test.ts
```

### Code Quality

```bash
# Linting with oxlint
bunx oxlint .

# Formatting with oxfmt
bunx oxfmt .

# Type checking
bun run build  # TypeScript compiler validates types
```

### Pre-commit Hooks

The project uses Husky for git hooks:

- **pre-commit:** Runs linting and formatting checks

## Project Structure

```
Canvas-CLI/
├── src/index.ts          # CLI entry point (Commander.js)
├── commands/             # Command implementations (15 files)
├── lib/                  # Shared utilities (6 files)
├── types/index.ts        # TypeScript interfaces
├── tests/                # Test suite (17 files)
└── dist/                 # Compiled output
```

## Adding New Commands

### 1. Create Command File

```typescript
// commands/mycommand.ts
import readline from "readline";
import { makeCanvasRequest } from "../lib/api-client.js";
import { Table } from "../lib/display.js";
import type { MyCommandOptions } from "../types/index.js";

export async function myCommand(
  rl: readline.Interface,
  options: MyCommandOptions
): Promise<void> {
  // Implementation
}
```

### 2. Add Type Definitions

```typescript
// types/index.ts
export interface MyCommandOptions {
  all?: boolean;
  verbose?: boolean;
}
```

### 3. Register in CLI

```typescript
// src/index.ts
import { myCommand } from "../commands/mycommand.js";

program
  .command("mycommand")
  .aliases(["mc", "my"])
  .description("Description of command")
  .option("-a, --all", "Include all items")
  .option("-v, --verbose", "Verbose output")
  .action(requireConfig(myCommand));
```

### 4. Add Tests

```typescript
// tests/mycommand.test.ts
import { describe, it, expect, mock, beforeEach, afterEach } from "bun:test";
import { myCommand } from "../commands/mycommand";

describe("myCommand", () => {
  it("should do expected behavior", async () => {
    // Test implementation
  });
});
```

## API Client Usage

### Making Requests

```typescript
import { makeCanvasRequest } from "../lib/api-client.js";

// GET request
const courses = await makeCanvasRequest<CanvasCourse[]>(
  "GET",
  "/api/v1/courses",
  ["per_page=100", "enrollment_state=active"]
);

// POST request with body
const result = await makeCanvasRequest<any>(
  "POST",
  "/api/v1/courses/123/assignments",
  [],
  JSON.stringify({ assignment: { name: "New Assignment" } })
);
```

### Course Selection

```typescript
import { getCanvasCourse, getCanvasCourses } from "../lib/api-client.js";

// Get all courses
const courses = await getCanvasCourses(true);

// Interactive course selection
const course = await getCanvasCourse("partial name", rl);
```

## Display Utilities

### Table Rendering

```typescript
import { Table } from "../lib/display.js";

const table = new Table([
  { key: "name", header: "Name", flex: 2 },
  { key: "grade", header: "Grade", width: 10 },
  { key: "status", header: "Status", width: 15 }
]);

for (const item of items) {
  table.addRow({
    name: item.name,
    grade: formatGrade(item.score),
    status: item.status
  });
}

table.renderWithResize();
```

### Interactive Prompts

```typescript
import { askQuestion, askConfirmation, selectFilesKeyboard } from "../lib/interactive.js";

// Text input
const answer = await askQuestion(rl, "Enter your choice: ");

// Yes/No confirmation
const confirmed = await askConfirmation(rl, "Proceed?");

// File selection
const files = await selectFilesKeyboard(rl, process.cwd(), [".pdf", ".docx"]);
```

## Testing Patterns

### Mocking API Responses

```typescript
// tests/mocks/mock-canvas-api.ts
export function mockCourseResponse() {
  return [
    { id: 1, name: "Test Course", enrollments: [...] }
  ];
}
```

### Test Fixtures

```typescript
// tests/mocks/fixtures.ts
export const testUser = {
  id: 1,
  name: "Test User",
  email: "test@example.com"
};

export const testCourse = {
  id: 123,
  name: "Introduction to Testing",
  course_code: "TEST101"
};
```

## Error Handling

### API Errors

The `makeCanvasRequest` function handles common HTTP errors:

| Status | Error Message                               |
| ------ | ------------------------------------------- |
| 401    | "Unauthorized. Please check your API token" |
| 403    | "Access denied. You don't have permission"  |
| 404    | "Resource not found"                        |

### User Errors

For user-facing errors, use chalk for colored output:

```typescript
import chalk from "chalk";

console.error(chalk.red("Error: File not found"));
console.warn(chalk.yellow("Warning: Assignment past due"));
console.log(chalk.green("Success: Submission uploaded"));
```

## Versioning

### Bumping Version

```bash
# Using the version bump script
bun run scripts/bump-version.ts [major|minor|patch]

# Manual update
# 1. Update version in package.json
# 2. Update CHANGELOG.md
# 3. Commit with version tag
```

### Release Process

1. Update `CHANGELOG.md` with release notes
2. Run `bun run scripts/bump-version.ts <type>`
3. Create GitHub release
4. npm publish workflow triggers automatically

## CI/CD

### GitHub Actions Workflows

| Workflow          | Trigger         | Purpose                        |
| ----------------- | --------------- | ------------------------------ |
| `tests.yml`       | Push/PR to main | Run tests on Ubuntu & Windows  |
| `npm-publish.yml` | Release created | Publish to npm with provenance |
| `lint-format.yml` | Push/PR         | Code quality checks            |
| `codeql.yml`      | Scheduled       | Security analysis              |

### Test Matrix

- **OS:** Ubuntu Latest, Windows Latest
- **Runtime:** Bun (latest)
- **Fail-fast:** Disabled

## Common Tasks

### Debug a Command

```bash
# Run with Node.js debugger
node --inspect dist/src/index.js <command>

# Add console.log statements
# Then rebuild and run
bun run build && canvas <command>
```

### Test API Responses

```bash
# Test Canvas API directly with curl
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://your-domain.instructure.com/api/v1/courses"
```

### Profile Performance

```bash
# Run with timing
time canvas list

# Memory usage
node --trace-gc dist/src/index.js list
```

## Troubleshooting

### Common Issues

**"Config not found" error:**

```bash
canvas config setup
```

**"Unauthorized" error:**

- Verify API token in Canvas settings
- Regenerate token if expired
- Check token scopes

**Build errors:**

```bash
# Clean rebuild
rm -rf dist && bun run build
```

**Test failures:**

```bash
# Run specific test with verbose output
bun test tests/failing.test.ts --verbose
```

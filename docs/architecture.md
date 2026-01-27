# Canvas-CLI Architecture Documentation

> Generated: 2026-01-27 | Scan Level: Exhaustive | Mode: Initial Scan

## Executive Summary

Canvas-CLI is a modern command-line interface for Canvas LMS built with TypeScript and Bun. It provides 15 commands for managing courses, assignments, grades, files, and more through the Canvas REST API.

**Key Characteristics:**

- **Architecture Pattern:** Command-based CLI with shared library utilities
- **Language:** TypeScript (ES2022, strict mode)
- **Runtime:** Bun (Node.js >=14.0.0 compatible)
- **API Integration:** Canvas LMS REST API v1
- **CLI Framework:** Commander.js v14

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                               │
│                   Terminal / Command Line                            │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                        ENTRY LAYER                                   │
│  src/index.ts - Commander.js Program                                 │
│  - Command registration and argument parsing                         │
│  - Config validation wrapper (requireConfig)                         │
│  - Version management                                                │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       COMMANDS LAYER                                 │
│  commands/*.ts - 15 Command Implementations                          │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐          │
│  │ list.ts     │ grades.ts   │ submit.ts   │ download.ts │          │
│  │ config.ts   │ gpa.ts      │ files.ts    │ calendar.ts │          │
│  │ assignments │ modules.ts  │ groups.ts   │ todo.ts     │          │
│  │ announce... │ profile.ts  │ star.ts     │             │          │
│  └─────────────┴─────────────┴─────────────┴─────────────┘          │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       LIBRARY LAYER                                  │
│  lib/*.ts - Shared Utilities (6 modules)                             │
│  ┌────────────────────┬────────────────────┬────────────────────┐   │
│  │ api-client.ts      │ display.ts         │ interactive.ts     │   │
│  │ - Canvas API calls │ - Table rendering  │ - User prompts     │   │
│  │ - HTTP methods     │ - Course picker    │ - File browser     │   │
│  │ - Error handling   │ - Grade formatting │ - Keyboard nav     │   │
│  ├────────────────────┼────────────────────┼────────────────────┤   │
│  │ config.ts          │ config-validator   │ file-upload.ts     │   │
│  │ - Load/save config │ - requireConfig()  │ - Canvas file API  │   │
│  │ - Config path      │ - ensureConfig()   │ - Multi-part upload│   │
│  └────────────────────┴────────────────────┴────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                        TYPES LAYER                                   │
│  types/index.ts - TypeScript Interfaces                              │
│  - CanvasCourse, CanvasAssignment, CanvasSubmission                  │
│  - CanvasEnrollment, CanvasGrade, CanvasUser                         │
│  - CanvasFile, CanvasModule, CanvasGroup, etc.                       │
│  - Command option interfaces                                         │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                      EXTERNAL SYSTEMS                                │
│  ┌─────────────────────┐  ┌─────────────────────┐                   │
│  │   Canvas LMS API    │  │   Local Filesystem  │                   │
│  │ - REST API v1       │  │ - Config file       │                   │
│  │ - Bearer token auth │  │ - File uploads      │                   │
│  │ - JSON responses    │  │ - Download storage  │                   │
│  └─────────────────────┘  └─────────────────────┘                   │
└─────────────────────────────────────────────────────────────────────┘
```

## Module Details

### Entry Point (`src/index.ts`)

The CLI entry point uses Commander.js to define all commands:

```typescript
const program = new Command();
program
  .name("canvas")
  .description("Canvas LMS Command Line Interface")
  .version("1.10.3");

// Commands wrapped with requireConfig() for auth validation
program.command("list").action(requireConfig(listCourses));
```

**Key Patterns:**

- Commands protected by `requireConfig()` wrapper
- Multiple aliases per command (e.g., `list`, `courses`, `course`)
- Consistent option naming (`-a` for all, `-v` for verbose)

### Library Modules

#### `lib/api-client.ts` - Canvas API Client

Core API communication module:

```typescript
// Main API function
async function makeCanvasRequest<T>(
  method: string,
  endpoint: string,
  queryParams: string[] = [],
  requestBody: string | null = null
): Promise<T>

// Course helpers
async function getCanvasCourses(getAllCourse: boolean): Promise<CanvasCourse[]>
async function getCanvasCourse(courseName: string, rl: readline.Interface): Promise<CanvasCourse | undefined>
```

**Features:**

- Generic HTTP method support (GET, POST, PUT, DELETE)
- Query parameter handling
- File upload body support (from file path with `@filename`)
- Comprehensive error handling (401, 403, 404, etc.)
- Domain-specific filtering (e.g., Swinburne ORG- courses)

#### `lib/config.ts` - Configuration Management

Handles persistent user configuration:

```typescript
// Config file: ~/.canvaslms-cli-config.json
interface CanvasConfig {
  domain: string;      // Canvas instance domain
  token: string;       // API bearer token
  tableTruncate?: boolean;  // Table display mode
  createdAt?: string;
  lastUpdated?: string;
}
```

**Functions:**

- `loadConfig()` - Load and validate configuration
- `saveConfig()` - Save configuration to disk
- `readConfig()` - Read raw configuration
- `configExists()` - Check if config file exists
- `getConfigPath()` - Get config file path
- `deleteConfig()` - Remove config file

#### `lib/display.ts` - Table Rendering

Rich terminal table display with live resize support:

```typescript
class Table {
  constructor(columns: ColumnDefinition[], options?: TableOptions)
  addRow(row: Record<string, any>): this
  render(): void
  renderWithResize(): void  // Live resize on terminal resize
  stopWatching(): void
}
```

**Features:**

- Adaptive column widths
- Flex-based layout system
- Color-coded values
- Text wrapping or truncation modes
- Unicode box drawing characters
- Live resize handling

#### `lib/interactive.ts` - User Interaction

Terminal interaction utilities:

```typescript
// Basic prompts
function askQuestion(rl: readline.Interface, question: string): Promise<string>
function askConfirmation(rl: readline.Interface, question: string): Promise<boolean>
function selectFromList<T>(rl: readline.Interface, items: T[]): Promise<T | null>

// Advanced file browser
function selectFilesKeyboard(
  rl: readline.Interface,
  currentDir: string,
  allowedExtensions?: string[]
): Promise<string[]>
```

**File Browser Features:**

- Keyboard navigation (arrow keys, space, enter)
- Multi-file selection
- File type icons
- Breadcrumb navigation
- Extension filtering
- Size display

#### `lib/file-upload.ts` - File Upload

Canvas file upload handling:

```typescript
async function uploadSingleFileToCanvas(
  courseId: number,
  assignmentId: number,
  filePath: string
): Promise<number>  // Returns file ID

async function submitAssignmentWithFiles(
  courseId: number,
  assignmentId: number,
  fileIds: number[]
): Promise<any>
```

**Upload Process:**

1. Get upload URL from Canvas API
2. POST file with FormData to upload URL
3. Retrieve file ID from response
4. Submit assignment with file IDs

### Command Implementations

| Command         | File               | Description                                   |
| --------------- | ------------------ | --------------------------------------------- |
| `list`          | `list.ts`          | List enrolled/starred courses                 |
| `config`        | `config.ts`        | Configuration management (setup, edit, show)  |
| `assignments`   | `assignments.ts`   | List assignments with filters                 |
| `grades`        | `grades.ts`        | View grades summary and detailed breakdown    |
| `gpa`           | `gpa.ts`           | GPA calculator (4.0 scale) and what-if grades |
| `announcements` | `announcements.ts` | View course announcements                     |
| `calendar`      | `calendar.ts`      | View upcoming due dates                       |
| `modules`       | `modules.ts`       | Browse course modules/content                 |
| `todo`          | `todo.ts`          | View pending todo items                       |
| `files`         | `files.ts`         | Browse and download course files              |
| `download`      | `download.ts`      | Bulk download all course files                |
| `submit`        | `submit.ts`        | Submit files to assignments                   |
| `groups`        | `groups.ts`        | View group memberships                        |
| `star`          | `star.ts`          | Star/unstar courses                           |
| `profile`       | `profile.ts`       | Display user profile                          |

### Type Definitions

Located in `types/index.ts`, providing TypeScript interfaces for:

**Canvas API Types:**

- `CanvasCourse` - Course data with enrollment info
- `CanvasAssignment` - Assignment with submission types
- `CanvasSubmission` - Submission status and grades
- `CanvasEnrollment` - Enrollment with grade data
- `CanvasAnnouncement` - Announcement content
- `CanvasFile` / `CanvasFolder` - File system items
- `CanvasModule` / `CanvasModuleItem` - Module content
- `CanvasGroup` - Group membership

**Command Option Types:**

- `ListCoursesOptions`
- `ListAssignmentsOptions`
- `ShowGradesOptions`
- `SubmitAssignmentParams`
- etc.

## Data Flow

### Authentication Flow

```
User → canvas config setup
         │
         ▼
    Interactive Wizard
    - Prompt for domain
    - Prompt for API token
         │
         ▼
    Save to ~/.canvaslms-cli-config.json
         │
         ▼
    requireConfig() wrapper validates on each command
```

### Command Execution Flow

```
User Command → Commander.js Parser
                    │
                    ▼
            requireConfig() wrapper
            - Check config exists
            - Prompt setup if needed
                    │
                    ▼
            Command Handler
            - Parse arguments/options
            - Call lib utilities
                    │
                    ▼
            makeCanvasRequest()
            - Build URL with params
            - Add auth header
            - Handle response/errors
                    │
                    ▼
            Display Results
            - Format with Table class
            - Apply colors
            - Handle terminal resize
```

### File Submission Flow

```
canvas submit
     │
     ▼
1. Course Selection
   - Interactive picker or name match
     │
     ▼
2. Assignment Selection
   - Filter online_upload types
   - Display with due dates
     │
     ▼
3. File Selection
   - selectFilesKeyboard()
   - Multi-file support
     │
     ▼
4. Confirmation
   - askConfirmation()
     │
     ▼
5. Upload Loop
   - uploadSingleFileToCanvas() per file
   - Collect file IDs
     │
     ▼
6. Submit
   - submitAssignmentWithFiles()
   - Show success URL
```

## Testing Architecture

### Test Structure

```
tests/
├── mocks/
│   ├── index.ts           # Mock exports
│   ├── fixtures.ts        # Test data fixtures
│   └── mock-canvas-api.ts # API mock utilities
├── *.test.ts              # Test files per feature
└── e2e.test.ts           # End-to-end tests
```

### Test Coverage

| Test File                    | Coverage Area            |
| ---------------------------- | ------------------------ |
| `list.test.ts`               | Course listing           |
| `grades.test.ts`             | Grade display            |
| `config.test.ts`             | Configuration management |
| `display.test.ts`            | Table rendering          |
| `display-edge-cases.test.ts` | Edge cases for display   |
| `submit.test.ts`             | Assignment submission    |
| `files.test.ts`              | File operations          |
| `calendar.test.ts`           | Calendar/due dates       |
| `modules.test.ts`            | Module browsing          |
| `announcements.test.ts`      | Announcements            |
| `profile.test.ts`            | User profile             |
| `groups.test.ts`             | Group memberships        |
| `todo.test.ts`               | Todo items               |
| `e2e.test.ts`                | End-to-end scenarios     |

### Testing Commands

```bash
bun test              # Run all tests
bun test --coverage   # With coverage report
bun test <file>       # Run specific test file
```

## Security Considerations

### API Token Storage

- Token stored in plain text in `~/.canvaslms-cli-config.json`
- File permissions should be user-read-only (0600)
- Token should have minimal required scopes

### HTML Sanitization

The `parseHtmlContent()` function in `display.ts` sanitizes HTML content:

```typescript
// Iterative sanitization to prevent bypass attacks
let prevText = "";
while (prevText !== text) {
  prevText = text;
  text = text
    .replace(/<style\b[^>]*>[\s\S]*?<\/style\s*[^>]*>/gi, "")
    .replace(/<script\b[^>]*>[\s\S]*?<\/script\s*[^>]*>/gi, "")
    // ... other tag removal
}
```

## Error Handling

### API Error Mapping

| HTTP Status | Error Message                               |
| ----------- | ------------------------------------------- |
| 401         | "Unauthorized. Please check your API token" |
| 403         | "Access denied. You don't have permission"  |
| 404         | "Resource not found"                        |
| Other       | "HTTP {status}: {message}"                  |

### Upload Error Handling

- File not found errors with clear paths
- Permission denied with helpful suggestions
- Partial upload recovery (continue with remaining files)
- Manual completion fallback on submit failure

## Configuration

### Config File Schema

```json
{
  "domain": "school.instructure.com",
  "token": "your-api-token-here",
  "tableTruncate": false,
  "createdAt": "2024-01-15T10:30:00.000Z",
  "lastUpdated": "2024-01-20T14:22:00.000Z"
}
```

### Environment Requirements

- Node.js >=14.0.0
- npm >=6.0.0
- Or Bun (recommended)

## Build & Distribution

### Build Process

```bash
bun run build    # Compile TypeScript to dist/
```

### Output Structure

```
dist/
├── src/
│   └── index.js     # CLI entry point
├── lib/             # Compiled library modules
├── commands/        # Compiled commands
├── types/           # Type definitions (.d.ts)
└── index.js         # Library export
```

### npm Package

```json
{
  "bin": {
    "canvas": "dist/src/index.js",
    "canvaslms-cli": "dist/src/index.js"
  },
  "main": "dist/index.js",
  "types": "dist/index.d.ts"
}
```

## Future Considerations

### Potential Enhancements

1. **Caching** - Local cache for course/assignment data to reduce API calls
2. **Offline Mode** - Queue submissions for later sync
3. **Plugins** - Extensible command system
4. **Config Encryption** - Encrypted token storage
5. **Batch Operations** - Bulk grade exports, multi-assignment submissions

### Technical Debt

1. Some commands have duplicated course selection logic
2. Table rendering could be extracted to a dedicated package
3. File browser could use a UI framework for better terminal support

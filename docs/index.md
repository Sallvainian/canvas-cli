# Canvas-CLI Documentation Index

> **Generated:** 2026-01-27 | **Scan Level:** Exhaustive | **Version:** 1.10.3

## Quick Reference

| Aspect | Value |
|--------|-------|
| **Package** | `canvaslms-cli` |
| **Language** | TypeScript (ES2022, strict) |
| **Runtime** | Bun (Node.js >=14.0.0 compatible) |
| **CLI Framework** | Commander.js v14 |
| **API** | Canvas LMS REST API v1 |
| **Commands** | 15 CLI commands |
| **Config Location** | `~/.canvaslms-cli-config.json` |

## Documentation Files

| Document | Purpose | Key Contents |
|----------|---------|--------------|
| [architecture.md](./architecture.md) | System design | Layers, modules, data flows |
| [source-tree-analysis.md](./source-tree-analysis.md) | Directory structure | File locations, import graph |
| [technology-stack.md](./technology-stack.md) | Tech decisions | Dependencies, CI/CD, configs |
| [development-guide.md](./development-guide.md) | Developer setup | Build, test, contribute |
| [existing-documentation.md](./existing-documentation.md) | Doc inventory | README, CHANGELOG locations |

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────┐
│                      USER INTERFACE                          │
│                  Terminal / Command Line                     │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      ENTRY LAYER                             │
│  src/index.ts - Commander.js Program                         │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    COMMANDS LAYER                            │
│  commands/*.ts - 15 Command Implementations                  │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    LIBRARY LAYER                             │
│  lib/*.ts - Shared Utilities                                 │
│  api-client | display | interactive | config | file-upload   │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                   EXTERNAL SYSTEMS                           │
│  Canvas LMS API (REST v1) | Local Filesystem                 │
└─────────────────────────────────────────────────────────────┘
```

## Command Reference

| Command | File | Description |
|---------|------|-------------|
| `canvas list` | `commands/list.ts` | List enrolled/starred courses |
| `canvas config` | `commands/config.ts` | Configuration management |
| `canvas assignments` | `commands/assignments.ts` | List assignments with filters |
| `canvas grades` | `commands/grades.ts` | View grades and breakdown |
| `canvas gpa` | `commands/gpa.ts` | Calculate overall GPA |
| `canvas announcements` | `commands/announcements.ts` | View course announcements |
| `canvas calendar` | `commands/calendar.ts` | View upcoming due dates |
| `canvas modules` | `commands/modules.ts` | Browse course modules |
| `canvas todo` | `commands/todo.ts` | View pending items |
| `canvas files` | `commands/files.ts` | Interactive file browser |
| `canvas download` | `commands/download.ts` | Bulk file download |
| `canvas submit` | `commands/submit.ts` | Submit assignments |
| `canvas groups` | `commands/groups.ts` | View group memberships |
| `canvas star` | `commands/star.ts` | Star/unstar courses |
| `canvas profile` | `commands/profile.ts` | Display user profile |

## Core Modules

### `lib/api-client.ts` (218 LOC)
Canvas API communication layer.

**Key Exports:**
- `makeCanvasRequest<T>(method, endpoint, params, body)` - Generic HTTP request
- `getCanvasCourses(getAllCourses)` - Fetch enrolled courses
- `getCanvasCourse(courseName, rl)` - Interactive course selection

### `lib/display.ts` (1097 LOC)
Terminal table rendering with live resize support.

**Key Exports:**
- `Table` class - Adaptive column width tables
- `pickCourse(courses, rl)` - Interactive course picker
- `formatGrade(grade)` - Color-coded grade display
- `parseHtmlContent(html)` - HTML sanitization

### `lib/interactive.ts` (835 LOC)
User interaction utilities.

**Key Exports:**
- `askQuestion(rl, question)` - Text input prompt
- `askConfirmation(rl, question)` - Yes/No prompt
- `selectFilesKeyboard(rl, dir, extensions)` - File browser

### `lib/config.ts` (137 LOC)
Configuration file management.

**Key Exports:**
- `loadConfig()` - Load and validate config
- `saveConfig(config)` - Persist config to disk
- `configExists()` - Check config existence
- `getConfigPath()` - Get config file path

### `lib/file-upload.ts` (141 LOC)
Canvas file upload handling.

**Key Exports:**
- `uploadSingleFileToCanvas(courseId, assignmentId, filePath)` - Upload file
- `submitAssignmentWithFiles(courseId, assignmentId, fileIds)` - Submit with files

## Type Definitions

Located in `types/index.ts` (337 LOC):

**Canvas API Types:**
- `CanvasCourse` - Course with enrollment info
- `CanvasAssignment` - Assignment with submission types
- `CanvasSubmission` - Submission status and grades
- `CanvasEnrollment` - Enrollment with grade data
- `CanvasFile` / `CanvasFolder` - File system items
- `CanvasModule` / `CanvasModuleItem` - Module content
- `CanvasAnnouncement` - Announcement content
- `CanvasGroup` - Group membership
- `CanvasUser` - User profile data
- `CanvasTodoItem` - Todo item

**Configuration Types:**
- `CanvasConfig` - Main config interface
- `InstanceConfig` - Instance-specific settings

**Command Option Types:**
- `ListCoursesOptions`, `ListAssignmentsOptions`
- `ShowGradesOptions`, `ShowAnnouncementsOptions`
- `ShowFilesOptions`, `ShowGroupsOptions`
- `ModulesOptions`, `CalendarOptions`, etc.

## Test Coverage

| Test File | Coverage Area |
|-----------|---------------|
| `tests/list.test.ts` | Course listing |
| `tests/grades.test.ts` | Grade display |
| `tests/config.test.ts` | Configuration |
| `tests/display.test.ts` | Table rendering |
| `tests/display-edge-cases.test.ts` | Display edge cases |
| `tests/submit.test.ts` | Assignment submission |
| `tests/files.test.ts` | File operations |
| `tests/calendar.test.ts` | Calendar/due dates |
| `tests/modules.test.ts` | Module browsing |
| `tests/announcements.test.ts` | Announcements |
| `tests/profile.test.ts` | User profile |
| `tests/groups.test.ts` | Group memberships |
| `tests/todo.test.ts` | Todo items |
| `tests/e2e.test.ts` | End-to-end scenarios |

## Key Patterns

### Authentication Flow
1. User runs `canvas config setup`
2. Interactive wizard prompts for domain and API token
3. Config saved to `~/.canvaslms-cli-config.json`
4. `requireConfig()` wrapper validates on each command

### Command Execution Flow
1. Commander.js parses user input
2. `requireConfig()` validates configuration
3. Command handler processes arguments
4. `makeCanvasRequest()` calls Canvas API
5. `Table` class renders results

### File Submission Flow
1. Course selection via `pickCourse()`
2. Assignment selection (filtered for online_upload)
3. File selection via `selectFilesKeyboard()`
4. Confirmation via `askConfirmation()`
5. Upload loop via `uploadSingleFileToCanvas()`
6. Submit via `submitAssignmentWithFiles()`

## Security Considerations

- **Token Storage:** Plain text in `~/.canvaslms-cli-config.json`
- **Recommended:** Set file permissions to 0600 (user-read-only)
- **HTML Sanitization:** `parseHtmlContent()` iteratively removes dangerous tags
- **API Scopes:** Token should have minimal required scopes

## Development Commands

```bash
# Setup
bun install

# Build
bun run build

# Test
bun test
bun test --coverage

# Quality
bunx oxlint .
bunx oxfmt .

# Version bump
bun run scripts/bump-version.ts [major|minor|patch]
```

## File Statistics

| Category | Files | Lines of Code |
|----------|-------|---------------|
| Commands | 15 | ~3,500 |
| Library | 6 | ~2,500 |
| Types | 1 | ~340 |
| Tests | 17 | ~2,000 |
| Config | 5 | ~150 |
| **Total** | **44** | **~8,500** |

## Entry Points

| Entry Point | Location | Purpose |
|-------------|----------|---------|
| CLI Binary | `dist/src/index.js` | Command-line interface |
| Library | `dist/index.js` | Programmatic API access |
| Development | `index.ts` | Source library export |
| Tests | `tests/*.test.ts` | Test runner entry |

---

*This documentation was generated via exhaustive codebase scan. For updates, re-run the document-project workflow.*

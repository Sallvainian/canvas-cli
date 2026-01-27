# Source Tree Analysis

> Generated: 2026-01-27 | Scan Level: Exhaustive

## Directory Structure

```
Canvas-CLI/
├── 📄 index.ts                    # Library export module (exports all public APIs)
├── 📄 package.json                # Package manifest (canvaslms-cli v1.10.3)
├── 📄 tsconfig.json               # TypeScript config (ES2022, strict mode)
├── 📄 bun.lock                    # Bun package lock file
├── 📄 README.md                   # Main documentation
├── 📄 CHANGELOG.md                # Version history
├── 📄 LICENSE                     # MIT License
│
├── 📁 src/                        # Distribution entry point
│   └── 📄 index.ts                # CLI program (Commander.js setup)
│
├── 📁 commands/                   # Command implementations (15 files)
│   ├── 📄 announcements.ts        # View course announcements
│   ├── 📄 assignments.ts          # List/filter assignments
│   ├── 📄 calendar.ts             # View upcoming due dates
│   ├── 📄 config.ts               # Configuration management
│   ├── 📄 download.ts             # Bulk file download
│   ├── 📄 files.ts                # File browser
│   ├── 📄 gpa.ts                  # GPA calculator + what-if
│   ├── 📄 grades.ts               # Grade display
│   ├── 📄 groups.ts               # Group memberships
│   ├── 📄 list.ts                 # Course listing
│   ├── 📄 modules.ts              # Module browser
│   ├── 📄 profile.ts              # User profile
│   ├── 📄 star.ts                 # Star/unstar courses
│   ├── 📄 submit.ts               # Assignment submission
│   └── 📄 todo.ts                 # Todo items
│
├── 📁 lib/                        # Shared utility libraries (6 files)
│   ├── 📄 api-client.ts           # Canvas API client (makeCanvasRequest)
│   ├── 📄 config.ts               # Config file management
│   ├── 📄 config-validator.ts     # Config validation wrapper
│   ├── 📄 display.ts              # Table rendering (Table class)
│   ├── 📄 file-upload.ts          # File upload utilities
│   └── 📄 interactive.ts          # User prompts & file browser
│
├── 📁 types/                      # TypeScript definitions
│   └── 📄 index.ts                # All type interfaces
│
├── 📁 tests/                      # Test suite (17 files)
│   ├── 📁 mocks/                  # Test utilities
│   │   ├── 📄 fixtures.ts         # Test data fixtures
│   │   ├── 📄 index.ts            # Mock exports
│   │   └── 📄 mock-canvas-api.ts  # API mocking
│   ├── 📄 announcements.test.ts
│   ├── 📄 calendar.test.ts
│   ├── 📄 config.test.ts
│   ├── 📄 display.test.ts
│   ├── 📄 display-edge-cases.test.ts
│   ├── 📄 e2e.test.ts
│   ├── 📄 files.test.ts
│   ├── 📄 grades.test.ts
│   ├── 📄 groups.test.ts
│   ├── 📄 list.test.ts
│   ├── 📄 modules.test.ts
│   ├── 📄 profile.test.ts
│   ├── 📄 submit.test.ts
│   └── 📄 todo.test.ts
│
├── 📁 scripts/                    # Development scripts
│   ├── 📄 bump-version.ts         # Version bumping utility
│   └── 📄 README.md               # Scripts documentation
│
├── 📁 assets/                     # Documentation images
│   ├── 📷 assignment_selection.png
│   ├── 📷 interactive.png
│   ├── 📷 grades_view.png
│   └── 📷 approved_integrations_canvas.png
│
├── 📁 .github/                    # GitHub configuration
│   └── 📁 workflows/              # CI/CD pipelines
│       ├── 📄 codeql.yml          # Security analysis
│       ├── 📄 lint-format.yml     # Code quality
│       ├── 📄 npm-publish.yml     # npm publishing
│       └── 📄 tests.yml           # Test & build
│
├── 📁 .husky/                     # Git hooks
│   └── 📄 pre-commit              # Pre-commit hook
│
└── 📁 dist/                       # Compiled output (generated)
    ├── src/index.js               # CLI binary
    ├── lib/                       # Compiled libraries
    ├── commands/                  # Compiled commands
    └── types/                     # Type definitions
```

## Critical Directories

### `/src` - CLI Entry Point

Contains the main CLI program that registers all commands with Commander.js.

**Key File:** `src/index.ts`
- Imports all command handlers
- Defines command structure with options
- Wraps commands with `requireConfig()` for auth validation

### `/commands` - Command Implementations

Each file exports a single command handler function:

| File | Export | Purpose |
|------|--------|---------|
| `list.ts` | `listCourses()` | List enrolled/starred courses |
| `config.ts` | `showConfig()`, `setupConfig()`, etc. | Config management |
| `assignments.ts` | `listAssignments()` | Assignment listing with filters |
| `grades.ts` | `showGrades()` | Grade display and breakdown |
| `gpa.ts` | `calculateOverallGPA()`, `calculateWhatIfGrade()` | GPA calculations |
| `announcements.ts` | `showAnnouncements()` | Announcement viewing |
| `calendar.ts` | `showCalendar()` | Due date calendar |
| `modules.ts` | `showModules()` | Module browser |
| `todo.ts` | `showTodo()` | Todo items |
| `files.ts` | `showFiles()` | File browser |
| `download.ts` | `bulkDownload()` | Bulk file download |
| `submit.ts` | `submitAssignment()` | Assignment submission |
| `groups.ts` | `showGroups()` | Group memberships |
| `star.ts` | `starCourse()`, `unstarCourse()` | Favorites management |
| `profile.ts` | `showProfile()` | User profile |

### `/lib` - Shared Libraries

Utility modules used across commands:

| File | Primary Exports | LOC | Purpose |
|------|----------------|-----|---------|
| `api-client.ts` | `makeCanvasRequest()`, `getCanvasCourses()` | 218 | Canvas API communication |
| `display.ts` | `Table`, `pickCourse()`, `formatGrade()` | 1097 | Table rendering, course selection |
| `interactive.ts` | `askQuestion()`, `selectFilesKeyboard()` | 835 | User prompts, file browser |
| `config.ts` | `loadConfig()`, `saveConfig()` | 137 | Config file management |
| `config-validator.ts` | `requireConfig()` | 71 | Config validation wrapper |
| `file-upload.ts` | `uploadSingleFileToCanvas()` | 141 | File upload handling |

### `/types` - Type Definitions

Single file containing all TypeScript interfaces:

**Canvas API Types:**
- `CanvasCourse`, `CanvasAssignment`, `CanvasSubmission`
- `CanvasEnrollment`, `CanvasGrade`, `CanvasUser`
- `CanvasFile`, `CanvasFolder`, `CanvasModule`, `CanvasModuleItem`
- `CanvasAnnouncement`, `CanvasGroup`, `CanvasTodoItem`

**Configuration Types:**
- `CanvasConfig`, `InstanceConfig`

**Command Option Types:**
- `ListCoursesOptions`, `ListAssignmentsOptions`, `ShowGradesOptions`
- `ShowAnnouncementsOptions`, `ApiQueryOptions`, `ShowTodoOptions`
- `ShowFilesOptions`, `ShowGroupsOptions`, `ModulesOptions`, `CalendarOptions`

### `/tests` - Test Suite

**Test Coverage by Feature:**
- Configuration: `config.test.ts`
- Display: `display.test.ts`, `display-edge-cases.test.ts`
- Commands: Individual test files per command
- E2E: `e2e.test.ts`

**Mocks:**
- `fixtures.ts` - Test data (courses, assignments, users)
- `mock-canvas-api.ts` - API response mocking

## File Statistics

| Category | Files | Lines of Code |
|----------|-------|---------------|
| Commands | 15 | ~3,500 |
| Library | 6 | ~2,500 |
| Types | 1 | ~340 |
| Tests | 17 | ~2,000 |
| Config | 5 | ~150 |
| **Total** | **44** | **~8,500** |

## Import Graph

```
src/index.ts (CLI Entry)
    │
    ├── commands/*.ts
    │       │
    │       ├── lib/api-client.ts
    │       │       └── lib/config.ts
    │       │       └── types/index.ts
    │       │
    │       ├── lib/display.ts
    │       │       └── lib/interactive.ts
    │       │       └── types/index.ts
    │       │
    │       ├── lib/interactive.ts
    │       │
    │       └── lib/file-upload.ts
    │               └── lib/api-client.ts
    │
    └── lib/config-validator.ts
            └── lib/config.ts
            └── lib/interactive.ts

index.ts (Library Export)
    │
    ├── lib/api-client.ts
    ├── lib/config.ts
    ├── lib/config-validator.ts
    ├── lib/file-upload.ts
    ├── lib/interactive.ts
    ├── commands/*.ts
    └── types/index.ts
```

## Entry Points

| Entry Point | Location | Purpose |
|-------------|----------|---------|
| CLI Binary | `dist/src/index.js` | Command-line interface |
| Library | `dist/index.js` | Programmatic API access |
| Development | `index.ts` | Source library export |
| Tests | `tests/*.test.ts` | Test runner entry |

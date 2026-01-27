# Existing Documentation Inventory

> Generated: 2026-01-27 | Part of Exhaustive Scan

## Documentation Files Found

| File | Type | Location | Description |
|------|------|----------|-------------|
| README.md | Main README | `/README.md` | Comprehensive usage guide with features, installation, setup, and all 15 commands |
| CHANGELOG.md | Changelog | `/CHANGELOG.md` | Version history and release notes |
| scripts/README.md | Scripts Docs | `/scripts/README.md` | Documentation for version bump script |
| LICENSE | License | `/LICENSE` | MIT License |

## README.md Summary

The main README covers:

### Installation
- Global npm installation: `npm install -g canvaslms-cli`

### Setup Process
1. Getting Canvas API Token (from Canvas Account → Settings → Approved Integrations)
2. Running `canvas config setup`
3. Entering Canvas domain and API token

### Configuration Storage
- Config file: `~/.canvaslms-cli-config.json`
- Settings: domain, token, truncate mode

### Commands Documented
All 15 commands with full usage examples:
- `config` - Configuration management
- `list` - List courses (starred/all)
- `star/unstar` - Manage favorite courses
- `assignments` - View assignments with filters (-s, -p, -a, -m, -w)
- `grades` - View detailed grades
- `gpa` - Calculate overall GPA
- `what-if` - What-if grade calculator (referenced as `canvas what-if` but implemented in grades command)
- `download` - Bulk download course files
- `announcements` - View course announcements
- `calendar` - View upcoming due dates
- `modules` - Browse course modules
- `todo` - View pending items
- `files` - Interactive file browser
- `groups` - View group memberships
- `submit` - Submit assignments
- `profile` - Display user profile

### Visual Assets
- `assets/assignment_selection.png` - Assignment submission interface
- `assets/interactive.png` - Interactive files selection
- `assets/grades_view.png` - Detailed grades view
- `assets/approved_integrations_canvas.png` - Canvas token setup guide

## User-Provided Context

No additional context or focus areas specified.

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Gemini, etc.) when working with code in this repository.

## Version Control Safety Protocol

### IMPORTANT: Backup Branch Creation Before Commits

**Always create a backup branch before making commits and pushes to preserve working state:**

```bash
# 1. Create backup branch with timestamp and descriptive name
# Format: backup-YYYY-MM-DD-HHMM-short-description
# Example: backup-2025-01-15-1430-auth-fix
git checkout -b backup-$(date +%Y-%m-%d-%H%M)-<short-description>

# 2. Push backup branch to remote
git push -u origin backup-$(date +%Y-%m-%d-%H%M)-<short-description>

# 3. Return to main branch for changes
git checkout main

# 4. NOW proceed with your commits and pushes
git add .
git commit -m "Your commit message"
git push origin main
```

**Why this matters:**
- Preserves known working state before changes
- Protects against sync conflicts when pulling from upstream fork
- Provides rollback points if issues arise
- Prevents loss of custom modifications during upstream merges

**Backup branch naming convention:**
- `backup-` prefix for easy identification
- `YYYY-MM-DD-HHMM` timestamp for chronological ordering
- Short descriptive suffix (e.g., `auth-fix`, `ui-update`, `config-change`)

## Build and Development Commands

### Building Geary
```bash
# Configure and build (first time)
meson setup build
ninja -C build

# Subsequent builds
ninja -C build

# Build with specific profile (development/beta/release)
meson setup build -Dprofile=development
```

### Running Geary
```bash
# Run directly from build directory (no installation needed)
./build/src/geary

# Run console tool for debugging
./build/src/geary-console

# Run mailer utility
./build/src/geary-mailer
```

### Testing
```bash
# Run all tests
meson test -C build

# Run specific test suite
meson test -C build engine-tests
meson test -C build client-tests

# Run with verbose output
meson test -C build --verbose
```

### Code Quality
```bash
# No built-in linting commands - Vala compiler handles validation during build
# Rebuild to check for compilation errors
ninja -C build
```

## Architecture Overview

Geary is a GTK3 email client written in Vala, organized into distinct architectural layers:

### Core Components

**Engine (`src/engine/`)**: Backend email handling
- `imap/` and `imap-engine/`: IMAP protocol and integration
- `smtp/`: SMTP for sending emails
- `db/` and `imap-db/`: SQLite-based storage with FTS search
- `rfc822/`: Email parsing and formatting
- `api/`: Public API interfaces

**Client (`src/client/`)**: GTK user interface
- `application/`: Main app window and lifecycle
- `conversation-viewer/`: Email reading interface
- `composer/`: Email composition
- `conversation-list/`: Email list display
- `plugin/`: Plugin system with built-in extensions

**Console (`src/console/`)**: CLI debugging tool

**Mailer (`src/mailer/`)**: Standalone email sender

### Key Design Patterns

1. **Conversation-based**: Emails grouped by conversation threads, not individual messages
2. **Asynchronous operations**: Uses Vala's async/yield for non-blocking I/O
3. **WebKit integration**: HTML emails rendered via WebKitGTK
4. **Plugin architecture**: LibPeas-based extensibility
5. **Database-backed**: SQLite with FTS3/FTS5 for local caching and search

### Important Implementation Details

- **Language**: Vala compiles to C, uses GObject type system
- **UI files**: GTK Builder XML in `ui/` directory
- **Resources**: Bundled via GResource system
- **Dark mode**: Uses DarkReader JavaScript library (`ui/darkreader.js`)
- **Email storage**: SQLite database per account in user data directory
- **Threading**: GLib main loop with async operations
- **Memory management**: Reference counting via GObject

### Development Considerations

When modifying Geary:
- Engine changes require understanding IMAP/SMTP protocols
- UI changes involve GTK3 and possibly WebKitGTK for email display
- Database schema changes need migration scripts in `sql/`
- Vala code follows GObject conventions (properties, signals, interfaces)
- Client-engine separation is strict - no direct database access from UI
- Conversation threading logic is complex and central to UX
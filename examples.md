---
layout: default
title: Real-World CLAUDE.md Examples
lang: en
---

# Real-World CLAUDE.md Examples

[Back to Guide](README.md)

These are real CLAUDE.md files from a production setup — anonymized but otherwise unedited. They cover a global config and six different project types, from iOS native to creative writing. Use them as starting points, not templates — the best CLAUDE.md files are shaped by your specific project, not copied from someone else's.

---

## Table of Contents

| Example | Stack | Lines | What Makes It Interesting |
|---------|-------|-------|--------------------------|
| [Global CLAUDE.md](#global-claudemd) | Cross-project | ~160 | Tool selection, orchestration model, memory routing, verification |
| [iOS Native App](#ios-native-app-swiftui--swiftdata) | SwiftUI + SwiftData | ~380 | Domain-specific rules, SwiftLint integration, pattern-counter sync |
| [Mobile Cross-Platform](#mobile-cross-platform-react-native--expo) | React Native + Expo + Supabase | ~95 | Clean Architecture (DDD), Result pattern, auth flows |
| [Backend API](#backend-api-nestjs--typeorm) | NestJS + TypeORM + PostgreSQL | ~320 | Layered architecture, migration workflows, memory guidelines |
| [CLI Tool](#cli-tool-typescript) | TypeScript + Vitest | ~90 | Verification commands, publishing workflow, gotchas |
| [Simple Website](#simple-website-vite--typescript) | Vite + TypeScript | ~30 | The minimal effective config |
| [Creative Writing](#creative-writing-project) | VitePress + TypeScript | ~100 | Non-code use case, narrative rules, agentic framework |

---

## Global CLAUDE.md

This is the file at `~/.claude/CLAUDE.md`. It applies to every session regardless of project. Notice how it focuses on **cross-cutting concerns** — tool selection, memory routing, verification, orchestration — and defers project-specific details to project CLAUDE.md files.

**~160 lines. Sections: Tools, Tool Selection, Memory Routing, Git, Context Hygiene, File Discipline, Code Quality, Verification, Orchestration Model, Active Projects, Session Start Checklist.**

```markdown
# Global Claude Code Instructions

## Tools & MCP Servers

| Tool | Purpose | Activation |
|------|---------|------------|
| **Graphiti** | Long-term knowledge graph memory | `search_memory_facts` at start; `add_memory` at end |
| **MCP_DOCKER** | Docker MCP catalog + GitHub MCP (40 tools) | `ToolSearch(query="+MCP_DOCKER <keyword>")` |
| **Atlassian** | Jira + Confluence (native OAuth, pre-installed) | `ToolSearch(query="jira")` |
| **JetBrains** | IDE builds, tests, inspections, refactoring | Available when IDE open |
| **Serena** | Token-efficient code navigation + editing | `activate_project("<name>")` from project CLAUDE.md |
| **Context7** | Up-to-date library docs | Call before writing integration code |
| **TypeScript LSP** | Type checking, completions, diagnostics | Active for TS/JS projects |

**Graphiti rules:** Always use project `group_id` from its CLAUDE.md. Store: arch decisions,
patterns, bug fixes, trade-offs. Skip: routine changes, secrets, verbose code dumps.

**Serena workflow:** `list_dir` -> `get_symbols_overview` -> `find_symbol(include_body=true)`.
Use `find_referencing_symbols` before any breaking change.
Edit via `replace_symbol_body`, `insert_after/before_symbol`.

### Tool Selection (Serena + Ripgrep + LSP + Built-ins)

| Task | Best Tool | Why |
|------|-----------|-----|
| Find a symbol/class/function | Serena `find_symbol` | Semantic, language-aware |
| Understand file structure | Serena `get_symbols_overview` | Token-efficient overview |
| Check references before refactoring | Serena `find_referencing_symbols` | Catches all usages semantically |
| Read a known file | `Read` | Direct, no overhead — use Serena only when you need structure |
| Search string literals, config values, error messages | `Grep` (ripgrep) | Text patterns Serena can't parse as symbols |
| Search non-code files (markdown, JSON, YAML, configs) | `Grep` (ripgrep) | Serena only indexes code symbols |
| Find files by name/pattern | `Glob` | Faster than Serena `find_file` for globs |
| Replace a function/method body | Serena `replace_symbol_body` | Precise, symbol-level |
| Small code fix (line-level tweak) | `Edit` | Faster than Serena for surgical changes |
| Edit config/non-code files | `Edit` | Serena doesn't handle non-code |
| Type errors and diagnostics | TypeScript LSP / Swift LSP | Real-time type checking without running build |
| Rename across codebase | Serena `rename_symbol` | Semantic rename, not text find-replace |
| Library/framework API docs | Context7 | Up-to-date docs, not training data |
| Run build, test, lint | `Bash` | Verification commands, scripts, CLI tools |
| Recall past decisions, patterns, context | Graphiti `search_memory_facts` | Cross-session memory |
| Quick project facts (IDs, status, keys) | MEMORY.md (auto-loaded) | Already in context — no tool call needed |

**Rule of thumb:** Serena for code symbols, Grep for text patterns and non-code, Glob for
file discovery.

---

## Memory Routing

| | MEMORY.md | Graphiti | Serena |
|-|-----------|----------|--------|
| Auto-loaded at start | Yes | — search manually | — activate project |
| Quick facts / identifiers | Yes | — | — |
| Deep decisions (WHY) | summary | Yes full | — |
| Code navigation (WHERE) | — | — | Yes |
| Persists across sessions | Yes | Yes | Yes |

---

## Git

**Commits:** `<type>(<scope>): <description>` (50 char max subject). Body: blank line + bullet
list, 72 char wrap. Types: `feat fix docs style refactor perf test build ci chore revert`.

**Rules:** `git commit` auto-approved. `git push` + PR creation require confirmation. Never
commit to `main`/`master`. No AI attribution in commits.

**CodeRabbit PR reviews:** Address all comments (including "nice to have" enhancements). After
fixing, resolve each review thread via GitHub GraphQL `resolveReviewThread` mutation. Then post
`@coderabbitai resolve` to clear outstanding comments, followed by `@coderabbitai full review`
to trigger a fresh review.

---

## Context Hygiene

- **Compaction awareness.** Re-read any file before editing if significant work has happened
  since the last read. Auto-compaction discards earlier file reads.
- **File read cap.** Each file read returns max 2,000 lines. Use `offset` and `limit` for
  large files.
- **Tool result truncation.** Large tool results are silently truncated. Re-run with narrower
  scope if results look suspicious.
- **Task tracking.** Use `TaskCreate` for multi-step work — tasks survive compaction.

---

## File Discipline

- **One class/module per file.** Inner/local types co-located with their parent are fine.
- **500-line cap.** Files exceeding 500 LOC should be flagged for splitting.

---

## Code Quality

- Read before modifying. Keep changes minimal and focused. Run linter after edits.
- Comments explain *why*, not *what*. DRY (abstract after 3+). SOLID principles.
- Strong typing — no `any`. Error handling — never swallow, log with context.

---

## Verification

### Per-Step (after each logical task or subtask)

1. Build / type-check (e.g. `tsc --noEmit`, `xcodebuild`)
2. Linter (e.g. `eslint . --quiet`, `swiftlint`)
3. Run unit tests relevant to the changed code

Sub-agents follow the same rules — never report work as done without verification evidence.

### Before PR

- Full unit test suite must pass
- All per-step checks clean

### TDD Workflow

Write test first -> write implementation -> verify tests pass -> move forward.

---

## Orchestration Model

The main session is the **orchestrator** — it plans, delegates, validates, and reviews.
Implementation work is delegated to sub-agents.

| Role | Model | Use When |
|------|-------|----------|
| Orchestrator (main session) | `opus` | Always — planning, review, validation, coordination |
| Implementation agent | `sonnet` | Writing code, tests, standard refactors |
| Exploration agent | `haiku` | Broad codebase search, quick lookups, file discovery |
| Critical work (sub-agent) | `opus` | When complexity or impact demands getting it right |

### Sub-Agent Rules

- **Parallel by default.** For tasks touching >5 independent files, launch parallel sub-agents.
- **Tool-aware.** Sub-agents follow the same tool selection rules as the main session.
- **Self-verifying.** Sub-agents MUST build, type-check, lint, and test before reporting done.
- **Orchestrator double-checks.** Main session reviews sub-agent results independently.

---

## Active Projects

| Project | Description | Jira |
|---------|-------------|------|
| **my-framework** | Modular AI-assisted dev framework | `MF` |
| **my-novel** | Fantasy novel project | `MN` |
| **knitting-app** | iOS knitting/crochet counter (SwiftUI) | `KA` |
| **couples-app** | Couples appreciation app (React Native) | `CA` |
| **marketing-site** | Landing page (Vite + TS) | — |

---

## Session Start Checklist

1. Read project CLAUDE.md -> get `group_id` (Graphiti) + `activate_project` name (Serena)
2. `search_memory_facts` in Graphiti for relevant context
3. `activate_project` in Serena
4. Check MEMORY.md (auto-loaded) for quick facts
```

**Why this works:**
- The tool selection table is the highest-leverage section — it changes how Claude works in every session
- Memory routing table prevents "where do I save this?" confusion
- Orchestration model + sub-agent rules prevent the most common delegation failures
- Context hygiene catches the silent killers of long sessions
- Active projects table gives Claude quick orientation
- Everything else (code style, git format, verification) is short and opinionated

---

## iOS Native App (SwiftUI + SwiftData)

A production iOS app with SwiftData persistence, widgets, Siri integration, and pattern-guided counters. This is a detailed CLAUDE.md that shows how domain-specific rules pay off — the SwiftData pitfalls section alone probably saves 30 minutes per session.

**~380 lines. Sections: Project Status, Quick Start, Architecture, Code Standards, Testing, Common Pitfalls, Pattern Support, SwiftLint, Development Workflow, External Documentation.**

```markdown
# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project Status

| Metric | Status |
|--------|--------|
| iOS Modernization | Complete |
| Test Coverage | ~70% |
| Deployment Target | iOS 17.0 |
| Widget Support | 5 sizes |
| Siri Integration | Complete |

**See**: `/CHANGELOG.md` for history, `docs/roadmap/future-enhancements.md` for roadmap.

---

## Quick Start

```bash
# Build
xcodebuild -project MyApp.xcodeproj -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 15' build

# Test
xcodebuild test -project MyApp.xcodeproj -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 15'

# Lint
swiftlint && echo "No violations"
```

---

## Architecture

### Data Models (SwiftData)

**Core Models:**
- `Project` -> root with sections, photos, optional pattern
- `ProjectSection` -> section with counter, optional pattern link
- `Counter` -> value, mode (`.simple`/`.complex`/`.patternGuided`), undo steps
- `RowMarker` -> bookmarks with notes
- `ProjectPhoto` -> references to App Group stored photos

**Pattern Models (SchemaV4):**
- `PatternTemplate` -> pattern with sections, glossary, attachments
- `PatternSection` -> groups ordered instructions
- `PatternInstruction` -> text, row ranges, completion state

### Key Files

| Purpose | Location |
|---------|----------|
| Entry point | `MyApp.swift` |
| Project list | `Views/ProjectListScreen.swift` |
| Counter UI | `Views/CounterView.swift` |
| Pattern reader | `Views/Pattern/PatternDetailView.swift` |
| Models | `Persistance/Models/` |
| Migrations | `Persistance/Migrations/` |
| Shared container | `Persistance/SharedContainer.swift` |
| Components | `Views/Components/` |
| Widget | `Counter.Widget/` |

### Persistence

- SwiftData in **shared App Group** (`group.com.myapp`)
- Photos stored as JPEGs in `ProjectPhotos/` subdirectory
- Current schema: **SchemaV4** (pattern support)
- Always call `WidgetCenter.shared.reloadTimelines()` after data changes

---

## Code Standards

### Critical Rules

| Rule | Enforcement |
|------|-------------|
| No `print()` | Use `Logger.category` |
| No `try?` in production | Use `do-catch` with logging |
| No force unwraps (`!`) | Use `guard let`/`if let` |
| No `@State` for SwiftData | Use `@Bindable` |
| File headers required | Every Swift file |
| View body < 50 lines | Extract subviews |

### Logging

```swift
import os.Logger

// Categories: persistence, haptics, photos, widget, pattern, counter, project
Logger.persistence.error("Failed to save: \(error)")
Logger.counter.debug("Progress: \(progress)")
```

### Error Handling

```swift
// CORRECT
do {
    try modelContext.save()
} catch {
    Logger.persistence.error("Failed: \(error)")
    showErrorAlert = true
}

// WRONG
try? modelContext.save()
```

### File Header Template

```swift
// FileName.swift
// Brief description (1-2 sentences)
//
// Key features:
// - Feature 1
// - Feature 2
```

---

## Testing

### TDD Workflow

1. **RED**: Write failing test first
2. **GREEN**: Minimum code to pass
3. **REFACTOR**: Improve without breaking tests

### Test Template

```swift
@Test("Description of behavior")
func testBehavior() async throws {
    let config = ModelConfiguration(isStoredInMemoryOnly: true)
    let container = try ModelContainer(for: MyModel.self, configurations: config)
    let context = ModelContext(container)

    // Arrange
    let entity = MyModel(...)
    context.insert(entity)
    try context.save()

    // Act
    let result = entity.performAction()

    // Assert
    #expect(result == expected)
}
```

### Coverage Targets

| Component | Target |
|-----------|--------|
| ViewModels | 90% |
| Services | 80% |
| Business Logic | 90% |
| SwiftData Models | 70% |

---

## Common Pitfalls

### SwiftUI

| Pitfall | Fix |
|---------|-----|
| `@State` for SwiftData | Use `@Bindable` |
| View body > 50 lines | Extract subviews |
| Business logic in views | Extract to ViewModel |
| Force unwrap in views | Use optional chaining |
| Missing `.onChange` cleanup | Use `.task` |

### SwiftData

| Pitfall | Fix |
|---------|-----|
| Forgetting `save()` | Always call after mutations |
| New property without default | Add default value (SwiftData handles automatically) |
| N+1 queries | Use predicates, eager loading |

> **Note**: SwiftData handles additive schema changes (new models/properties with defaults)
> automatically. No explicit migration plan needed for these.

### Widget

| Pitfall | Fix |
|---------|-----|
| Stale widget data | Call `WidgetCenter.shared.reloadTimelines()` |
| Missing App Group | Use `SharedContainer` |
| Widget crashes | Add models to widget target |

### Concurrency

| Pitfall | Fix |
|---------|-----|
| `DispatchQueue` | Use `async/await` |
| UI not on main thread | Add `@MainActor` |
| Sync I/O on main | Use async operations |

---

## Pattern Support

### Counter Modes
- `.simple` -> Basic single counter
- `.complex` -> Multi-section counter
- `.patternGuided` -> Synced with pattern instructions

### Pattern-Counter Sync
When mode is `.patternGuided`:
- Increment -> marks current instruction complete
- Decrement -> unmarks next instruction
- Counter value = instruction index (1-based)

### Key Services
- `SectionProgressTracker` -> read-only progress computation
- `SectionProgressSynchronizer` -> counter-instruction sync
- `PatternTemplateSeeder` -> loads bundled JSON patterns

---

## SwiftLint

### Zero Tolerance (Build Fails)
- Unused variables/functions
- Force unwrap/cast/try
- Deprecated API usage
- `print()` statements
- Missing file headers
- Line length > 150
- Function body > 80 lines

### Running

```bash
swiftlint          # Check
swiftlint --fix    # Auto-fix
```

---

## Development Workflow

### Before Commit
1. Run tests
2. Run `swiftlint`
3. Verify file headers present
4. Check no `print()` statements

### Adding SwiftData Model
1. Create with `@Model` macro
2. Add to `SharedContainer.build()`
3. Ensure new properties have default values (for automatic migration)
4. Write tests

### Adding View
1. Add file header
2. Use `@Bindable` for models
3. Keep body < 50 lines
4. Add accessibility labels

---

## Tool Integration

- **Graphiti:** `group_id="my-knitting-app"`
- **Serena:** `activate_project("knitting-app")` at session start
```

**Why this works:**
- The pitfalls tables are gold — Claude hits these exact issues and the tables prevent them
- Counter modes and pattern-counter sync explain domain logic Claude couldn't guess
- SwiftLint zero-tolerance rules prevent back-and-forth ("oh, you need a file header")
- The "Adding SwiftData Model" checklist catches the steps Claude would otherwise miss
- Verification is baked into workflow, not a separate afterthought

---

## Mobile Cross-Platform (React Native + Expo)

A couples appreciation app using Clean Architecture (DDD). Notice how the architecture section is the star — when your codebase has a specific pattern (like Result\<T\>), documenting it explicitly saves endless confusion.

**~95 lines. Sections: Architecture, Tech Stack, Critical Pattern, Data Model, Auth Flow, Key Screens, Testing, Tool Integration.**

```markdown
# Couples App

Couples appreciation app — write notes about what you value your partner doing for you.

## Architecture

**Clean Architecture (DDD).** Inner layers never import from outer layers.
Dependencies point inward.

```text
app/
├── src/
│   ├── domain/           # Business entities & logic (no framework deps)
│   │   ├── entities/     # Note, Connection, User, Reaction
│   │   ├── value-objects/ # NoteStatus, ConnectionStatus, SubscriptionTier
│   │   └── repositories/ # INoteRepository, IConnectionRepository (contracts)
│   ├── application/      # Use cases & orchestration
│   │   ├── use-cases/    # CreateNote, SendNote, PairConnection, etc.
│   │   ├── services/     # DataSyncService, GuestUserService, NotificationService
│   │   └── di/           # Dependency injection container
│   ├── infrastructure/   # External services
│   │   ├── repositories/ # AsyncStorage + Supabase implementations
│   │   ├── supabase/     # Supabase client setup
│   │   ├── push/         # Expo Push Notifications
│   │   └── storage/      # AsyncStorage helpers
│   ├── presentation/     # UI layer
│   │   ├── screens/      # Connections, Journal, Gifts, Compose, Settings, Activity
│   │   ├── components/   # NoteCard, Envelope, Polaroid, ReactionBar, EmojiPicker
│   │   ├── hooks/        # useAuth, useNotes, useConnections, useGifts
│   │   ├── navigation/   # Expo Router layout files
│   │   └── stores/       # Zustand (authStore, connectionStore, noteStore)
│   └── shared/           # Cross-cutting concerns
│       ├── types/        # Result<T>, common types
│       ├── constants/    # App constants, color palette, typography
│       └── utils/        # Formatting, date helpers
├── supabase/
│   ├── migrations/       # SQL migrations
│   └── functions/        # Edge functions
└── e2e/                  # E2E tests
```

## Tech Stack

TypeScript (strict) + React Native + Expo SDK 52+ + Expo Router + Supabase
(PostgreSQL, Auth, Edge Functions, Storage) + Expo Push Notifications + Zustand +
React Native Reanimated + react-native-skia + Vitest.

## Critical Pattern: Result<T>

Domain entities use Result pattern. Access value with `.value!` (NOT `getValue()`):

```typescript
const result = Note.create(props);
if (result.isSuccess) {
  const note = result.value!;
}
```

## Data Model

19 entities. Core: `users`, `connections`, `notes`, `note_photos`, `reactions`,
`subscriptions`. See design spec for full schema.

## Auth Flow

Guest mode by default (AsyncStorage, no signup). Optional cloud save via magic link
email. Guest data migrates to Supabase on auth. Solo connections upgrade to paired
on invite acceptance.

## Key Screens

| Screen | Purpose | Auth |
|--------|---------|------|
| Connections (Home tab) | List of connections | No (Guest) |
| Journal | Your notes about a person | No (Guest) |
| Gifts | Notes from your partner, envelope animation | Yes (Paired) |
| Compose | Write a new note | No (Guest = draft) |
| Activity (tab) | Reaction feed | Yes |
| Settings (tab) | Profile, notifications, themes, subscription | Yes |

## Testing

| Type | Location | Command |
|------|----------|---------|
| Unit (Vitest) | `app/src/**/*.test.ts(x)` | `cd app && npm test` |

## Tool Integration

- **Graphiti:** `group_id="my-couples-app"`
- **Serena:** `activate_project("couples-app")` at session start

## Reference Docs

| Doc | Content |
|-----|---------|
| VISION.md | Product vision and north star |
| Design Spec | Full design specification |
| Implementation Plan | Epic-by-epic build plan |
```

**Why this works:**
- The architecture tree is worth 1,000 words — Claude immediately knows where things go
- The `Result<T>` pattern section prevents the most common DDD mistake (using `getValue()`)
- Auth flow is short but covers the non-obvious transitions (guest -> cloud, solo -> paired)
- Key screens table gives Claude enough context to navigate without reading every file

---

## Backend API (NestJS + TypeORM)

A production NestJS backend with layered architecture. This one is intentionally thorough — it covers not just "what the code looks like" but "how to add things to it." The common tasks section is especially valuable for backends where the workflow (generate migration, run it, update module) has multiple steps.

**~320 lines. Sections: Project Identity, Overview, Tech Stack, Structure, Commands, Architecture Details, Code Standards, Memory Guidelines, Common Tasks.**

```markdown
# Backend API Project Context

## Project Identity

- **Graphiti group_id:** `my-backend`
- **Project Type:** Backend API (NestJS)
- **Primary Language:** TypeScript
- **Framework:** NestJS
- **Serena:** Run `activate_project("my-backend")` at session start

## Overview

RESTful API built with NestJS following domain-driven design principles. Provides data
services for the frontend with PostgreSQL database integration via TypeORM.

**Architecture Pattern:** Layered architecture with Controllers, Services, Repositories,
and Entities. NestJS dependency injection and modular structure.

## Tech Stack

- **Framework:** NestJS 10+
- **Runtime:** Node.js 18+
- **ORM:** TypeORM with PostgreSQL
- **Database:** PostgreSQL 14+
- **Testing:** Jest (unit/integration), Supertest (E2E)
- **Validation:** class-validator, class-transformer
- **Documentation:** Swagger/OpenAPI

## Project Structure

```text
src/
  main.ts                    # Application entry point
  app.module.ts              # Root module

  modules/                   # Feature modules
    analysis/                # Analysis module
      controllers/           # HTTP endpoints
      services/              # Business logic
      repositories/          # Data access layer
      entities/              # TypeORM entities
      dto/                   # Data transfer objects
      analysis.module.ts

    common/                  # Shared module
      decorators/
      filters/
      guards/
      interceptors/
      pipes/

  config/                    # Configuration files
    database.config.ts
    app.config.ts

  migrations/                # TypeORM migrations

test/                        # E2E tests
```

## Key Commands

**Development:**

```bash
npm run start              # Start in production mode
npm run start:dev          # Start with hot-reload
npm run start:debug        # Start in debug mode (port 9229)
```

**Database:**

```bash
npm run migration:generate -- -n MigrationName    # Generate migration
npm run migration:run      # Run pending migrations
npm run migration:revert   # Rollback last migration
```

**Testing:**

```bash
npm run test               # Run unit tests
npm run test:cov           # Generate coverage report
npm run test:e2e           # Run E2E tests
```

**Linting:**

```bash
npm run lint               # ESLint check
npm run format             # Prettier format
```

## Architecture Details

### Layered Architecture

**Controllers:** Handle HTTP requests, validate input using DTOs with class-validator,
delegate business logic to services.

**Services:** Business logic, orchestrate data access via repositories, handle
transactions and error handling. Injectable via NestJS DI.

**Repositories:** Data access layer. Use TypeORM QueryBuilder for complex queries.

**Entities:** TypeORM entities mapping to database tables. Define relationships,
validation constraints, column definitions.

**DTOs:** Request validation (`CreateXDto`, `UpdateXDto`), response serialization.
Use class-validator decorators. Separate DTOs per operation.

### Error Handling

- Use NestJS built-in exceptions (`HttpException`, `BadRequestException`, etc.)
- Global exception filters for consistent error responses
- Validation pipes for automatic DTO validation
- Logging with NestJS Logger

## Code Standards

### NestJS
- One module per feature/domain
- Controllers handle routing, services handle logic
- DTOs for all request/response data

### TypeScript
- Strict mode. Explicit return types for public methods.
- Avoid `any`. Use interfaces for contracts.

### Database
- Always use migrations (never sync in production)
- Use transactions for multi-step operations
- Index frequently queried columns
- Avoid N+1 queries (use relations/joins)

### Testing
- Unit tests for services (mock dependencies)
- Integration tests for database operations
- E2E tests for API endpoints
- TDD for new features. Aim for 80%+ coverage.

## Common Tasks

### Creating a New Module
1. Generate module: `nest g module modules/feature-name`
2. Generate controller: `nest g controller modules/feature-name`
3. Generate service: `nest g service modules/feature-name`
4. Create entity in `entities/` directory
5. Create DTOs in `dto/` directory
6. Register in `app.module.ts` if needed
7. Write unit tests for service
8. Write E2E tests for endpoints

### Creating a Database Entity
1. Create entity class with TypeORM decorators
2. Define columns, relationships, constraints
3. Generate migration: `npm run migration:generate -- -n CreateFeatureTable`
4. Review generated migration (never trust auto-generated SQL blindly)
5. Run migration: `npm run migration:run`
6. Create repository if custom queries needed

### Running Tests
- Via CLI: `npm run test`
- E2E tests: `npm run test:e2e`
- Debug tests: `npm run test:debug` (port 9229)

## Notes

- Always run migrations in order (never skip or reorder)
- Use transactions for operations affecting multiple tables
- Keep services thin — delegate to repositories for data access
- Use DTOs to validate all incoming data
- Never commit `.env` file
- Swagger docs auto-generated at `/api` endpoint
```

**Why this works:**
- Common Tasks sections are a backend-specific superpower — "Creating a New Module" is 8 steps that Claude would otherwise guess at (and get wrong)
- The architecture summary is compact but explains the *purpose* of each layer, not just the name
- Database commands are listed because they're easy to forget and dangerous to get wrong
- Notes section catches the non-obvious constraints (migration order, transaction requirements)

---

## CLI Tool (TypeScript)

A TypeScript CLI tool with Vitest tests and automated npm publishing. This shows a mid-size project config — more than minimal, but focused on the things that actually matter for a CLI project.

**~90 lines. Sections: Tech Stack, Commands, Verification, Structure, Publishing, Local Dev, Gotchas.**

```markdown
# Config Sync Tool

CLI tool that stores Claude Code configuration in a git repo and syncs between machines.

## Tech Stack

- TypeScript (strict mode, ESM)
- Node.js built-ins for file operations
- Vitest for testing
- ESLint + Prettier for linting/formatting
- Husky + lint-staged for pre-commit/pre-push hooks

## Development Scripts

```bash
npm run build          # Compile TypeScript
npm run dev            # Watch mode
npm test               # Run Vitest tests
npm run lint           # ESLint check
npm run format         # Prettier format
```

## Verification Commands

```bash
npm run build          # Must build before linking
npx tsc --noEmit       # Type-check only
npx eslint . --quiet   # Lint (also runs via pre-commit hook)
npm test               # Unit tests
```

## Repo Structure

```text
src/
  cli.ts              # Entry point, command registration
  cli-utils.ts        # Shared CLI helpers
  commands/           # One file per CLI subcommand
  __tests__/          # Vitest test files (mirror src/ structure)
  paths.ts            # File discovery
  files.ts            # File I/O utilities (copy, backup, chmod)
  diff.ts             # Unified diff generation
  filter.ts           # Filtering logic
  machine.ts          # Machine config loading
  types.ts            # Shared TypeScript types
dist/                 # Compiled JS output (gitignored)
scripts/              # Build helpers
.github/workflows/    # CI: tests on PR, npm publish on tag
```

## Publishing

npm publish is **automated via CI** — push a `v*` tag after bumping the version:

```bash
# 1. Bump version in package.json + add CHANGELOG entry, commit
# 2. Tag and push:
git tag v0.4.1 && git push origin v0.4.1
```

CI validates the tag matches `package.json` version, then runs typecheck -> test -> build ->
npm publish.

## Local Development / Testing

```bash
npm run build             # Must build before linking
npm link                  # Installs globally from dist/
my-tool --version         # Verify correct version is active
npm unlink my-tool        # Remove when done
```

## Gotchas

### Pre-push hook blocks on failures

Husky pre-push runs **typecheck -> test -> build** in sequence. A type error or failing test
blocks the push. Fix the root cause; never use `--no-verify`.

### ESM mocking of Node built-ins in Vitest

`vi.spyOn` cannot mock properties on native ESM modules (`node:fs`, etc.). Use the factory
overload instead:

```typescript
vi.mock("node:fs", async (importOriginal) => {
  const actual = await importOriginal<typeof import("node:fs")>();
  return { ...actual, existsSync: vi.fn(), readdirSync: vi.fn() };
});
vi.mocked(existsSync).mockReturnValue(false);
```

## Tool Integration

- **Graphiti:** `group_id="my-config-sync"`
- **Serena:** `activate_project("config-sync")` at session start
```

**Why this works:**
- Gotchas section is the MVP — ESM mocking with Vitest is a trap Claude hits repeatedly
- Publishing workflow is documented because it's a multi-step process with a CI dependency
- Structure is shown because CLI tools have a non-obvious layout (commands/, one file per subcommand)
- Verification commands are separate from dev scripts — Claude needs to know what to run to check its work

---

## Simple Website (Vite + TypeScript)

A marketing landing page. This is the minimal effective CLAUDE.md — proof that not every project needs 300 lines. If your project is small and conventional, a short config works.

**~30 lines.**

```markdown
# Marketing Website

Marketing/landing page for the mobile app.

## Tech Stack

TypeScript + Vite + (see `package.json` for full dependencies).

## Development

```bash
npm install
npm run dev       # Dev server
npm run build     # Production build
```

## Verification Commands

```bash
npm run build          # Build (includes type-check)
npx tsc --noEmit       # Type-check only
npx eslint . --quiet   # Lint
```

## Tool Integration

- **Graphiti:** `group_id="my-marketing-site"`
- **Serena:** `activate_project("marketing-site")` at session start
```

**Why this works:**
- It's short because the project is simple. Don't pad a simple project with rules it doesn't need.
- Verification commands are always worth including, even in a 30-line config
- Tool integration section ensures Claude activates the right tools every session
- The global CLAUDE.md handles everything else (code quality, git format, orchestration)

---

## Creative Writing Project

A fantasy novel project that uses an agentic framework for AI-assisted writing. This shows that CLAUDE.md isn't just for code — it works for any structured project where Claude needs domain context.

**~100 lines. Sections: Framework Modules, Agents, Book Project, Key Files, Commands, Writing Workflow, Narrative Approach, Tool Integration.**

```markdown
# Project Entry Point

Framework: Agentic Development Framework v1.0.0

## Installed Modules

- Framework Core (core)
- Fantasy Book Writer (writer)

## Available Agents (Slash Commands)

- /ai-framework-manager
- /ai-book-writer
- /ai-book-writer-slim

## Quick Start

1. Fill in context files in `ai/context/`
2. Use slash commands to interact with agents
3. Add more modules with `npx agentic-framework add <module>`

## Navigation

- `ai/agents/` - Agent documentation
- `ai/skills/` - Skill documentation
- `ai/context/` - Project-specific context
- `ai/registries/` - Discovery metadata
- `book/` - All book content (manuscript, world, characters, gallery)

---

## Book Project: The Awakening

**Genre**: Arcanepunk coming-of-age fantasy
**Timeline**: 4 years (protagonist ages 13-17)
**Style**: Third-person omniscient, Tolkien-Zelazny synthesis (70/30)

### Key Files

| Purpose                     | File                                        |
| --------------------------- | ------------------------------------------- |
| Book metadata & synopsis    | `book/index.md`                             |
| Story structure             | `book/manuscript/index.md`                  |
| World state at book opening | `book/world/baseline/BASELINE.md`           |
| Character registry          | `book/characters/_index.md`                 |
| Prose style guidelines      | `.claude/skills/writer-style-framework/SKILL.md` |

### Development Commands

```bash
npm run docs:dev                     # Development preview (live reload)
npm run preview                      # Build and preview
npm run build:all                    # Build everything
npx agentic-framework writer build   # Build manuscript to single file
```

### Writing Workflow

1. Use `/ai-book-writer` for orchestrated writing tasks
2. Review PART.md files for chapter breakdowns
3. Follow time skip approach (episodic chapters, months between)
4. Apply power unlock trope (emotional trauma catalyzes abilities)
5. Commit with descriptive messages after each scene

### Narrative Approach

- **POV**: Protagonist (90%), supporting characters (10%)
- **Prologue**: Treaty signing (~100 years before main story)
- **Time Skips**: Narrator summary, character memory, dialogue, physical markers
- **Power Unlock**: Grief/trauma opens magical abilities (crystal amplifies emotional state)

## Tool Integration

- **Graphiti:** `group_id="my-novel"`
- **Serena:** `activate_project("my-novel")` at session start
```

**Why this works:**
- Narrative approach section gives Claude creative constraints that produce consistent output
- Writing workflow is step-by-step — Claude follows it instead of improvising
- Key files table prevents Claude from reading the entire `book/` directory to find things
- The framework integration shows how agents and skills extend Claude for non-code work
- Style reference is in a skill (`.claude/skills/`), not in CLAUDE.md — loaded on demand

---

## Patterns to Notice

Looking across all these examples, a few patterns emerge:

1. **Every project has Tool Integration.** Even the 30-line config specifies `group_id` and `activate_project`. This ensures Claude activates the right tools in every session.

2. **Verification commands are always explicit.** Even when they seem obvious (`npm test`), listing them means Claude doesn't have to guess or read `package.json`.

3. **Domain-specific rules are the highest value.** The iOS pitfalls table, the `Result<T>` pattern, the counter-instruction sync rules — these are things Claude can't figure out from reading the code. They prevent the most expensive mistakes.

4. **Architecture summaries use structure trees.** A directory tree with one-line descriptions per folder gives Claude more spatial understanding than paragraphs of prose.

5. **Common tasks are step-by-step.** "Creating a New Module" as 8 ordered steps works better than "follow NestJS conventions."

6. **Length matches project complexity.** A simple website gets 30 lines. An iOS app with widgets, patterns, and SwiftData gets 380. Don't pad short configs or compress complex ones.

7. **Global handles the universal; project handles the specific.** Code quality, git format, orchestration, memory routing — those go in global. Architecture, pitfalls, build commands, domain rules — those go in project.

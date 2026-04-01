---
layout: default
title: Real-World CLAUDE.md Examples
lang: en
---

# Real-World CLAUDE.md Examples

[Back to Guide](README.md)

These are real CLAUDE.md files from a production setup — anonymized but otherwise unedited. Use them as starting points, not templates — the best CLAUDE.md files are shaped by your specific project, not copied from someone else's.

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

## Patterns to Notice

Looking across these examples, a few patterns emerge:

1. **Global handles the universal; project handles the specific.** Code quality, git format, orchestration, memory routing — those go in global. Architecture, pitfalls, build commands, domain rules — those go in project.

2. **Domain-specific rules are the highest value.** The iOS pitfalls table, the pattern-counter sync rules, the SwiftLint zero-tolerance list — these are things Claude can't figure out from reading the code. They prevent the most expensive mistakes.

3. **Verification commands are always explicit.** Even when they seem obvious, listing them means Claude doesn't have to guess or read config files.

4. **Architecture summaries use structure trees.** A key files table with one-line descriptions gives Claude more spatial understanding than paragraphs of prose.

5. **Length matches project complexity.** A simple website might need 30 lines. An iOS app with widgets, patterns, and SwiftData needs 380. Don't pad short configs or compress complex ones.

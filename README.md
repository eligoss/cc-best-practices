# Claude Code Best Practices

**English** | [Русский](ru) | [Українська](uk) | [Español](es)

> **Who is this for?** Developers who already use Claude Code and want to go from "it works" to "this is genuinely 10x." Whether you've been using it for a week or six months, there's something here for you.

Most people install Claude Code, type some prompts, and get decent results. But there's a massive gap between using it casually and setting it up so that Claude actually *understands* your projects, remembers your decisions, and works the way you do. This guide bridges that gap.

We'll start with the building blocks — plugins, MCP servers, and commands — then work through the practices that compound over time: how to structure your instructions, when to use which tool, and how to make sessions feel less like starting from scratch every time.

---

## Table of Contents

| #  | Section                                                                | What You'll Learn                                                                   |
|----|------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| 1  | [The Extension Ecosystem](#1-the-extension-ecosystem)                  | Plugins, MCP servers, and slash commands — the three ways to extend Claude Code     |
| 2  | [CLAUDE.md Files](#2-claudemd--your-instructions-that-stick)           | Global vs project, path-specific rules, and `/init` — what goes where and why       |
| 3  | [Skills](#3-skills--load-knowledge-when-you-need-it)                   | Turn any project knowledge into on-demand context that doesn't bloat every session  |
| 4  | [Superpowers](#4-superpowers--the-plugin-that-changes-everything)      | Structured workflows that turn Claude from assistant to engineering partner         |
| 5  | [Serena](#5-serena--code-navigation-that-understands-your-code)        | Navigate and edit code by *meaning*, not just text search                           |
| 6  | [Ripgrep](#6-ripgrep--when-text-search-is-the-right-tool)              | Fast text search for everything Serena can't see                                    |
| 7  | [Tool Selection](#7-teaching-claude-when-to-use-what)                  | The single most impactful table you can add to your config                          |
| 8  | [Graphiti](#8-graphiti--memory-that-lasts-across-sessions)             | A knowledge graph so Claude remembers what you decided and why                      |
| 9  | [Memory Systems](#9-memory--the-right-information-in-the-right-place)  | Auto memory, MEMORY.md, Graphiti, CLAUDE.md — which goes where                      |
| 10 | [Context7](#10-context7--docs-that-are-actually-current)               | Live library documentation because your training data is already stale              |
| 11 | [Hooks and Guardrails](#11-hooks-and-guardrails--automate-and-protect) | Session automation, security hooks, and git hooks working together                  |
| 12 | [Permissions and Security](#12-permissions-and-security)               | Let Claude move fast without letting it do anything dangerous                       |
| 13 | [Subagents and Agent Teams](#13-subagents-and-agent-teams)             | The orchestration model, custom agents, model tiers, and multi-agent coordination   |
| 14 | [Context Window Management](#14-context-window-management)             | `/compact`, `/clear`, `/btw`, context hygiene, and keeping sessions focused         |
| 15 | [Git Worktrees](#15-git-worktrees--parallel-sessions)                  | Run multiple Claude sessions on the same repo without conflicts                     |
| 16 | [Git and Code Review](#16-git-and-code-review)                         | Conventional commits, per-step verification, CodeRabbit, and local subagent reviews |
| 17 | [Session Management](#17-session-management)                           | Resume, rename, and navigate sessions like a pro                                    |
| 18 | [Status Line](#18-status-line--see-whats-happening)                    | Real-time context usage, cost, and session stats at a glance                        |
| 19 | [Task Tracking](#19-task-tracking--dont-lose-your-place)               | Why in-session tasks matter more than you think                                     |
| 20 | [Common Failure Patterns](#20-common-failure-patterns--what-to-avoid)  | The mistakes everyone makes and how to break the cycle                              |
| 21 | [Config Sync](#21-config-sync--same-setup-everywhere)                  | Keep two machines in lockstep                                                       |

---

## 1. The Extension Ecosystem

Claude Code out of the box is good. Claude Code with the right extensions is a different experience entirely.

There are three ways to extend it, and they work together:

### Plugins — the big one

Plugins bundle everything: slash commands, specialized agents, hooks, and sometimes MCP tools. Think of them as "feature packs" for Claude Code.

```bash
# Browse the plugin marketplace
/plugins

# Or install directly
claude plugin install superpowers@claude-plugins-official
```

Here are the plugins worth installing right away — we'll go deeper into the important ones later:

| Plugin                   | Why You Want It                                                                                                         |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------|
| **superpowers**          | Planning, brainstorming, TDD, debugging, worktrees, code review — the structured workflows that make Claude disciplined |
| **claude-md-management** | AI-assisted CLAUDE.md writing and auditing                                                                              |
| **skill-creator**        | Turn any project knowledge into a reusable skill                                                                        |
| **serena**               | Semantic code navigation — find symbols, references, edit by meaning                                                    |
| **context7**             | Live library docs so Claude doesn't hallucinate outdated APIs                                                           |
| **code-review**          | PR review workflows with parallel subagent reviewers                                                                    |
| **coderabbit**           | [CodeRabbit](https://www.coderabbit.ai/) AI review integration                                                          |
| **feature-dev**          | Guided feature development with codebase exploration                                                                    |
| **playwright**           | Browser automation and testing                                                                                          |
| **plugin-dev**           | Tools for building your own plugins                                                                                     |

### MCP Servers — external tools

MCP (Model Context Protocol) servers connect Claude to external systems — your IDE, issue tracker, knowledge graph, monitoring tools. They provide raw tools that Claude calls directly.

```bash
claude mcp add <name> -- <command>
claude mcp list
```

Some commonly used ones:

| Server         | What It Does                                       |
|----------------|----------------------------------------------------|
| **Graphiti**   | Knowledge graph for persistent memory              |
| **JetBrains**  | Build, test, and refactor from your IDE            |
| **Atlassian**  | Jira + Confluence (often pre-installed via OAuth)  |
| **Sentry**     | Error tracking and issue investigation             |
| **Docker MCP** | A catalog of MCP servers you can install on demand |

### Slash Commands — the user interface

Slash commands are how you invoke skills from plugins. Type `/` and you'll see what's available:

```text
/commit              Create a git commit
/code-review         Review a pull request
/revise-claude-md    Update CLAUDE.md with session learnings
/skill-creator       Create a new skill from knowledge
/simplify            Review and simplify changed code
```

That's the quick tour. Now let's get into what actually makes the difference.

---

## 2. CLAUDE.md — Your Instructions That Stick

If you only do one thing from this guide, make it this: **write a good CLAUDE.md.**

Every time you start a session, Claude reads your CLAUDE.md files. They're your persistent instructions — the stuff you'd otherwise repeat in every conversation. Get the split between global and project right, and Claude starts every session already knowing how you work.

### Two files, two jobs

|                    | Global (`~/.claude/CLAUDE.md`) | Project (`<repo>/CLAUDE.md`) |
|--------------------|--------------------------------|------------------------------|
| **Loaded**         | Every session, every project   | Only in that project         |
| **Think of it as** | Your developer profile         | The project handbook         |
| **Contains**       | How *you* work                 | How *this codebase* works    |

### Your global file: the developer profile

This tells Claude who you are across all projects. Write it once, refine it over time:

- **What tools are available** and when to activate each one
- **How you like commits formatted** and your branching strategy
- **Your code quality bar** — DRY, SOLID, strong typing, whatever you care about
- **How thoroughly to verify work** before saying "done"
- **Which AI model for which task** — Haiku for lookups, Opus for architecture
- **Your active projects** — names, paths, Jira keys for quick reference

Here's what a good global CLAUDE.md skeleton looks like:

```markdown
# Global Claude Code Instructions

## Tools & MCP Servers
| Tool | Purpose | When to Activate |
|------|---------|-----------------|
| Serena | Code navigation | activate_project("<name>") at session start |
| Graphiti | Persistent memory | search_memory_facts at session start |
| Context7 | Library docs | Before writing integration code |

## Git
Commits: <type>(<scope>): <description>
Never commit to main. Never force push. PRs require confirmation.

## Code Quality
Read before modifying. Comments explain *why*. No `any` types.

## Subagents
- haiku — exploration, quick lookups
- sonnet — implementation, tests, code review
- opus — architecture, security-critical decisions
```

> **Pro tip:** Target roughly 100-200 lines. Boris Cherny (creator of Claude Code) has said "about 100 lines works better than 800." Each line should pass the test: *would removing this cause Claude to make mistakes?* If Claude does the right thing without a rule, delete the rule.

### Your project file: the codebase handbook

This is where project-specific knowledge lives. Think of it as the onboarding doc you'd hand a new team member:

- **Architecture** — data models, key files, how things connect
- **Build/test/lint commands** — the exact commands, no guessing
- **Code standards specific to this project** — "use `Logger`, not `print()`"
- **Common pitfalls** — the things that bite you in *this* codebase
- **Integration pointers** — Jira key, Graphiti `group_id`, Serena project name

> **What doesn't belong here:** Generic coding advice (that's global), tool activation instructions (also global), anything temporary (that's tasks or memory).

> **Want to see real examples?** The [Examples page](examples.md) has annotated, production CLAUDE.md files — a global config and a detailed iOS project — with analysis of why each section works.

### Starting from scratch? Use `/init`

If you're setting up CLAUDE.md for an existing project, don't start from a blank file. Claude Code's `/init` command explores your codebase — package files, config, directory structure, existing docs — and generates a starter CLAUDE.md automatically:

```bash
# Interactive multi-phase flow that explores before writing
CLAUDE_CODE_NEW_INIT=1 claude /init
```

It's a solid starting point. Then refine it over time with `/revise-claude-md` (see below).

### Path-specific rules: load the right context for the right files

Instead of loading all your rules all the time, you can scope rules to specific file patterns using `.claude/rules/`. This works both globally (`~/.claude/rules/`) and per-project (`.claude/rules/`).

Rules come in two flavors: **unconditional** (no `paths:` — always loaded alongside CLAUDE.md) and **path-scoped** (only loaded when Claude works on matching files):

```yaml
# ~/.claude/rules/git.md — unconditional, loaded every session
---
description: Git commit format and push rules
---

# Git Rules
## Commit Format
`<type>(<scope>): <description>` — 100 char max subject.
...
```

```yaml
# ~/.claude/rules/typescript.md — only loads for TS/TSX files
---
description: TypeScript coding rules
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# TypeScript Rules
- Enable `strict: true` in tsconfig.json.
- No `any` types. Use `unknown` for external data...
```

This is the right home for language rules (TypeScript, Swift), workflow rules (CodeRabbit loop, git conventions), context hygiene, and code quality standards. Each loads only when relevant — fixing a CSS bug won't load your TypeScript strict-mode rules. Same idea as skills (Section 3), but file-path-triggered rather than task-triggered.

### Don't write it alone

The **claude-md-management** plugin has two skills that do the heavy lifting:

```bash
claude plugin install claude-md-management@claude-plugins-official
```

| Command               | What It Does                                                                             |
|-----------------------|------------------------------------------------------------------------------------------|
| `/claude-md-improver` | Audits your existing CLAUDE.md for gaps, inconsistencies, and missed best practices      |
| `/revise-claude-md`   | After a working session, captures what you learned — new patterns, pitfalls, corrections |

**The habit that pays off:** Run `/revise-claude-md` at the end of any session where you discovered something about the project. Over time, your CLAUDE.md evolves from a stub into a comprehensive, battle-tested handbook — without you having to write it from scratch.

### CLAUDE.md inheritance (monorepos and nested directories)

Claude Code walks up the directory tree and loads every CLAUDE.md it finds:
- `~/.claude/CLAUDE.md` — your global profile
- `./CLAUDE.md` or `./.claude/CLAUDE.md` — project root (check into git for the team)
- Child directory CLAUDE.md files — lazily loaded when Claude works in those directories

In monorepos, this means the root CLAUDE.md covers shared conventions, and each package directory can have its own overrides. Use `claudeMdExcludes` in settings if other teams' files create noise.

---

## 3. Skills — Load Knowledge When You Need It

Here's a problem you'll run into quickly: your CLAUDE.md gets long. You keep adding rules — API conventions, testing patterns, database standards, deployment checklists — and eventually, every session loads 2,000 tokens of context whether it needs it or not.

Skills solve this. They're like CLAUDE.md sections that load **on demand** instead of always.

### The idea in 30 seconds

A skill is a markdown file with a description. When Claude starts a task, it checks: "Does any skill match what I'm about to do?" If yes, it loads that skill's content. If not, it stays out of the way.

```text
You: Fix the flaky test in UserService

Claude: [sees this is a testing task]
        [loads test-driven-development skill]
        [loads your project's testing-conventions skill]
        [works with full testing context — nothing else loaded]
```

### The before and after

**Before** — everything crammed into CLAUDE.md:

```markdown
## API Design Rules
- All endpoints return { data, error, meta }
- Use Zod for request validation
- Rate limit: 100 req/min per user
- Error codes follow RFC 7807
... (50 more lines)

## Database Conventions
- All tables have created_at, updated_at
- Use soft deletes
- Indexes on all foreign keys
... (40 more lines)
```

Every session loads all of this. Fixing a CSS bug? You're still carrying API and database rules in context.

**After** — lean CLAUDE.md, knowledge on demand:

```markdown
## Available Skills
- `/api-design` — API conventions, response format, error codes
- `/db-conventions` — Database patterns, migrations, indexes
```

Now the API context only loads when you're actually working on endpoints. Same knowledge, less waste, better focus.

### How to build a skill

You don't need to write skill files by hand. The **skill-creator** plugin does it:

```bash
claude plugin install skill-creator@claude-plugins-official
```

Then just tell it what you know:

```text
You: /skill-creator

Claude: What knowledge do you want to turn into a skill?

You: Our API always returns responses in this format: { data, error, meta }.
     We use Zod for all request validation. Rate limit is 100 req/min.
     Errors follow RFC 7807. All endpoints are versioned under /api/v1/...

Claude: [Creates a well-structured skill file with proper frontmatter,
         trigger conditions, and organized instructions]
```

You provide the knowledge, the plugin handles the formatting and trigger descriptions.

### What makes a skill file tick

```markdown
---
name: api-design
description: Use when creating or modifying API endpoints — covers response
  format, validation, error handling, and versioning conventions
---

## Response Format
All endpoints return:
{ "data": ..., "error": null | { ... }, "meta": { ... } }

## Validation
Use Zod schemas for all request bodies...
```

That `description` field is doing most of the work. It tells Claude *when* to load the skill, so write it like a trigger condition: "Use when [doing X] — covers [Y and Z]."

> **Rule of thumb:** If you've explained something to Claude more than twice across different sessions, it should be a skill.

---

## 4. Superpowers — The Plugin That Changes Everything

If you install one plugin from this guide, make it this one.

Here's the thing about Claude: it's *capable* of great engineering practices — planning before coding, writing tests first, debugging methodically. But left to its own devices, it tends to skip straight to writing code. We all do. The difference is that Superpowers makes these practices **automatic and enforced**, not aspirational.

```bash
claude plugin install superpowers@claude-plugins-official
```

### What changes after you install it

Superpowers adds a set of **process skills** that activate based on what you're doing:

| When you're...             | Superpowers activates...    | Which means Claude will...                                     |
|----------------------------|-----------------------------|----------------------------------------------------------------|
| Starting something new     | **Brainstorming**           | Present 2-3 approaches with trade-offs before writing any code |
| Ready to build             | **Writing Plans**           | Create a step-by-step implementation plan for your approval    |
| Building                   | **Executing Plans**         | Follow the plan step by step, with progress tracking           |
| Writing any feature or fix | **Test-Driven Development** | Write a failing test *first*, then the implementation          |
| Hitting a bug              | **Systematic Debugging**    | Form hypotheses, test them methodically, narrow down the cause |
| Finishing a chunk of work  | **Code Review**             | Review against the plan and project standards                  |
| About to say "done"        | **Verification**            | Run final checks before claiming completion                    |
| Facing independent tasks   | **Parallel Agents**         | Dispatch multiple agents simultaneously                        |
| Starting feature work      | **Git Worktrees**           | Isolate work in a dedicated worktree branch                    |
| Completing a branch        | **Finishing a Branch**      | Clean branch finalization and PR readiness                     |

### Why this is critically helpful

**Without Superpowers**, a typical session looks like:

```text
You: Implement user authentication with OAuth

Claude: Sure! Let me create the auth middleware...
        [writes 200 lines of code]
        [realizes it needs a different approach]
        [rewrites half of it]
        Done! I've implemented OAuth authentication.
        [no tests, no plan, no verification]
```

**With Superpowers:**

```text
You: Implement user authentication with OAuth

Claude: [Brainstorming activates]
        I see three approaches here:
        1. Passport.js — mature, lots of strategies, but heavy
        2. Auth.js (NextAuth) — built for Next.js, simpler
        3. Custom with jose — lightweight, full control
        Here are the trade-offs...

You: Option 2, let's go with Auth.js

Claude: [Planning activates]
        Here's the implementation plan:
        Step 1: Install dependencies and configure providers
        Step 2: Create auth API routes
        Step 3: Add session middleware
        Step 4: Protect routes with auth guards
        ...

You: Looks good, go

Claude: [Execution activates]
        Starting Step 1...
        [TDD activates] Writing test for OAuth provider config...
        [Test fails — good]
        [Writes implementation to pass the test]
        [Test passes]
        Step 1 complete. Moving to Step 2...
        ...
        [All steps done]
        [Verification activates]
        Running full test suite... All passing.
        Checking against plan... All steps covered.
        Done.
```

Same request, completely different outcome. The plan catches wrong turns early. The tests catch regressions. The verification prevents "it works on my machine" surprises.

### The planning workflow prevents the #1 time sink

The biggest time waster in AI-assisted coding isn't slow models or bad code — it's **building the wrong thing**. You say "add authentication," Claude interprets it one way, you wanted something different, and you've burned 20 minutes on code you'll throw away.

Superpowers' plan-then-execute workflow eliminates this. Claude shows you the plan *before* writing any code. You review it, adjust it, and only then say "go." The 2 minutes you spend reviewing a plan saves the 20 minutes you'd spend undoing the wrong approach.

### Thinking triggers for critical decisions

When you need Claude to think extra hard — architecture decisions, complex refactors, tricky debugging — you can push reasoning to the max:

- **`ultrathink`** — include this word in any prompt for maximum reasoning effort on that specific request
- **`/effort high`** — set a persistent effort level for the entire session
- **EXPLORE -> PLAN -> CODE -> COMMIT** — have Claude read files first without writing, then `ultrathink` on the plan, critique it for edge cases, *then* execute

These are especially powerful combined with Superpowers' planning workflow. Let Claude brainstorm at normal effort, then `ultrathink` on the final plan before execution.

---

## 5. Serena — Code Navigation That Understands Your Code

Most of the time, when Claude needs to understand your code, it does one of two things: reads the whole file, or runs a text search. Both work, but both are wasteful. Reading a 500-line file when you only need one function burns context. Grepping for a class name finds the definition but misses what's actually *using* it.

Serena changes this. It's a language-aware MCP server that understands your code as **symbols** — classes, functions, methods, variables — not just text. It knows what a function does, where it's defined, and everything that references it.

### What the difference feels like

| The old way                                            | With Serena                                                  |
|--------------------------------------------------------|--------------------------------------------------------------|
| `grep -r "class UserService"` across the whole project | `find_symbol("UserService")` — goes straight there           |
| Read the whole file to understand its structure        | `get_symbols_overview("user-service.ts")` — just the outline |
| Find-and-replace to rename something                   | `rename_symbol("oldName", "newName")` — safe, semantic       |
| Manually search for "who calls this?"                  | `find_referencing_symbols("UserService")` — complete answer  |

The token savings alone are significant. But the real win is accuracy — Serena finds the *definition*, not just string matches.

### Setting it up for a project

Two steps. First, create a Serena config in your project:

```yaml
# .serena/settings.yml
project_name: "my-project"
language_server:
  language: typescript  # or python, swift, go, etc.
  root_path: "src"
```

Second, tell Claude to activate it. In your project CLAUDE.md:

```markdown
## Serena (Semantic Code Analysis)
At session start, call `activate_project("my-project")`.
```

(If you set up a session start hook — covered in [Section 11](#11-hooks-and-guardrails--automate-and-protect) — this happens automatically.)

### The Serena workflow

The most efficient way to explore code with Serena follows a **zoom-in** pattern:

```text
list_dir                → What's in this directory?
get_symbols_overview    → What symbols are in this file? (no bodies — just names and signatures)
find_symbol             → Show me the full body of this one function
find_referencing_symbols → Who uses this? (before I change it)
```

And for editing:

```text
replace_symbol_body   → Rewrite a function body precisely
insert_before_symbol  → Add something above a symbol
insert_after_symbol   → Add something below a symbol
```

### Where Serena doesn't help

Serena only indexes **code symbols**. It can't see:
- Config files (JSON, YAML, TOML)
- Markdown and documentation
- String literals, error messages, comments
- Non-code files of any kind

For those, you need text search — which brings us to...

---

## 6. Ripgrep — When Text Search Is the Right Tool

Ripgrep (`rg`) is a fast text search tool, and Claude Code uses it under the hood for its `Grep` tool. By default, Claude uses a bundled version, but you can point it at your system install for better performance.

### Quick setup

```jsonc
// ~/.claude/settings.json
{
  "env": {
    "USE_BUILTIN_RIPGREP": "0"
  }
}
```

```bash
# Install if you haven't
brew install ripgrep        # macOS
sudo apt install ripgrep    # Ubuntu/Debian
```

### Serena vs Ripgrep — the split

These two tools complement each other perfectly, and teaching Claude when to use each one is one of the highest-value things you can do (more on this in [Section 7](#7-teaching-claude-when-to-use-what)):

| Reach for Ripgrep when you need   | Reach for Serena when you need         |
|-----------------------------------|----------------------------------------|
| String literals, error messages   | Function/class definitions             |
| Config values across many files   | Who references this symbol?            |
| Non-code files (docs, JSON, YAML) | The body of a specific method          |
| Regex patterns anywhere           | Safe symbol renaming                   |
| "Find every file mentioning X"    | "What does this file's API look like?" |

Neither tool replaces the other. Together, they cover everything.

---

## 7. Teaching Claude When to Use What

This is possibly the single highest-leverage thing in this entire guide, and it's just a table.

Claude has a lot of tools available — Serena, Ripgrep, Glob, Context7, Graphiti, the Edit tool, the Read tool. Without guidance, it falls back on its training defaults: read full files and grep for text. That's fine, but it's like using a hammer for everything when you also have a screwdriver and a drill.

### The table

Add this to your global CLAUDE.md. Seriously, just paste it in:

| Task                                                  | Best Tool                         | Why                                                           |
|-------------------------------------------------------|-----------------------------------|---------------------------------------------------------------|
| Find a symbol/class/function                          | Serena `find_symbol`              | Semantic, language-aware                                      |
| Understand file structure                             | Serena `get_symbols_overview`     | Token-efficient — just the outline                            |
| Check references before changing something            | Serena `find_referencing_symbols` | Complete — catches every usage                                |
| Read a known file                                     | `Read`                            | Direct, no overhead — use Serena only when you need structure |
| Search string literals, config values, error messages | `Grep` (ripgrep)                  | Text patterns that aren't code symbols                        |
| Search non-code files (markdown, JSON, YAML)          | `Grep` (ripgrep)                  | Serena only indexes code                                      |
| Find files by name or pattern                         | `Glob`                            | Faster than any search for file discovery                     |
| Replace a function/method body                        | Serena `replace_symbol_body`      | Precise, symbol-level replacement                             |
| Small code fix (line-level tweak)                     | `Edit`                            | Faster than Serena for surgical changes                       |
| Edit config or non-code files                         | `Edit`                            | Serena doesn't handle non-code                                |
| Type errors and diagnostics                           | TypeScript LSP / Swift LSP        | Real-time type checking without a full build                  |
| Rename across codebase                                | Serena `rename_symbol`            | Semantic rename, not text find-replace                        |
| Look up a library's current API                       | Context7                          | Real-time docs — training data may be stale                   |
| Run build, test, lint                                 | `Bash`                            | Verification commands and scripts                             |
| Recall a past decision or pattern                     | Graphiti `search_memory_facts`    | Cross-session knowledge graph                                 |
| Quick project facts (IDs, keys, status)               | MEMORY.md                         | Already in context — no tool call needed                      |

**The rule of thumb:** Serena for code symbols. Grep for text and non-code. Glob for files. Context7 for docs. Graphiti for memory. LSP for type checking. Read/Edit for direct file work.

### Why this matters more than you'd think

Without this table, a typical Claude session might:
1. Read an entire 400-line file to find one function (40 tokens/line = 16,000 tokens wasted)
2. Grep for a class name and get string matches in comments, tests, and docs
3. Try to remember if Zustand's API changed since its training cutoff

With the table:
1. `get_symbols_overview` -> 200 tokens for the whole file outline, then read one function body
2. `find_symbol` -> goes directly to the definition
3. Context7 -> fetches the current docs

Same task, a fraction of the tokens, better accuracy. Over a session, this compounds.

---

## 8. Graphiti — Memory That Lasts Across Sessions

Here's a frustrating pattern: you spend 30 minutes explaining your architecture to Claude. You make good progress. Next session, you explain it all over again. CLAUDE.md helps — but it's static, and you have to remember to update it.

Graphiti is different. It's a **knowledge graph** that Claude writes to during sessions and reads from at the start. Over time, it builds up a web of entities, relationships, and facts about your project — with timestamps so you know when something was decided and whether it's still current.

### What goes in Graphiti (and what doesn't)

Think of Graphiti as the place for **durable knowledge** — anything a future session would need to understand the project deeply, that isn't obvious from reading the code:

**Yes:**
- Project vision, goals, and product direction — what you're building and why
- Key entities and how they're used — data model concepts, relationships between systems
- Architectural design decisions — chosen patterns, structure, technology choices + rationale
- UI/layout design intent — screen flows, component philosophy, visual decisions
- Bug root causes and non-obvious fixes — the kind that took an hour to find
- Trade-offs considered and rejected alternatives — so future sessions don't re-litigate settled decisions
- "The auth middleware rewrite is driven by legal compliance, not tech debt — scope decisions should favor compliance over ergonomics."

**No:**
- Routine code changes (that's what git log is for)
- Secrets or credentials (never)
- Verbose code dumps (store in the code, not in memory)
- Anything already in CLAUDE.md (don't duplicate)
- Session-specific ephemeral state

### How it works in practice

Each project gets a `group_id` to scope its knowledge. Add this to your project CLAUDE.md:

```markdown
## Memory (Graphiti Knowledge Graph)
When using Graphiti tools, always use `group_id="my-project"`.
```

**Session start** — Claude loads relevant context:

```text
search_memory_facts(
  query="project vision goals entities architecture design decisions patterns",
  group_ids=["my-project"]
)
```

**Session end** — Claude saves what it learned:

```text
add_memory(
  group_id="my-project",
  content="Decided to use event sourcing for order processing
           because we need audit trail and temporal queries."
)
```

The best part? You can automate both of these with hooks (see [Section 11](#11-hooks-and-guardrails--automate-and-protect)), so it just *happens* without you thinking about it.

---

## 9. Memory — The Right Information in the Right Place

Claude Code has multiple memory systems, and they each do something different. Using the wrong one is like filing a receipt in your journal — it's stored, but you'll never find it when you need it.

### The cheat sheet

| What You're Storing                              | Where It Goes   | Why There                                     |
|--------------------------------------------------|-----------------|-----------------------------------------------|
| Persistent instructions and rules                | **CLAUDE.md**   | Always loaded, always followed — you write it |
| Quick-reference facts (IDs, names, status flags) | **MEMORY.md**   | Auto-loaded per project, scannable            |
| Patterns Claude discovers while working          | **Auto Memory** | Claude writes these itself as it works        |
| Vision, entities, design, decisions, bug roots   | **Graphiti**    | Searchable, timestamped, relational           |
| Current task progress                            | **Tasks**       | Survives context compaction                   |
| Deep topic-specific knowledge                    | **Skills**      | Loaded on demand, not always                  |

### Auto Memory — Claude teaching itself

This is distinct from MEMORY.md that you write. Claude Code automatically writes notes to itself as it works — build commands that worked, debugging insights, architecture it discovered, code style corrections you gave it. These accumulate in `~/.claude/projects/<project>/memory/`.

The key distinction:
- **CLAUDE.md** = rules *you* write, loaded every session
- **Auto Memory** = patterns *Claude* learns, also loaded every session
- Don't duplicate between them — if Claude already learned something via auto memory, you don't need to add it to CLAUDE.md

### MEMORY.md in practice

Located at `~/.claude/projects/<slug>/memory/MEMORY.md`, this file loads automatically in every session for that project.

**Good MEMORY.md content:**
- "Supabase project ID: `abc123`"
- "Migration to v2 auth: complete as of 2026-01-15"
- "Edge function names: `process-payment`, `send-notification`"

**Bad MEMORY.md content:**
- Code patterns (read the codebase instead)
- Git history summaries (run `git log`)
- Fix recipes (the fix is in the code)
- Anything already in CLAUDE.md

**Keep it under 200 lines.** For deeper topics, link to separate files:

```markdown
<!-- MEMORY.md -->
- [Supabase setup](supabase.md) — project ID, edge functions, RLS policies
- [Localization](localization.md) — supported languages, string key format
```

---

## 10. Context7 — Docs That Are Actually Current

Claude's training data has a cutoff. Libraries ship updates after that cutoff. This means Claude might confidently write code using an API that changed two months ago. Context7 fixes this by fetching **live documentation** at the moment Claude needs it.

```bash
claude plugin install context7@claude-plugins-official
```

### When it matters

- Writing integration code for any library (React, Express, Prisma, Tailwind — all of them)
- Upgrading or migrating between library versions
- Debugging library-specific behavior
- Using a library Claude seems uncertain about

### Teach Claude to reach for it

Add to your global CLAUDE.md:

```markdown
## Context7
Call Context7 before writing integration code for any library.
Training data may be outdated — always verify current API.
```

### When to skip it

Context7 is for library docs. Don't use it for general programming concepts, refactoring your own code, business logic, or code review.

---

## 11. Hooks and Guardrails — Automate and Protect

Hooks serve two purposes: **automation** (so you don't forget to do things) and **protection** (so Claude doesn't do things it shouldn't). The combination of Claude Code hooks and git hooks creates a solid safety net.

### Session start: load everything automatically

Set up a hook that runs at the start of every session:

```jsonc
// ~/.claude/settings.json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/session-start.sh",
            "statusMessage": "Loading project context"
          }
        ]
      }
    ]
  }
}
```

The hook script reads your project CLAUDE.md, extracts the Graphiti `group_id` and Serena project name, and injects "Action required" reminders into Claude's context:

```bash
#!/bin/bash
INPUT=$(cat)
CWD=$(echo "$INPUT" | python3 -c \
  "import sys,json; print(json.load(sys.stdin).get('cwd',''))" 2>/dev/null)

GROUP_ID=""
SERENA_NAME=""

if [ -f "$CWD/CLAUDE.md" ]; then
  GROUP_ID=$(sed -n 's/.*group_id="\([^"]*\)".*/\1/p' "$CWD/CLAUDE.md" | head -1)
  # Match activate_project("name") with constrained identifier characters
  SERENA_NAME=$(
    sed -n \
      -e 's/.*activate_project("\([A-Za-z0-9._-]\+\)").*/\1/p' \
      "$CWD/CLAUDE.md" | head -1
  )
fi

MSGS="Action required: Read ~/.claude/CLAUDE.md for global instructions before starting work.\n"
[ -n "$GROUP_ID" ] && MSGS="${MSGS}Action required: Call search_memory_facts(query=\"project vision goals entities architecture design decisions patterns\", group_ids=[\"$GROUP_ID\"]) to load Graphiti memory. Do this BEFORE responding.\n"
[ -n "$SERENA_NAME" ] && MSGS="${MSGS}Action required: Call activate_project(\"$SERENA_NAME\") to enable Serena code navigation.\n"
[ -z "$GROUP_ID" ] && MSGS="${MSGS}Note: No Graphiti group_id found in project CLAUDE.md.\n"
MSGS="${MSGS}Note: MEMORY.md is auto-loaded — scan it for quick facts before proceeding.\n"

echo -e "$MSGS"
```

Note the ordering: Graphiti runs before Serena activation, matching the required session-start sequence.

> **Tip — identity injection:** If you use an `IDENTITY.md` or `SOUL.md` file for Claude's personality and working style, inject it here too via an "Action required: Read" reminder. Session hooks only fire for interactive sessions — sub-agents spawned with the Agent tool don't run them, so personality context stays out of focused task agents where it would just waste tokens.

### Between edits: nudge verification

A PostToolUse hook fires after each file write. Use it to remind Claude to lint and type-check without blocking:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "You just edited or wrote a file. If this completes a logical unit of work (not mid-edit in a series of changes), run the appropriate linter and type-checker for this project now. Use the commands from the project CLAUDE.md. Skip if you are mid-task and plan to make more changes immediately.",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

This nudges Claude to verify incrementally rather than accumulating errors across many files.

### Session end: save what you learned

A Stop hook runs when Claude is about to finish. Use it for three checks:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Before stopping, do all three: (1) If code was modified, run type-check and lint — report results. (2) If non-obvious decisions, architecture choices, or design rationale were established, save them to Graphiti using add_memory with the project group_id. Skip routine edits. (3) If any stable facts (IDs, status flags, confirmed patterns) changed, update MEMORY.md.",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

> **Why Haiku?** The stop hook is a quick audit — it doesn't need heavy reasoning. Using Haiku keeps it fast and cheap.

### Hook types: more than just shell scripts

Hooks come in three flavors, each for a different level of verification:

| Type      | What It Does                                                | Best For                             |
|-----------|-------------------------------------------------------------|--------------------------------------|
| `command` | Runs a shell script, checks exit code                       | Linting, formatting, notifications   |
| `prompt`  | Sends a prompt to a Claude model for single-turn evaluation | Lightweight review, security checks  |
| `agent`   | Spawns a subagent with full tool access (up to 50 turns)    | Deep verification, multi-file checks |

A `prompt` hook is perfect for "is this code safe?" checks without the overhead of a full subagent. An `agent` hook can read files, run searches, and verify conditions across the codebase before allowing a tool call through.

### Git hooks: the other safety layer

Don't rely only on Claude Code hooks. Git hooks catch things at commit time regardless of how the code was written:

- **pre-commit** — run linters, formatters, type checks
- **commit-msg** — enforce conventional commit format
- **pre-push** — run tests before pushing

The two hook systems complement each other:
- **Claude Code hooks** catch issues *during* the session (before Claude even writes a file)
- **Git hooks** catch issues *at commit time* (the final gate before code enters history)

Together, they create defense in depth. Claude Code hooks prevent bad patterns from being written. Git hooks prevent anything that slipped through from being committed.

### Common hook patterns

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review this file change for security issues (injection, XSS, exposed secrets). Return {\"ok\": true} if safe, {\"ok\": false, \"reason\": \"...\"} if not.",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

### The result

With both hook systems in place, your workflow has multiple checkpoints:
- **Session start:** tools activated, memory loaded, context ready
- **During work:** security checks on edits, formatting on saves
- **At commit:** linting, tests, message format
- **Session end:** decisions saved, memory updated, nothing lost

---

## 12. Permissions and Security

Claude Code's permission system controls what Claude can do without asking you. The right setup lets Claude move fast on safe operations while keeping guardrails on anything dangerous.

### The three tiers

```jsonc
{
  "permissions": {
    "allow": [
      "Read(**)",                          // Read anything
      "Edit(**)",                          // Edit anything
      "Bash(git:*)",                       // All git commands
      "Bash(npm:*)",                       // All npm commands
      "mcp__plugin_serena_serena__*"       // All Serena tools
    ],
    "deny": [
      "Bash(sudo *)",                      // Never sudo
      "Bash(git push --force *)",          // Never force push
      "Read(.env)",                        // Never read secrets
      "Read(**/*.key)",                    // Never read key files
      "Read(~/.ssh/**)",                   // Never read SSH keys
      "Read(~/.aws/**)",                   // Never read AWS creds
      "Read(~/.kube/**)",                  // Never read kube config
      "Read(~/.npmrc)",                    // Never read npm auth
      "Read(~/.git-credentials)",          // Never read git creds
      "Edit(~/.bashrc)",                   // Never modify shell config
      "Edit(~/.zshrc)"                     // Never modify shell config
    ],
    "ask": [
      "Bash(curl * | bash*)"              // Ask before pipe-to-bash
    ]
  }
}
```

### The principles

1. **Be generous with reads** — Claude works better when it can explore freely
2. **Be strict with destructive operations** — force push, schema drops, `rm -rf` on system paths
3. **Always deny secret access** — `.env`, credentials, keys, PEM files, SSH, AWS, kube configs
4. **Wildcard MCP tools** — `mcp__plugin_serena_serena__*` grants all Serena tools at once
5. **Use `ask` for gray areas** — things that are sometimes fine but sometimes dangerous

### Prevent malicious MCP servers in cloned repos

Repositories can include `.mcp.json` files that auto-activate MCP servers when you open them. This is a supply chain risk — a malicious repo could inject tools. Disable auto-activation:

```json
{
  "enableAllProjectMcpServers": false
}
```

With this set, project-level MCP servers require explicit opt-in.

### Bypass mode (for power users)

If you trust your deny list and don't want permission prompts for everything else:

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  },
  "skipDangerousModePermissionPrompt": true
}
```

This means Claude will only stop to ask when it hits something explicitly denied. Fast, but requires a well-configured deny list. Don't enable this until your deny rules are solid.

---

## 13. Subagents and Agent Teams

Claude Code can spawn smaller Claude instances to handle subtasks. This is one of the most powerful features once you go beyond the defaults.

### The orchestration model

The most effective way to use Claude Code is to think of your main session as an **orchestrator** — it plans, delegates, validates, and reviews. The actual implementation work gets delegated to sub-agents.

This isn't just a nice mental model — it's how the best configurations are structured:

| Role                            | Model    | Use When                                             |
|---------------------------------|----------|------------------------------------------------------|
| **Orchestrator** (main session) | `opus`   | Always — planning, review, validation, coordination  |
| **Implementation agent**        | `sonnet` | Writing code, tests, standard refactors              |
| **Exploration agent**           | `haiku`  | Broad codebase search, quick lookups, file discovery |
| **Critical sub-agent**          | `opus`   | When complexity or impact demands getting it right   |

Add this to your global CLAUDE.md:

```markdown
## Orchestration Model
Main session = orchestrator (plans, delegates, validates, reviews)
- haiku — exploration, quick lookups
- sonnet — implementation, tests, code review, standard QA
- opus — architecture, security-critical, complex refactors
```

### Sub-agent rules that pay off

These rules prevent the most common sub-agent failures:

- **Parallel by default.** For tasks touching 5+ independent files, launch parallel sub-agents (5-8 files each). Sequential processing of large tasks guarantees context decay in the orchestrator.
- **Tool-aware.** Sub-agents follow the same tool selection rules as the main session — Serena for code, Grep for text, etc. Don't let them default to reading entire files.
- **Self-verifying.** Sub-agents must build, type-check, lint, and run relevant tests before reporting done. "It compiles" is not verification evidence.
- **Orchestrator double-checks.** The main session reviews sub-agent results independently before accepting. Trust, but verify.

### Custom subagent definitions

Beyond the built-in agents, you can define your own specialized agents in `.claude/agents/`. These are reusable across sessions and can accumulate knowledge over time.

Example — a project-aware code reviewer:

```markdown
<!-- .claude/agents/code-reviewer.md -->
---
name: code-reviewer
description: Reviews code changes for bugs, security issues, and adherence to project conventions
model: sonnet
memory: project
tools:
  - Read
  - Grep
  - Glob
---

You are a code reviewer for this project. Review changes for:
- Logic errors and edge cases
- Security vulnerabilities
- Adherence to project conventions in CLAUDE.md
- Test coverage gaps

Be specific. Cite line numbers. Suggest fixes, don't just flag problems.
```

The `memory: project` setting means this agent accumulates codebase knowledge in `~/.claude/agent-memory/code-reviewer/` across sessions. The more you use it, the better it understands your codebase.

Defining agents that match your specific needs is a game-changer — a security reviewer, a test writer, an architecture advisor. Each one can have its own model, tools, and memory.

### Agent Teams

> *I haven't used Agent Teams yet, but they're worth knowing about for complex projects.*

Agent Teams go beyond simple subagents. Instead of one-way parent-child delegation, teams have:
- **Shared task lists** with dependency tracking
- **Peer-to-peer messaging** between agents
- **Parallel execution** — each teammate is a full Claude Code instance

Think of it as going from "one developer with assistants" to "a small team working together."

Enable it:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Best use cases: parallel code review (security + performance + test reviewers), independent module development, cross-layer changes where frontend and backend need to move together.

**Sizing guidance:** 3-5 teammates, 5-6 tasks per teammate. Each teammate burns its own tokens, so this is a power tool — use it when the parallelism justifies the cost.

---

## 14. Context Window Management

This might be the most important operational insight in this guide: **Claude performs best when the context window is under 200K tokens.** Not 500K, not 1M — under 200K. Yes, the window *supports* 1M, but quality degrades as it fills. Think of it like RAM: your computer has 64 GB, but if you're using 60 GB, everything slows down.

This has a direct practical consequence: **compact and continue is almost always better than starting a new session** for similar work. When you `/compact`, Claude preserves the key decisions and context while shedding the noise. A compacted session with 50K of focused context outperforms a fresh session where Claude has to re-read everything from scratch — and it *far* outperforms a bloated session at 500K where important instructions are buried in old tool outputs.

### The key commands

| Command                             | What It Does                                            | When to Use                                                      |
|-------------------------------------|---------------------------------------------------------|------------------------------------------------------------------|
| `/compact`                          | Compresses conversation history, preserving key context | When the session feels sluggish or you're switching subtasks     |
| `/compact Focus on the API changes` | Guided compression — tells Claude what to preserve      | When you want specific context to survive                        |
| `/clear`                            | Wipes conversation history entirely                     | Between unrelated tasks — the most underused command             |
| `/btw`                              | Ask a side question that doesn't enter history          | Quick lookups that shouldn't grow context                        |
| `/context`                          | Shows per-category token breakdown                      | Diagnosing what's eating your context                            |
| `Ctrl+B`                            | Background a running subagent                           | When a subagent is taking long and you want to do something else |

### Compact vs clear vs new session

This is a decision you'll make dozens of times. Here's the rule:

| Situation                                             | Best Action              | Why                                                                 |
|-------------------------------------------------------|--------------------------|---------------------------------------------------------------------|
| Switching to **unrelated** work                       | `/clear`                 | Old context is pure noise for the new task                          |
| Continuing **similar** work after a chunk is done     | `/compact`               | Keep the decisions, shed the tool outputs                           |
| Session feels sluggish or Claude forgets instructions | `/compact` with guidance | Reset the noise while preserving what matters                       |
| Starting a completely **different project**           | New session              | Different CLAUDE.md, different Serena project, different everything |

The key insight: **`/compact` + continue beats a new session** when the work is related. A compacted session retains your architectural decisions, agreed-upon patterns, and task progress. A new session starts cold and has to rediscover all of that.

### Practical habits

**Use `/clear` aggressively between unrelated tasks.** Finished debugging that API endpoint? `/clear` before starting the frontend work. The cost of re-reading a few files is nothing compared to carrying 100K of irrelevant debugging context.

**Compact proactively, not reactively.** Don't wait until Claude starts getting confused. After completing a major step, `/compact` to keep context lean. Think of it as clearing your desk between tasks — a small habit that compounds.

**Custom `/compact` instructions work.** Instead of just `/compact`, try:

```text
/compact Keep the architectural decisions and test commands,
         summarize the debugging session
```

This guides what survives compression. Useful when you're about to shift focus but want to keep specific decisions in context.

**`/btw` for quick lookups.** Need to check a function signature or a config value but don't want it cluttering your conversation? `/btw what's the return type of getUserById?` — the answer appears in a dismissible overlay, never enters history.

**Subagents are your best context defense.** Delegate exploratory work to subagents. They run in their own context, return just the answer, and keep your main session clean. "Use a subagent to investigate the test failures in the auth module" is better than investigating directly and filling your session with 50 file reads.

### Context hygiene — defensive habits

These are the silent killers of long sessions. None of them produce obvious errors — things just gradually get worse:

- **Re-read before editing after significant work.** Auto-compaction discards earlier file reads. If you've done 20 tool calls since you last read a file, Claude may be editing against stale context. The fix is simple: re-read, then edit.
- **Large files need chunked reads.** Each `Read` call returns max 2,000 lines. For files approaching that, use `offset` and `limit` parameters. Never assume a single read captured everything.
- **Watch for truncated tool results.** Large search results get silently truncated. If a search returns suspiciously few results, re-run with a narrower scope — specific directory, stricter glob pattern.
- **500-line file cap.** Files exceeding 500 lines are hard for Claude to work with efficiently. If you notice a file growing past this, split it — one class or module per file is a good default.

### What survives compaction

- CLAUDE.md: re-read from disk on every compaction — always survives
- Auto Memory / MEMORY.md: always loaded — always survives
- Tasks: survive compaction — this is why they matter
- Conversation instructions: **do NOT survive** unless written to CLAUDE.md

If an instruction you gave Claude disappeared after compaction, it was only in conversation. Write it to CLAUDE.md or a skill if it should be permanent.

---

## 15. Git Worktrees — Parallel Sessions

Claude Code is *designed* for parallelism. It's not an afterthought — it's a core architectural principle. Subagents run in parallel within a session. Multiple sessions run in parallel across worktrees. And you can mix both: a main session orchestrating subagents while other sessions work independently in their own worktrees.

Once you internalize this, you stop thinking of Claude as "one assistant doing one thing" and start thinking of it as "a team I can deploy across my codebase."

### The idea

A git worktree is a separate checkout of the same repo in a different directory, on its own branch. Each Claude session works in its own worktree — different files, different branch, no conflicts.

### Built-in support

Claude Code has first-class worktree support, and the Superpowers plugin includes a **Using Git Worktrees** skill that manages the workflow:

```bash
# Start a session in a new worktree
claude --worktree feature-auth
```

This creates `.claude/worktrees/feature-auth/` with a dedicated branch.

### The power-user pattern

High-output teams run 5-15 Claude sessions simultaneously — some in terminal tabs, some in the web app — each in its own worktree. Frontend in one, backend in another, tests in a third. They all work in parallel without conflicts, and each session opens its own PR when done.

### Two levels of parallelism

Think of it as two complementary strategies:

**Within a session — subagents.** Claude dispatches Haiku/Sonnet/Opus subagents to handle subtasks in parallel. Research three approaches simultaneously. Run code review and tests at the same time. Explore the codebase in one subagent while planning in another. This keeps your main session's context clean while getting work done faster.

**Across sessions — worktrees.** Multiple full Claude sessions, each in its own worktree, working on independent tasks. This is how you parallelize at the feature level — authentication in one session, API endpoints in another, database migrations in a third.

The combination is where the real power is. Each worktree session can *itself* dispatch subagents. You end up with a tree of parallel work that would take a human team days to coordinate.

### Setup tips

**Gitignore the worktrees directory:**

```gitignore
.claude/worktrees/
```

**Copy env files into new worktrees** with `.worktreeinclude`:

```text
# .worktreeinclude — gitignored files to copy into each new worktree
.env.local
.env.development
```

Only gitignored files are copied — tracked files come from the git checkout automatically.

### Subagent worktrees

You can also give subagents their own worktrees for truly isolated parallel work:

```markdown
<!-- .claude/agents/my-agent.md -->
---
isolation: worktree
---
```

The worktree is auto-cleaned if the agent makes no changes.

---

## 16. Git and Code Review

### Conventional commits

Define your format once in global CLAUDE.md and never think about it again:

```markdown
## Git
Commits: <type>(<scope>): <description> (100 char max subject)
Body: blank line + bullet list, 72 char wrap
Types: feat fix docs style refactor perf test build ci chore revert
Rules: Never commit to main. Never force push. Never skip hooks.
git commit is auto-approved. git push requires confirmation.
```

### Code review: use both approaches

The best code review setup uses **two complementary approaches**:

**1. [CodeRabbit](https://www.coderabbit.ai/) — cloud-based AI review on PRs**

CodeRabbit reviews your pull requests automatically on GitHub. It catches issues from a fresh perspective — it hasn't seen your session context, so it finds things you and Claude both missed.

```bash
claude plugin install coderabbit@claude-plugins-official
```

```text
/coderabbit:review    — Run CodeRabbit review on current changes
```

**Autonomous CodeRabbit loop** — run this after every PR push, without waiting to be asked:
1. Wait 2–3 minutes, then fetch all `coderabbitai[bot]` inline comments
2. Fix every comment in one pass — do not cherry-pick
3. Commit and push
4. Wait 2–3 minutes, re-check; repeat until no new comments on the latest push
5. Wait up to 5 minutes for CodeRabbit to flip to `APPROVED`
6. Post `@coderabbitai resolve` to collapse resolved threads

If comments persist after 3 full iterations, stop and report to the user. Use `@coderabbitai full review` if the incremental review seems incoherent (e.g., after a large rebase). Never dismiss reviews via the GitHub API — CodeRabbit approves when it's satisfied.

**2. Local subagent reviews — immediate, contextual feedback**

Claude Code can spawn review subagents that check your work against the plan and project standards right in your session. The Superpowers plugin does this automatically after completing major implementation steps.

You can also trigger it manually:

```text
/code-review
```

Or define a custom review agent (see [Section 13](#13-subagents-and-agent-teams)) that accumulates knowledge about your codebase over time.

**Why both?** Local reviews are fast and contextual — they know the plan and catch deviations immediately. CodeRabbit reviews are fresh and independent — they catch issues that session bias would miss. Together, they give you defense in depth.

### Verification: per-step, not just at the end

Don't wait until the PR to verify. After each logical step:

1. **Build / type-check** (e.g., `tsc --noEmit`, `xcodebuild`)
2. **Lint** (e.g., `eslint . --quiet`, `swiftlint`)
3. **Run relevant unit tests**

This applies to sub-agents too — they should verify before reporting done (see [Section 13](#13-subagents-and-agent-teams)). Before creating a PR, run the full test suite. Add your project's verification commands to its CLAUDE.md so Claude always knows how to check its own work.

---

## 17. Session Management

Sessions in Claude Code aren't disposable — you can resume them, name them, and navigate between them. This matters for multi-day work and when you need to context-switch.

### Key commands

| Command                   | What It Does                                    |
|---------------------------|-------------------------------------------------|
| `claude --continue`       | Resume the most recent session                  |
| `claude --resume`         | Open a session picker to choose which to resume |
| `claude -n auth-refactor` | Start a named session (easier to find later)    |
| `/rename auth-refactor`   | Rename the current session                      |
| `claude --from-pr 123`    | Resume a session linked to a specific PR        |

### Session picker shortcuts

When in the `/resume` picker:
- **`P`** — preview a session
- **`R`** — rename
- **`B`** — filter to current git branch
- **`A`** — toggle between current directory and all projects

### The `!` prefix

Need to run a quick shell command without wasting a tool call round-trip? Prefix it with `!`:

```text
! git status
! npm test
! ls -la src/
```

The output lands directly in conversation context. Faster than asking Claude to run it, and sometimes that's all you need.

---

## 18. Status Line — See What's Happening

Claude Code supports a custom status line at the bottom of the terminal that shows real-time information about your session. Set it up with a shell script at `~/.claude/statusline.sh`.

A good status line shows:
- **Context usage** — how full the context window is (green < 50%, yellow 50-79%, red 80%+)
- **Session cost** — how much you've spent this session
- **Elapsed time** — how long you've been working
- **Prompt cache hit rate** — how efficiently context is being reused

This gives you at-a-glance awareness of when to `/compact` or `/clear`, and helps you develop intuition for what's expensive and what's cheap.

---

## 19. Task Tracking — Don't Lose Your Place

When your conversation gets long, Claude Code compresses older messages to stay within context limits. This is usually fine — but it means Claude can forget where it was in a multi-step plan.

Tasks survive this compression. They're always visible, even after compaction.

```text
TaskCreate   — Create a task
TaskUpdate   — Mark in_progress, completed, or blocked
TaskList     — See all current tasks
```

**The practice:** For any work with more than 2-3 steps, create tasks upfront. Mark each one `in_progress` when you start it, `completed` when you're done — immediately, not batched at the end.

> Tasks are for the current session. For information that should persist across sessions, use MEMORY.md or Graphiti.

---

## 20. Common Failure Patterns — What to Avoid

These are the mistakes that waste the most time with Claude Code. Every one of them is easy to fall into and easy to fix once you know what to look for.

### The kitchen sink session

**The pattern:** You debug an API endpoint, then fix a CSS bug, then write a migration, then refactor a test — all in the same session. By the end, Claude is carrying 500K of mixed context and its responses are getting vague.

**The fix:** `/clear` between unrelated tasks. It feels wasteful ("I just had all that context!") but a fresh start with the right context beats a bloated session every time.

### The correction loop

**The pattern:** Claude does something wrong. You correct it. It does it wrong again, slightly differently. You correct it again. Three rounds in, you're frustrated and Claude is confused — each correction added conflicting context.

**The fix:** After two failed corrections on the same issue, stop correcting. `/clear` and write a better initial prompt. The problem isn't Claude's understanding — it's that the original instruction wasn't clear enough, and each correction is making the context noisier.

### The over-specified CLAUDE.md

**The pattern:** Your CLAUDE.md is 800 lines of detailed rules. Claude ignores half of them because important rules are buried in noise. You add more rules to compensate. It gets worse.

**The fix:** Each line in CLAUDE.md should pass the test: *would removing this cause Claude to make a specific mistake?* If Claude already does the right thing without a rule, delete the rule. Target ~100-200 lines. Move deep knowledge to skills that load on demand.

### The trust-then-verify gap

**The pattern:** Claude writes plausible-looking code. You glance at it, say "looks good," and move on. Later you discover it doesn't handle edge cases, or it subtly broke something else.

**The fix:** Always provide verification criteria — tests to run, scripts to execute, specific behaviors to check. Superpowers' verification skill does this automatically, but you can also just say: "Before we're done, run the test suite and show me the output."

### The infinite exploration

**The pattern:** You ask Claude to "investigate the performance issues." Claude reads 80 files, runs 30 searches, and fills your context with a comprehensive analysis you didn't need.

**The fix:** Scope your asks. "Check if the N+1 query in getUserOrders is causing the slowdown" instead of "investigate performance." Or delegate to a subagent: "Use a subagent to investigate the performance issues in the orders module" — the investigation happens in a separate context and returns just the findings.

### The "I'll add tests later" lie

**The pattern:** Claude implements a feature without tests. You plan to add them later. You never do. The next change breaks the feature silently.

**The fix:** Superpowers' TDD workflow handles this automatically — tests come first, not last. If you're not using Superpowers, make it a rule in your CLAUDE.md: "Write failing tests before implementation. No exceptions."

---

## 21. Config Sync — Same Setup Everywhere

If you develop on more than one machine — say, a work laptop and a personal one — you want your Claude Code setup to be the same on both. Same CLAUDE.md, same hooks, same permissions, same plugins.

### cc-config-sync

[cc-config-sync](https://www.npmjs.com/package/cc-config-sync) is a CLI tool that stores your Claude Code config in a git repo and pushes/pulls between machines:

```bash
npm install -g cc-config-sync

cc-config-sync pull        # Capture current machine's config into repo
cc-config-sync push --yes  # Apply repo config to current machine
cc-config-sync status      # Check what's different
cc-config-sync push --dry-run  # Preview without writing
```

### What gets synced

- `~/.claude/CLAUDE.md` (global instructions)
- `~/.claude/settings.json` (permissions, hooks, plugins)
- `~/.claude/hooks/*.sh` (hook scripts)
- `~/.claude/plugins/installed_plugins.json`
- Per-project CLAUDE.md, settings, and memory files

### One thing to watch

`settings.local.json` contains machine-specific overrides — like paths that differ between machines. Review these before pushing across environments. What works on one machine's filesystem layout may not apply to another.

---

## Getting Started

If you're setting up from scratch, here's the order that gives you the most value fastest:

1. **Install the core plugins** — superpowers, claude-md-management, skill-creator, context7, serena, code-review, coderabbit
2. **Generate your first CLAUDE.md** — use `/init` to scaffold, then refine with `/claude-md-improver`
3. **Add the tool selection table** to your global CLAUDE.md — this single table changes how Claude works
4. **Configure permissions** — allow reads freely, deny the dangerous stuff and secrets
5. **Set up hooks** — session start for automatic tool activation, session end for memory saves
6. **Write your first project CLAUDE.md** — architecture, conventions, key files, build commands
7. **Set up Serena** for your main project — the token savings start immediately
8. **Set up Graphiti** for persistent memory — decisions stop being lost between sessions
9. **Create your first skill** — take something you keep explaining and make it on-demand
10. **Trust Superpowers** — let it guide the planning, TDD, and verification workflows

Each step builds on the previous ones. You don't have to do them all at once — even steps 1-5 will make a noticeable difference.

**Need inspiration?** Check the [Real-World Examples](examples.md) page for annotated, production CLAUDE.md files with analysis of what makes each section effective.

---

## Community Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code Plugins](https://github.com/anthropics/claude-code-plugins)
- [CodeRabbit — AI Code Review](https://www.coderabbit.ai/)
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — curated list of skills, hooks, agents, plugins
- [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) — 135+ agents, 400K+ skills
- [Trail of Bits Claude Code Config](https://github.com/trailofbits/claude-code-config) — security-focused defaults
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- [Graphiti Knowledge Graph](https://github.com/getzep/graphiti)
- [cc-config-sync](https://www.npmjs.com/package/cc-config-sync)

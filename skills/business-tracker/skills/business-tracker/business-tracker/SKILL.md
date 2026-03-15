---
name: business-tracker
version: 1.0.0
description: "Auto-extract business logic from any codebase into living documentation. Scans code to discover business rules, tracks changes across sessions, and maintains always-up-to-date rule docs organized by module. Supports Flutter, Node.js, Go, Python, Java, .NET, Rust, Ruby, PHP, iOS. Triggers: 'biz scan', 'scan business logic', 'biz update', 'record business change', 'biz query', 'query business rules', 'what are the business rules', 'document business logic'. Use this skill whenever the user wants to understand, extract, document, or track business rules and domain logic in a codebase — even if they don't use the exact command names."
---

# Business Tracker

Auto-extract, continuously track, and maintain living documentation of business logic from any codebase.

Works with any tech stack: Flutter, Node.js, Go, Python, Java/Kotlin, .NET, Rust, Ruby, PHP, iOS (Swift/Objective-C).

## How It Works

Trigger via `/business-tracker` + natural language. The skill matches user intent to the right command:

| User says | Action | Command file |
|-----------|--------|-------------|
| "scan business logic", "biz scan", "scan src/services/" | Scan | `commands/biz-scan.md` |
| "record business change", "biz update", "update business docs" | Track change | `commands/biz-update.md` |
| "what's the payment rule?", "biz query", "query business rules" | Query | `commands/biz-query.md` |

**Intent matching:**
- Contains "scan" → run scan (if a path follows, scope to that path)
- Contains "record", "update", "change" → track a change
- Contains "query", "what rule", "how does X work" → query rules
- Ambiguous → ask the user

**On receiving a command:** read the corresponding file under this skill's `commands/` directory and follow its instructions. Scan strategy and detection rules live in `references/` — load them on demand, not upfront.

## Output Structure

```
biz_scan/
├── index.md                    # Master index (project overview + module list)
├── modules/
│   ├── <module_name>.md        # Detailed rules per business module
│   └── ...
├── changelog.md                # Business change history (append-only)
└── .scan-cache.json            # Scan cache (git hash + progress, do not edit)
```

### Progressive Disclosure (`@` references)

- `index.md` contains only the project overview + one-line summary per module + `@biz_scan/modules/xxx.md` reference
- Queries load `index.md` first, then only the relevant module file(s)
- Each module file is self-contained and small — context-friendly

Commit everything to Git for team sharing.

## Core Principles

1. **Cover every rule** — Every flow, every branch, every constraint gets documented. Not a high-level summary.
2. **Write product docs, not code manuals** — Organize by process steps and natural language, not by code structure.
3. **Flows first, rules second** — Describe end-to-end flows (numbered steps) before listing standalone rules.
4. **Don't repeat what code already tells you** — Skip class names, field lists, method signatures, API params. Only keep entry-point file paths.
5. **Auto-split large files** — When a module exceeds 150 lines, auto-split into sub-files by business topic.
6. **Record the why, not just the what** — If the reason is unknown, mark it "reason TBD".
7. **Mark uncertainty** — Uncertain rules get a `[speculative]` tag. Encourage human confirmation.
8. **Transparent coverage** — After scanning, report exactly what was scanned, skipped, and remaining.
9. **Human-AI collaboration** — Auto-generate the first draft; encourage manual edits and additions.

## Supported Stacks

The skill auto-detects project type and loads only the relevant scan rules:

- **Mobile:** Flutter/Dart, iOS (Swift/Objective-C)
- **Frontend:** React/Vue/Angular (TypeScript/JavaScript)
- **Backend:** Node.js, Go, Python (Django/FastAPI), Java/Kotlin (Spring), .NET, Rust, Ruby (Rails), PHP (Laravel)
- **Monorepo:** Detects each subdirectory independently

## Reference Files

| File | Content | When to load |
|------|---------|-------------|
| `references/scan-rules/common.md` | Project type detection, business signal patterns, batching strategy, cache format | biz-scan and biz-update |
| `references/scan-rules/<stack>.md` | Stack-specific file priority rules (e.g., `flutter.md`, `node.md`) | After detecting project type |
| `references/index-template.md` | Template for index.md | First-time doc generation |
| `references/module-template.md` | Template for module files | Creating new module files |
| `references/changelog-template.md` | Template for changelog entries | biz-update |

# Scan Rules — Common

Shared rules for biz-scan and biz-update. Load this file first, then load the stack-specific file after detecting project type.

---

## 1. Project Type Detection

Scan root directory files to determine the tech stack:

| Detection file | Project type | Tag | Stack rules file |
|----------------|-------------|-----|-----------------|
| `pubspec.yaml` | Flutter/Dart | `flutter` | `flutter.md` |
| `package.json` + (`next.config.*` / `nuxt.config.*` / `angular.json` / `vite.config.*`) | Frontend framework | `node-frontend` | `node.md` |
| `package.json` + (`src/server` / `src/app.ts` / `nest-cli.json`) | Node.js backend | `node-backend` | `node.md` |
| `go.mod` | Go | `go` | `go.md` |
| `Cargo.toml` | Rust | `rust` | `rust.md` |
| `*.csproj` / `*.sln` | .NET | `dotnet` | `dotnet.md` |
| `pom.xml` / `build.gradle` / `build.gradle.kts` | Java/Kotlin | `jvm` | `jvm.md` |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python | `python` | `python.md` |
| `Gemfile` | Ruby | `ruby` | `ruby.md` |
| `composer.json` | PHP | `php` | `php.md` |
| `*.xcodeproj` / `*.xcworkspace` + `*.swift` | iOS (Swift) | `ios-swift` | `ios.md` |
| `*.xcodeproj` / `*.xcworkspace` + `*.m` / `*.h` | iOS (Obj-C) | `ios-objc` | `ios.md` |

Monorepo projects may have multiple tags — detect each subdirectory separately.

After detection, load the corresponding stack rules file from this skill's `references/scan-rules/` directory.

---

## 2. Universal File Priority

These apply across all stacks. Stack-specific files add concrete file patterns.

**P0 — Must scan (highest business logic density)**
- State management / business logic layer (Store, Service, UseCase, Handler)
- Business constants / enum definitions (enums/consts with business vocabulary)
- Domain models with business methods (not pure data carriers)

**P1 — Should scan (likely contains business rules)**
- Data models (focus on field validation, defaults, business methods)
- Repository / DAO layer (data transformation, business filtering)
- Routes / middleware (flow structure, permission guards)
- API definitions (business constraints in parameters)

**P2 — May scan (occasionally contains business logic)**
- Utils / helpers with business vocabulary in filename
- Configuration files with business settings
- Data migrations / seed files

**Skip — Do not scan**
- Generated code, build output, dependency directories
- Pure UI components (layout/style/animation only)
- Test files
- Asset files (images, fonts, etc.)
- CI/CD config, Docker, build scripts

---

## 3. Extraction Principles — Product Docs, Not Code Manuals

### Goal: Complete Coverage in Product-Doc Style

**Complete coverage** means: every business flow, every if/else branch, every business constraint is recorded. Not a high-level summary — detailed enough to construct a complete plan from.

**Product-doc style** means: organized by process steps and natural language, not by code structure. The difference is in presentation, not detail level.

### What to Extract vs. Skip

**Extract (describe code logic in a different way):**
- End-to-end business flows — numbered steps covering the full chain, including all branches
- Business constraints and rules — "when X happens, Y occurs" for every condition
- Cross-module interactions — how module A affects module B, what data passes between them
- State transitions — entity states, transition triggers, and conditions
- Data flow — where data is stored, how it syncs, when it's cleared, isolation strategy
- Current special states — A/B tests, disabled features, migrating logic
- Non-obvious design decisions — why two IDs, why two API sets
- Analytics and side effects — which operations trigger tracking events or write local state

**Do NOT extract (reading code is more accurate for these):**
- Class names, method signatures, function parameters — grep can find these
- Data model field lists — read the model file directly
- API paths and request parameter details — read the API file directly
- UI layout and styling — read the widget/component directly

### Quality Check

After writing a module's docs, verify:
1. "Could a developer who doesn't know this module draw a complete business flow diagram from this doc?" — If not, flows are incomplete.
2. "If I need to add a feature to this module, can this doc help me assess the impact?" — If not, cross-module interactions and constraints are lacking.
3. "Is there information here I could get by just grepping the code?" — If yes, trim it.

### Output Format

- Use **numbered steps** for process chains, not "call XX method"
- Use **natural language** for rules, not code snippets
- Keep only **top-level entry points** (3-5 files per module), not every implementation file
- A module's doc should read like a **product PRD's technical supplement**, not an API reference

### Auto-Split

When a single module file exceeds **150 lines**, split by business topic:
- Module file becomes an index (overview + sub-file reference table)
- Sub-files go in `biz_scan/modules/<module_name>/` directory
- Each sub-file covers one business topic
- See `module-template.md` for the split template

---

## 4. Business Logic Signals

These patterns strongly indicate business logic (language-agnostic):

### Strong Signals (high probability of business rules)
- Money/pricing: `price`, `cost`, `fee`, `discount`, `amount`, `rate`, `tax`
- Permissions: `isVip`, `hasPermission`, `canAccess`, `isAllowed`, `authorize`, `guard`, `policy`
- State transitions: state machine definitions, order/payment/approval status changes, `switch (status)`, `transition`
- Business validation: field length limits, format checks (non-technical), business preconditions
- Time rules: expiry, cooldown, free trial period, `expire`, `ttl`, `cooldown`
- Quantity limits: max devices, daily free quota, inventory check, `limit`, `quota`, `threshold`
- Business flow orchestration: step sequences, approval chains, workflows

### Medium Signals (context-dependent)
- Conditional branches with user type/role checks
- Enum definitions (could be business or technical)
- Comments: `// business rule`, `// product requirement`, `// TODO: PM said`, `// BUSINESS:`
- Config items (could be business or technical)
- Event/message definitions (may reflect business flows)

### Weak Signals (usually not business logic)
- Exception handling / error code mapping
- Logging
- Cache logic
- Network retry strategy
- UI state management (loading/error/success)
- Serialization/deserialization
- DB connection/transaction management

---

## 5. Batching Strategy

### Why Batch
A single context window can't hold all code from large projects. Batching ensures each batch gets thorough analysis.

### Batching Rules
- Max **8-10 files per batch** (adjust by file size — keep total code per batch analyzable)
- Priority order: P0 first, then P1, then P2
- Keep files from the same business module in the same batch for context coherence
- In monorepos, process each sub-project separately

### Scan Loop

```
Start scan
  → Detect project type (Section 1)
  → Read .scan-cache.json (if exists)
  → Run `git ls-files -s` for all file hashes
  → Apply file priority rules by project type
  → Compare with cache: filter files needing scan (new + changed hash)
  → Sort by priority
  → Batch processing loop:
      Each batch:
        1. Read this batch's files
        2. Extract business logic
        3. Update corresponding module files (incremental)
        4. Update .scan-cache.json (record scanned file hashes)
        5. Output progress
      → Continue to next batch until done or user stops
  → Generate coverage report
End
```

### User Interruption Handling
If user stops mid-scan (Ctrl+C or stop):
- Already-scanned results stay in docs
- Cache is already up-to-date, next biz-scan continues from checkpoint
- Coverage report notes "scan incomplete"

---

## 6. Scan Cache Format

File: `biz_scan/.scan-cache.json`

```json
{
  "version": 1,
  "project_type": "flutter",
  "project_types": ["flutter"],
  "last_scan": "2026-03-02T14:30:00Z",
  "scan_complete": true,
  "files": {
    "lib/features/payment/stores/subscription_store.dart": {
      "git_hash": "a1b2c3d4e5f6",
      "last_scanned": "2026-03-02T14:30:00Z",
      "priority": "P0",
      "rules_extracted": 5,
      "status": "scanned"
    }
  },
  "deleted_files": [
    {
      "path": "lib/features/payment/stores/old_payment_store.dart",
      "detected_at": "2026-03-02T14:30:00Z",
      "had_rules": true
    }
  ]
}
```

### Cache Comparison Logic

```
For each business-relevant file:
  Not in cache → new file, needs scanning
  In cache but git_hash differs → file changed, needs re-scanning
  In cache and git_hash matches → skip

For files in cache but not in `git ls-files`:
  → File deleted, add to deleted_files
  → Flag in coverage report: related business rules may need cleanup
```

---

## 7. Coverage Report Format

Appended to `biz_scan/index.md`:

```markdown
---

## Scan Coverage Report

> Scan time: 2026-03-02 14:30 | Status: Complete | Project type: flutter

### Scanned — 23 files, 18 rules extracted
| Module | Files | Rules | Key files |
|--------|-------|-------|-----------|
| payment | 5 | 7 | subscription_store.dart, payment_service.dart |
| reader | 4 | 5 | chapter_service.dart, reading_store.dart |

### Skipped (unchanged hash) — 8 files
No changes since last scan; existing rules retained.

### Not scanned (low priority) — 12 files
- `lib/utils/format_helper.dart` — P2, may contain formatting business rules
To scan: `/business-tracker biz scan lib/utils/format_helper.dart`

### Deleted files — 1
- `lib/features/payment/stores/old_payment_store.dart` — had 3 rules
  Please verify if related rules are still valid
```

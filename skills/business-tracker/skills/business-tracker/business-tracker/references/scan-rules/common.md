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

## 2. Two-Phase Scanning Strategy

### Phase 1: Structure Mapping ("draw the map")

Before reading any source file for business logic, build a lightweight map of the project — like a human developer browsing the folder structure before diving into code:

1. **Directory tree** — understand the folder layout, infer layers from directory names
2. **Sub-library discovery** — find all internal dependencies (path/git/workspace packages)
3. **Entry point reading** — read 2-5 entry files to understand the app's wiring (DI, routing, top-level modules)
4. **Module grouping** — combine directory structure + entry point context to group files into business modules

**Key principle: only read entry files, infer everything else from directory structure and naming conventions.** No need to parse every file's imports — the folder layout is the map.

This phase produces `.project-structure.json` — the "map" that guides Phase 2.

### Phase 2: Business Logic Extraction ("walk the route")

With the map in hand, scan files **layer by layer, module by module**:

```
Layer 0: Entry points (main, app bootstrap)
Layer 1: DI / Config / Router (wiring layer)
Layer 2: Business logic (stores, services, controllers, use cases)
Layer 3: Data layer (API clients, repositories, persistence, DB)
Layer 4: Models / Entities / DTOs
Layer 5: Utils / Helpers / Extensions
```

**Key principle:** Scan each module as a **vertical slice** (Layer 2 + its Layer 3-5 dependencies together), not as horizontal layers. This ensures the scanner sees the full context of each business flow in one batch.

### Why two phases?

- Scanning without a map leads to random file ordering and missed context
- Directory-structure-based mapping is **fast** (no source file reading) and gives a good-enough module grouping
- Reading entry files reveals the app's wiring, which pattern-matching alone can't infer
- Gap scan (after Phase 2) catches files with non-standard names that directory grouping missed
- Sub-library discovery ensures vendored/local dependencies aren't overlooked

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

### Code references: what to record vs. skip

**Record these (stable, rarely change):**
- Core Classes table — class name + file path + one-line role description
- Flow entry points — when a step triggers code, annotate the entry class inline, e.g. "User taps Subscribe → create order (via `SubscriptionStore`)"
- Cross-module references — use `@biz_scan/modules/xxx.md` format

**Do NOT record these (change frequently, read code directly):**
- Method signatures, parameter lists — outdated after one refactor
- Field lists, data structure definitions — read the model file
- Import paths, dependency chains — IDE can navigate
- Internal implementation classes — only record entry-point classes
- API paths and request parameter details — read the API file directly
- UI layout and styling — read the component directly

### Quality Check

After writing a module's docs, verify:
1. "Could a developer who doesn't know this module draw a complete business flow diagram from this doc?" — If not, flows are incomplete.
2. "If I need to add a feature to this module, can this doc help me assess the impact?" — If not, cross-module interactions and constraints are lacking.
3. "Is there information here I could get by just grepping the code?" — If yes, trim it (except the Core Classes table — that stays).

### Output Format

- Use **numbered steps** for process chains, not "call XX method"
- Use **natural language** for rules, not code snippets
- Use `@biz_scan/modules/xxx.md` references for cross-module links
- Include a **Core Classes** table with class name, file path, and role
- File coverage is tracked in `.scan-cache.json`, NOT in the md file
- A module's doc should read like a **product PRD's technical supplement**, not an API reference

### Extraction Completeness

The goal is **complete extraction** — every business flow, every branch, every constraint that exists in the code must appear in the docs. There is no fixed "N rules per M lines" quota. Instead, the standard is: if a branch exists in code, it must exist in the doc.

**How to verify completeness:**
- After extracting from a file, mentally walk through the code's control flow — is every path represented?
- If a method has 5 if-branches but only 2 were extracted, the extraction is incomplete

**Banned writing patterns (these indicate incomplete extraction):**
- "handles different cases based on condition" — must list each case explicitly
- "calls the relevant service to complete the operation" — must name the service and what it does
- "performs various validations" — must list each validation
- Any use of vague qualifiers ("various", "relevant", "different", "multiple") that obscure specific branches — replace with the actual list of items

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
- **Batch by module vertical slice**, not by file type or layer alone
- Include a module's business logic files AND their direct data-layer dependencies in the same batch
- In monorepos / multi-package projects, process each sub-project separately
- Sub-libraries: each sub-lib gets its own batch(es)

**Parallel execution constraints:**
- Maximum 2 scan agents running concurrently
- The main agent acts as dispatcher only — does NOT read source files or extract logic itself
- Each scan agent independently: reads files → extracts logic → writes module .md → updates .scan-cache.json
- Agents must NOT pass scan results to each other (prevents double-compression / detail loss)

### Batch Order
```
Batch 1:     Entry + DI/Bootstrap (Layer 0-1) — understand wiring
Batch 2-N:   Business modules (Layer 2 + dependencies) — one module per vertical slice
Batch N+1-M: Sub-libraries — each sub-lib as 1+ batches
(Gap scan runs after all batches — see biz-scan Step 5)
```

### Scan Loop

```
Start scan
  → Detect project type (Section 1)
  → Map project structure: directory tree + entry files (biz-scan Step 2)
  → Read .scan-cache.json (if exists)
  → Build scan manifest from module groups
  → Compare with cache: filter files needing scan
  → Batch processing loop:
      Each batch:
        1. Read this batch's files (vertical slice)
        2. Extract business logic
        3. Update corresponding module files (incremental)
        4. Update .scan-cache.json
        5. Output progress
      → Continue to next batch until done or user stops
  → Run gap scan (unassigned files)
  → Generate coverage report
End
```

### User Interruption Handling
If user stops mid-scan (Ctrl+C or stop):
- Already-scanned results stay in docs
- Cache is already up-to-date, next biz-scan continues from checkpoint
- Structure map (`.project-structure.json`) is preserved
- Coverage report notes "scan incomplete"

---

## 6. Scan Cache Format

File: `biz_scan/.scan-cache.json`

**Version migration:** If existing cache has `version` < 2, discard the cache and re-scan from scratch (keep existing module md files).

```json
{
  "version": 2,
  "project_type": "flutter",
  "project_types": ["flutter"],
  "last_scan": "2026-03-16T14:30:00Z",
  "scan_complete": true,
  "structure_mapped": true,
  "files": {
    "lib/stores/user_store.dart": {
      "git_hash": "a1b2c3d4e5f6",
      "last_scanned": "2026-03-16T14:30:00Z",
      "layer": 2,
      "module": "user",
      "rules_extracted": 5,
      "status": "scanned"
    }
  },
  "sub_libraries": {
    "purchase_service": {
      "path": "/path/to/purchase_service/",
      "files_total": 20,
      "files_scanned": 20,
      "status": "complete"
    }
  },
  "gap_scan": {
    "checked": 39,
    "found_business_logic": 5,
    "skipped": 34
  },
  "deleted_files": []
}
```

### Cache Comparison Logic

```
For each file in module groups + unassigned files:
  Not in cache → new file, needs scanning
  In cache but git_hash differs → file changed, needs re-scanning
  In cache and git_hash matches → skip

For files in cache but not in current file set:
  → File deleted, add to deleted_files
  → Flag in coverage report

Structure re-mapping trigger:
  → When entry point files changed (git hash differs)
  → When dependency config files changed (pubspec.yaml, package.json, go.mod, etc.)
  → When new directories appear or existing ones are removed
  Otherwise, reuse existing .project-structure.json
```

---

## 7. Coverage Report Format

Appended to `biz_scan/index.md`:

```markdown
---

## Scan Coverage Report

> Scan time: 2026-03-16 14:30 | Status: Complete | Project type: flutter

### Project Structure
- Source roots: 1 main + 14 sub-libraries + 4 vendored
- Entry points: 3
- Modules identified: 21
- Layers: entry(3) → bootstrap(8) → business(75) → data(120) → model(45) → util(30)

### Scanned — 200 files, 400 rules extracted
| Module | Files | Rules | Layers covered | Key classes |
|--------|-------|-------|---------------|-------------|
| user | 13 | 25 | 2-3-4-5 | UserStore, AuthService, AccessTokenStore |
| reader | 18 | 50 | 2-3-4-5 | ReaderStore, ChapterStore, BookLoadStore |

### Sub-libraries — 14 scanned
| Library | Files scanned | Files total | Status |
|---------|--------------|-------------|--------|
| purchase_service | 20 | 20 | complete |
| yj_network | 28 | 28 | complete |

### Gap scan — 39 unassigned files checked
- 5 files contained business logic → scanned and added to modules
- 34 files had no business signals → skipped

### Skipped (unchanged hash) — 0 files
(First scan — no files skipped)

### Deleted files — 0
(No deletions detected)
```

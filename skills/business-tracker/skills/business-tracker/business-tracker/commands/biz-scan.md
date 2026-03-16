Scan the current codebase, extract business logic rules, and generate/update business documentation. Supports full scan, module scan, and incremental scan (skips unchanged files via git hash cache).

First scan auto-initializes (creates `biz_scan/` directory). No separate init step needed.

Scans run in automatic batched loops until complete or stopped by the user.

Optional arguments:
- `biz-scan` — full project scan
- `biz-scan [path]` — scope scan to a specific path, e.g. `biz-scan lib/features/payment/`
- `biz-scan [module]` — scan a specific business module, e.g. `biz-scan user`, `biz-scan payment`

---

## Pre-scan: Load Scan Strategy

Read this skill's `references/scan-rules/common.md` to understand:
- Two-phase scanning strategy (structure mapping → business logic extraction)
- Business logic signal patterns
- Batching strategy and cache format

---

## Step 0: Auto-Initialize & Scope Resolution

### 0a. Initialize directory

Check if `biz_scan/` exists in project root:

- **Does not exist** (first use):
  ```bash
  mkdir -p biz_scan/modules
  ```

### 0b. Resolve scan scope

Determine what to scan based on user input:

| User input | Scope | Behavior |
|-----------|-------|----------|
| `biz-scan` (no args) | Full project | Run all steps |
| `biz-scan lib/features/payment/` | Path | Only scan files under that path |
| `biz-scan payment` | Module | Look up module in structure data, scan its files |

**How to disambiguate module vs path:**
- Contains `/` or `.` → treat as **path**
- Otherwise → check if it's a known module name in `.project-structure.json` or `.scan-cache.json`
- If neither matches → treat as path, report error if not found

**Module scan resolution:**
1. Check if `biz_scan/.project-structure.json` exists
   - **Exists**: Look up the target module's file list from `module_groups`. Jump to Step 3 (skip structure mapping).
   - **Does not exist**: Must run Step 1 + Step 2 first (structure mapping), then scope to the target module from Step 3 onward.
2. Also check `biz_scan/.scan-cache.json` for existing module assignments
   - Files already assigned to the target module in cache can be filtered by hash for incremental scan

**Path scan resolution:**
1. Check if `biz_scan/.project-structure.json` exists
   - **Exists**: Filter files under the target path from structure data. Jump to Step 3.
   - **Does not exist**: Must run Step 1 + Step 2 first, then scope to the target path from Step 3 onward.

## Step 1: Project Detection

- Read project root directory structure, detect project type per `common.md` Section 1
- Load the corresponding stack-specific rules from this skill's `references/scan-rules/<stack>.md`
- For monorepos, detect each subdirectory's stack and apply corresponding rules
- Read CLAUDE.md, README.md for existing business context

## Step 2: Project Structure Mapping (the "draw map" phase)

**Skip condition:** If `biz_scan/.project-structure.json` already exists AND the dependency config files (pubspec.yaml, package.json, go.mod, etc.) haven't changed since last mapping, skip this step and reuse existing structure data. Otherwise, re-run mapping.

**This step must complete before any file scanning begins.** Its purpose is to build a lightweight map of the project's architecture — like a human developer first browsing the folder structure before diving into code.

**Principle: progressive discovery, not exhaustive parsing.** Only read a few key files (entry points + config), infer the rest from directory structure and naming conventions.

### 2a. Map the full directory tree (read 0 source files)

```bash
# Get directory structure (depth 4, exclude build/generated/assets)
find . -maxdepth 4 -type d \
  | grep -v node_modules | grep -v .dart_tool | grep -v build \
  | grep -v .git | grep -v __pycache__ | grep -v .gradle \
  | grep -v Pods | grep -v .pub-cache | grep -v vendor \
  | sort
```

Output: a tree of all source directories. From directory names alone, infer the layer structure:

| Directory naming pattern | Layer |
|-------------------------|-------|
| `stores/`, `blocs/`, `controllers/`, `usecases/`, `handlers/` | Layer 2 (business logic) |
| `services/`, `api/`, `repositories/`, `dao/`, `persistence/`, `db/` | Layer 3 (data) |
| `models/`, `entities/`, `dto/`, `schemas/` | Layer 4 (model) |
| `utils/`, `helpers/`, `extensions/`, `common/`, `shared/` | Layer 5 (util) |
| `routes/`, `router/`, `di/`, `config/`, `providers/` | Layer 1 (bootstrap) |

Stack-specific rules file adds more patterns per language.

### 2b. Discover all internal dependencies (sub-libraries / sub-packages / sub-modules)

Depending on the project type:
- **Flutter/Dart**: Parse `pubspec.yaml` for `path:` and `git:` dependencies. Parse `.dart_tool/package_config.json` for resolved paths.
- **Node.js**: Parse `package.json` workspaces. Check for `packages/`, `libs/`, `modules/` directories.
- **Go**: Parse `go.mod` replace directives. Check for `internal/`, `pkg/` directories.
- **Python**: Parse `pyproject.toml` or `setup.py` for local packages. Check for `src/` sub-packages.
- **Java/Kotlin**: Parse `settings.gradle` for included modules. Check for multi-module project structure.
- **.NET**: Parse `*.sln` for project references. Check `src/` for sub-projects.
- **Rust**: Parse `Cargo.toml` workspace members.
- **Monorepo**: Detect each subdirectory independently.

Also scan for vendored/local dependencies (e.g. `third/`, `vendor/`, `plugins/`).

**Output: a complete list of all code locations** (main project + every sub-library + vendored dependencies), with their resolved paths.

### 2c. Read entry points to understand wiring (read 2-5 files)

Find and **read** the application's entry files:
- **Flutter**: `main.dart`, `main_*.dart`, `app.dart`
- **Node.js**: `package.json` → `"main"` / `"scripts.start"`, `index.ts`, `app.ts`, `server.ts`
- **Go**: `cmd/*/main.go`, `main.go`
- **Python**: `__main__.py`, `app.py`, `manage.py`, `wsgi.py`
- **Java/Kotlin**: `*Application.java`, `*Application.kt`, classes with `@SpringBootApplication`
- **.NET**: `Program.cs`, `Startup.cs`
- **Rust**: `main.rs`, `lib.rs`
- **iOS**: `AppDelegate.swift`, `SceneDelegate.swift`

From entry files, extract:
- What top-level modules/services are registered (DI container, provider list, router config)
- What the application's main feature areas are
- How the app is structured (single module, feature-based, layer-based, etc.)

This is the **only step that reads source files** during structure mapping.

### 2d. Infer module groups from directory structure + entry point context

Combine what you learned from 2a (directory tree) and 2c (entry point wiring) to group files into business modules:

**Strategy: directory-path-based grouping, not import-chain tracing.**

```
From directory tree:
  lib/stores/user_store.dart        → module: user,    layer: 2
  lib/stores/sign_in_store.dart     → module: user,    layer: 2
  lib/services/api/user.dart        → module: user,    layer: 3
  lib/models/user_model.dart        → module: user,    layer: 4
  lib/stores/reader_store.dart      → module: reader,  layer: 2
  lib/services/api/reader.dart      → module: reader,  layer: 3
```

**Module inference rules (in priority order):**
1. **Feature directory structure** (e.g. `features/payment/`): all files under a feature directory → same module
2. **Filename keyword matching**: `user_store.dart`, `user_api.dart`, `user_model.dart` → all share keyword `user` → same module
3. **Entry point context**: if entry file shows `UserStore` registered alongside `AuthService`, they likely belong to the same `user` module
4. **Ambiguous files**: files that don't clearly belong to one module → mark as `shared` or `unassigned`, resolve during scan

**Don't try to be perfect here** — module assignment will be refined during the actual scan (Step 4b). This is a best-guess grouping to enable vertical-slice batching.

### 2e. Generate structure report

Write `biz_scan/.project-structure.json`:
```json
{
  "project_type": "flutter",
  "mapped_at": "2026-03-16T14:30:00Z",
  "entry_points": ["lib/main.dart", "lib/entry_main.dart"],
  "source_roots": [
    {"path": "lib/", "type": "main", "file_count": 719},
    {"path": "/path/to/sub_lib/lib/", "type": "git_dependency", "name": "purchase_service", "file_count": 20}
  ],
  "layer_dirs": {
    "2_business": ["lib/stores/", "lib/features/*/stores/"],
    "3_data": ["lib/services/api/", "lib/services/persistence/", "lib/db/"],
    "4_model": ["lib/models/"],
    "5_util": ["lib/utils/"]
  },
  "module_groups": {
    "user": {
      "files": ["lib/stores/user_store.dart", "lib/stores/sign_in_store.dart", "lib/services/api/user.dart", "lib/models/user_model.dart"],
      "layers": [2, 3, 4]
    },
    "reader": {
      "files": ["lib/stores/reader_store.dart", "lib/services/api/reader.dart"],
      "layers": [2, 3]
    },
    "payment": {
      "files": ["lib/stores/subscription_store.dart"],
      "sub_library": "purchase_service",
      "layers": [2, 3]
    }
  },
  "unassigned_files": ["lib/TestPage.dart", "lib/old_feature.dart"],
  "total_source_files": 719
}
```

Output summary to user:
```
Project structure mapped (read 3 entry files + directory tree):
  Source roots: 1 main + 14 git dependencies + 4 vendored
  Entry points: 3 (main.dart, main_develop.dart, main_production.dart)
  Total source files: 950 (main: 719, sub-libraries: 231)
  Modules identified: 15 (user, reader, payment, ...)
  Unassigned files: 39 (will be classified during scan)
  Starting scan...
```

## Step 3: Build Scan Manifest

Using the structure from Step 2, build the scan file list:

**File discovery is directory-structure-based:**
1. Start with all source files under known source roots (from Step 2a/2b)
2. Exclude generated files, test files, pure UI files, assets (per stack-specific rules)
3. Assign each file a layer based on its directory (from Step 2a layer_dirs)
4. Group into modules (from Step 2d module_groups)

For each file, assign priority based on **layer**:
- **P0**: Layer 2 (business logic) — all files
- **P1**: Layer 3 (data layer) — all files
- **P1**: Layer 1 (bootstrap/DI) — all files
- **P2**: Layer 4-5 (model/util) — files with business signal keywords in filename
- **P2**: Unassigned files with business signal keywords in filename
- **Skip**: Generated, test, pure UI, assets, CI/CD

Compare with `biz_scan/.scan-cache.json`:
- Same hash → mark "skip"
- Different hash or new file → mark "needs scan"
- In cache but not in current files → mark "deleted"

Output scan plan and **immediately start scanning**:
```
Scan plan:
  To scan: 185 files (P0: 75, P1: 68, P2: 42)
  Skipped (unchanged): 0 files (first scan)
  Deleted: 0 files
  Estimated: 20 batches
  Starting batch 1...
```

## Step 4: Batched Scan Loop

### Batch ordering: follow the layer structure

Instead of sorting by filename pattern, batch by **module vertical slices**:
```
Batch 1: Layer 0-1 (entry + bootstrap/DI) — understand the wiring
Batch 2-N: Business modules — each module as a vertical slice (Layer 2 + its Layer 3-5 files)
Batch N+1-M: Sub-libraries — each sub-lib as one or more batches
(Gap scan runs after all batches complete — see Step 5)
```

**Within each module batch**, include the full vertical slice:
```
Module "user" batch:
  Layer 2: user_store.dart, sign_in_store.dart
  Layer 3: auth_service.dart, user_api.dart, user_persistence.dart
  Layer 4: user_model.dart
  Layer 5: (none)
```

This ensures the scanner sees the complete context for each module in one batch.

Max **8-10 files per batch** (adjust by file size). If a module's vertical slice exceeds this, split into sub-batches but keep related files together.

### Each batch:

**4a. Read this batch's source files**

**4b. Extract business logic (product-doc style)**

Follow `common.md` Section 3 "Extraction Principles" strictly:

- **Extract flows**: Chain multiple functions/files into end-to-end business flows with numbered steps
- **Extract rules**: Identify business constraints, conditions, limits — describe "when X, then Y"
- **Extract state transitions**: If state machines or lifecycles exist, describe states and triggers
- **Extract cross-module interactions**: Record module dependencies and data flow
- **Extract special states**: A/B tests, disabled features, migrating logic
- **Record code references**: Per `common.md` Section 3 "Code references" — Core Classes table + flow entry point annotations
- Mark uncertain rules with `[speculative]`
- **Group by business module**: Determine which module each rule belongs to

**4c. Write module files (progressive disclosure + auto-split)**

For each affected module:
- If `biz_scan/modules/<module_name>.md` doesn't exist:
  Read this skill's `references/module-template.md`, create from template
- If it exists:
  Find the relevant section, incrementally update (don't overwrite other sections)
- **MUST include** the `## Core Classes` section (see module-template.md)
- File coverage is tracked in `.scan-cache.json` (each file's `module` field), NOT in the md file

**Auto-split check** (after each batch write):
- Check module file line count. If over **150 lines**:
  1. Create `biz_scan/modules/<module_name>/` directory
  2. Split content into sub-files by business topic
  3. Replace original module file with index format (overview + sub-file reference table)
  4. Keep cross-module interactions, special states, and core classes in the index file
- See `references/module-template.md` for split template

**4d. Update index.md**

- If `biz_scan/index.md` doesn't exist:
  Read this skill's `references/index-template.md`, create from template
- Update module index: ensure each module has a one-line summary + `@biz_scan/modules/<module_name>.md` reference
- Update scan date in header

**4e. Update cache**
- Write this batch's scanned files to `biz_scan/.scan-cache.json`
- Record git_hash, scan time, rules extracted count, layer, module assignment

**4f. Output progress**
```
Batch 3/20 complete (module: user)
  This batch: 8 files across 4 layers, 12 rules extracted
  Total: 24/185 files, 35 rules
  Remaining: 161 files
  Continuing to next batch (module: reader)...
```

### Loop until:
- All files scanned → go to Step 5
- User stops → save progress, output "Scan paused. Next biz-scan will resume from checkpoint."

## Step 5: Gap Scan (catch what directory-based grouping missed)

After completing all module batches, check for files that were unassigned or skipped:

1. List files that were in `unassigned_files` (from `.project-structure.json`) and not yet scanned
2. For each unscanned file:
   - Quick-read the file header (first 50 lines)
   - Check for business signal keywords (from `common.md` Section 4)
   - If signals found → scan it fully and assign to the appropriate module
   - If no signals → add to skip list with reason
3. Report gap scan results

```
Gap scan:
  Checked: 39 unassigned files
  Found business logic: 5 files → scanned and added to modules
  No business logic: 34 files → skipped
```

**Note:** This step is the safety net. The directory-based grouping (Step 2d) may miss files with non-standard names or unusual locations. Gap scan ensures nothing important falls through.

## Step 6: Handle Deleted Files

For files marked "deleted" in cache:
- Find related rules in the corresponding module file
- Add tag: `[source file deleted — please verify if this rule is still valid]`
- List these rules for user confirmation

## Step 7: Generate Coverage Report

Generate/update the coverage report at the end of `biz_scan/index.md` (format per `common.md` Section 7).

Report includes:
- **Structure summary**: source roots, entry points, layer distribution
- **Scanned files**: by module and layer, with rule counts
- **Sub-libraries**: each sub-lib with scanned/total file counts
- **Skipped files**: unchanged hash (incremental scan)
- **Gap scan results**: unassigned files checked
- **Deleted files**: and affected rules

## Step 8: Final Summary

```
Scan complete

Structure:
  Source roots: 1 main + 14 sub-libraries + 4 vendored
  Total source files: 950
  Modules identified: 21 (+ 5 added from gap scan)

Results:
  Files scanned: 200 (skipped 0 unchanged)
  Rules extracted: 400 (12 marked [speculative])
  Business modules: 21
  Core classes indexed: 150+

Updated files:
  - biz_scan/index.md
  - biz_scan/.project-structure.json
  - biz_scan/modules/user.md (25 rules, 8 classes)
  - biz_scan/modules/reader/ (50 rules, 20 classes)
  - ...
  - biz_scan/.scan-cache.json

Coverage:
  - Layer 0-1 (entry/bootstrap): 100%
  - Layer 2 (business logic): 100%
  - Layer 3 (data layer): 95% (5 pure-technical files skipped)
  - Sub-libraries: 14/14 scanned
  - Gap scan: checked 39 unassigned files, 5 had business logic → added

Next run will auto-skip unchanged files and only scan new/modified ones.
```

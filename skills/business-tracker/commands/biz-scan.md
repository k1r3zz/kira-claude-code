Scan the current codebase, extract business logic rules, and generate/update business documentation. Supports full scan and incremental scan (skips unchanged files via git hash cache).

First scan auto-initializes (creates `biz_scan/` directory). No separate init step needed.

Scans run in automatic batched loops until complete or stopped by the user.

Optional argument: `biz-scan [path]` — scope scan to a specific path, e.g. `biz-scan lib/features/payment/`

---

## Pre-scan: Load Scan Strategy

Read this skill's `references/scan-rules/common.md` to understand:
- File priority system (P0/P1/P2/Skip)
- Business logic signal patterns
- Batching strategy and cache format

---

## Step 0: Auto-Initialize (first scan only)

Check if `biz_scan/` exists in project root:

- **Does not exist** (first use):
  ```bash
  mkdir -p biz_scan/modules
  ```
  Then continue to Step 1.

- **Already exists**: Skip, go to Step 1.

## Step 1: Project Detection

- Read project root directory structure, detect project type per `common.md` Section 1
- Load the corresponding stack-specific rules from this skill's `references/scan-rules/<stack>.md`
- For monorepos, detect each subdirectory's stack and apply corresponding rules
- Read CLAUDE.md, README.md for existing business context
- Determine business code directories based on stack-specific file priorities

## Step 2: Build Scan Manifest

```bash
# Get all git-tracked files with hashes
git ls-files -s
```

- Filter to business-relevant files (using stack-specific priority rules)
- If user specified a path, keep only files under that path
- Read `biz_scan/.scan-cache.json` (if exists)
- Compare git hashes:
  - Same hash → mark "skip"
  - Different hash or new file → mark "needs scan"
  - In cache but not in git → mark "deleted"

Output scan plan and **immediately start scanning** (don't wait for user confirmation):
```
Scan plan:
  To scan: 15 files (P0: 6, P1: 5, P2: 4)
  Skipped (unchanged): 8 files
  Deleted: 1 file
  Estimated: 2 batches
  Starting batch 1...
```

## Step 3: Batched Scan Loop

Sort by priority, 8-10 files per batch, keep same-module files together.

### Each batch:

**3a. Read this batch's source files**

**3b. Extract business logic (product-doc style)**

Follow `common.md` Section 3 "Extraction Principles" strictly:

- **Extract flows**: Chain multiple functions/files into end-to-end business flows with numbered steps
- **Extract rules**: Identify business constraints, conditions, limits — describe "when X, then Y"
- **Extract state transitions**: If state machines or lifecycles exist, describe states and triggers
- **Extract cross-module interactions**: Record module dependencies and data flow
- **Extract special states**: A/B tests, disabled features, migrating logic
- **Do NOT extract**: Class names, field lists, method signatures, API parameter details, UI layout
- Mark uncertain rules with `[speculative]`
- **Group by business module**: Determine which module each rule belongs to (e.g. payment, user, reader)
- Code locations: only top-level entry files (3-5 per module), not every implementation file

**3c. Write module files (progressive disclosure + auto-split)**

For each affected module:
- If `biz_scan/modules/<module_name>.md` doesn't exist:
  Read this skill's `references/module-template.md`, create from template
- If it exists:
  Find the relevant section, incrementally update (don't overwrite other sections)

**Auto-split check** (after each batch write):
- Check module file line count. If over **150 lines**:
  1. Create `biz_scan/modules/<module_name>/` directory
  2. Split content into sub-files by business topic
  3. Replace original module file with index format (overview + sub-file reference table)
  4. Keep cross-module interactions, special states, and code entries in the index file
- See `references/module-template.md` for split template

**3d. Update index.md**

- If `biz_scan/index.md` doesn't exist:
  Read this skill's `references/index-template.md`, create from template
- Update module index: ensure each module has a one-line summary + `@biz_scan/modules/<module_name>.md` reference
- Update scan date in header

**3e. Update cache**
- Write this batch's scanned files to `biz_scan/.scan-cache.json`
- Record git_hash, scan time, rules extracted count

**3f. Output progress**
```
Batch 1/2 complete
  This batch: 6 files, 8 rules extracted (payment: 3, reader: 5)
  Total: 6/15 files, 8 rules
  Remaining: 9 files
  Continuing to next batch...
```

### Loop until:
- All files scanned → go to Step 4
- User stops → save progress, output "Scan paused. Next biz-scan will resume from checkpoint."

## Step 4: Handle Deleted Files

For files marked "deleted" in cache:
- Find related rules in the corresponding module file
- Add tag: `[source file deleted — please verify if this rule is still valid]`
- List these rules for user confirmation

## Step 5: Generate Coverage Report

Generate/update the coverage report at the end of `biz_scan/index.md` (format per `common.md` Section 7).

Report includes:
- Scanned files list with extracted rule counts
- Skipped files (unchanged hash)
- Unscanned low-priority files
- Deleted files and affected rules

## Step 6: Final Summary

```
Scan complete

Results:
  Files scanned: 15 (skipped 8 unchanged)
  Rules extracted: 18 (3 marked [speculative])
  Business modules: 5 (payment, reader, user, recommend, notification)

Updated files:
  - biz_scan/index.md
  - biz_scan/modules/payment.md (7 rules)
  - biz_scan/modules/reader.md (5 rules)
  - biz_scan/modules/user.md (4 rules)
  - biz_scan/modules/recommend.md (1 rule)
  - biz_scan/modules/notification.md (1 rule)
  - biz_scan/.scan-cache.json

Attention needed:
  - 3 [speculative] rules need human confirmation
  - 1 deleted file's rules need review
  - 12 low-priority files not scanned (to cover them: biz scan lib/utils/)

Next run will auto-skip unchanged files and only scan new/modified ones.
```

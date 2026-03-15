Analyze code changes in the current session, identify business logic changes, and update business documentation.

**Also handles corrections**: If existing rules don't match actual code behavior, fix them too.

---

## Pre-check

Check if `biz_scan/index.md` exists:
- **Does not exist** → output message and stop:
  ```
  No business scan found. Please run `biz scan` first to create the initial documentation.
  ```
- **Exists** → continue.

---

## Pre-load: Scan Strategy

Read this skill's `references/scan-rules/common.md` for business logic signal patterns.

---

## Step 1: Get Change Scope

Try in priority order:

```bash
# 1. Unstaged changes
git diff --name-only

# 2. Staged but uncommitted
git diff --cached --name-only

# 3. Last commit
git log -1 --name-only --pretty=format:""
```

If none found, use session context (what was discussed, which files were modified).

## Step 2: Filter Business-Relevant Changes

Using file priority rules from scan-rules:
- P0/P1 file changes → analyze in detail
- P2 file changes → quick check
- Skip-category files → ignore

## Step 3: Analyze Business Impact

For each business-relevant file, check the actual diff with `git diff`.

### Is It a Business Change?

**Yes — business change:**
- Payment/pricing/discount logic changes
- User permission/tier rule changes
- Core flow state transition changes
- Data validation rule changes
- Business constant/enum additions or modifications
- New business module or feature
- Business API interface changes

**No — ignore:**
- Pure UI adjustments
- Code refactoring (behavior unchanged)
- Bug fixes (restoring expected behavior, not changing rules)
- Performance optimization, dependency upgrades

### If No Business Changes Found

```
No business logic changes detected in this session.
Changed files: [list]
Reason: [brief explanation]
```

Then proceed to Step 5 (still run correction check).

## Step 4: Record Business Changes

### 4a. Update changelog

Read this skill's `references/changelog-template.md` for the template.

Generate a change entry including:
- Change title (one-line summary)
- Old rule → New rule
- Change reason (extract from session context; if unknown, mark "reason TBD")
- Affected scope (file paths)
- Related info (ticket links, commit hash, etc. if available)

**Prepend to `biz_scan/changelog.md`** (newest first).
If file doesn't exist, create it.

### 4b. Update module files

- Determine which business modules are affected
- Read the corresponding `biz_scan/modules/<module_name>.md`
- Update affected rule content
- Leave other rules untouched
- If a new module is involved, create a new module file (from `references/module-template.md`)

### 4c. Update index.md

- Read `biz_scan/index.md`
- Update "last incremental update" date
- If new modules were added, add index entry (one-line summary + `@biz_scan/modules/<module_name>.md`)

### 4d. Update scan cache

Update `biz_scan/.scan-cache.json` with new git hashes for affected files.

## Step 5: Correction Check

**Run this step regardless of whether new business changes were found.**

Read `biz_scan/index.md`, identify modules related to the changed files, load the corresponding `biz_scan/modules/<module_name>.md`, and compare against actual code:
- Do documented rules match actual code behavior?
- Are there outdated rules (code changed but docs not updated)?
- Can any `[speculative]` rules now be confirmed?

### If Inconsistencies Found

```
Found doc-code inconsistencies, corrected:

1. payment module > Pricing:
   Doc said: VIP $15/mo
   Code actual: VIP $18/mo (likely a price change that wasn't recorded)
   → Updated biz_scan/modules/payment.md

2. user module > Device limit:
   Doc marked [speculative]: max 3 devices
   Code confirms: indeed 3 devices (MAX_DEVICES = 3)
   → Removed [speculative] tag
```

Also append correction records to changelog (clearly marked as corrections, not business changes).

## Step 6: Output Summary

```
Business changes recorded

Changes: [one-line summary]
Corrections: [fixed N inconsistencies] (if any)
Updated files:
  - biz_scan/changelog.md (1 entry added)
  - biz_scan/modules/payment.md (updated "Pricing")
  - biz_scan/index.md (date updated)
```

## Edge Cases

- **Mixed changes** (business + technical): only record the business parts
- **Uncertain if it's a business change**: ask the user to confirm
- **Large number of changes**: list by module, let user confirm which to record
- **biz_scan/ directory missing**: prompt to run `biz scan` first

# [Module Name]

> One-line summary: [What problem does this module solve? For what user scenario?]
> Last updated: [date]

---

<!--
  Writing principles:
  1. Write like a product doc, not a code manual
  2. Cover ALL business logic — every flow, every branch, every constraint
  3. Organize by flows and rules, not by class names or method signatures
  4. Uncertain rules get [speculative] tag
  5. Cross-module references use @biz_scan/modules/xxx.md format
  6. Core Classes table is MANDATORY — it's the bridge between docs and code
  7. File coverage is tracked in .scan-cache.json, NOT in this md file

  Auto-split rule:
  - When this file exceeds 150 lines, split by business topic into sub-files
  - This file becomes a module index (overview + sub-file reference list)
  - Sub-files go in biz_scan/modules/<module_name>/
  - Each sub-file covers one business topic, using the "Sub-file template" below
  - Core Classes section STAYS in the index file (not split)
-->

## Core Classes

<!--
  MANDATORY section. Maps business concepts to actual code locations.
  Purpose: enable code navigation and help developers know where to START reading.

  Rules:
  - Include every class/type that implements business logic in this module
  - Include classes from sub-libraries that belong to this module's domain
  - "Role" is a 5-10 word description of what the class DOES, not what it IS
  - Keep sorted by layer (business → data → model → util)
  - Mark the 1-3 most important entry classes with ★ — these are where a developer should start reading
-->

| Class | File | Role |
|-------|------|------|
| ★ [MainEntryClass] | [path/to/file.ext] | [What this class does — START HERE] |
| [ClassName] | [path/to/file.ext] | [What this class does in 5-10 words] |

---

## Core Flows

<!--
  Most important section. Must cover every business flow in this module.
  Each flow uses numbered steps — like the text version of a product flow diagram.
  Include: trigger conditions, prerequisites, all steps, all branches, constraints, error handling.
  Exclude: specific function names, parameter names, field mappings.

  Detail level:
  - Every if/else branch must be visible
  - Business conditions must be explicit
  - Data passing between steps: describe WHAT passes, not WHICH variable

  Cross-module references:
  - When a flow triggers logic in another module, use @biz_scan/modules/xxx.md
  - Example: "Triggers wallet refresh — see @biz_scan/modules/wallet.md"
-->

### Flow 1: [Flow Name]

> Trigger: [When does this flow start?]
> Prerequisites: [What must be true before executing?]

1. Step one
2. Step two
   - If [condition A]: take branch X
     - X.1 ...
     - X.2 ...
   - If [condition B]: take branch Y
3. Step three

**Constraints:**
- [Rules that must not be violated]
- [Boundary conditions, limits]

**Analytics / Side Effects:**
- [What tracking events fire, if any]
- [What local state gets written, if any]

---

## Business Rules

<!--
  Rules that exist independently of flows. E.g.: pricing, permissions, quotas, time rules.
  Each rule: one sentence describing "when X, then Y".
  Include the reason if known.
  Must cover ALL rules — don't skip any.
-->

- **[Rule Name]**: [When condition X, then Y happens]
  - Reason: [Why designed this way, if known]

---

## Data Flow & Storage

<!--
  How data moves, where it lives, how it syncs.
  Don't list fields — but do describe:
  - Storage mechanism (e.g. SQLite, Redis, SharedPreferences, localStorage)
  - Data lifecycle (when written, when cleared)
  - Relationships between data copies (local model vs API model vs UI model)
  - Data isolation strategy (e.g. per-user, per-tenant)
-->

- [Data storage description]

---

## State & Lifecycle

<!--
  If this module has state machines or clear lifecycles, describe them here.
  Text description of state transitions — no need for diagrams.
-->

- [Entity] states: [State A] → [State B] → [State C]
  - A → B trigger: ...
  - B → C trigger: ...

---

## Cross-Module Interactions

<!--
  How this module works with other modules.
  This is one of the hardest things to read from code — scattered across many files.
  Be specific about interaction scenarios and data flow direction.

  MUST use @biz_scan/modules/xxx.md references for every mentioned module.
-->

- **With [Module X]** (@biz_scan/modules/x.md): [Interaction pattern and scenario]
- **With [Module Y]** (@biz_scan/modules/y.md): [Interaction pattern and scenario]

---

## Current Special States

<!--
  Temporary info: A/B tests, disabled features, migrating logic, known hacks.
  These are things that would confuse someone reading the code — document them to save time.
-->

- [Description of current special situation]

---

<!--
  File coverage tracking:
  Which files belong to this module and their scan status are tracked in
  biz_scan/.scan-cache.json (under each file's "module" field), NOT in this md file.
  This keeps the md focused on business logic only.
-->

# ===== Templates for split-file scenario =====

## Module Index Template (replaces this file when content needs splitting)

<!--
# [Module Name]

> One-line summary: [What problem does this module solve?]
> Last updated: [date]

---

## Module Overview

[2-3 sentences describing the module's overall responsibility and architecture approach]

## Sub-topics

| Topic | Description | Details |
|-------|-------------|---------|
| Sync | Data incremental sync flow | @biz_scan/modules/module_name/sync.md |
| Purchase | In-app purchase and unlock | @biz_scan/modules/module_name/purchase.md |
| Settings | User preferences and config | @biz_scan/modules/module_name/settings.md |

## Core Classes

[STAYS in index — not split into sub-files. Mark 1-3 entry classes with ★]

| Class | File | Role |
|-------|------|------|
| ★ ... | ... | ... (START HERE) |
| ... | ... | ... |

## Cross-Module Interactions

[STAYS in index — this is module-level info]

- **With [Module X]** (@biz_scan/modules/x.md): ...

## Current Special States

[STAYS in index]

  (File coverage tracked in .scan-cache.json, not in md)
-->

## Sub-file Template

<!--
# [Module Name] — [Sub-topic Name]

> Parent module: @biz_scan/modules/module_name.md (or /index.md)
> Last updated: [date]

---

## Core Flows

[Same format as main template, but only flows for this sub-topic]

## Business Rules

[Same format as main template, but only rules for this sub-topic]

## Data Flow & Storage

[If this sub-topic has independent storage logic]
-->

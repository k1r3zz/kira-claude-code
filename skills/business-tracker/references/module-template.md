# [Module Name]

> One-line summary: [What problem does this module solve? For what user scenario?]
> Last updated: [date]

---

<!--
  Writing principles:
  1. Write like a product doc, not a code manual
  2. Cover ALL business logic — every flow, every branch, every constraint
  3. Organize by flows and rules, not by class names or method signatures
  4. Code locations: only top-level entry points, not every file
  5. Uncertain rules get [speculative] tag

  Auto-split rule:
  - When this file exceeds 150 lines, split by business topic into sub-files
  - This file becomes a module index (overview + sub-file reference list)
  - Sub-files go in biz_scan/modules/<module_name>/
  - Each sub-file covers one business topic, using the "Sub-file template" below
-->

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
  - Storage mechanism (e.g. Sembast, SQLite, SharedPreferences)
  - Data lifecycle (when written, when cleared)
  - Relationships between data copies (local model vs API model vs UI model)
  - Data isolation strategy (e.g. per-user)
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
-->

- **With [Module X]**: [Interaction pattern and scenario]
- **With [Module Y]**: [Interaction pattern and scenario]

---

## Current Special States

<!--
  Temporary info: A/B tests, disabled features, migrating logic, known hacks.
  These are things that would confuse someone reading the code — document them to save time.
-->

- [Description of current special situation]

---

## Code Entry Points

<!--
  Only the top-level entry files (3-5). Not every implementation file.
  Purpose: help people know where to START reading code, not replace code reading.
-->

- [Role description]: `path/to/entry.dart`

---
---

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
| Sync | Bookshelf data incremental sync flow | @biz_scan/modules/bookshelf/sync.md |
| Auto-pull | Auto-return to reader on app exit | @biz_scan/modules/bookshelf/auto-pull.md |
| Offline download | Chapter content download and storage | @biz_scan/modules/bookshelf/download.md |
| Purchased | Unlocked content display and navigation | @biz_scan/modules/bookshelf/purchased.md |

## Cross-Module Interactions

[Stays in index — this is module-level info]

## Current Special States

[Stays in index]

## Code Entry Points

[Stays in index]
-->

## Sub-file Template

<!--
# [Module Name] — [Sub-topic Name]

> Parent module: [Module Name]
> Last updated: [date]

---

## Core Flows

[Same format as main template, but only flows for this sub-topic]

## Business Rules

[Same format as main template, but only rules for this sub-topic]

## Data Flow & Storage

[If this sub-topic has independent storage logic]
-->

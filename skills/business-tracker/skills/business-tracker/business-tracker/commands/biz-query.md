Query business logic rules from the generated documentation. Locate relevant content and answer the user's question.

---

## Pre-check

Check if `biz_scan/index.md` exists:
- **Does not exist** → output message and stop:
  ```
  No business scan found. Please run `biz scan` first to create the initial documentation.
  ```
- **Exists** → continue.

---

## Steps

### 1. Load Index

Read `biz_scan/index.md` to understand which business modules exist.

### 2. Load Relevant Modules (progressive disclosure)

Based on the user's question, determine which modules are relevant:
- Only load the relevant `biz_scan/modules/<module_name>.md` file(s)
- **Do NOT load all module files at once**
- If the question involves change history, also read `biz_scan/changelog.md`

### 3. Answer

- Answer in natural language, citing specific rules from the docs
- Include code entry points for easy navigation
- Flag any `[speculative]` rules as unconfirmed
- If the docs don't cover the answer, try finding it directly in code, then suggest running `biz update` to add it to the docs

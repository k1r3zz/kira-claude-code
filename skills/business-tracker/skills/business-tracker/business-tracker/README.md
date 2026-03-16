# Business Tracker

**Extract living business-rule documentation from any codebase.**

Business logic hides in `if/else` branches, state machines, and scattered service files. New team members spend weeks reverse-engineering rules that nobody ever wrote down. Business Tracker fixes this — it scans your code, extracts the rules, and keeps them up to date as your code evolves.

## What It Does

| Command | What happens |
|---------|-------------|
| `biz scan` | Scans your codebase, extracts business rules, and generates structured documentation organized by module |
| `biz update` | Detects business logic changes in your current session and updates the docs + changelog |
| `biz query` | Answers questions about your business rules by consulting the generated docs |
| `关联 openspec` | Links `biz_scan/index.md` as context into `openspec/config.yaml`, so OpenSpec artifacts get business logic context |

## Before / After

**Before:** A developer joins the team and asks "how does our subscription pricing work?"
→ Answer: "Ask Mike, he wrote that 2 years ago. Or read through 15 files in `lib/features/payment/`."

**After:** The developer runs `biz query subscription pricing` and gets:
```
## Subscription Flow
1. User taps "Subscribe" → app checks current subscription status
2. If already subscribed → show "Manage Subscription" screen
3. If new user → show pricing tiers:
   - Basic: $4.99/mo (50 chapters/day)
   - VIP: $9.99/mo (unlimited + ad-free)
   - Annual VIP: $89.99/yr (save 25%)
4. After payment confirmed → update user role, unlock content, record purchase event
   ...

Rules:
- Trial period: 3 days free for first-time users
- Downgrade: takes effect at end of current billing cycle
- [speculative] Family sharing: up to 3 devices per account
```

## Key Features

- **Multi-stack support** — Flutter, Node.js, Go, Python, Java, .NET, Rust, Ruby, PHP, iOS
- **Incremental scanning** — Uses git hash caching so re-scans only touch changed files
- **Auto-split** — Large modules auto-split into sub-files by business topic
- **Change tracking** — Records what changed, old rule → new rule, and why
- **Uncertainty marking** — Rules inferred from code (not confirmed by humans) are tagged `[speculative]`
- **Coverage reporting** — See exactly what was scanned, skipped, and remaining

## Quick Start

```bash
# Install
cp -r business-tracker .claude/skills/

# First scan
/business-tracker biz scan

# After making changes
/business-tracker biz update

# Ask questions
/business-tracker biz query "how does the payment system work?"
```

## Output Example

After scanning, you get a `biz_scan/` directory:

```
biz_scan/
├── index.md              # "Table of contents" for all business logic
├── modules/
│   ├── payment.md        # Subscription, pricing, purchase flows
│   ├── reader.md         # Reading progress, chapter unlocking
│   ├── user.md           # Registration, roles, device limits
│   └── notification.md   # Push notification rules
├── changelog.md          # History of business rule changes
└── .scan-cache.json      # Scan progress (auto-managed)
```

Each module file reads like a product spec — not an API reference.

## Who Is This For?

- **Teams inheriting legacy codebases** — Finally understand what the code actually does
- **Technical leads** — Keep business logic documented without manual effort
- **New team members** — Onboard faster by querying rules instead of reading every file
- **Product managers** — Verify that code matches intended business requirements

## Optional: Auto-Remind on Session End

Add a Stop hook to remind yourself to run `biz update` after coding sessions:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "echo 'If you changed business logic, run: /business-tracker biz update'"
      }]
    }]
  }
}
```

## License

MIT

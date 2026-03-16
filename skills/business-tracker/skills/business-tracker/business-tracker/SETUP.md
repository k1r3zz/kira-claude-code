# Business Tracker — Setup

Works with any tech stack (Flutter, Node.js, Go, Python, Java, .NET, Rust, Ruby, PHP, iOS).

## Install

In Claude Code, run:
```
/plugin
```
Then select `business-tracker` from the plugin list to install.

After installation, run `/business-tracker biz scan` in your project. The first scan auto-detects your project type and creates the `biz_scan/` directory.

## Team Members

After `git pull`, team members can use `biz query` and `biz update` immediately — the `biz_scan/` directory is already in the repo.

## Optional: Stop Hook Reminder

To get a reminder to run `biz update` at the end of each session, add this to `.claude/settings.local.json`:

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

> **Note:** This triggers on every session stop. If you find it noisy, remove the hook and run `biz update` manually when needed.

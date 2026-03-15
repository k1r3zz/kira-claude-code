# kira-claude-code

Kira 的 Claude Code 技能集合，用于增强 AI Agent 的能力。

## Install

```bash
/plugin marketplace add https://github.com/k1r3zz/kira-claude-code
/plugin install business-tracker
```

## Skills

### 📊 business-tracker

自动提取、持续追踪代码库中的业务逻辑，生成活文档。

支持 10+ 技术栈：Flutter、Node.js、Go、Python、Java/Kotlin、.NET、Rust、Ruby、PHP、iOS。

| 命令 | 说明 |
|------|------|
| `biz scan [path]` | 扫描代码库，提取所有业务规则并生成文档 |
| `biz update` | 记录业务变更，追加到 changelog |
| `biz query <关键词>` | 查询已有的业务规则 |

## License

MIT

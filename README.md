# kira-cluade-code

Kira 的 Claude Code 技能集合，用于增强 AI Agent 的能力。

## Install

```bash
# 添加技能源
/plugin marketplace add https://github.com/k1r3zz/kira-cluade-code

# 安装技能
/plugin install kira-cluade-code
```

或手动克隆：

```bash
git clone https://github.com/k1r3zz/kira-cluade-code.git
# 将 skills/ 下的技能目录复制或软链接到你的 Claude Code skills 目录
```

## 目录结构

```
kira-cluade-code/
├── README.md
├── config.yaml
└── skills/
    └── business-tracker/    # 业务逻辑追踪技能
```

## Skills

### 📊 business-tracker

**自动提取、持续追踪代码库中的业务逻辑，生成活文档。**

支持 10+ 技术栈：Flutter、Node.js、Go、Python、Java/Kotlin、.NET、Rust、Ruby、PHP、iOS (Swift/ObjC)。

**核心命令：**

| 命令 | 说明 |
|------|------|
| `biz scan [path]` | 扫描代码库，自动提取所有业务规则并生成文档 |
| `biz update` | 记录业务变更，追加到 changelog |
| `biz query <关键词>` | 查询已有的业务规则（如 "支付规则是什么？"） |

**特点：**
- 按业务模块组织，而非代码结构
- 渐进式加载，对 AI 上下文友好
- 自动检测项目类型，加载对应扫描规则
- 支持 Monorepo 多子项目独立检测
- 不确定的规则标记 `[speculative]`，鼓励人工确认

## License

MIT

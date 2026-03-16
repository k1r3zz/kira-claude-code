# biz-link-openspec — 关联业务文档到 OpenSpec

将 `biz_scan/index.md` 作为知识库上下文写入 `openspec/config.yaml`，使 OpenSpec 创建工件时自动获取业务逻辑上下文。

## 执行步骤

### 1. 检查前置条件

1. 检查 `biz_scan/index.md` 是否存在
   - **不存在** → 提示用户：`biz_scan/index.md 不存在，请先运行 /business-tracker biz scan 扫描业务逻辑。` → 停止
2. 检查 `openspec/config.yaml` 是否存在
   - **不存在** → 提示用户：`openspec/config.yaml 不存在，请先初始化 OpenSpec。` → 停止

### 2. 读取并更新 config.yaml

1. 读取 `openspec/config.yaml` 的完整内容
2. 查找 `context` 字段：
   - **已存在 `context` 字段**：检查内容中是否已包含 `biz_scan/index.md` 相关描述
     - 已包含 → 提示"已关联，无需重复操作"→ 停止
     - 未包含 → 在现有 context 内容末尾追加业务知识库指令
   - **不存在 `context` 字段**：添加 `context` 字段
3. `context` 字段写入以下内容（使用 YAML 多行字符串 `|`）：

```yaml
context: |
  业务知识库在 biz_scan/index.md，propose 前必须先阅读。
```

4. 如果 context 已有其他内容，则追加这一行，不覆盖原有内容
5. 写回 `openspec/config.yaml`，保持其余字段不变

### 3. 输出结果

输出以下信息：

```
✅ 已将 biz_scan/index.md 关联到 openspec/config.yaml 的 context 字段。
OpenSpec 创建工件时将自动加载业务逻辑上下文。
```

## 注意事项

- 不要覆盖 `config.yaml` 中已有的其他 context 条目
- 保持 YAML 文件的格式和注释
- 使用相对路径 `biz_scan/index.md`（相对于项目根目录）

# LLM Wiki 知识库 — Agent 规则入口（AGENTS.md）

> 本文件遵循 [AGENTS.md 开放标准](https://agents.md/)（Linux Foundation / Agentic AI Foundation 托管），为纯 Markdown 薄壳。

## 必读

在操作本仓库之前，必须先完整阅读并严格遵守：

**`docs/agent-rules.md`**

该文件是全部规则的**唯一事实来源**，涵盖：

- 语言设定：始终使用简体中文思考、回复和编写知识库内容
- 目录权限边界：`raw/` 绝对只读（唯一真相来源）、`assets/` 媒体层、`wiki/` 为 Agent 专属可写层
- Wiki Schema：`wiki/index.md`（总目录）与 `wiki/log.md`（操作日志，Append-only）的维护契约
- 内容分类：`wiki/concepts/`、`wiki/entities/`、`wiki/sources/`、`wiki/syntheses/`
- 强制双向链接：每个 wiki 页面必须包含 `## 关联连接` 区域，禁止孤岛页面
- 矛盾处理：冲突不静默覆盖，新建 `## 知识冲突` 区块保留对比
- 页面 Frontmatter 规范：`title / type / tags / sources / last_updated`
- 工作流：`/ingest`、`/query`、`/lint` 的核心逻辑

若同时使用 Claude Code，`.claude/skills/` 下另有专用技能可配合工作流使用。

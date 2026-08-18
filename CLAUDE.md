# LLM Wiki 知识库 — Agent 规则入口（Claude Code 版）

> 本文件为规则入口（薄壳），全部规则见唯一事实来源：

@docs/agent-rules.md

## 必读

- 立即完整阅读 `docs/agent-rules.md`，其中全部规则（语言设定、目录权限边界、Wiki Schema、双链、矛盾处理、Frontmatter、工作流）必须严格遵守。
- 本文件不再重复规则正文，避免双份内容漂移。

## Claude Code 专有说明

- 本仓库在 `.claude/skills/` 提供专用技能：`ingest/`（raw→wiki 编译）、`query/`（带双链引用检索）、`lint/`（知识体检）、`obsidian-markdown/`、`mermaid-visualizer/` 等，可配合规则文件中的工作流使用。
- `@import` 语法（如上方的 `@docs/agent-rules.md`）为 Claude Code 专有特性；使用其他工具时请读取 `AGENTS.md`。

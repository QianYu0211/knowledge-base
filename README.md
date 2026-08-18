# LLM Wiki 知识库

本项目是一个基于 [Karpathy 的 LLM Wiki 理念](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 构建的 Obsidian 知识库。

## 核心理念

将碎片化的信息编译成**结构化、高度相互链接**的知识网络，便于 AI 辅助学习和研究。

## 目录结构

```
🏛️ 你的知识库文件夹 (LLM-Wiki-Vault)
├── 🖼️ assets/                 ← 统一媒体资源层：存放图片、PDF、附件（Obsidian设置附件路径至此）
│
├── 📥 raw/                    ← 原始资料收件箱（只读事实层，文件处理后移动至 archive）
│   ├── 📄 01-articles/        ← 网页剪藏、技术文章 (.md)
│   ├── 🎓 02-papers/          ← 论文、深度研报、PDF文档
│   ├── 🎙️ 03-transcripts/     ← 视频/播客转录文本、会议记录
│   └── 🗃️ 09-archive/         ← 已归档区：`/ingest` 执行成功后，源文件自动移动至此
│
├── 🧠 wiki/                  ← 知识编译输出层（LLM 拥有完全写权限，人类阅读层）
│   ├── 📑 index.md           ← 全局内容字典：记录所有 wiki 页面及其一句话索引
│   ├── 📜 log.md             ← 行为流水线：以 Grep-friendly 格式记录 ingest/query 历史
│   ├── 🏗️ concepts/          ← 抽象层：方法论、架构模式、第一性原理
│   ├── 👥 entities/          ← 实体层：人名、公司、工具软件、项目
│   ├── 🔍 sources/           ← 摘要层：针对 raw 文件的一对一核心观点提炼
│   └── 💎 syntheses/         ← 综合层：针对复杂提问生成的深度研究报告
│
├── 📋 AGENTS.md              ← 跨工具规则入口（AGENTS.md 开放标准，Linux Foundation 托管）
│
├── 🤖 CLAUDE.md              ← Claude Code 规则入口（@import 引入 docs/agent-rules.md）
│
├── 📚 docs/                  ← 规则层
│   └── 📄 agent-rules.md     ← 全部规则的唯一事实来源（SSOT）：语言协议、权限边界、Wiki Schema
│
└── ⚙️ .claude/               ← Claude Code 官方配置目录
    └── 🛠️ skills/            ← Agent Skill 中心（供 Claude Code 使用，其他工具按 docs/agent-rules.md 工作流执行）
        ├── ⚙️ ingest/        ← 自定义：编译收件箱 raw 文件到 wiki，并执行 09-archive 归档
        ├── 🔎 query/         ← 自定义：检索 wiki/index 并读取相关页面，生成带双链引用的回答
        ├── 🩺 lint/          ← 自定义：知识体检，修复死链、补充 index、发现认知冲突
        ├── 📝 obsidian-markdown/  ← Obsidian官方：Obsidian 风格 Markdown（双链/嵌入/引用块/属性）
        └── 📊 mermaid-visualizer/ ← 自定义：文本转专业 Mermaid 图表（流程图/架构图/思维导图）
```

## 规则文件体系

| 文件 | 受众 | 职责 |
|---|---|---|
| `docs/agent-rules.md` | 所有 Agent | **唯一事实来源 (SSOT)**：语言协议、权限边界、Wiki Schema、双链、矛盾处理、Frontmatter、工作流 |
| `CLAUDE.md` | Claude Code | 薄壳入口：`@import` 引入规则文件，并说明 `.claude/skills/` 专用技能 |
| `AGENTS.md` | Codex / Gemini CLI / Cursor / pi 等 | 薄壳入口（纯 Markdown 开放标准）：指引阅读规则文件 |

**设计原则**：规则正文只维护一份（`docs/agent-rules.md`），入口文件只做引用，避免多份规则内容漂移。

## 使用方式

在 Obsidian 中打开本 vault，使用任意主流 AI 编码 Agent 执行操作（Claude Code、OpenAI Codex、Gemini CLI、Cursor、pi 等）。所有 Agent 均通过 `AGENTS.md` / `CLAUDE.md` 入口读取 `docs/agent-rules.md` 中的规则。

### 常用命令

- `/query <问题>` — 在知识库中搜索相关内容
- `/ingest <路径>` — 将新的原始资料编译到知识库
- `/lint` — 检查知识库健康度（死链、孤儿页面）

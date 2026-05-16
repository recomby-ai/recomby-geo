# Agent 集合

主流 AI agent / coding CLI 列表，作为 AI 员工方案的「身体」一层。每条标注是否开源、是否支持本地模型、扩展机制。

> 📌 信息基于 2025-2026 自动调研，链接与许可证以官方页面为准。欢迎 PR 补充 / 勘误。
>
> **Verified: 2026-05** — 所有条目最后核对于此日期；提交新条目 / 勘误时请同步更新自己条目的 verified 日期。

---

## 国外（开源 / 社区主导）

| 名称 | 链接 | 一句话定位 | 开源 | 本地模型 | 扩展机制 |
|------|------|-----------|------|---------|---------|
| **Claude Code** | [claude.com/product/claude-code](https://claude.com/product/claude-code) | Anthropic 官方终端 AI 编码代理，支持 worktree / subagents | 客户端闭源 / SDK 开源 | × | Skills + Plugins + MCP |
| **OpenAI Codex CLI** | [github.com/openai/codex](https://github.com/openai/codex) | OpenAI 官方开源 coding agent CLI | ✓ Apache 2.0 | ✓ | Tools / MCP |
| **OpenCode** | [opencode.ai](https://opencode.ai/) | 多 LLM 后端编码 agent，隐私优先架构 | ✓ | ✓ | 多智能体 |
| **Cline** | [github.com/cline/cline](https://github.com/cline/cline) | 开源 AI 编码 agent，支持 30+ LLM + MCP | ✓ MIT | ✓ | MCP |
| **Aider** | [aider.chat](https://aider.chat/) | 终端结对编程，原子化 Git 提交 | ✓ Apache 2.0 | ✓ (Ollama) | 插件 |
| **Goose** | [goose-docs.ai](https://goose-docs.ai/) | Block 开源编程 agent，15+ LLM + 70+ MCP 扩展 | ✓ Apache 2.0 | ✓ | MCP 生态 |
| **Continue.dev** | [continue.dev](https://continue.dev/) | VS Code / JetBrains 开源插件，本地优先 | ✓ Apache 2.0 | ✓ | 自定义 LLM / 工具 |
| **gptme** | [gptme.ai](https://gptme.ai/) | 首批开源终端 agent，持久化自主运行 | ✓ MIT | ✓ (llama.cpp) | 工具扩展 |
| **Cursor CLI** | [cursor.com](https://cursor.com/) | IDE 原生 Composer 模型，多 agent 并行 | × 商业 | × | 模型可切换 |

---

## 国内

| 名称 | 链接 | 一句话定位 | 开源 | 本地模型 | 扩展机制 |
|------|------|-----------|------|---------|---------|
| **字节 Trae-Agent** | [github.com/bytedance/trae-agent](https://github.com/bytedance/trae-agent) | 字节开源通用工程 agent | ✓ | × | 工具系统 |
| **腾讯 CodeBuddy** | [codebuddy.cn](https://www.codebuddy.cn/) | 腾讯混元驱动 agent，首家国产 MCP 支持 | × 云服务 | × | MCP |
| **阿里 通义灵码** | [lingma.aliyun.com](https://lingma.aliyun.com/) | 阿里云编程助手，100+ 语言，工程级变更 | × 云服务 | × | IDE 插件 |
| **百度 文心快码** | [comate.baidu.com](https://comate.baidu.com/) | 百度多智能体编程 agent，Craft 无代码模式 | × 云服务 | × | IDE 集成 |

---

## 选型建议（用于本项目）

按本项目 **Local-first + Skills 可加载 + MCP 接入办公 CLI** 三条原则筛选：

| 推荐场景 | 首选 | 理由 |
|----------|------|------|
| 国外团队、需要稳定 skills 生态 | **Claude Code** | Skills + Plugins + MCP 三层扩展最成熟 |
| 严格本地化 / 离线 | **Cline / Goose / Aider** | 任意本地 LLM + MCP |
| 国内团队、本地化 + 开源 | **字节 Trae-Agent** | 国内开源选择，可对接本地模型路径 |
| 想跨多 LLM 测试 | **OpenCode / Cline** | LLM-agnostic |

---

## 贡献

新增 / 勘误：提交 PR 修改本文件。每条请提供：名称、官方链接、≤30 字定位、许可证、本地模型支持、扩展机制。

# 办公 CLI 集合

让 agent 接入业务数据 + 操作客户协作空间的 CLI / MCP server 列表，作为 AI 员工方案的「数据源 + 操作脚手架」一层。

> 📌 信息基于 2025-2026 自动调研，链接与许可证以官方页面为准。MCP（Model Context Protocol）是 Anthropic 提出、目前事实标准的 agent 工具协议。欢迎 PR 补充 / 勘误。
>
> **Verified: 2026-05** — 所有条目最后核对于此日期；提交新条目 / 勘误时请同步更新自己条目的 verified 日期。

---

## 国内

| 产品 | 链接 | 一句话定位 | CLI | MCP | 官方 / 社区 | 粒度 |
|------|------|-----------|-----|-----|------------|------|
| **飞书 (Lark)** | [larksuite/lark-openapi-mcp](https://github.com/larksuite/lark-openapi-mcp) | 官方 OpenAPI MCP，文档 / 日历 / 任务 / Base | ✓ | ✓ | 官方 | 读写 |
| **钉钉 (DingTalk)** | [dingtalk-workspace-cli](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli) | dws CLI，AI-native 设计 + 审计 | ✓ | ✓ | 官方 + 社区 | 读写 |
| **企业微信** | [shellus/qiye_wechat_mcp](https://github.com/shellus/qiye_wechat_mcp) | 社区 MCP，企业消息 / 通讯录 | ✓ | ✓ | 社区 | 读写 |
| **语雀 (Yuque)** | [yuque/yuque-mcp-server](https://github.com/yuque/yuque-mcp-server) | 官方 MCP，知识库读写 / 搜索 / 分析 | ✓ | ✓ | 官方 | 读写 |

---

## 国外

| 产品 | 链接 | 一句话定位 | CLI | MCP | 粒度 |
|------|------|-----------|-----|-----|------|
| **Slack** | [docs.slack.dev/ai/slack-mcp-server](https://docs.slack.dev/ai/slack-mcp-server/) | 官方 MCP，搜索 / 消息 / Canvas | 第三方 | ✓ 官方 | 读写 |
| **Notion** | [developers.notion.com/docs/mcp](https://developers.notion.com/docs/mcp) | 官方 MCP，页面 / 数据库完整操作 | 社区 | ✓ 官方 | 读写 |
| **GitHub** | [github/github-mcp-server](https://github.com/github/github-mcp-server) | 官方 MCP，基于 `gh` CLI，PR / Issue / 代码 | ✓ (gh) | ✓ 官方 | 读写 |
| **Linear** | [linear.app/docs/mcp](https://linear.app/docs/mcp) | 官方 MCP，任务 / 周期 / 项目 | 第三方 | ✓ 官方 | 读写 |
| **Google Workspace** | [developers.google.com/workspace/guides/configure-mcp-servers](https://developers.google.com/workspace/guides/configure-mcp-servers) | 官方 MCP + CLI，Gmail / Drive / Calendar / Docs | ✓ | ✓ 官方 | 读写 |
| **Jira / Confluence** | [atlassian/atlassian-mcp-server](https://github.com/atlassian/atlassian-mcp-server) | Atlassian 官方 MCP（GA），Issue / Page / Compass | 第三方 | ✓ 官方 | 读写 |
| **Microsoft 365 / Teams** | [microsoft-agent-365](https://learn.microsoft.com/en-us/microsoft-agent-365/tooling-servers-overview) | 官方 Work IQ MCP，邮件 / 日历 / Teams / 文档 | ✓ (m365 CLI) | ✓ 官方 | 读写 |
| **ClickUp** | [developer.clickup.com/docs/connect-an-ai-assistant-to-clickups-mcp-server](https://developer.clickup.com/docs/connect-an-ai-assistant-to-clickups-mcp-server) | 官方 MCP + CLI，任务 / 列表 / 文档 | ✓ | ✓ 官方 | 读写 |
| **Asana** | 社区实现 | 社区 MCP，任务 / 项目（官方尚无原生支持） | 待验证 | ✓ 社区 | 读写 |

---

## 选型建议（用于本项目）

按本项目 **数据不出本地 + 官方维护优先 + MCP 标准协议** 原则：

| 客户类型 | 首选 CLI / MCP |
|----------|----------------|
| 国内一般企业 | **飞书 MCP** + **钉钉 dws CLI** |
| 国内技术团队 / 知识库重 | **语雀 MCP** + **飞书 MCP** |
| 国外 SaaS 重度用户 | **Slack MCP** + **Notion MCP** + **GitHub MCP** |
| 跨国企业 | **Microsoft 365 MCP** + **Google Workspace MCP** |
| 研发驱动型 | **GitHub MCP** + **Linear MCP** + **Jira MCP** |

---

## 关于 MCP

MCP（Model Context Protocol）是 Anthropic 提出的 agent 工具调用标准协议，现已被 OpenAI、Google、字节、腾讯等主流厂商支持。本集合优先收录支持 MCP 的实现，理由：

- 跨 agent 通用（CC、Codex、Cline、Goose 等都能用同一 MCP server）
- 协议层标准化，不绑死某一家 agent
- 部署上 agent 跟 MCP server 都在本地，数据不上第三方云

---

## 贡献

新增 / 勘误：提交 PR 修改本文件。每条请提供：产品名、官方链接、≤30 字定位、是否有 CLI / MCP、官方还是社区维护、数据访问粒度（只读 / 读写）。

# 贡献指南

欢迎为 recomby-geo 提交 PR。本仓库分两层，三类贡献对应不同规则。

---

## 1. 给 `agents.md` / `clis.md` 新增 / 勘误条目

每条必须提供以下字段（顺序参照现有表格）：

- **名称** — 产品官方名
- **官方链接** — 优先官网；GitHub 项目链接也可
- **一句话定位** — ≤30 字，避免市场话术
- **许可证** — MIT / Apache 2.0 / 商业 / 闭源 等
- **本地模型支持** — ✓ / × / 备注
- **扩展机制** — Skills / Plugins / MCP / Tools / 其他

PR 末尾请附 `Verified: YYYY-MM`（你最后一次核对官方页面的年月），用于后续刷新。

---

## 2. 给 `plugins/recomby-geo/skills/` vendor 新 skill

硬性规则：

- **许可证必须宽松**：MIT、Apache 2.0、BSD、ISC、CC0。**拒绝 GPL / AGPL / 自定义限制条款**（避免传染性 / 模糊条款污染整个仓库）
- **必须更新 [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md)**：补充来源 repo URL + commit hash + 许可证类型
- **必须模型可自动调用**：skill 的 `SKILL.md` 顶部 frontmatter 必须有清晰的 `description` 字段，描述触发场景（不写 description → 模型永远不会调它）
- **必须在 `plugin.json` 的 `skills` 数组里注册**
- 拷贝时保留原始 `LICENSE` 文件在 skill 目录内

PR 描述里请说明：为什么需要这个 skill、与现有 6 个的差异（避免功能重叠）。

---

## 3. 修改 `plugins/recomby-geo/commands/` 命令

硬性规则：

- 命令的产出必须**通过对应 JSON Schema 校验**（见 `plugins/recomby-geo/schemas/`）
- 如果改了产出结构 → 同步改 Schema → 同步改下游消费此产出的命令
- 不要在命令里硬编码客户路径；客户目录由 `argument-hint` 传入
- 不要破坏「专家槽位 = 硬门槛」的约束：05-production 必须在 brief 状态 ≠ `ready-for-production` 时硬拒绝执行，**绝对不允许用 AI 内容自动填写专家槽位**

本地校验命令产出：

```bash
python3 -c "import json,jsonschema; \
  s=json.load(open('plugins/recomby-geo/schemas/<schema>.schema.json')); \
  d=json.load(open('clients/<slug>/<artifact>.json')); \
  jsonschema.validate(d,s); print('OK')"
```

---

## PR 流程

1. Fork → 新建 branch（命名如 `add-agent-X` / `fix-cmd-04`）
2. 单 PR 单主题，便于 review
3. 大改动先开 Issue 讨论
4. 使用 [`PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) 默认模板填写

---

## 行为准则

技术讨论 only。质量高于速度，宁缺毋滥。

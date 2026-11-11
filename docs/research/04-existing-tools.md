# 04 现成工具评估

> 调研时间：2026-11。候选信息多来自第三方页面和目录站，未逐一核实维护状态、许可和实际支持范围（⚠️）。本文是评估计划，结果在试用后填写。

## 背景：方向调整

- 只靠目录和文件约定太弱：读不读记忆、收不收尾、统计准不准，都取决于 agent 是否自觉；也看不到各家 client 的 session。
- 所以需要一个**伴随 app**：不取代 Cursor / Claude Code / Codex，而是在旁边管理记忆、session、工具和统计，并通过 hooks 和 MCP 把约束接进各家 client。
- 分工：**hooks 管“必须发生”的事**（注入上下文、记录调用、触发收尾），不经过模型判断；**MCP 管“agent 主动发起”的事**（提议修改记忆、查询上下文），由模型决定何时调用。
- 调研发现：伴随 app 的各个部分都已有开源项目在做。因此策略是**组装现成工具，只补缺口**，不从零做 app。

## 需求清单

| # | 需求 | 优先级 |
|---|---|---|
| R1 | 跨 client（Cursor、Claude Code、Codex）浏览和搜索 session | 高 |
| R2 | session 按主题分组，查看主题下的 session 和关联主题 | 高 |
| R3 | session 开始时自动注入个人原则和对应主题的上下文 | 高 |
| R4 | 工具调用自动记录（工具、成败、耗时），不依赖 agent 自觉 | 高 |
| R5 | 工具效果的主观评价（ok / partial / fail + 备注），可与调用记录关联 | 中 |
| R6 | session 结束时自动触发或提醒收尾 | 中 |
| R7 | 记忆写回需人工审核，审核体验优于在聊天窗口里看 diff | 中 |
| R8 | 记忆底层是纯文本、可导出、可进 git | 高 |
| R9 | 全部可在本地运行，数据不外发（或外发可控） | 高 |
| R10 | 维护活跃，许可允许自用和修改 | 高 |

## 候选

### A. 跨 client 的 session 浏览

| 候选 | 形态 | 覆盖需求 | 备注 |
|---|---|---|---|
| [Agent Sessions](https://github.com/jazzyalex/agent-sessions) | macOS app，开源 | R1 | 本地运行，可恢复 session |
| [claude-code-history-viewer](https://github.com/jhlee0409/claude-code-history-viewer) | 桌面 app + 网页服务，开源 | R1 | 离线；有网页版，适合作为 fork 基础 |
| [cass](https://github.com/Dicklesworthstone/coding_agent_session_search) | 终端 TUI / CLI | R1 | 支持输出 JSON 给 agent 用 |

### B. 跨 client 的 hooks 框架

| 候选 | 形态 | 覆盖需求 | 备注 |
|---|---|---|---|
| [agenthooks（Speakeasy）](https://pkg.go.dev/github.com/speakeasy-api/agenthooks) | Go，MIT | R3、R4、R6 的适配层 | 自动处理各家差异 |
| [@premierstudio/ai-hooks](https://libraries.io/npm/@premierstudio%2Fai-hooks) | npm，MIT | 同上 | 一份配置 |
| [agent-hooks](https://docs.rs/agent-hooks) | Rust，MIT | 同上 | |

### C. 监听与统计

| 候选 | 形态 | 覆盖需求 | 备注 |
|---|---|---|---|
| 官方遥测（[Claude Code](https://code.claude.com/docs/en/monitoring-usage)、Codex） | 内置，OpenTelemetry | R4 | 默认关闭；工具名、成败、耗时都有。Cursor 情况待核实 |
| [AI Observer](https://github.com/tobilg/ai-observer) | 单文件、本地部署 | R4 的接收端 | 专为本地 coding agent 设计 |
| [OpenLIT](https://docs.openlit.io/latest/openlit/coding-agents/overview) | 自部署平台 | R4 + 看板 | 覆盖 Claude Code、Cursor、Codex |
| [panopticon](https://github.com/fml-inc/panopticon) | 本地，SQLite | R1、R4 | 同时采集 hooks、遥测、session 文件；可通过 MCP 查询。未列出 Cursor |
| [observer](https://github.com/lujuma/observer) | 本地 | R4 | 号称 29 种 client |

### D. 记忆采集与存储

| 候选 | 形态 | 覆盖需求 | 备注 |
|---|---|---|---|
| [claude-mem](https://www.augmentcode.com/learn/claude-mem-74k-stars-agent-memory) | 插件 + hooks + MCP | R3、R6 | 很热门；Claude Code、Cursor；Codex 支持情况不一。自动采集，需评估是否符合“人工审核”原则 |
| [Basic Memory](https://docs.basicmemory.com/integrations/harness-capture) | 本地 markdown + MCP | R3、R8 | 底层纯文本，与本项目原则最接近 |
| [repo-memory](https://enterprisedna.co/directories/mcp/yubinkim444-repo-memory) | CLI + MCP，git 文件 | R8 | 无数据库 |
| [Memorix](https://mcpservers.org/en/servers/avids2/memorix)、[knowl](https://enterprisedna.co/directories/mcp/dat999zx-knowl)、[DecisionNode](https://enterprisedna.co/directories/mcp/decisionnode-decisionnode) | MCP 服务 | R3 | 较早期 |

### 预期的缺口（需要自己补）

调研中没有找到覆盖这些的现成工具：

- **R2 主题分组**：把 session、记忆、工具关联到主题。
- **R7 写回审核**：集中审核收尾提议的界面。
- **R5 主观评价**：与调用记录关联的效果评价。
- 原则和规范本身（已写好，见 prompts/）。

## 评估标准

每个候选按以下几项打分（0–2 分）：

| 维度 | 说明 |
|---|---|
| 需求覆盖 | 覆盖了哪些 R，覆盖得是否完整 |
| client 覆盖 | Cursor、Claude Code、Codex 三家各自是否真的能用（实测，不看宣传） |
| 本地与数据 | 能否完全本地运行；有没有数据外发，能否关闭 |
| 底层格式 | 数据能否导出；是否纯文本或标准格式（JSONL、SQLite、OTel） |
| 维护状态 | 最近提交、发布频率、issue 响应 |
| 可改造性 | 许可、代码规模、能否 fork 或通过插件扩展 |
| 安全 | hooks 运行什么代码、以什么权限运行；装之前要读完 hooks 脚本 |

## 试用步骤

1. **先核实**（半天）：逐个看仓库的 README、最近提交、许可、对三家 client 的支持说明，排除明显不合适的。每类最多留 2 个。
2. **最小组合试用**（1–2 天），建议组合：
   - 官方遥测 + AI Observer（或 panopticon）→ 验证 R4
   - 一个 hooks 框架写一个最简单的 session 开始 hook，注入个人原则 → 验证 R3 在三家 client 的实际效果
   - Agent Sessions 或 claude-code-history-viewer → 验证 R1，并看能否扩展出 R2
   - Basic Memory 或 repo-memory → 验证 R8，与现有记忆约定对比
3. **记录**：每个候选按评估标准打分，写下实际遇到的问题。
4. **决定**：
   - 现成组合能覆盖大部分高优先级需求 → 只做缺口（R2、R5、R7）
   - 某个开源项目适合当基础 → fork 后加功能
   - 都不合适 → 再考虑自己做伴随 app，范围只限缺口部分

## 安全注意

- hooks 会在每次事件发生时以你的权限自动运行。安装第三方 hooks 前，读完它会执行的脚本。
- 确认数据流向：有的工具会把 prompt、代码片段发到外部服务。只选能完全本地运行、或外发可以关闭的。
- 遥测数据可能包含 prompt 内容和文件路径，接收端只放在本地。

## 评估结果

（试用后填写）

| 候选 | 需求覆盖 | client 覆盖 | 本地与数据 | 底层格式 | 维护 | 可改造性 | 安全 | 结论 |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

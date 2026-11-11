# 02 现状调研

> 调研时间：2026-11。部分信息来自二手资料，落地前以各工具官方文档为准。标 ⚠️ 的条目未经核实。

## 使用中的 client

主力：Cursor > Claude Code ≈ Codex；也用过 Grok、Pi。

## 各 harness 已有的“记忆”机制

| 方式 | 代表 | 特点 |
|---|---|---|
| 仓库内的指令文件 | `CLAUDE.md`（Claude Code）、`AGENTS.md`（Codex 等） | 按目录分层继承，天然对应 monorepo 的分组 |
| 带作用范围的规则 | Cursor rules | 可按文件路径匹配生效 |
| Project 级知识 | Claude / ChatGPT Projects | 项目内的对话共享资料 |
| 自动记忆 | 多家都在做 | 粒度偏粗（全局 / 项目级） |

结论：**记忆的存储和注入不需要重造。**

## 现有方案的空白

1. 只能按目录分组，横切主题（性能、鉴权重构）没有归属。
2. 写回和维护薄弱：缺少“提议 → 人确认 → 去重 / 过期清理”的流程。
3. 记忆和产生它的 session 之间没有关联，无法溯源。
4. 跨 client 割裂：各家只管自己的 session，记忆文件格式也各不相同。

## 社区标准

- **AGENTS.md**：指令文件的事实标准，据称已归入 Linux Foundation 体系（⚠️）。Codex、Cursor 读取。**Claude Code 只读 `CLAUDE.md`**，需要在 `CLAUDE.md` 里写 `@AGENTS.md` 引用，或做软链接。
- **Agent Skills（SKILL.md）**：可复用流程的标准，据称被 30–40 个工具采纳（⚠️）。各家目录不一致：Claude 读 `.claude/skills/`，Codex 读 `.agents/skills/`；Cursor 的兼容程度说法不一（⚠️）。
- **MCP**：工具接入标准，与本项目关系不大。
- Pi、Grok CLI 对 AGENTS.md 的支持未核实（⚠️）。

## session 本地存储（供以后做跨 client 索引）

- Claude Code：`~/.claude/projects/` 下的 jsonl
- Codex：`~/.codex/sessions/`
- Cursor：本地 SQLite，格式未公开，版本升级可能变化（⚠️）

## Magpie（usemagpie.ai）

- yetone 开发，开源（MIT），Go 编写；有菜单栏应用、TUI、CLI 三种形态。
- 解决的是**模型 / provider 管理**：给 Claude Code、Codex、Gemini CLI 等几十个 agent 统一选模型、切 provider、多账号故障切换、路由。
- 与本项目不在同一层（它是模型接入层，本项目是上下文 / 知识层），可以并存。
- 可借鉴：“一份统一配置 → 适配器生成各 agent 的配置”这个模式，对应本项目的 sync 脚本。
- 官网当时无法直接访问，以上信息来自第三方介绍。

## 待核实的风险

- ⚠️ 印象中今年有研究发现：**自动生成的、冗长的 context 文件效果可能持平甚至更差**（增加 token 成本、引入噪音）。未当场核实，但已作为设计约束（见 design/principles.md）。

## 参考来源

- https://alphasignal.ai/news/magpie-gives-developers-one-gateway-to-rule-20-coding-agents
- https://andrew.ooo/posts/magpie-review-yetone-menu-bar-model-switcher-coding-agents/
- https://www.besthub.dev/articles/magpie-open-source-ai-agent-manager-unifies-45-tools-with-smart-routing-4e94f13e8df2
- https://codex.danielvaughan.com/2026/04/07/agents-md-open-standard-cross-tool-portability/
- https://codex.danielvaughan.com/2026/05/05/agent-skills-open-standard-portable-skills-codex-cli-cross-agent/
- https://www.noqta.tn/en/news/skill-md-open-standard-30-ai-coding-tools-adoption-2026
- https://mcp.directory/blog/claude-md-vs-agents-md-vs-skills-2026
- https://blog.buildbetter.ai/agents-md-vs-cursorrules-vs-claude-skills-2026-comparison/
- 官方规范：https://agents.md 、https://agentskills.io

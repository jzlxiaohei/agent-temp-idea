# 02 现状调研

> 调研时间：2026-11。部分信息来自二手资料或印象，落地前以官方文档为准。标 ⚠️ 的条目未经核实。

## 使用中的 client

主力：Cursor > Claude Code ≈ Codex；也用过 Grok、Pi。网页对话只做临时用途，不在记忆共享范围内。

## 各 client 已有的记忆机制

| 方式 | 代表 | 特点 |
|---|---|---|
| 个人级指令文件 | `~/.claude/CLAUDE.md`、`~/.codex/AGENTS.md`、Cursor User Rules | 跨项目生效，各家各一份 |
| 项目级指令文件 | `CLAUDE.md`、`AGENTS.md` | 可按目录分层继承（只对有目录分层的项目有意义） |
| 带作用范围的规则 | Cursor rules | 可按文件路径匹配生效 |
| Project 级知识 | Claude / ChatGPT Projects | 网页端，不在本项目范围 |
| 自动记忆 | 多家都在做 | 粒度偏粗，绑定单一厂商 |

结论：**单个 client 内的记忆存储和注入不需要重造。**

## 现有方案的空白

1. 分组要么跟目录走，要么没有；“主题”没有一等的位置。
2. 写回和维护薄弱：缺少“提议 → 人确认 → 去重 / 过期清理”的流程。
3. 记忆无法溯源到产生它的 session。
4. 跨厂商割裂：个人级配置要在各家分别维护，格式也不同。厂商没有动力解决这一点。

## 社区标准

- **AGENTS.md**：指令文件的事实标准，据称已归入 Linux Foundation 体系（⚠️）。Codex、Cursor 读取。**Claude Code 只读 `CLAUDE.md`**，需要在里面写 `@AGENTS.md` 引用或做软链接。本质是“文件夹里的 markdown + agent 读取约定”，不限于代码项目。
- **Agent Skills（SKILL.md）**：可复用流程的标准，据称被 30–40 个工具采纳（⚠️）。目录不一致：Claude 读 `.claude/skills/`，Codex 读 `.agents/skills/`；Cursor 兼容程度说法不一（⚠️）。
- **MCP**：工具接入标准；第三方记忆服务大多通过 MCP 接入多个 client。
- Pi、Grok CLI 对 AGENTS.md 的支持未核实（⚠️）。

## 记忆存储：用现成的，不自建

| 方案 | 形态 | 适合 |
|---|---|---|
| **私有 git 仓库 + markdown**（基线） | 本地 client 直接读文件；云端 agent（Claude Code 网页版、Codex cloud）clone 仓库 | 起步阶段；零基建、版本化、归自己所有 |
| Basic Memory（⚠️） | 本地 markdown 文件 + MCP 服务 | 想保留纯文本，又要 MCP 检索 |
| mem0 / OpenMemory（⚠️） | 记忆服务，MCP 接入多个 client | 需要自动抽取、语义检索 |
| Supermemory（⚠️） | 云端记忆服务，提供 MCP | 需要云端、跨设备 |
| Letta、Zep（⚠️） | agent 记忆层 / 框架 | 偏向自己构建 agent，不太适合本项目 |

选型原则：

- 先用 git + markdown 基线，**只有在实验中出现基线解决不了的需求时**才引入第三方（例如需要语义检索，或要在没有文件系统的环境里用记忆）。
- 优先选**底层仍是纯文本、可导出**的方案，避免把记忆锁进另一个厂商。
- 引入前需要单独做一次选型核实（功能、维护状态、数据存放位置、隐私）。

## session 本地存储（供以后做跨 client 索引，暂不做）

- Claude Code：`~/.claude/projects/` 下的 jsonl
- Codex：`~/.codex/sessions/`
- Cursor：本地 SQLite，格式未公开（⚠️）

## Magpie（usemagpie.ai）

- yetone 开发，开源（MIT），Go 编写；菜单栏应用、TUI、CLI 三种形态。
- 解决**模型 / provider 管理**：给多个 coding agent 统一选模型、切 provider、多账号故障切换、路由。
- 与本项目不在同一层（它是模型接入层，本项目是上下文层），可以并存。
- 可借鉴：“一份统一配置 → 适配器生成各 agent 的配置”的模式，对应以后可能写的同步脚本。
- 官网当时无法直接访问，信息来自第三方介绍。

## 待核实的风险

- ⚠️ 印象中今年有研究发现：**自动生成的、冗长的 context 文件效果可能持平甚至更差**（增加 token 成本、引入噪音）。未核实，但已作为设计约束。

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

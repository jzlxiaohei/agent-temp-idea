# agent-context-kit（暂定名）

一套面向 monorepo 的 AI coding 上下文规范 + skills + 脚本，让 Cursor / Claude Code / Codex 等多个 client 共享同一份项目知识。

- 不发明新格式：对齐 AGENTS.md、Agent Skills（SKILL.md）等社区标准
- 分两条路径：新 monorepo 初始化（init）与已有 monorepo 改造（adopt）
- 按需生成、少即是多：不一次铺满，只写 agent 从代码里推断不出来的东西
- 首要目标是自用；可以直接拿现成的东西用，或以它为基准改造

## 文档

| 文档 | 内容 |
|---|---|
| [docs/research/01-problem.md](docs/research/01-problem.md) | 问题拆解：session 平铺列表的不足、分组/图谱的风险 |
| [docs/research/02-landscape.md](docs/research/02-landscape.md) | 现状调研：各 harness 的记忆机制、社区标准、Magpie |
| [docs/design/principles.md](docs/design/principles.md) | 设计原则（含“按需生成”约束） |
| [docs/design/kit-overview.md](docs/design/kit-overview.md) | set 的组成、init / adopt 流程、验证方法 |

## 状态

调研阶段完成，下一步：

1. 写规范（spec）一页纸
2. 写 `repo-adopt` 的 SKILL.md，挑一个已有 monorepo 试点
3. 写 sync 脚本（CLAUDE.md 转接、skills 目录软链）

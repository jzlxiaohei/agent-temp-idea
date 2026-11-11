# agent-context-kit（暂定名）

一套个人 AI 工作方法：用**原则 prompt + 纯文本记忆约定 + 收尾写回流程**，让 Cursor / Claude Code / Codex 等多个 client 共享同一份上下文。

- 适用于任何项目，不限 monorepo，也不限编码项目
- 不发明新格式：对齐 AGENTS.md、Agent Skills 等社区标准
- 记忆存储和基建用现成的（git 仓库里的 markdown 起步，需要时再接第三方记忆服务），不自建
- 按需生成、少即是多
- 定位是“半天搭好、靠习惯维持的方法”，不是持续开发的工具项目；先用两周实验验证意义

## 文档

| 文档 | 内容 |
|---|---|
| [docs/research/01-problem.md](docs/research/01-problem.md) | 问题拆解：session 平铺列表的不足、分组和图谱的风险 |
| [docs/research/02-landscape.md](docs/research/02-landscape.md) | 现状调研：各 client 的记忆机制、社区标准、第三方记忆服务、Magpie |
| [docs/research/03-assessment.md](docs/research/03-assessment.md) | 意义评估：正反理由、分项判断 |
| [docs/design/principles.md](docs/design/principles.md) | 设计原则 |
| [docs/design/architecture.md](docs/design/architecture.md) | 结构：三层记忆、主题即组、存储后端、写回闭环 |
| [docs/design/tools.md](docs/design/tools.md) | 脚本与工具：脚本化标准、沉淀触发、skill 格式、统计与评估 |
| [tools/](tools/INDEX.md) | 工具库（skill 格式）、候选记录 |
| [docs/design/llm-invocation.md](docs/design/llm-invocation.md) | 如何调用大模型：交互、headless CLI、API/网关，以及 Magpie 的位置 |
| [docs/design/experiment.md](docs/design/experiment.md) | 两周验证实验 |
| [docs/design/sketches/product-sketch.svg](docs/design/sketches/product-sketch.svg) | 产品草图：一次完整的使用流程 |
| [docs/design/sketches/architecture.svg](docs/design/sketches/architecture.svg) | 系统架构线框图 |
| [prompts/principles.md](prompts/principles.md) | 原则 prompt（个人层，放进各 client 的个人配置） |
| [prompts/distill.md](prompts/distill.md) | 收尾 prompt（session 结束时运行） |

## 状态

- [x] 调研、意义评估
- [x] 起草原则 prompt、收尾 prompt
- [ ] 两周实验（见 experiment.md）
- [ ] 根据实验结果决定是否写同步脚本、lint，是否接入第三方记忆服务

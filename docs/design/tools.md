# 脚本与工具

## 两个独立的判断

| 判断 | 依据 | 谁决定 |
|---|---|---|
| 这个任务用不用脚本做 | 任务的性质，和次数无关 | agent 当场判断，直接写临时脚本 |
| 这个脚本要不要沉淀成工具 | 是否会复用（同类任务再次出现，或我说明常做） | agent 提议，我确认 |

原来的“三次升级”（命令 → 脚本 → skill → 工具）把两者混在一起，已废弃。

## 1. 脚本化的标准

满足任一条，就直接写临时脚本执行，不要逐步推理或手工逐个改：

- 批量或重复的机械操作
- 多步骤、手工容易出错的操作
- 需要精确结果的计算、统计、数据转换
- 结果需要可复现或可验证

不需要脚本的：改一两处、一次性的简单问答。

临时脚本放临时目录，不放进项目。写法规范见 [tools/scripting-guide/SKILL.md](../../tools/scripting-guide/SKILL.md)。

这部分标准写进了原则 prompt（[prompts/principles.md](../../prompts/principles.md)），始终生效。

## 2. 沉淀的触发

- 同类任务**再次**出现（靠记忆判断，见下文“候选记录”），或
- 我明确说这是经常要做的事（第一次就可以提议）。

提议时说明：用途、放在哪里、维护成本。我确认后才创建。

### 候选记录

agent 本身不知道跨 session 的次数，所以收尾流程会把本次用过、可能再用的临时脚本记到 `tools/candidates.md`，每条一行：

```
日期 · 用途（一句话）· 所在项目 · 关键做法 / 脚本要点
```

下次遇到同类任务时，agent 查到这条记录，就提议沉淀。沉淀后从候选里删除；超过 90 天没再出现的候选，收尾时提议清理。

## 3. 沉淀成什么形式：按用途选，不逐级升级

| 情况 | 形式 | 位置 |
|---|---|---|
| 只在一个项目里用 | 脚本 | 项目内，如 `scripts/` |
| 多个项目都用 | 工具（skill 格式） | 本仓库 `tools/<id>/` |
| 除了执行还需要判断和步骤说明 | 工具（skill 格式，正文写步骤） | 同上 |
| 需要在多个 client 里像函数一样调用，或要连外部服务 | MCP 工具 | 个人层；成本最高，最少用 |

## 4. 工具的格式：skill 格式 + 渐进式披露

工具库里的每个工具都用 Agent Skills 的格式打包，天然获得渐进式披露：

1. **平时**：只加载 `name` 和 `description`（一两句话）。
2. **用到时**：读 SKILL.md 正文（用法、参数、注意事项）。
3. **执行时**：才运行 `scripts/` 里的代码，代码本身不进上下文。

```
tools/
  INDEX.md                 # 自动生成：id + 一句话说明（给不支持 skill 的 client 用）
  candidates.md            # 候选记录
  bin/                     # 必要的小脚本（如生成 INDEX.md），按需添加
  scripting-guide/         # 写脚本的规范，本身也是一个 skill
    SKILL.md
  <id>/
    SKILL.md               # frontmatter: name（= id）、description、metadata
    scripts/main.*         # 入口
```

SKILL.md 模板：

```markdown
---
name: csv-to-events
description: 把埋点导出的 CSV 转成事件 JSON。用于每周报表数据准备。
metadata:
  version: "1"
  created: 2026-11-12
---

## 用法
python3 $AGENT_HOME/tools/csv-to-events/scripts/main.py <输入.csv> <输出.json>

## 注意
- ...
```

### id 规范

- 小写 kebab-case，即目录名，也是 SKILL.md 的 `name`。
- 一旦创建不改名（统计按 id 累计）。行为有不兼容的变化时，`metadata.version` 加一。

### 让各 client 找到工具

- 设环境变量 `AGENT_HOME` 指向本仓库的本地 clone。
- 支持 skill 的 client：把 `tools/<id>/` 软链到它的个人 skills 目录（Claude Code 为 `~/.claude/skills/`；Codex、Cursor 的个人 skills 目录待核实 ⚠️）。实验阶段手工链，以后用同步脚本。
- 不支持的 client：原则 prompt 让 agent 读 `$AGENT_HOME/tools/INDEX.md`，效果相同。

## 5. 统计与评估

### 原则：采集不依赖 agent 自觉

早期设计要求所有工具都通过 `tools/bin/run` 包装脚本调用，由它记录日志。问题是 agent 一旦直接运行脚本，就不会被记录。现在改为**从外部捕获所有工具调用**，再识别出哪些是工具库里的工具：

- **采集**：用 client 官方遥测（Claude Code、Codex 支持 OpenTelemetry），或工具调用后的 hook。每次调用都有工具名、命令、是否成功、耗时。
- **识别**：执行的命令路径落在 `$AGENT_HOME/tools/<id>/` 下的，就是工具库里的工具 `<id>`。版本从该工具 SKILL.md 的 `metadata.version` 读取。
- **接收与存储**：本地的 OTel 接收端或监听工具（候选见 [04-existing-tools.md](../research/04-existing-tools.md)），不自己写。

记录的字段（以采集来源实际提供的为准）：

```
时间 · client · session id · 项目 · 工具 id · 版本 · 是否成功 · 耗时
```

### 主观评价

退出码只说明“有没有报错”，不说明“有没有用”。所以收尾流程会列出本次调用过的工具，提议评价（`ok` / `partial` / `fail` + 一句话），我确认后记录下来，并通过 session id + 工具 id 与调用记录关联。评价存放方式待定（可以是本地 JSONL，或评估选中的工具自带的标注功能）。

### 看什么

每个工具的：调用次数、成功率、评价分布、最近使用时间、版本。优先用监听工具自带的看板；不够时再写一个汇总脚本。

### 用来做什么

- **淘汰**：超过 60 天没用，或 `fail` 评价占多数的，收尾时提议删除或改进。
- **改进**：`partial` / `fail` 的备注就是改进清单。
- **验证意义**：两周实验评估时，看沉淀下来的工具是否真的被反复调用。

### 工具内部调用模型

能用确定性代码解决的，不调模型。确实需要的，走 API 或网关，并在统计里额外记录模型名和 token 用量。见 [llm-invocation.md](llm-invocation.md)。

### 待定

- **采集方案**：官方遥测 + 本地接收端，还是 hooks + 监听工具，见 [04-existing-tools.md](../research/04-existing-tools.md) 的试用结果。Cursor 的采集能力需要实测。
- **主观评价的存放**：本地 JSONL，或选中工具自带的标注功能。
- 实现推迟到评估完成之后。

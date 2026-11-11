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
  bin/                     # 统计用的小脚本：run / feedback / stats / index
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
$AGENT_HOME/tools/bin/run csv-to-events <输入.csv> <输出.json>

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

### 记录什么

所有工具都通过 `tools/bin/run <id> [参数]` 调用。`run` 执行入口脚本，并追加一行 JSONL：

```json
{"run_id":"r-20261112-8f3a","ts":"2026-11-12T10:03:00+08:00","id":"csv-to-events","version":"1",
 "project":"my-project","client":"claude-code","exit":0,"duration_ms":842}
```

效果评价分两部分：

- **客观**：退出码（是否报错）、耗时，由 `run` 自动记录。
- **主观**：结果是否真的有用。收尾流程会列出本次调用过的工具，提议评价（`ok` / `partial` / `fail` + 一句话），我确认后由 `tools/bin/feedback <run_id> <评价> [备注]` 追加记录。

### 看什么

`tools/bin/stats` 输出每个工具的：调用次数、成功率（退出码）、评价分布、最近使用时间、版本。

### 用来做什么

- **淘汰**：超过 60 天没用，或 `fail` 评价占多数的，收尾时提议删除或改进。
- **改进**：`partial` / `fail` 的备注就是改进清单。
- **验证意义**：两周实验评估时，看沉淀下来的工具是否真的被反复调用。

### 待定

- **日志放哪里**：
  - 方案 A（推荐先用）：本机 `~/.local/state/agent-tools/usage.jsonl`，不进 git。简单，但云端 session 的记录会随容器丢失，多台机器之间不合并。
  - 方案 B：进本仓库 `usage/<主机名>.jsonl`，按机器分文件避免冲突。能汇总，但会频繁产生提交。
- **统计依赖 agent 遵守“通过 run 调用”**：直接运行脚本不会被记录。实验中观察遵守程度。
- `tools/bin/` 的四个小脚本还没写；它们本身是确定性代码，符合“确定性交给脚本”的原则。

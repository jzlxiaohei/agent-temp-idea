# 结构

## 分层定位

| 层 | 管什么 | 代表 |
|---|---|---|
| 模型接入层 | agent 用哪个模型、怎么计费和路由 | Magpie（可选）；调用方式见 [llm-invocation.md](llm-invocation.md) |
| 原则层 | agent 按什么方法干活 | **本项目：原则 prompt** |
| 上下文 / 记忆层 | agent 知道什么 | **本项目：记忆约定 + 收尾流程**；存储用 git 或第三方 |
| 会话组织层 | session 怎么找回、怎么关联 | 不做，交给厂商 |

## 三层记忆

| 层 | 内容 | 位置 | 生效方式 |
|---|---|---|---|
| 个人层 | 工作原则、偏好、工具库 | 本仓库 `prompts/principles.md` 为源头，复制或同步到各 client 的个人配置 | 所有项目自动加载 |
| 项目层 | 项目目标、约定、真实命令、禁止事项、主题索引 | 项目根目录 `AGENTS.md`（加上 `CLAUDE.md` 转接） | 进入项目自动加载 |
| 主题层 | 某个主题的背景、决策、坑、未完成事项 | 项目内 `.agents/memory/topics/<主题>.md`（暂定） | 由入口索引指引，按需读取 |

个人配置的位置：

- Claude Code：`~/.claude/CLAUDE.md`
- Codex：`~/.codex/AGENTS.md`
- Cursor：Settings → Rules → User Rules

## 调用大模型

核心流程不直接调用模型：交互工作由 client 调用；以后的自动化任务用 headless agent CLI（`claude -p`、`codex exec` 等）经适配脚本调用；工具内部的纯文本小调用走 API 或网关。详见 [llm-invocation.md](llm-invocation.md)。

## 工具库

跨项目复用的脚本沉淀在本仓库 `tools/`，每个工具用 skill 格式打包（渐进式披露），调用记录由官方遥测或 hooks 从外部捕获，不依赖 agent 自觉。详见 [tools.md](tools.md)。

## 主题即组

- 一个主题文件就是一个组：带长度上限的上下文 + 关联的 session。
- 入口 `AGENTS.md` 里维护两个小节：
  - **主题索引**：主题名、一句话说明、什么时候该读。
  - **待归类**：第一次出现、还没有主题文件的内容（日期、主题名、一句话）。同一主题第二次出现时，由收尾流程提议建主题文件并迁移。
- 主题之间的关系写在主题文件里（“相关：xxx”），由收尾流程提议、人确认。不做全局图谱。
- 编码项目可选增强：给主题配路径规则（如 `apps/web/**`），或用子目录 `AGENTS.md`。

## 目录结构（项目内，按需长出来）

```
AGENTS.md                      # 入口：项目说明、约定、命令、禁止事项、主题索引
CLAUDE.md                      # 一行：@AGENTS.md
.agents/
  memory/
    topics/<主题>.md           # 第二次出现时才建
    decisions.md               # 重要决策（日期 + 决策 + 理由），按需建
  skills/                      # 以后若把收尾流程做成 skill 放这里
```

## 主题文件模板

```markdown
# <主题名>

> 一句话说明。相关：<其他主题>

## 背景
## 决策（日期 · 决策 · 理由）
## 约定
## 踩过的坑
## 进行中（日期 · 事项）
```

## 写回闭环

1. session 结束时运行 [收尾 prompt](../../prompts/distill.md)。
2. agent 提取候选内容，判断归属层和目标文件，以 diff 形式提议，记录工具候选、提议沉淀，并评价本次用过的工具（见 [tools.md](tools.md)）。
3. 人确认、删减后写入；在 git 项目中随正常提交一起入库。
4. “进行中”类条目带日期，过期的在下次收尾时提示清理。

## 伴随 app 方向

只靠文件约定太弱，需要一个伴随 app：hooks 管“必须发生”的事（注入上下文、记录调用、触发收尾），MCP 管“agent 主动发起”的事。各部分已有开源项目，策略是组装现成工具、只补缺口（主题分组、写回审核、主观评价）。评估计划见 [04-existing-tools.md](../research/04-existing-tools.md)。

## 存储后端

- **基线**：项目文件夹本身（项目层、主题层）+ 本仓库（个人层），均为 git 管理的 markdown。
- **按需引入第三方**：实验中若出现基线解决不了的需求（语义检索、无文件系统环境、跨设备），再做选型，见 [02-landscape.md](../research/02-landscape.md)。

## 以后可能的工具（实验后再决定）

- 同步脚本：一份源头 → 各 client 的个人配置、`CLAUDE.md` 转接、skills 目录软链
- lint：长度预算、过期条目、失效引用
- 把收尾 prompt 做成 skill，各 client 通用

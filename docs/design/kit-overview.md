# set 总览

## 分层定位

| 层 | 管什么 | 代表 |
|---|---|---|
| 模型接入层 | agent 用哪个模型、怎么计费和路由 | Magpie |
| 上下文 / 知识层 | agent 知道什么、按什么规范干活 | **本项目** |
| 会话组织层 | session 怎么找回、怎么关联 | 以后再说（可选） |

## 组成

```
1. 规范（spec）      一页纸：目录结构 + 每类文件写什么、不写什么、写多长
2. Skills（.agents/skills/，标准 SKILL.md 格式）
   - repo-init        新 monorepo：生成最小骨架
   - repo-adopt       已有 monorepo：扫描 → 起草 → 以 PR 形式交给人审
   - session-distill  session 收尾：提取结论，以 diff 形式提议写回；按需提议新建文件
   - context-lint     检查：长度超预算、内容过时、引用失效、重复或冲突
3. 脚本
   - sync：生成 CLAUDE.md 转接（@AGENTS.md）、skills 目录软链、必要时生成 Cursor rules
   - lint：pre-commit / CI 中跑长度和引用检查
4. 跨 client 的 session 索引（可选，暂缓）
```

## 目标目录结构（完整形态；实际按需逐步长出来）

```
AGENTS.md                  # 全局：真实命令、非显然约定、禁止事项、topics 索引
CLAUDE.md                  # 一行：@AGENTS.md（由 sync 生成）
apps/web/AGENTS.md         # 按需创建
services/api/AGENTS.md     # 按需创建
docs/topics/perf.md        # 横切主题，第二次出现时才建；由根 AGENTS.md 索引，按需读取
docs/decisions/            # 重要决策（简版 ADR）
.agents/skills/            # skills 源头；.claude/skills 软链到这里
```

## init：新 monorepo

1. 读取 workspace 配置（pnpm / turbo / nx 等），识别包结构。
2. 只生成根 AGENTS.md（真实命令 + 空的约定、禁止事项区）和目录骨架。
3. 跑 sync。
4. 目标：10 分钟内完成。

## adopt：已有 monorepo

1. **迁移**：合并散落的 `.cursorrules`、`CLAUDE.md`、`.cursor/rules` 到 AGENTS.md，去重并标出冲突。
2. **推断**：从 package.json scripts、CI 配置、lint 配置、README 提取真实的命令和约定。
3. **最小范围**：只产出根文件，外加最多 1–2 个最常改的包（可按 git 提交频率判断）。其余包不动，等 session-distill 按需提议。
4. **草稿 → PR**：人工删减后合入。
5. 可以把各 client 自带的 `/init` 作为起点，生成后按规范大幅删减。

## 写回闭环（session-distill）

session 结束时运行，在三个 client 中提供同一份指令：

> 回顾本次 session，提取：1) 新确定的约定或决策；2) 踩过的坑；3) 未完成事项。判断各自应写入哪个文件；目标文件不存在时说明理由再提议新建。以 diff 形式提议，不要直接写入。

“进行中”类内容要标日期，由 context-lint 提示过期。

## 验证

从真实任务里挑 5–10 个，比较有无规范时的：返工次数、重复交代背景的次数、犯同类错误的次数。

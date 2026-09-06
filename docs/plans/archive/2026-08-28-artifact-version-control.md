# 产物纳入版本控制

> Opened: 2026-08-28 · commit 4229fe1
> Closed: 2026-09-06
> 来源：原 backlog D-08 + D-09，同源合并

## 目标

让项目的两个产物都有唯一权威副本，且都在 git 内。

现状两个缺口：

| 缺口 | 表现 |
|---|---|
| `owncc.md` 不在任何仓库 | 唯一副本在 `~/.claude/output-styles/`，误删无退路，design.md 里「owncc 管什么」无实体可对照 |
| `explain/SKILL.md` 有两份完全相同的实体 | `skills/` 与 `.claude/skills/` 各一份，均被 git 追踪，改一份忘另一份必漂移 |

根因是同一个：**开发位置 ≠ 生效位置**。owncc 必须在 `~/.claude/output-styles/` 才生效，skill 必须在 `.claude/skills/` 才被本地加载，而版本控制只认项目目录。

## 边界

- **不做**同步自动化（文件监听、git hook）——先把权威副本定下来，自动化是之后的事
- **不改** owncc.md 与 SKILL.md 的内容，那是下一个任务的事

## 推断的前提

| 前提 | 结果 |
|---|---|
| `skills/` 是权威位置、`.claude/skills/` 是本地生效副本 | 成立，但**下半句作废**——见下 |
| `.claude/skills/` 可用 junction 指向 `skills/`，同卷免提权 | **未验证，也不必验证**——方案已被否 |
| git 对 junction 的处理需要 `.claude/skills/` 移出追踪 | 同上 |

**关键前提被推翻**：整个 junction 方案建立在「本地需要 `.claude/skills/` 加载 skill 做开发」之上。实际使用方式是通过插件安装，开发阶段不走本地加载路径，因此 `.claude/skills/` 没有存在理由——**直接删除即可，不需要任何联接或同步机制**。

## 完成判据

- [x] `git ls-files` 里 `explain/SKILL.md` 只出现一次
- [x] `git ls-files` 里出现 `output-styles/owncc.md`
- [x] 项目内 `output-styles/owncc.md` 与 `~/.claude/output-styles/owncc.md` 内容一致
- [~] ~~本地 `.claude/skills/explain/SKILL.md` 仍可被 Claude Code 加载~~ — **作废**，该路径已删除，本地加载不在使用方式内
- [x] `docs/context.md` 的结构性坑一条据实更新

## 实际解法

| 产物 | 权威位置 | 同步方式 |
|---|---|---|
| `explain/SKILL.md` | `skills/` | 无需同步，唯一副本 |
| `owncc.md` | `output-styles/` | `cp` 推送到 `~/.claude/output-styles/`，跨卷别无选择 |

删除 `.claude/skills/` 后，`.claude/` 目录在本项目内已无内容。

## 进度

- 2026-08-28 从 backlog 拆出
- 2026-09-06 owncc.md 落库（`acfbf58`）
- 2026-09-06 删除 `.claude/skills/` 重复副本，junction 方案因前提推翻而取消，任务完成

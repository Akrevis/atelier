# 产物纳入版本控制

> Opened: 2026-08-28 · commit 4229fe1
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

## 推断的前提（待确认）

- 假设 `skills/` 是权威位置、`.claude/skills/` 是本地生效副本——依据是插件规范要求 `skills/` 在包根
- 假设 `.claude/skills/` 可以用 Windows 目录联接（junction）指向 `skills/`，同卷、免提权；**未实测**
- 假设 git 对 junction 的处理是「跟进去当普通目录」，因此 `.claude/skills/` 必须同时移出 git 追踪；**未实测**

## 完成判据

- [ ] `git ls-files` 里 `explain/SKILL.md` 只出现一次
- [ ] `git ls-files` 里出现 `output-styles/owncc.md`
- [ ] 项目内 `output-styles/owncc.md` 与 `~/.claude/output-styles/owncc.md` 内容一致（`diff` 为空）
- [ ] 本地 `.claude/skills/explain/SKILL.md` 仍可被 Claude Code 加载（新开会话验证 explain skill 出现在清单里）
- [ ] `docs/context.md` 的结构性坑一条据实更新为最终采用的同步方式

## 进度

- 2026-08-28 从 backlog 拆出，未开始

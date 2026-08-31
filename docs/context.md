# atelier · context

> Last updated: 2026-08-28 · commit 4229fe1
> 本文件注入每次会话。只写 git 推导不出、且 AI 不知道自己缺的事实。

## 这是什么

让 AI 的输出可学——从「给你结果」变成「给你理解」。两个刻意解耦的产物：`owncc.md`（output style，管**怎么说**）＋ `atelier`（Claude Code 插件，管**做什么**）。两者可单独使用，组合才完整。

## 不做什么

- MVP 只做**日常模式**（两个 skill + 一个 command）；沉浸模式整块留第二版，交付前不碰
- **不依赖任何其他包**
- **不要求用户改 CLAUDE.md 或 settings.json**——装上就用，卸载就干净
- `owncc.md` **不收进插件**，保持独立分发

## 顶层模块

| 模块 | 职责 |
|---|---|
| `docs` | 设计快照 · 决策日志 · 任务计划 · 演进路线 |
| `skills` | 插件分发的 skill 实体 |

## 活跃任务

| 任务 | 计划文档 | 状态 |
|---|---|---|
| 产物纳入版本控制 | `plans/2026-08-28-artifact-version-control.md` | 未开始 |
| explain skill 瘦身 | `plans/2026-08-28-explain-skill-slim.md` | 未开始 |
| 写 code-walkthrough skill | `plans/2026-08-28-code-walkthrough-skill.md` | 未开始 |
| 写 /palette command | `plans/2026-08-28-palette-command.md` | 未开始 |
| 打包三件套 | `plans/2026-08-28-plugin-packaging.md` | 未开始 |

推进顺序即上表顺序，逐条做完再动下一条。

## 结构性坑

- **`owncc.md` 的生效位置与项目不同卷**：必须放在 `C:\Users\IceLine\.claude\output-styles\` 才生效，而项目在 `F:` 盘。跨卷硬链接不可用、符号链接需提权，因此项目内副本与生效副本只能靠复制同步。改完 `owncc.md` 必须推送到生效位置，否则改的是个没人读的文件。

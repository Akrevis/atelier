# atelier · context

> Last updated: 2026-09-06 · commit cc315a5
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
| `output-styles` | `owncc.md` 的权威副本，独立于插件分发 |

## 活跃任务

| 任务 | 计划文档 | 状态 |
|---|---|---|
| explain skill 瘦身 | `plans/2026-08-28-explain-skill-slim.md` | 未开始 |
| 写 code-walkthrough skill | `plans/2026-08-28-code-walkthrough-skill.md` | 未开始 |
| 写 /palette command | `plans/2026-08-28-palette-command.md` | 未开始 |
| 打包三件套 | `plans/2026-08-28-plugin-packaging.md` | 未开始 |

推进顺序即上表顺序，逐条做完再动下一条。

## 结构性坑

- **改完 `owncc.md` 必须手动推送到生效位置**：权威副本在项目 `output-styles/`，但只有 `~/.claude/output-styles/owncc.md` 会被加载。两者跨卷，硬链接不可用、符号链接需提权，只能复制。漏推的失败是静默的——改的是个没人读的文件。
- **owncc 的规则文本是英文，它约束的输出是中文**：改它时别把模板里的固定字符串（`▍需你拍板` 等）和示例一并译掉，那些必须保持中文，否则模型会照着英文样例输出英文。
- **开发阶段不走本地 skill 加载**：使用方式是插件安装，项目内不保留 `.claude/skills/` 副本。要在本地验证 skill，得先装插件，不能靠往 `.claude/` 里放一份。

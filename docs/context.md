# atelier · context

> Last updated: 2026-09-06 · commit cccdea7
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

## 并行开发约定

多个会话同时开分支时，以下三条无条件遵守——**会话之间看不到彼此，本文件是唯一的共享通道**：

1. **功能分支不改 `context.md` 与 `roadmap.md`**。只改自己那份 `plans/` 文档。这两个文件是所有分支共享的索引，同时改必冲突，且是手动解的文本冲突。合并回 main 后由一个会话统一更新。
2. **功能分支不改 `output-styles/owncc.md`**。生效副本是 `~/.claude/output-styles/` 里的全局单例，切分支它不跟着变——A 分支改完切到 B 会话，读的仍是 A 的版本，且没有任何提示。要改 owncc 就停下并行，单独在 main 上做。
3. **插件测试是串行的**。装进测试环境的插件读的是当前 checkout 出来的那个分支；一个会话切分支，另一个会话正在测的内容就变了。同名插件也无法同时装两份。要测就约一个独占时间窗。

分支命名与工作面划分见 `roadmap.md` 阶段二。

## 结构性坑

- **改完 `owncc.md` 必须手动推送到生效位置**：权威副本在项目 `output-styles/`，但只有 `~/.claude/output-styles/owncc.md` 会被加载。两者跨卷，硬链接不可用、符号链接需提权，只能复制。漏推的失败是静默的——改的是个没人读的文件。
- **owncc 的规则文本是英文，它约束的输出是中文**：改它时别把模板里的固定字符串（`▍需你拍板` 等）和示例一并译掉，那些必须保持中文，否则模型会照着英文样例输出英文。
- **验证 skill 行为必须装插件**：项目内不保留 `.claude/skills/` 副本，往 `.claude/` 里放一份不是可选方案。改完任何 skill 或 command，要验证就走 `claude plugin marketplace add <本项目路径>` 装进测试环境跑，静态检查（搜关键词、比对内容）代替不了行为验证。

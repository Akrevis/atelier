# 写 /palette command

> Opened: 2026-08-28 · commit 4229fe1
> 来源：原 backlog T-16

## 目标

ASCII 撑不住的图（>15 节点）的出口。用户主动敲，生成可视化 HTML。

| 环节 | 机制 |
|---|---|
| 存储 | `~/.claude/atelier/projects/<项目路径编码>/<会话 ID>.html`，照 Claude Code 存会话的规矩分目录 |
| 复看 | 同会话内多图做成 tab 切换 |
| 生命周期 | 路径即生命周期：同一会话追加进同一个文件，新会话落新文件名；建新文件时清掉同目录 30 天没动过的旧画板 |
| 调用权 | frontmatter 设 `disable-model-invocation: true`，从机制上保证 AI 不能自己调、只能建议用户敲 |

放 `~/.claude/` 而非当前目录，是为了不污染项目、不被 git 追踪——它是临时白板，不是存档。

**存储方式 2026-09-07 改过一次**，原设计是单文件加会话标识比对，在并行会话下会互相清空。理由与代价见 `devlog.md`。

## 边界

- **不做持久化知识地图**——「我想留下这张图」是另一个需求，第二版单独设计。两个需求混在一个文件里通常两边都做不好
- **不做**跨项目的图检索。按项目分了目录，但没有「列出我所有画过的图」这种入口——要找就自己去 `~/.claude/atelier/projects/` 翻

## 推断的前提

| 前提 | 现在的状态 |
|---|---|
| `disable-model-invocation: true` 让 description 不进上下文、只有用户敲才加载 | 官方文档确认字段对 command 文件同样生效（command 与 skill 共用一套 frontmatter，只有 `name` 和 `paths` 被忽略）。**行为实测仍缺**——要装插件 |
| 「新会话第一次调用就重置」靠文件内写入会话标识判断 | **不成立，已换掉**。会话标识改作文件名而非文件内容，重置这个动作本身随之消失 |
| tab 切换用纯 HTML + 少量内联 JS，无外部依赖 | 成立。整份模板零依赖，`file://` 下直接可用 |

## 完成判据

- [x] `commands/palette.md` 存在，frontmatter 含 `disable-model-invocation: true`
- [x] 敲一次在 `~/.claude/atelier/projects/<项目>/<会话 ID>.html` 生成文件，浏览器打开能看到图
- [x] 同会话敲第二次，两张图以 tab 并存，第一张仍在
- [x] 两个会话各敲各的，各写各的文件，谁也不清空谁
- [ ] 实测：AI 在不该调用时没有自行调用（问一个 20 节点的结构，它应当画主干并**建议**用户敲，而不是自己调）

前四条走脚本跑完整流程 + headless Edge 截图核对，不是静态检查。最后一条要装插件才能验，等独占时间窗。

## 进度

- 2026-08-28 从 backlog 拆出，未开始
- 2026-09-06 分支 `feat/palette` 开工
- 2026-09-07 实现完成，交付 `commands/palette.md` + `assets/palette-template.html`
- 2026-09-07 存储改为按项目、按会话分文件（用户拍板），原单文件方案在并行会话下会互相清空

### 自己定的那几件事

| 待定项 | 选了什么 | 为什么 |
|---|---|---|
| 存储与生命周期 | 会话 ID（`CLAUDE_CODE_SESSION_ID`）当文件名，项目路径编码当目录名，编码规则与 `~/.claude/projects/` 一致 | 文件在就追加、不在就新建，**「是不是新会话」这个判断整个不需要了**。并行会话天然隔离 |
| 图怎么画 | **模型只写数据，页面负责布局**。JSON 给 `{id,label,note,kind,col,row}` 与边表，坐标、连线、避让由内联 JS 算 | 让模型手写 SVG 坐标必错位；给它网格序号则错不了。也彻底避开 Mermaid 的 CDN 依赖 |
| 多图并存 | 每张图一个 `<script class="figure" type="application/json">`，插在 `<!-- FIGURES:END -->` 之前 | 追加只需替换那一行锚点，模型不必读写整份 HTML。单行 JSON 让替换无歧义 |
| tab 状态 | `location.hash` 记当前页 | 按 `R` 刷新后仍停在原处；`sessionStorage` 做不到且首次读出 `null` 会被 `+null` 静默转成 0 |

### 连线的四种走法

自环 / 同列直上下 / 向右折线 / 绕行下方通道。向右折线会先做**矩形-线段相交检测**，撞上中间节点就降级走通道；同一通道里的多条线按边序错开 9 像素，不叠在一起。列数建议 ≤6，再多一屏放不下。

## 分派契约

| 项 | 值 |
|---|---|
| 分支 | `feat/palette` |
| 起点 | main 的 `674ed25` |
| 交付物 | `commands/palette.md` + HTML 模板 |
| 完成后接手 | `docs/readme` 分支需要这个 command 的能力清单 |

### 已冻结的前置——拿着就能开工，不要等人也不要改

| 前置 | 在哪 | 状态 |
|---|---|---|
| 存储位置、生命周期、tab 切换、调用权 | `docs/design.md` 第五节 | 冻结 |
| 什么时候该建议用户敲（>15 节点） | `output-styles/owncc.md` 画图第 5 条 | 冻结 |
| `disable-model-invocation: true` | 本文档「目标」节 | 冻结 |

与另外两个包**零文件重叠**，`/palette` 不参与 skill 的触发竞争（它靠用户主动敲），因此没有语义依赖。

### 不许碰

`docs/context.md` · `docs/roadmap.md` · `output-styles/owncc.md`。

### 唯一允许回来问决策者的情况

实测发现 `disable-model-invocation` 的行为与文档描述不符，导致「AI 不能自己调」这个设计前提不成立。那要改设计，不是改实现。

HTML 模板长什么样、tab 用什么实现、新会话重置的判断机制——**全部自己定，在「进度」里记下选了什么**。

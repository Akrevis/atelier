# 写 /palette command

> Opened: 2026-08-28 · commit 4229fe1
> 来源：原 backlog T-16

## 目标

ASCII 撑不住的图（>15 节点）的出口。用户主动敲，生成可视化 HTML。

| 环节 | 机制 |
|---|---|
| 存储 | 单文件 `~/.claude/atelier/palette.html`，不分全局/项目 |
| 复看 | 同会话内多图做成 tab 切换 |
| 生命周期 | 会话内累积 → 新会话第一次 `/palette` 全部重置 |
| 调用权 | frontmatter 设 `disable-model-invocation: true`，从机制上保证 AI 不能自己调、只能建议用户敲 |

放 `~/.claude/` 而非当前目录，是为了不污染项目、不被 git 追踪——它是临时白板，不是存档。

## 边界

- **不做持久化知识地图**——「我想留下这张图」是另一个需求，第二版单独设计。两个需求混在一个文件里通常两边都做不好
- **不做**全局/项目区分

## 推断的前提（待确认）

- 假设 `disable-model-invocation: true` 的效果是 description 不进上下文、只有用户敲才加载；**来自文档，未实测**
- 假设「新会话第一次调用就重置」可以靠**文件内写入会话标识**判断，而非依赖 AI 记得自己是不是新会话；**具体机制未定**
- 假设 tab 切换用纯 HTML + 少量内联 JS，无外部依赖

## 完成判据

- [ ] `commands/palette.md` 存在，frontmatter 含 `disable-model-invocation: true`
- [ ] 敲一次生成 `~/.claude/atelier/palette.html`，浏览器打开能看到图
- [ ] 同会话敲第二次，两张图以 tab 并存，第一张仍在
- [ ] 新开会话敲第一次，旧内容被清空
- [ ] 实测：AI 在不该调用时没有自行调用（问一个 20 节点的结构，它应当画主干并**建议**用户敲，而不是自己调）

## 进度

- 2026-08-28 从 backlog 拆出，未开始

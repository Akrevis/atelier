# 打包三件套

> Opened: 2026-08-28 · commit 4229fe1
> 来源：原 backlog T-11

## 目标

`.claude-plugin/plugin.json` + `marketplace.json` + 双语 README，参照 groundwork 的结构。

## 边界

- **`owncc.md` 不进插件**——插件技术上支持分发 output style，但会毁掉三件事：独立性（想要格式规范就得装整个插件）、占掉全局唯一的 style 位、`force-for-plugin` 静默覆盖用户设置
- **安全底线不进 output style**——它可被 `/config` 随时切换，搬进去等于给底线装了个开关

## 依赖

前四个任务全部完成。**前三步定型才知道 README 写什么。**

## 推断的前提（待确认）

- 假设 marketplace 走自建仓库分发，不投官方源
- 假设双语指中文 + 英文，中文为主

## 完成判据

- [ ] `.claude-plugin/plugin.json` 通过 Claude Code 本地安装校验
- [ ] 从干净环境装一次，两个 skill 与 `/palette` 均出现在清单里
- [ ] 卸载后 `~/.claude/` 下无残留（`atelier/palette.html` 除外，需在 README 说明）
- [ ] README 读起来像一份从头写成的成品——不留演化痕迹、不写项目私史、不出现「原本 X 后来改成 Y」
- [ ] README 里每条功能描述都对应实际存在的机制（对照 `skills/` 与 `commands/` 逐条核）

## 进度

- 2026-08-28 从 backlog 拆出，未开始

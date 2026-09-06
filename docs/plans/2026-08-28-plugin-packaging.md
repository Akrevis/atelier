# 打包三件套

> Opened: 2026-08-28 · commit 4229fe1
> 来源：原 backlog T-11

## 目标

`.claude-plugin/plugin.json` + `marketplace.json` + 双语 README，参照 groundwork 的结构。

## 边界

- **`owncc.md` 不进插件**——插件技术上支持分发 output style，但会毁掉三件事：独立性（想要格式规范就得装整个插件）、占掉全局唯一的 style 位、`force-for-plugin` 静默覆盖用户设置
- **安全底线不进 output style**——它可被 `/config` 随时切换，搬进去等于给底线装了个开关

## 依赖

README 依赖前三步定型才知道写什么。

**但两个 JSON 提前做了**：删掉 `.claude/skills/` 后本地已无路径能加载 skill，行为实测全部阻塞。`plugin.json` 本来就要写，提前写不浪费，写完每一步都能装进测试环境真跑。这是拆分执行，不是提前完成。

## 推断的前提（待确认）

- 假设 marketplace 走自建仓库分发，不投官方源
- 假设双语指中文 + 英文，中文为主

## 完成判据

- [x] `.claude-plugin/plugin.json` 与 `marketplace.json` 存在且是合法 JSON
- [ ] 通过 Claude Code 本地安装校验（VM 测试环境待跑）
- [ ] 从干净环境装一次，两个 skill 与 `/palette` 均出现在清单里
- [ ] 卸载后 `~/.claude/` 下无残留（`atelier/palette.html` 除外，需在 README 说明）
- [ ] README 读起来像一份从头写成的成品——不留演化痕迹、不写项目私史、不出现「原本 X 后来改成 Y」
- [ ] README 里每条功能描述都对应实际存在的机制（对照 `skills/` 与 `commands/` 逐条核）

## 进度

- 2026-08-28 从 backlog 拆出，未开始

## 已完成的部分

`plugin.json` 与 `marketplace.json`（`0.0.1`）。参照本机已装的 groundwork 1.1.1 的真实结构写的，不是凭文档记忆。

**两个字段刻意留空**：`repository` 与 `license`。项目没有远程仓库、没有 LICENSE 文件，写上去就是声明不存在的东西。有了再补。

**description 只描述当前真有的能力**——一个 explain skill。`code-walkthrough` 与 `/palette` 尚未写，描述里不出现。

## 进度

- 2026-08-28 从 backlog 拆出
- 2026-09-06 两个 JSON 完成，为解除实测阻塞而提前；README 仍待前三步定型

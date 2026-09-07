---
description: 把刚才讲的结构画成一张可交互的 HTML 图——ASCII 框图撑不住的时候用。同一会话内多张图以标签页并存，按项目和会话分文件存放，多个会话同时开也互不覆盖。
argument-hint: [画什么，留空则画刚讨论过的结构]
disable-model-invocation: true
allowed-tools: Read, Edit, Bash(mkdir:*), Bash(sed:*), Bash(tr:*), Bash(pwd:*), Bash(test:*), Bash(explorer.exe:*), Bash(open:*), Bash(xdg-open:*)
---

# /palette —— 画板

终端里的 ASCII 框图有硬上限：87 列、8 个节点。超过就该出图。这个命令把结构画进一张本地 HTML，浏览器打开，节点可悬停看连线。

**画什么**：`$ARGUMENTS` 说了就画它；没说就画**刚才这段对话里讨论的那个结构**——上一轮讲的调用链、状态机、模块依赖、数据流。找不到可画的东西就直说，别硬凑一张。

## 一、准备画布

存储照搬 Claude Code 存会话的那套：**按项目分目录，按会话分文件**。

```
~/.claude/atelier/projects/<项目路径编码>/<会话 ID>.html
                           F--workspace-projects-atelier/034c08c0-9e65-….html
```

| 段 | 怎么来 |
|---|---|
| 项目路径编码 | 当前工作目录里的 `:` `\` `/` 全换成 `-`。Windows 上要取 Windows 形式的路径（`pwd -W`），`F:\x\y` 编码成 `F--x-y` |
| 会话 ID | 环境变量 `CLAUDE_CODE_SESSION_ID` |

**这套路径本身就是生命周期**，不需要再判断「是不是新会话」：新会话必然落到新文件名，同一会话再敲就是往同一个文件里追加。两个会话同时开着也互不干扰——各写各的。

建新文件时顺手清掉同目录下 30 天没动过的旧画板，跟 Claude Code 清理会话记录的周期对齐。这一步**没有预授权**（`find -delete` 不该被无条件放行），用户拒了就跳过——清理失败不影响画图，别停下来。

Bash（Windows 的 git bash 同样适用）：

```bash
SID="${CLAUDE_CODE_SESSION_ID:-nosession}"
PROJ=$(pwd -W 2>/dev/null || pwd)
DIR=~/.claude/atelier/projects/$(echo "$PROJ" | tr ':\\/' '-')
FILE="$DIR/$SID.html"
mkdir -p "$DIR"
test -f "$FILE" || {
  find "$DIR" -maxdepth 1 -name '*.html' -mtime +30 -delete
  sed -e "s/__SESSION__/$SID/" -e "s#__PROJECT__#$PROJ#" \
      "${CLAUDE_PLUGIN_ROOT}/assets/palette-template.html" > "$FILE"
}
echo "$FILE"
```

PowerShell：

```powershell
$sid  = if ($env:CLAUDE_CODE_SESSION_ID) { $env:CLAUDE_CODE_SESSION_ID } else { 'nosession' }
$dir  = Join-Path "$HOME\.claude\atelier\projects" ($PWD.Path -replace '[:\\/]', '-')
$file = Join-Path $dir "$sid.html"
New-Item -ItemType Directory -Force $dir | Out-Null
if (-not (Test-Path $file)) {
  Get-ChildItem $dir -Filter *.html |
    Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } | Remove-Item -Confirm:$false
  (Get-Content "$env:CLAUDE_PLUGIN_ROOT\assets\palette-template.html" -Raw).
    Replace('__SESSION__', $sid).Replace('__PROJECT__', $PWD.Path) |
    Set-Content $file -Encoding utf8
}
$file
```

`CLAUDE_CODE_SESSION_ID` 读不到时文件名退化成 `nosession.html`，**该项目下所有读不到 ID 的会话共用它、只追加不清空**。真遇到，就在报告路径时说一句这个文件是共用的。

## 二、追加一张图

用 Edit 把结束锚点替换成「新图 + 结束锚点」：

| 找 | 换成 |
|---|---|
| `<!-- FIGURES:END -->` | `<script class="figure" type="application/json">{…}</script>` 换行 `<!-- FIGURES:END -->` |

**JSON 压成一行**，否则锚点替换会撞上多行内容。文本里的小于号一律写成 `\u003c`——原样写进去会提前关掉 `script` 标签，整页白屏。

### 数据格式

```json
{
  "title": "四到八字标题",
  "summary": "一句话说明这张图在讲什么",
  "nodes": [
    {"id": "a", "label": "节点名", "note": "框内小字注解，可省", "kind": "accent", "col": 0, "row": 0}
  ],
  "edges": [
    {"from": "a", "to": "b", "label": "边上的字，可省", "style": "dashed"}
  ]
}
```

| 字段 | 说明 |
|---|---|
| `col` / `row` | 网格坐标，从 0 起的整数。**只管相对次序，不管像素**——坐标、连线、避让全部由页面算 |
| `kind` | 省略为默认灰。`accent` 主干 · `danger` 出问题的地方 · `ok` 正常路径 · `muted` 次要（虚线灰） |
| `style` | 省略为实线。`dashed` 表示异步、可选、失败分支 |
| `note` | 一句话，不是段落。放约束、耗时、持有的资源这类**读图时要同时看到**的信息 |

### 摆位约定

**`col` 是流向，`row` 是同层并列。** 认准这一条，图就不会乱：

| 内容形状 | 怎么摆 |
|---|---|
| 调用链、数据流、管道 | `col` 递增（左→右），分支占不同 `row` |
| 层级、包含关系 | 上层 `row` 小，同层的兄弟节点排 `col` |
| 状态机 | 主路径沿 `col` 铺开，回边自然从下方绕回，不用特殊处理 |

同一 `(col, row)` 允许放多个节点，但它们会叠在一起——**这是坐标写错了**，不是特性。

**`col` 最多到 5**（六列约 1560 像素，再宽就要横向滚动，一屏看不全等于白画）。层数超了就拆成两张图。

## 三、打开

`$FILE` 就是上一步算出来的那条路径：

| 平台 | 命令 |
|---|---|
| Windows | `explorer.exe "$(cygpath -w "$FILE")"` 或 PowerShell `Invoke-Item $file` |
| macOS | `open "$FILE"` |
| Linux | `xdg-open "$FILE"` |

**页面已经开着的时候不用再开一次**——换成一句「已追加第 N 张，页面里按 `R` 刷新」。浏览器不会自己重载 `file://`。

## 四、画得好的判据

1. **一张图只讲一件事**。20 个节点塞进一张，等于把 ASCII 的失败照搬到 HTML 上。拆：主干一张，局部展开各一张，靠标签页并排——**同一次调用可以出多张图**
2. **`label` 是名字，不是句子**。解释放 `note`，因果放 `edges` 的 `label`
3. **画完不要用文字把图再讲一遍**。图旁边只留 `summary` 那一句
4. **没有边的一堆框不是图**，是表格——那种内容退回终端用 markdown 表格，别开画板

## 反面模式

1. 把线性推理、取舍权衡画成框图。**推理是因果链，装进框里连接词就丢了**
2. 为了「看起来专业」给每个节点都配 `note` 和 `kind`。默认灰、无注解才是常态，标记只给真正特殊的节点
3. 追加时手写整份 HTML。**永远只动 `FIGURES:END` 那一行**，其余内容不碰
4. 用户没敲这个命令，你自己拿它去画图。它对你是不可调用的——该出图时**建议用户敲**，然后照常给终端里的 ASCII 主干

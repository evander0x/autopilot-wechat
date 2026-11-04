---
name: wechat-editorial
description: |
  公众号排版模板「编辑手记」：把 Markdown 渲染成可直接粘贴进微信编辑器的 HTML
  （全内联样式、无脚本、无外部字体）。固定版式：栏目标签、序号块、下划线高亮、
  引用块、清单卡片、金句区。
  触发关键词：公众号排版、微信排版、排一版、套模板、发公众号、编辑手记风格。
  不负责选题、写作和发布——那些交给 wewrite。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
---

# wechat-editorial — 公众号排版模板

## 何时用

用户要「排版 / 出一版 / 套编辑手记风格 / 发公众号」时，用本模板生成 HTML。
正文内容可以由本会话写作，也可以来自 `wewrite`（选题/写稿/审稿）的产出。

## 怎么做

1. 把待排版内容写成 Markdown，遵循下面的约定。
2. 运行：

```bash
python3 ~/.workbuddy/skills/wechat-editorial/wechat_layout.py article.md -o article.html
# 可选：--theme ink（墨色版）| teal（默认，深青）
# 可选：--fragment 只输出正文片段（不含页面外壳），供推草稿箱使用。
#       fragment 会自动省略 front matter 的 label 和文首大标题——
#       公众号编辑器里标题单独填，正文重复出现是冗余；导语保留。
```

3. 用 present_files 打开 `article.html` 给用户看。
4. 需要推草稿箱时，用 `~/.workbuddy/skills/wechat-autopilot/push_draft.py`
   （吃 `--fragment` 的产物，样式原样上传，不经 wewrite 的主题转换）。

## Markdown 约定

```markdown
---
title: 页面标题（仅用于浏览器标签）
label: 职场观察 · 效率          # 可选，标题上方的青色栏目标签
lead: 一句话导语                 # 可选，标题下方的灰色引子
---
# 文章标题（可用 <br> 手动折行）

正文段落，正常写。需要高亮的句子用 ==两个等号== 包住，会渲染成浅色下划线。

## 01 小节标题                  # 以数字开头 → 自动生成深青圆角序号块
> 引用或示例                    # 淡青底 + 左竖线
![配图](img/xx.jpg "图注文字")   # 居中大图 + 灰色图注；路径可相对可绝对
1. 有序列表                     # 序号用主色
:::card 卡片标题
1. 条目一
2. 条目二
:::
:::punch
第一行（17px 加粗居中）
第二行（15px 灰色）
:::
```

## 版式规范（改样式只改这里）

| 元素 | 规格 |
|---|---|
| 主色 | `#0e7490`（墨色版 `#4a4a4a`） |
| 正文字号 | 16px / 行高 1.95 / 字距 .4px / 两端对齐 |
| 大标题 | 26px / 700 / `#171717` / 行高 1.45 |
| 小节标题 | 18px / 700 + 30×30 圆角 8px 序号块 |
| 引用块 | 底色 `#f4f8f9` + 3px 左竖线 + 右侧圆角 8px |
| 卡片 | 底色 `#faf9f7` + 1px `#ecebe6` 边框 + 圆角 14px |
| 高亮 | `linear-gradient(to top,#e6f4f6 40%,transparent 40%)` |
| 金句区 | 上下 1px 细线，居中 |
| 页面 | 420px 白卡 + 外层米灰底（仅预览用，复制时不会带上） |

## 注意

- 输出必须是全内联样式：微信编辑器会剥离 `<style>`、外链字体和脚本。
- 公众号正文不接受 `position`、`flex` 之外的复杂布局，也不支持自定义字体加载。
- 改版式时只改 `THEMES` 字典和 `render_*` 函数里的样式串，不要引入外链资源。

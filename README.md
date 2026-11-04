# autopilot-wechat

公众号全流程自动化：**选题 → 写稿 → 审稿 → 排版（编辑手记）→ 推送草稿箱**。

一套给 AI Agent（WorkBuddy / Claude 类环境）用的 skill 集合 + 两个独立脚本。
内容能力来自 [wewrite](https://github.com/imraywang/wewrite)，视觉版式由本项目的「编辑手记」模板负责，发布走微信公众号官方 API。

## 它解决什么问题

| 痛点 | 方案 |
|---|---|
| AI 排版产出是通用模板，不像「你的号」 | 固定版式写进代码，序号块 / 高亮 / 卡片 / 金句区一键生成 |
| Markdown 转 HTML 后推草稿箱会丢样式 | `wechat_layout.py --fragment` 产出全内联样式片段，原样上传 |
| 正文外链图片会被微信剥离 | `push_draft.py` 自动扫描 `<img>`，本地图片先传 `media/uploadimg` 换微信 URL |
| 发布过程不可控 | 默认不发布；明确授权才调 API，发布后回读校验 |

## 仓库结构

```
wechat-autopilot/
├── skills/
│   ├── wechat-autopilot/     # 编排层：全流程 SKILL.md + 推送脚本
│   │   ├── SKILL.md
│   │   └── push_draft.py     # 上传封面/插图 → draft/add → 回读校验
│   └── wechat-editorial/     # 视觉层：「编辑手记」排版模板
│       ├── SKILL.md          # Markdown 约定 + 版式规范
│       └── wechat_layout.py  # Markdown → 全内联样式 HTML
└── examples/                 # 一篇样例文章（源文件与两种产物）
    ├── sample.md             # 编辑手记语法的源文件
    ├── sample.html           # 完整页面（预览用）
    └── sample.fragment.html  # 正文片段（推送用）
```

## 快速开始

### 1. 排版一篇文章

```bash
python3 skills/wechat-editorial/wechat_layout.py article.md -o article.html
# --theme ink     墨色版（默认为 teal 深青）
# --fragment      只输出正文片段（推草稿箱用）
```

源文件写法见 [`skills/wechat-editorial/SKILL.md`](skills/wechat-editorial/SKILL.md)，核心约定：

```markdown
---
label: 职场观察 · 效率        # 标题上方的栏目标签
lead: 一句话导语
---
# 文章标题

## 01 小节标题                 # 数字开头 → 自动生成序号块
==这句话会高亮==
> 引用块
![配图](img/x.jpg "图注")
:::card 卡片标题
1. 条目
:::
:::punch
金句大字
灰色补充
:::
```

### 2. 推送草稿箱

前置条件：公众号后台拿到 `appid` + `secret`，写入 `~/.wewrite/config.yaml`（权限 600），并把本机公网 IP 加入开发者平台的 API IP 白名单。

```bash
python3 skills/wechat-autopilot/push_draft.py article.fragment.html \
    --cover cover.png \
    --title "标题（≤25 个汉字）" \
    --digest "摘要（≤40 个汉字）"
```

脚本行为：上传封面为永久素材 → 扫描正文 `<img>` 逐张上传换 URL → `draft/add` 建草稿 → 打印 media_id。报错对照：`40164` IP 不在白名单、`48001` 账号无草稿权限（个人未认证号）、`45003` 标题超长。

### 3. 作为 Agent skill 安装

把 `skills/` 下两个目录复制或软链到你的技能目录（如 `~/.workbuddy/skills/`、`~/.claude/skills/`）即可。配合 [wewrite](https://github.com/imraywang/wewrite) 的选题/写稿/审稿能力，一句话触发全流程。

## 版式规范（编辑手记）

| 元素 | 规格 |
|---|---|
| 主色 | `#0e7490`（墨色版 `#4a4a4a`） |
| 正文 | 16px / 行高 1.95 / 无首行缩进 / 两端对齐 |
| 小节标题 | 18px + 30×30 圆角序号块 |
| 引用块 | `#f4f8f9` 底 + 3px 左竖线 |
| 卡片 | `#faf9f7` 底 + 1px 边框 + 圆角 14px |
| 高亮 | `linear-gradient` 浅青下划线 |
| 金句区 | 上下 1px 细线 + 居中 |

输出全部为内联样式，不含 `<style>`、外链字体与脚本，可直接粘进微信编辑器。

## 安全提醒

- `appid` / `secret` 只存本机 `~/.wewrite/config.yaml`，**不要**提交进仓库或贴进对话
- secret 等同于公众号的发布权，怀疑泄露立即在后台重置
- 本项目不包含也不读取任何密钥，凭证由脚本运行时从本地配置载入

## License

MIT

---
name: wechat-autopilot
description: |
  公众号全流程自动化：选题 → 写稿 → 审稿 → 排版（编辑手记）→ 推草稿箱。
  把 wewrite 的内容能力与 wechat-editorial 的版式模板串成一条链路。
  触发关键词：写一篇公众号文章、完整制作一篇、公众号自动化、选题写稿一条龙、
  推草稿箱、排一版发公众号。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebSearch
  - WebFetch
---

# wechat-autopilot — 公众号全流程

## 链路总览

```
风格(style.yaml) → 选题 → 写稿(article.md) → 审稿 → 排版 → [授权后] 推草稿箱
   wewrite-style    wewrite-topic  写作+审稿      wechat-editorial  push_draft.py
```

- 内容判断（选题好不好、稿子行不行）：由 Agent 按 wewrite 的 SKILL.md 执行。
- 确定性操作（转 HTML、调微信 API）：走脚本，不要手搓。
- **默认不生图、不发布**；用户明确说「推到草稿箱」才允许调 push_draft.py。

## Step 0 风格

`~/.wewrite/style.yaml` 不存在时，先问三句：公众号名+方向、主要话题、调性
（专业分析/朋友聊天/犀利锐评/温暖故事/幽默吐槽）。写成 style.yaml 后继续，
不要为风格单独停一轮。

## Step 1 选题

用户给了题目就直接用；否则：

```bash
env -u PYTHONPATH ~/.local/bin/wewrite hotspots --limit 20
```

结合账号定位挑 2-3 个候选，向用户报一句话理由后开工（连续模式可自行定夺并说明）。

## Step 2-3 写稿 + 审稿

按 `~/wewrite/skills/wewrite-write/SKILL.md` 与 `wewrite-review/SKILL.md` 的标准执行：
任务书 → 主张与来源清单（可核实的走 WebSearch，不可核实的只写经验判断）→ 初稿 →
事实/观点/实用性/可读性自审 → 终稿。

### 写作基调：实战优先，不写废话

**每篇文章都要能被读者"照着做"或"据此判断"，否则不写。**

- **开头第一段就给结论或结果**：一句话说清读者能得到什么（跑通什么、省下什么、避掉什么坑），不铺垫时代背景、不写"随着……的发展"
- **每一节必须有硬货**：命令、参数、路径、价格、版本号、对比数字、具体坑位与解法。连续三段没有可执行信息或可核实事实，就删
- **步骤写成祈使句**：先做什么、在哪做、报什么错、怎么改。能贴命令就不描述命令
- **砍掉所有填充**：过渡句、排比抒情、空洞总结、"希望对你有帮助"、复述小节标题的结尾段
- **禁用套话**："在这个时代""不得不说""值得注意的是""众所周知""赋能/闭环/抓手/打法""让我们一起"
- **篇幅宁短勿水**：默认 1200-1800 字，每段不超过 3 行；素材不够就缩小主张，绝不注水
- **结尾给行动或边界**：一句可执行的下一步，或一句"什么情况下这套不适用"，二选一

审稿时专门过一遍"废话体检"：删掉不影响理解的句子后，如果信息量没变，那些句子就该删。

终稿用**编辑手记语法**保存（见 `~/.workbuddy/skills/wechat-editorial/SKILL.md`）：
front matter（label/lead）+ `## 01 小节` + `==高亮==` + `> 引用` + `:::card` + `:::punch`。

## Step 4 排版

```bash
# 给用户看的预览
python3 ~/.workbuddy/skills/wechat-editorial/wechat_layout.py article.md -o article.html
# 供推送用的正文片段
python3 ~/.workbuddy/skills/wechat-editorial/wechat_layout.py article.md -o article.fragment.html --fragment
```

## Step 5 发布（需用户明确授权）

```bash
~/wewrite/.venv/bin/python ~/.workbuddy/skills/wechat-autopilot/push_draft.py \
    article.fragment.html --cover cover.png \
    --title "标题" --digest "摘要"
```

约束：

- 标题 ≤25 个汉字（约 78 字节），超了微信报 45003
- 摘要 ≤120 字节（约 40 个汉字）
- 封面 900×383；没有现成图就先问用户，或经其同意后用图像生成工具做一张
- 凭证在 `~/.wewrite/config.yaml`；API 报 40164 = IP 不在白名单，48001 = 账号无草稿权限

## Step 5.5 更新已有草稿（用户在后台手改过时）

**铁律：先拉取远程草稿并备份，才能覆盖。绝不盲推。**

脚本与本 SKILL.md 同目录（`skills/wechat-autopilot/`）。

远程草稿（media_id 可用 `fetch_draft.py list` 列出）：

```bash
~/wewrite/.venv/bin/python <skill_dir>/fetch_draft.py list                 # 列草稿箱
~/wewrite/.venv/bin/python <skill_dir>/fetch_draft.py get <media_id>       # 拉取原文落盘
```

就地覆盖（脚本内置强制备份：自动 draft/get 存 `draft_remote_backup-<时间戳>.html/.json`，
并提示远程有而本地没有的文字，拉取失败即中止）：

```bash
~/wewrite/.venv/bin/python <skill_dir>/update_draft.py <media_id> article.fragment.html \
    --title "标题" --digest "摘要" --cover-media-id <thumb_media_id>
```

- 若用户在后台手改过内容，先 diff 远程与本地，把手改并入本地源稿再覆盖
- `draft/update` 覆盖后 media_id 不变，不会产生新草稿
- 凭证只从 `~/.wewrite/config.yaml` 运行时读取，本仓库永不落 AppID/AppSecret

## Step 6 汇报

完成后必须交代：标题、md/预览/片段的路径、草稿 media_id、来源数量、
以及发生过的降级（没生图 / 没发布 / 哪一步失败回退了）。

## 环境备忘

- wewrite CLI 必须 `env -u PYTHONPATH` 运行（WorkBuddy 的 Python shim 会拦它的目录创建）
- push_draft.py 用 `~/wewrite/.venv/bin/python`（自带 requests）
- 二者都不依赖对方，可单独使用

# 主题样式演示（style-demos）

两个「从真实公众号文章提炼版式 → 渲染引擎还原 → 案例文章验证」的完整示例。
每篇文章都是**样式演示用途**：正文内容为写作示例（案例文章为虚构示例），不代表任何真实报道或产品。

## 目录

| 目录 | 主题 | 版式参考来源 | 风格一句话 |
|---|---|---|---|
| `yicai/` | `--theme yicai`（财经蓝快讯流） | 第一财经 新闻推送 | 17px 两端对齐正文 + 财经蓝加粗小节标题 + 纯段落极简流 |
| `deepseek/` | `--theme deepseek`（DeepSeek 蓝发布体） | 深度求索（DeepSeek）官方公众号 技术发布 | 15px 深灰左对齐正文 + 18px 黑色加粗小节标题 + 品牌蓝链接行 |

每个目录里：

- `article.md` — Markdown 源稿（含 front matter，展示该主题适用的写作配套结构）
- `article.html` — 浏览器直接打开即可预览，白卡内正文全选复制进公众号编辑器可保留全部样式

## 怎么复现 / 套用到自己的文章

```bash
# 财经蓝快讯体
python3 skills/wechat-editorial/wechat_layout.py my_article.md -o my_article.html --theme yicai

# DeepSeek 蓝发布体
python3 skills/wechat-editorial/wechat_layout.py my_article.md -o my_article.html --theme deepseek

# 其他可选主题：teal（默认深青）/ ink（墨色）/ orange（橙色科技流）
```

Markdown 约定（栏目标签、高亮、卡片、金句、地址行等）见 `skills/wechat-editorial/SKILL.md`。

## 两套版式规格速查

### yicai · 财经蓝快讯流

- 正文：17px / 行高 1.75 / 字色 90% 黑 / 字距 1px / 左右 8px / 两端对齐
- 小节标题：财经蓝 `#0098d6` 16px 加粗独立成段，无序号块无底色
- 图注：12px 灰 `#888` / 字距 1px
- 气质：纯段落流，无卡片无分割线，靠蓝色小标题 + 留白分节
- 写作配套：导语段（"记者获悉/近日……"）→ 蓝色小节标题 3-4 个 → 结尾「来源 | xxx」行

### deepseek · DeepSeek 蓝发布体

- 正文：15px / 行高 1.75 / 深灰 `rgb(79,79,79)` / 左右 8px / **左对齐（不两端对齐）**
- 小节标题：18px 加粗黑色独立成段，品牌蓝不上标题
- 品牌蓝 `#4d6bfe`：只用于 `地址：` 链接行（纯文本蓝字）
- 图注：12px 灰 `#888`
- 气质：纯段落极简流，技术发布体语气
- 写作配套：开篇宣布发布 + 设计初衷 → 特性小节 → API 支持 / 定价调整 → 开源 + `地址：` 链接行 → 合作致谢收尾

> 版式规格仅从公开文章的排版特征（字号/颜色/间距/结构）归纳，用于学习排版，与原账号无关联。

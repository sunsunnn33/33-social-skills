---
name: 33-social-content
description: Prepares platform-optimized publish briefs from a video project folder. Reads a Pages script, researches trending tags per platform, generates adapted titles/descriptions/tags for each selected platform, and writes a publish-brief.md ready for social-publisher. Use when user says "准备发布内容", "生成发布文案", "帮我写各平台文案", "prepare publish brief", or provides a folder with a video and script.
version: 1.0.0
---

# Social Content Prep

Reads raw video project assets, generates platform-optimized content for each target platform, and produces a `publish-brief.md` for use by the `social-publisher` skill.

---

## Language

Respond in the user's language. Chinese input → Chinese output. Keep file paths and field names in English.

---

## Prerequisites

- macOS `textutil` (built-in, no install needed)
- `ffmpeg` (for first-frame extraction fallback): `brew install ffmpeg`
- `web-access` skill installed (for tag research)

---

## Step 1 — Locate Assets

Ask the user for the project folder path if not already provided.

Scan the folder for:

| Asset | Detection |
|-------|-----------|
| Script | First `*.pages` file found |
| Video | First `*.mp4 / *.mov / *.avi / *.webm` file found |
| Cover images | All `*.jpg / *.jpeg / *.png` files — detect aspect ratio of each |

**Cover aspect ratio mapping:**

Compute width:height ratio for each image file using:
```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 cover.jpg
```

| Detected ratio | Assigned to |
|----------------|-------------|
| ~16:9 (1.7–1.9) | YouTube, Bilibili, Weibo, 微信视频号, X |
| ~9:16 (0.5–0.6) | 抖音 |
| ~3:4 or ~4:5 (0.7–0.85) | 小红书 |

If multiple images match the same ratio, use the largest file by size.  
If no image matches a platform's required ratio → mark that platform as `cover: [first-frame]` (extracted in Step 6).

Report the mapping to the user:
```
资产识别结果：
脚本：script.pages ✓
视频：video.mp4 ✓
封面匹配：
  16:9 → cover-landscape.jpg（YouTube / Bilibili / 微博 / 视频号 / X）
  9:16 → cover-portrait.jpg（抖音）
  3:4  → [未找到] → 将使用视频第一帧（小红书）
```

---

## Step 2 — Read Script

Extract text from the `.pages` file:
```bash
textutil -convert txt "{script_path}" -stdout
```

If the output is empty or garbled (newer Pages format):
```bash
unzip -p "{script_path}" QuickLook/Preview.pdf 2>/dev/null | strings | head -200
```

If both fail, ask the user: "无法自动读取 .pages 文件，请将脚本内容粘贴到这里，或导出为 .txt 后告诉我路径。"

From the extracted text, identify:
- **核心主题**：这个视频讲的是什么
- **关键信息点**：3-5 个最重要的内容点
- **目标受众**：面向什么人群
- **内容风格**：干货教程 / 生活分享 / 娱乐 / 评测 / Vlog 等

Summarize and confirm with the user:
```
脚本解读：
主题：[X]
关键点：[A] / [B] / [C]
受众：[X]
风格：[X]

这个理解准确吗？如有偏差请告诉我。
```

---

## Step 3 — Select Target Platforms

Ask the user which platforms to include this time:

```
本次发布到哪些平台？（可多选）

1. 微信视频号
2. 微博
3. X（Twitter）
4. Bilibili
5. 抖音
6. 小红书
7. YouTube

请输入平台编号，用逗号分隔，例如：4,5,6,7
```

---

## Step 4 — Research Trending Tags

For each selected platform, use the `web-access` skill to search for trending tags related to the topic.

Run searches in parallel where possible.

| Platform | Search query pattern | Selection criteria |
|----------|---------------------|-------------------|
| 小红书 | `小红书 #{topic} 话题 热门 浏览量` | 1 超大话题(>10亿) + 2 垂直话题(>1亿) + 2 精准小话题 |
| 抖音 | `抖音 #{topic} 热门话题 上升` | 3-5 个，优先上升趋势，避开已饱和 |
| 微博 | `微博 {topic} 热搜 话题广场` | 2-3 个相关热搜或话题 |
| X | `{topic} trending hashtags twitter` | 3-5 个英文 hashtag |
| Bilibili | `bilibili {topic} 热门tag 分区` | 5-8 个，含分区名 |
| YouTube | `{topic} youtube keywords SEO search volume` | 5-8 个，长尾词优先 |
| 微信视频号 | `微信视频号 #{topic} 话题` | 2-3 个话题 |

For each tag, note the source/reason (e.g., "浏览量 82亿", "本周上升 340%", "搜索量高").

If `web-access` is unavailable, generate tags from built-in knowledge and clearly mark them as `[未验证热度]`.

---

## Step 5 — Generate Platform Content

For each selected platform, generate title, body, and tags. Apply platform rules strictly.

### 微信视频号
- 标题：≤50字，生活化，可加emoji
- 描述：支持 #话题，2-3 行，简洁
- 标签格式：#话题名（嵌入描述）
- 风格：接近朋友圈，真实自然

### 微博
- 正文：≤140字（短微博）或长文转头条文章
- 标签格式：#话题# 放正文中
- 风格：口语化，有传播性

### X（Twitter）
- 正文：≤280字（非Premium）
- 标签格式：#hashtag 放正文末尾
- 风格：简洁直接，英文或中英混合视受众而定

### Bilibili
- 标题：≤80字，含关键词，描述性强
- 简介：结构化，含关键词，可加时间戳
- 标签：5-10个，每个≤20字
- 分区：根据内容判断（生活/知识/科技/游戏等）
- 风格：信息密度高，面向 B站用户

### 抖音
- 标题/描述：≤55字，口语化，制造好奇心或情绪
- 标签格式：#话题 嵌入描述末尾
- 风格：有爆点，开头抓眼球

### 小红书
- 标题：≤20字，有情绪价值，制造悬念或共鸣（如"xx，我后悔没早知道"）
- 正文：≤1000字，分段，多换行，适量emoji，像朋友分享
- 话题：#话题 嵌入正文，3-5个
- 风格：真实、生活化、有画面感

### YouTube
- 标题：≤100字，含核心关键词，SEO友好
- 描述：结构化，前2行最重要（展开前可见），含时间戳、相关链接、关键词
- Tags：5-8个，长尾关键词优先
- 风格：信息清晰，可中英文视频内容而定

---

## Step 6 — Extract First-Frame Covers (if needed)

For any platform marked `cover: [first-frame]` in Step 1:

```bash
ffmpeg -i "{video_path}" -vframes 1 -q:v 2 "{folder}/cover-firstframe.jpg"
```

Note: one extraction is enough — reuse the same first-frame file for all platforms that need it.

---

## Step 7 — Present Draft for Review

Display the complete draft in Claude Code, one platform at a time:

```
=== 草稿预览 ===

【小红书】
封面：cover-3x4.jpg
标题：[title]
正文：
[body]
话题：#XX #XX #XX

---

【抖音】
封面：cover-9x16.jpg
标题/描述：[title]
话题：#XX #XX #XX

---
（其余平台同格式）
```

Then ask:
```
以上草稿有需要修改的吗？
可以说"小红书标题改成XX"、"抖音去掉第二个话题"，或"全部确认"。
```

Apply all edits immediately. Re-display the changed platform(s) after each edit.
Repeat until the user says "确认" or "全部确认".

---

## Step 8 — Write publish-brief.md

Save the finalized content to `{folder}/publish-brief.md`:

```markdown
# Publish Brief
生成时间：{datetime}
来源文件夹：{folder}
视频文件：{video_filename}

---

## 小红书
封面：{cover_file}
标题：{title}
正文：
{body}
话题：{topics}

---

## 抖音
封面：{cover_file}
标题：{title}
正文：{body}
话题：{topics}

---

## Bilibili
封面：{cover_file}
标题：{title}
简介：
{description}
标签：{tags}
分区：{category}

---

## YouTube
封面：{cover_file}
标题：{title}
描述：
{description}
Tags：{tags}

---

## 微信视频号
封面：{cover_file}
标题：{title}
描述：{description}
话题：{topics}

---

## 微博
封面：{cover_file}
正文：{body}
话题：{topics}

---

## X
封面：{cover_file}
正文：{body}
标签：{tags}
```

Confirm completion:
```
✓ publish-brief.md 已生成：{folder}/publish-brief.md

包含 {N} 个平台的发布内容，可交给 social-publisher skill 发布。
```

**Skill 1 ends here. No browser opened. Nothing published.**

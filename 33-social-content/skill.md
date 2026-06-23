---
name: 33-social-content
description: 从视频项目文件夹自动生成各平台发布文案，输出 publish-brief.md 供 33-social-publish 使用。支持口播脚本格式：.pages / .docx / .doc / .txt / .md / .pdf / .rtf / .odt / .fountain / .fdx。触发词：准备文案、写文案、生成发布内容、帮我写各平台内容、准备发布、写发布文案、social content prep、prepare publish brief。当用户提供包含视频和脚本的文件夹时自动触发。
version: 1.2.0
---

# 33-social-content

从视频项目文件夹读取脚本和素材，为每个目标平台生成优化后的发布文案，输出 `publish-brief.md`。

---

## Language

用中文回复。文件路径和字段名保持英文。

---

## Prerequisites

必须：
- macOS `textutil`（系统内置）
- `ffmpeg` + `ffprobe`：`brew install ffmpeg`

按需（读取特定格式时提示安装）：
- `pdftotext`（读取 PDF）：`brew install poppler`
- `pandoc`（读取 .odt）：`brew install pandoc`

依赖 skill：
- `web-access`（用于搜索各平台热门标签，未安装时降级为内置知识生成）
- `33-social-learn`（用于读取账号风格记忆，未安装时跳过）

---

## Step 0 — 读取账号风格记忆

在做任何事之前，先检查是否存在 ACCOUNT.md：

```
检查顺序：
1. {folder}/ACCOUNT.md（项目级，优先）
2. ~/.33-skills/ACCOUNT.md（全局）
```

如果找到，静默读取其中内容，将其中的账号风格偏好、高效标签记录、平台经验应用到后续所有文案生成步骤。不需要向用户展示读取过程。

如果未找到，正常继续，所有内容依赖通用知识生成。

---

## Step 1 — 定位素材

如果用户未提供文件夹路径，询问。

扫描文件夹，识别以下三类文件：

### 1a. 脚本文件

按优先级检测：

| 格式 | 检测方式 |
|------|---------|
| `.pages` | 找 `*.pages` |
| `.docx` / `.doc` | 找 `*.docx` 或 `*.doc` |
| `.txt` | 找 `*.txt` |
| `.md` | 找 `*.md`（排除 `publish-brief.md`）|
| `.pdf` | 找 `*.pdf` |
| `.rtf` | 找 `*.rtf` |
| `.odt` | 找 `*.odt` |
| `.fountain` | 找 `*.fountain` |
| `.fdx` | 找 `*.fdx` |

如果找到多个文档文件，列出让用户选择哪个是脚本。  
如果一个都没找到，告知用户并停止。

### 1b. 视频文件

找第一个 `*.mp4 / *.mov / *.avi / *.webm` 文件。

### 1c. 封面图片 + 第一帧提取

找所有 `*.jpg / *.jpeg / *.png` 文件，用 ffprobe 检测每张图的宽高比：

```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 "{image_path}"
```

| 检测比例 | 分配平台 |
|---------|---------|
| ~16:9（1.7–1.9） | YouTube / Bilibili / 微博 / 微信视频号 / X |
| ~9:16（0.5–0.6） | 抖音 |
| ~3:4 或 ~4:5（0.7–0.85） | 小红书 |

多张图片匹配同一比例时取文件最大的。

**找不到匹配比例的平台 → 立即提取视频第一帧作为封面：**

```bash
ffmpeg -i "{video_path}" -vframes 1 -q:v 2 "{folder}/cover-firstframe.jpg" -y
```

一次提取，所有缺封面的平台共用此文件。**在本步骤完成封面准备，后续步骤中每个平台的封面文件名均为真实文件名，不使用占位符。**

### 1d. 报告识别结果

```
素材识别结果：
脚本：script.pages ✓（Apple Pages）
视频：video.mp4 ✓
封面匹配：
  16:9 → cover-landscape.jpg（YouTube / Bilibili / 微博 / 视频号 / X）
  9:16 → cover-portrait.jpg（抖音）
  3:4  → cover-firstframe.jpg（小红书，已从视频第一帧提取）
```

---

## Step 2 — 读取脚本

根据脚本格式选择读取方式：

### `.pages`（Apple Pages）

**首选：AppleScript 导出**
```applescript
tell application "Pages"
    open POSIX file "{script_path}"
    export front document to POSIX file "/tmp/script_export.txt" as plain text
    close front document
end tell
```
```bash
cat /tmp/script_export.txt
```

**fallback 1：textutil**
```bash
textutil -convert txt "{script_path}" -output /tmp/script_export.txt && cat /tmp/script_export.txt
```

**fallback 2：unzip 提取内容**
```bash
unzip -p "{script_path}" index.xml 2>/dev/null | sed 's/<[^>]*>//g' | tr -s ' \n' | head -300
```

**全部失败时**：提示用户在 Pages 中「文件 → 导出为 → 纯文本」，然后告知路径。

---

### `.docx` / `.doc` / `.rtf`

```bash
textutil -convert txt "{script_path}" -output /tmp/script_export.txt && cat /tmp/script_export.txt
```

---

### `.txt` / `.md` / `.fountain`

直接读取文件内容。（`.fountain` 是纯文本剧本格式，无需特殊处理）

---

### `.pdf`

**首选：pdftotext**
```bash
pdftotext "{script_path}" /tmp/script_export.txt && cat /tmp/script_export.txt
```

如果 `pdftotext` 未安装：提示 `brew install poppler`，或请用户导出为 .txt。

---

### `.odt`（LibreOffice）

**首选：pandoc**
```bash
pandoc "{script_path}" -t plain -o /tmp/script_export.txt && cat /tmp/script_export.txt
```

如果 `pandoc` 未安装：提示 `brew install pandoc`，或请用户导出为 .docx 后重试。

---

### `.fdx`（Final Draft）

Final Draft 文件是 XML 格式，提取 `<Text>` 标签内容：
```bash
grep -o '<Text>[^<]*</Text>' "{script_path}" | sed 's/<[^>]*>//g' | tr '\n' ' '
```

---

### 解读脚本内容

从提取的文本中识别：
- **核心主题**：这个视频讲的是什么
- **关键信息点**：3-5 个最重要的内容点
- **目标受众**：面向什么人群
- **内容风格**：干货教程 / 生活分享 / 娱乐 / 评测 / Vlog 等

展示解读结果并确认：
```
脚本解读：
主题：[X]
关键点：[A] / [B] / [C]
受众：[X]
风格：[X]

理解有偏差的话告诉我，没问题就直接说继续。
```

---

## Step 3 — 选择目标平台

询问本次发布平台（支持多选）：

```
本次发布到哪些平台？

1. 微信视频号
2. 微博
3. X（Twitter）
4. Bilibili
5. 抖音
6. 小红书
7. YouTube

输入编号，逗号分隔，例如：4,5,6，或输入"全部"
```

---

## Step 4 — 搜索热门标签

为每个选中的平台，用 `web-access` 访问平台内搜索页获取真实标签热度数据。

尽量并行搜索。

| 平台 | 搜索 URL | 说明 |
|------|---------|------|
| 小红书 | `https://www.xiaohongshu.com/search_result?keyword={topic}&source=web_explore_feed` | 看话题浏览量和笔记数 |
| 抖音 | `https://www.douyin.com/search/{topic}?type=challenge` | 看话题播放量和上升趋势 |
| 微博 | `https://s.weibo.com/weibo?q=%23{topic}%23` | 看话题阅读量和讨论量 |
| X | `https://x.com/search?q=%23{topic}&f=live` | 看话题实时热度 |
| Bilibili | `https://search.bilibili.com/all?keyword={topic}` | 看视频数量和播放量 |
| YouTube | `https://www.youtube.com/results?search_query={topic}` | 看搜索结果量和热门视频 |
| 微信视频号 | `https://channels.weixin.qq.com/search?keyword={topic}` | 看话题内容数量 |

**标签选取标准：**

| 平台 | 推荐组合 |
|------|---------|
| 小红书 | 1 个超大话题（>10亿浏览）+ 2 个垂直话题（>1亿）+ 2 个精准小话题 |
| 抖音 | 3-5 个，优先选上升趋势，避开播放量停滞的饱和话题 |
| 微博 | 2-3 个，优先当前有热度的话题 |
| X | 3-5 个英文 hashtag，混合大小话题 |
| Bilibili | 5-8 个，含分区相关词，每个 ≤20字 |
| YouTube | 5-8 个，长尾关键词优先（具体词比泛词转化更好）|
| 微信视频号 | 2-3 个话题 |

如果 `web-access` 未安装或搜索失败，用内置知识生成标签并标注 `[未验证热度]`。

---

## Step 5 — 生成各平台文案

为每个选中的平台生成**两个标题选项（A/B）**、正文、标签，以及最佳发布时间建议。

如果 ACCOUNT.md 中有该平台的风格记忆或历史高效标签，优先采用。

严格遵守以下平台规则：

### 微信视频号
- 标题 A/B：各 ≤50字，生活化，可加 emoji
- 描述：2-3 行，支持 #话题，简洁自然
- 风格：接近朋友圈，真实
- 最佳发布时间：工作日 12:00-13:00 或 20:00-22:00，周末 10:00-12:00

### 微博
- 正文：≤2000字（普通微博），有传播性
- 标题 A/B：微博无独立标题字段，A/B 为正文开头两个版本（前20字）
- 标签格式：#话题# 嵌入正文中
- 风格：口语化，有情绪或观点
- 最佳发布时间：工作日 7:00-9:00 / 12:00-14:00 / 21:00-23:00

### X（Twitter）
- 正文 A/B：各 ≤280字，内容不同切角
- 标签格式：#hashtag 放正文末尾
- 风格：简洁直接，英文或中英混合视受众而定
- 最佳发布时间：周二-周四 9:00-15:00（UTC+8）

### Bilibili
- 标题 A/B：各 ≤80字，含关键词，描述性强，侧重点不同（一个主题导向，一个情绪/好奇心导向）
- 简介：结构化，含关键词，可加时间戳
- 标签：5-10个，每个 ≤20字
- 分区：根据内容判断（生活 / 知识 / 科技 / 游戏 / 娱乐等）
- 风格：信息密度适中，面向 B站用户
- 最佳发布时间：工作日 17:00-21:00，周末 10:00-12:00

### 抖音
- 描述 A/B：各 ≤55字，切角不同（一个制造好奇心，一个强调结果/价值）
- 标签格式：#话题 嵌入描述末尾
- 风格：有爆点，直接
- 最佳发布时间：工作日 7:00-9:00 / 12:00-14:00 / 18:00-21:00

### 小红书
- 标题 A/B：各 ≤20字，情绪价值不同（一个制造悬念，一个强调共鸣/干货）
- 正文：≤1000字，分段，多换行，适量 emoji，像朋友分享
- 话题：#话题 嵌入正文，3-5个
- 风格：真实、生活化、有画面感
- 最佳发布时间：工作日 7:00-8:30 / 12:00-13:30 / 21:00-23:00，周末全天

### YouTube
- 标题 A/B：各 ≤100字，一个 SEO 导向，一个点击率导向（更有情绪/悬念）
- 描述：结构化，前 2 行最重要（展开前可见），含时间戳和关键词
- Tags：5-8个，长尾关键词优先
- 风格：信息清晰，中英文视内容而定
- 最佳发布时间：周四-周六 14:00-17:00（目标受众时区）

---

## Step 6 — 展示草稿并接受修改

逐平台展示，**提供 A/B 标题选项**，**标签附带来源说明**，**附最佳发布时间**：

```
=== 发布草稿 ===

【小红书】
封面：cover-firstframe.jpg
标题 A：在家做出咖啡店同款拿铁🤫
标题 B：我研究了30天，终于搞懂了拿铁的秘密
正文：
之前以为拿铁很难做...（完整正文）
话题：#咖啡（浏览量 120亿）#在家咖啡（上升趋势，45亿）#拿铁（32亿）#咖啡爱好者（8亿）#下午茶（15亿）
最佳发布时间：今晚 21:00 或明天早上 7:30

---

【抖音】
封面：cover-portrait.jpg
描述 A：原来在家做拿铁这么简单！学会了再也不用去咖啡店 #咖啡 #拿铁 #生活技巧
描述 B：咖啡店的拿铁你知道怎么做的吗？其实3步就够了 #咖啡 #拿铁 #生活技巧
话题：#咖啡（播放 2800亿）#拿铁（上升趋势，180亿）#生活技巧（620亿）
最佳发布时间：今晚 18:30-20:00

---

（其余平台同格式）
```

然后询问：
```
以上草稿有需要修改的吗？
- 说"小红书用标题B"可切换
- 说"抖音描述改成XX"可修改内容
- 说"全部确认"进入下一步
```

每次修改后，只重新显示被修改的那个平台。重复直到用户确认。

---

## Step 7 — 写入 publish-brief.md

用户全部确认后，写入 `{folder}/publish-brief.md`：

```markdown
# Publish Brief
生成时间：{datetime}
来源文件夹：{folder}
视频文件：{video_filename}

---

## 小红书
封面：{cover_file}
标题：{selected_title}
正文：
{body}
话题：{topics}
最佳发布时间：{time}

---

## 抖音
封面：{cover_file}
描述：{selected_body}
话题：{topics}
最佳发布时间：{time}

---

## Bilibili
封面：{cover_file}
标题：{selected_title}
简介：
{description}
标签：{tags}
分区：{category}
最佳发布时间：{time}

---

## YouTube
封面：{cover_file}
标题：{selected_title}
描述：
{description}
Tags：{tags}
最佳发布时间：{time}

---

## 微信视频号
封面：{cover_file}
标题：{selected_title}
描述：{description}
话题：{topics}
最佳发布时间：{time}

---

## 微博
封面：{cover_file}
正文：{body}
话题：{topics}
最佳发布时间：{time}

---

## X
封面：{cover_file}
正文：{selected_body}
标签：{tags}
最佳发布时间：{time}
```

完成后确认：
```
✓ publish-brief.md 已生成：{folder}/publish-brief.md
包含 {N} 个平台的发布内容，可交给 33-social-publish skill 发布。
```

**本 skill 到此结束。不打开浏览器，不发布任何内容。**

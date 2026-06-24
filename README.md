<div align="center">

<h1>33-social-skills</h1>

<p>
  <b>拍完视频之后的所有事，交给 AI 做。</b><br/>
  <i>Everything after you finish filming — let AI handle it.</i>
</p>

<p>
  <a href="#-这是什么">这是什么</a> ·
  <a href="#-你做什么ai做什么">工作流</a> ·
  <a href="#-支持平台">平台</a> ·
  <a href="#-安装">安装</a>
  &nbsp;|&nbsp;
  <a href="#-what-is-this">English</a> ·
  <a href="#-workflow">Workflow</a> ·
  <a href="#-install">Install</a>
</p>

</div>

---

## 🇨🇳 中文

### 💡 这是什么

一套让 AI 帮你完成视频发布全流程的工具包。

你只需要把拍好的视频、脚本和封面素材放进一个文件夹，告诉 AI 发哪几个平台，剩下的——研究标签、写文案、做封面、逐平台发布——AI 来做。

**适合你，如果你：**
- 在多个平台发布同一个视频内容
- 每次发布都要花时间研究标签、改写文案，觉得枯燥
- 想要一个助手帮你填好每个平台，你只负责最后点"发布"

---

### 🔄 你做什么，AI 做什么

```
你做的事                        AI 做的事
──────────────────────────────────────────────────────
拍好视频，准备好脚本和封面素材
                                读取脚本，理解视频内容
告诉 AI 发哪几个平台
                                去各平台搜索真实的热门标签
                                按每个平台的规则写专属文案
                                给每个平台生成两个标题让你选
看一眼草稿，确认或修改
                                打开浏览器，逐平台上传视频
                                填好标题、文案、标签、封面
每个平台上手动点"发布"
                                记录这次发布的情况
下次发布时，AI 已经记住了
你的账号风格和上次踩过的坑
```

**封面也可以交给 AI：**
告诉 AI 这期的主题、选个背景色、上传一张运动照片，AI 直接生成封面图，还能自动裁剪成抖音、YouTube 等不同平台的比例。

---

### 📦 四个工具，一条流水线

| 工具 | 你什么时候用它 | 它帮你做什么 |
|------|-------------|------------|
| `33-social-cover` | 发布前，做封面 | 问你几个问题，生成品牌风格封面图，自动裁多平台比例 |
| `33-social-content` | 发布前，准备文案 | 读脚本 → 搜标签 → 写各平台文案 → 让你确认 |
| `33-social-publish` | 文案确认后 | 打开浏览器，逐平台填写并等你点发布 |
| `33-social-learn` | 每次发完后 | 总结这次经验，更新账号记忆，让下次发布更准 |

---

### 🌏 支持平台

小红书 · 抖音 · Bilibili · YouTube · 微信视频号 · 微博 · X (Twitter)

---

### ⚡ 安装

```bash
npx skills add sunsunnn33/33-social-skills
```

或者直接告诉 AI：
```
帮我安装 https://github.com/sunsunnn33/33-social-skills
```

**需要提前装好：**
```bash
brew install ffmpeg
```
以及 Chrome 浏览器（用你平时登录各平台的那个）。

---

### ▶️ 怎么用

装好后，直接用自然语言告诉 AI 你想做什么：

```
"帮我准备 /Users/我/这期视频 的各平台发布内容"
"发到小红书、抖音和 B 站"
"帮我做封面"
"开始发布"
"复盘一下这次发布"
```

不需要记命令，不需要填表格，跟 AI 说话就行。

---

### ⚠️ 注意

通过浏览器自动化操作社交平台有一定风险。建议先用小号试用，确认稳定后再用主号。

---

<br/>

## 🌐 English

### 💡 What Is This

A toolkit that lets AI handle your entire video publishing workflow.

Put your finished video, script, and cover image in a folder. Tell AI which platforms to publish to. Everything else — researching hashtags, writing captions, making covers, uploading to each platform — AI does it.

**This is for you if you:**
- Publish the same video across multiple platforms
- Spend too much time rewriting captions and researching trending tags for each platform
- Want an assistant to fill in everything so you just click "Publish"

---

### 🔄 Workflow — What You Do vs. What AI Does

```
You                             AI
──────────────────────────────────────────────────────
Film your video, prepare
your script and cover image
                                Reads your script, understands the content
Tell AI which platforms to use
                                Searches real trending tags on each platform
                                Writes platform-specific captions from scratch
                                Generates 2 title options per platform for you to pick
Review the drafts, confirm
or make edits
                                Opens Chrome, uploads your video to each platform
                                Fills in title, caption, tags, and cover image
Click "Publish" on each platform
                                Logs what happened this session
Next time, AI already remembers
your brand voice and past lessons
```

**Cover images too:**
Tell AI your topic, pick a background color, upload an action photo — AI generates a branded cover image and automatically crops it to the right ratio for each platform (9:16 for TikTok, 16:9 for YouTube, 3:4 for Xiaohongshu).

---

### 📦 Four Skills, One Pipeline

| Skill | When You Use It | What It Does |
|-------|----------------|--------------|
| `33-social-cover` | Before publishing — make cover | Asks a few questions, generates a branded cover, crops to multi-platform ratios |
| `33-social-content` | Before publishing — prepare captions | Reads script → researches tags → writes platform captions → waits for your approval |
| `33-social-publish` | After captions are ready | Opens Chrome, fills in each platform, waits for you to click Publish |
| `33-social-learn` | After each publish session | Summarizes what happened, updates account memory for next time |

---

### 🌏 Supported Platforms

Xiaohongshu · Douyin · Bilibili · YouTube · WeChat Channels · Weibo · X (Twitter)

---

### ⚡ Install

```bash
npx skills add sunsunnn33/33-social-skills
```

Or just tell your AI agent:
```
Install this for me: https://github.com/sunsunnn33/33-social-skills
```

**Prerequisites:**
```bash
brew install ffmpeg
```
Plus Chrome (the one where you're already logged into your social accounts).

---

### ▶️ How to Use

After installing, just talk to your AI agent in plain language:

```
"Prepare publishing content for /Users/me/this-video"
"Publish to Xiaohongshu, Douyin, and Bilibili"
"Make a cover for this video"
"Start publishing"
"Let's review this publish session"
```

No commands to memorize. No forms to fill. Just talk.

---

### ⚠️ Heads Up

Browser automation on social platforms carries some risk of account restrictions. Test with a secondary account first before using your main one.

---

<div align="center">
<br/>
MIT · by <a href="https://github.com/sunsunnn33">33</a>
</div>

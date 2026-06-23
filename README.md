# 33-social-skills

> 33 出品的自媒体 Skills 系列 — 专为视频内容创作者设计的 Claude Code 自动化工具

## 这个系列是什么

`33-social-*` 系列专注于视频内容的多平台分发，帮你把一条视频高效发布到 7 个主流平台，并自动适配每个平台的规则、风格和热门标签。

## Skills

### `33-social-content` · 内容准备

从视频项目文件夹自动生成各平台发布文案。

**你只需要准备：**
- 口播脚本（.pages 格式）
- 成品视频
- 封面图片（按比例分版本）

**它会自动：**
- 读取脚本，理解主题、受众、风格
- 为每个目标平台研究当前热门标签
- 按各平台规则生成独立文案（标题 / 正文 / 话题标签）
- 在 Claude Code 里展示草稿，接受你的修改
- 确认后输出 `publish-brief.md`

---

### `33-social-publish` · 多平台发布

读取 `publish-brief.md`，逐平台发布视频内容。

**流程：**
- 选择本次要发布的平台
- 自动打开 Chrome，上传视频和封面，填入文案
- 每个平台发布前停下来让你确认
- 你手动点发布，它继续下一个
- 最后输出发布报告

---

**支持平台：**

微信视频号 · 微博 · X (Twitter) · Bilibili · 抖音 · 小红书 · YouTube

---

## 安装

```bash
npx skills add sunsunnn33/33-social-skills
```

## 前置要求

- macOS
- Google Chrome 或 Chromium
- `bun`：`brew install oven-sh/bun/bun`
- `ffmpeg`：`brew install ffmpeg`
- 各平台在 Chrome 里手动登录一次（session 自动保留）
- `web-access` skill（用于搜索热门标签）：

```bash
npx skills add eze-is/web-access
```

## 使用流程

```
第一步：准备文案
告诉 Claude："帮我准备这个文件夹的发布内容"
→ 触发 33-social-content
→ 输出 publish-brief.md

第二步：发布
告诉 Claude："开始发布"
→ 触发 33-social-publish
```

## 关于 33-skills 系列

这是 33 出品的 Skills 合集之一，专注自媒体场景。
其他系列持续更新中。

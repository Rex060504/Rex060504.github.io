# rex060504 的博客

基于 [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题的个人博客，通过 GitHub Actions 自动部署到 GitHub Pages（<https://rex060504.github.io/>）。

## 技术栈

- 静态站点生成器：[Hugo](https://gohugo.io/)（extended）
- 主题：[PaperMod](https://github.com/adityatelange/hugo-PaperMod)（以 git submodule 引入）
- 部署：[GitHub Actions](https://github.com/features/actions) → GitHub Pages
- 站点地址：<https://rex060504.github.io/>

## 目录结构

```
my-blog/
├── archetypes/default.md       # 新建文章时的默认模板
├── content/
│   ├── _index.md               # 首页（个人简介 + 最新文章，首页显示 5 篇）
│   ├── about.md                # 关于页
│   ├── archives.md             # 归档页
│   ├── search.md               # 搜索页（可选）
│   └── posts/
│       ├── _index.md           # 博客列表页
│       ├── first-post.md       # 示例文章
│       └── hello-world.md      # 已有测试文章（draft，不会发布）
├── hugo.toml                   # 站点配置（菜单、主题、分类、社交等）
├── themes/PaperMod/            # 主题（submodule）
└── .github/workflows/hugo.yaml # 自动部署工作流
```

## 快速开始（本地开发）

### 1. 克隆并初始化主题子模块

```bash
git clone https://github.com/rex060504/rex060504.github.io.git
cd rex060504.github.io
git submodule update --init --recursive   # 拉取 PaperMod 主题
```

> 注意：仓库名必须是 `rex060504.github.io`，GitHub Pages 用户站点只接受这个名字。

### 2. 本地预览（含草稿）

```bash
hugo server -D
```

浏览器打开 <http://localhost:1313/> 即可预览，文件改动会实时刷新。

- `-D` 表示包含 `draft: true` 的草稿文章；去掉 `-D` 则只显示正式文章。
- 若希望局域网内其它设备也能访问，可用 `hugo server -D --bind 0.0.0.0`。

### 3. 构建

```bash
hugo --minify          # 普通构建
hugo --minify --gc     # 构建并清理缓存（与 CI 一致）
```

构建产物输出到 `public/` 目录（已被 `.gitignore` 忽略，无需提交）。

## 创建新文章

```bash
hugo new content posts/我的新文章.md
```

该命令会按 `archetypes/default.md` 生成带如下 front matter 的模板：

```markdown
---
title: '我的新文章'
date: '2026-09-18T...'
draft: true
author: 'rex060504'
tags: []
categories: []
summary: ''
toc: false
---
```

填好 `tags` / `categories` / `summary`，把 `draft` 改为 `false` 即可发布。常用 front matter 字段：

| 字段 | 说明 |
| --- | --- |
| `title` | 文章标题 |
| `date` | 发布日期 |
| `draft` | `true` 为草稿（本地 `-D` 可见，不发布） |
| `tags` | 标签列表，如 `['Hugo', '博客']` |
| `categories` | 分类列表，如 `['随笔']` |
| `summary` | 列表页摘要（不写则自动截取正文） |
| `toc` | `true` 时显示文章目录 |
| `author` | 作者（默认取 `hugo.toml` 中的 `author`） |

## 部署到 GitHub Pages

部署是自动的：将改动推送到 `main` 分支即可，GitHub Actions 会执行 `.github/workflows/hugo.yaml`，构建并发布到 <https://rex060504.github.io/>。

```bash
git add .
git commit -m "更新博客"
git push
```

首次部署前，请在 GitHub 仓库 **Settings → Pages** 中将 **Source** 设为 **GitHub Actions**。

工作流要点（已配置好，无需改动）：

- `permissions`: `pages: write`、`id-token: write`
- `concurrency`: 避免并发部署冲突
- 构建命令：`hugo --minify --gc`，产物目录 `public/`
- 通过 `actions/configure-pages` 自动匹配 `baseURL`

## 个性化配置

所有配置集中在 `hugo.toml`，改完保存后本地预览即可看到效果。

### 修改站点标题 / 作者

```toml
title = 'rex060504 的博客'      # 站点标题
[params]
  author = 'rex060504'          # 作者名
  description = '个人博客：记录技术、学习与生活'
```

### 修改首页个人简介

`homeInfoParams` 控制首页顶部的自我介绍：

```toml
[params.homeInfoParams]
  Title = '你好，我是 rex060504'
  Content = '欢迎来到我的博客。这里记录我的技术笔记、学习心得与生活随笔。'
  AlignSocialIconsTo = 'left'
```

### 社交链接

在 `[[params.socialIcons]]` 中添加/修改，`name` 需为 PaperMod 支持的图标名（如 `github`、`twitter`、`email`、`linkedin`、`rss` 等）：

```toml
[[params.socialIcons]]
  name = 'github'
  url = 'https://github.com/rex060504'
```

### 切换为「个人主页模式」

默认是「简介 + 最新文章」。若想首页只显示头像 + 标题 + 按钮的个人主页，把 `profileMode` 的 `enabled` 改为 `true`，并取消 `homeInfoParams`（二者二选一）：

```toml
[params.profileMode]
  enabled = true
  title = 'rex060504'
  subtitle = '个人博客'
  imageUrl = ''   # 填入头像地址，如 /avatar.png
```

### 暗色模式

```toml
defaultTheme = 'auto'   # auto / light / dark
disableThemeToggle = false
```

### 启用站内搜索（可选）

1. 在 `hugo.toml` 的 `[menu]` 中取消「搜索」菜单项的注释；
2. `content/search.md` 已就绪，`[params.fuseOpts]` 已配置，无需其它改动。

### 评论 / 统计（可选，暂未接入）

`hugo.toml` 末尾已留注释占位。接入 giscus 评论或 Google/Bing 统计时，按注释示例填入你自己的配置即可，**请勿提交真实密钥**。

## 常见问题

- **本地 `hugo server -D` 正常，但线上没有某篇文章**：检查该文章 `draft` 是否为 `true`（线上构建不含草稿）。
- **`baseURL` 与站点不匹配**：确认 `hugo.toml` 中 `baseURL = 'https://rex060504.github.io/'` 与仓库名一致。
- **主题子模块未初始化**：运行 `git submodule update --init --recursive`。

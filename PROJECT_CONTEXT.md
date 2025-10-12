# 项目上下文概要 (Project Context Summary)

**最后更新时间:** 2025年10月12日

本文档旨在为本项目提供一个快速、全面的背景介绍，以便在开始新的开发或维护工作时，能迅速理解项目的状态、技术选型和历史决策。

---

### 1. 项目概览

本项目是一个基于 **GitHub Pages** 和 **Jekyll** 构建的个人静态网站。其主要目的是分享高质量的数字资源、实用教程和效率工具。

项目从最初一个专注于“网盘推广”的垂直教程站，经过重新定位，现已转型为一个内容更广泛、更具品牌感的数字资源分享中心。

### 2. 品牌识别 (Brand Identity)

* **网站名称:** 像素信标 (Pixel Beacon)
* **网站描述:** `像素信标 (Pixel Beacon) - 你的数字生活导航。我们致力于在浩瀚的互联网中，为你搜寻并点亮那些值得关注的效率工具、实用软件、精品资源与优质教程。`

### 3. 技术栈 (Tech Stack)

* **托管平台 (Hosting):** GitHub Pages
* **静态网站生成器 (SSG):** Jekyll
* **Jekyll 主题 (Theme):** `minima`
* **网站统计 (Analytics):** Google Analytics 4

### 4. 关键配置 (`_config.yml`)

项目的核心配置如下，其中 `url` 和 `baseurl` 已根据 GitHub Pages 的最佳实践进行了精确设置。

```yaml
# 网站标题
title: 像素信标

# 网站描述
description: >-
  像素信标 (Pixel Beacon) - 你的数字生活导航。我们致力于在浩瀚的互联网中，为你搜寻并点亮那些值得关注的效率工具、实用软件、精品资源与优质教程。

# 网站发布后的根 URL，末尾不要加 /
url: "[https://x-wuxl.github.io](https://x-wuxl.github.io)" 
baseurl: ""

# 主题
theme: minima

# 插件
plugins:
  - jekyll-feed
  - jekyll-seo-tag
  - jekyll-sitemap

# Google Analytics ID
google_analytics: G-G21L5FTFZH
```

### 5. 内容与文件结构

* **文章 (`_posts/`):** 所有的博客文章都存放于此目录，遵循 Jekyll 的 `YYYY-MM-DD-文件名.md` 命名规范。
* **首页 (`index.md`):** 使用主题默认的 `home` 布局。
* **头部定制 (`_includes/head.html`):** 为了集成 Google Analytics，我们创建了此文件来覆盖主题的默认设置。该文件包含了 GA4 的跟踪脚本，并使用 `_config.yml` 中的变量进行管理。
* **依赖管理 (`Gemfile`):** 项目包含一个 `Gemfile` 文件，用于锁定 `github-pages` 这个 gem，以解决 Jekyll 主题和插件的依赖问题，确保在 GitHub Pages 环境中能稳定构建。

---
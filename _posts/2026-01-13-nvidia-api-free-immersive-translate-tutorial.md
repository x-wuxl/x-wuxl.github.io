---
layout: post
title: "Nvidia API 免费使用教程：配合沉浸式翻译实现无限额度翻译"
date: 2026-01-13 19:30:00 +0800
categories: [AI工具, 实用教程]
tags: [Nvidia, API, 沉浸式翻译, Kimi, 免费翻译, AI翻译, 大模型]
description: "详细教程：如何免费申请 Nvidia API Key，配合沉浸式翻译浏览器插件使用 Kimi K2 等大模型进行网页翻译，无额度限制，完全免费。"
excerpt: "英伟达免费提供大模型 API，无额度限制！本文手把手教你申请 Nvidia API Key，并配置沉浸式翻译，实现免费的 AI 网页翻译体验。"
author: 像素信标
keywords: 
  - Nvidia API
  - 沉浸式翻译
  - 免费翻译API
  - Kimi K2
  - AI翻译
  - 大模型API
  - 免费AI
seo:
  type: BlogPosting
  canonical: "https://xwuxl.com/2026/01/13/nvidia-api-free-immersive-translate-tutorial"
  publisher: "像素信标"
og:
  type: article
  locale: zh_CN
twitter:
  card: summary_large_image
---

对于经常阅读英文技术文档或海外资讯的朋友来说，**沉浸式翻译**是一款非常好用的浏览器插件。它支持多种翻译服务，包括自定义 AI 翻译 API。

好消息是，**英伟达（Nvidia）**免费提供了大模型 API，**没有额度限制**，完全免费！唯一的限制是速率（约 20-40 RPM），但对于日常翻译来说完全够用。

用来当沉浸式翻译的后端 API，简直不要太香！

---

## 申请 Nvidia API Key 步骤

### 第一步：注册 Nvidia 开发者账号

访问 [Nvidia Build](https://build.nvidia.com/) 官网，完成账号注册。

### 第二步：验证手机号

**注意**：注册后必须验证手机号，否则无法申请 API Key。未验证时会看到以下错误提示：

```
No API Key Permissions

You do not have permissions to create API Keys in this Organization. 
Switch Orgs or contact your admin.
```

验证方法：注册成功后，网站顶部会显示横幅提示，点击即可进行手机号验证。

### 第三步：创建 API Key

1. 点击网站右上角的**头像**
2. 选择 **API Keys**
3. 点击创建，即可获取你的 API Key

> **提示**：Nvidia 平台提供了众多免费模型，我现在使用的 **Kimi K2**（`moonshotai/kimi-k2-instruct`），翻译速度快。

---

## 沉浸式翻译配置

### 添加自定义翻译服务

1. 打开沉浸式翻译插件设置
2. 左侧菜单「翻译服务」→「添加自定义翻译服务」→「自定义 AI」
3. 填写以下配置信息：

| 配置项 | 值 |
|--------|-----|
| **API 接口地址** | `https://integrate.api.nvidia.com/v1/chat/completions` |
| **API Key** | 你申请的 Nvidia API Key |
| **自定义模型名称** | `moonshotai/kimi-k2-instruct` |

<img src="/assets/images/translation_settings.png" width="60%" alt="沉浸式翻译自定义服务设置">

### 启用翻译服务

配置完成后，左侧菜单「基本设置」→「翻译服务」，选择刚添加的自定义服务即可。

**免费 + 无限量**，这还不香吗？

可以不用，但不能没有！🎉
